# Large Language AIRL

Train a small language model to write **persuasive debate rebuttals** using **adversarial inverse reinforcement learning** (AIRL) plus **PPO**.

The goal is not “sound like a helpful chatbot.” It is to recover a reward for *successful persuasion* from expert human arguments, then use that reward to improve a generator. Expert demonstrations come from Reddit **ChangeMyView** threads where a rebuttal actually changed the original poster’s mind.

---

## What this project is

Online debate is a sparse-reward problem. You rarely get a clean scalar for “this argument worked.” Inverse RL treats **winning arguments as expert trajectories** and learns a discriminator that scores how expert-like a generated rebuttal is.

This repo implements that loop for language:

1. A **policy** (`Qwen/Qwen2.5-0.5B`) generates a rebuttal given a CMV thread.
2. A **discriminator** (tiny BERT classifier) is trained to tell policy text from expert winning arguments.
3. The discriminator is converted into an **AIRL-style reward**.
4. **PPO** updates the policy so generated rebuttals look more like arguments that actually changed minds.

The training prompt is:

```text
Generate a rebuttal to change the opinion of this post.
Post: <conversation context>
Rebuttal:
```

---

## Why adversarial IRL (not just SFT or RLHF)

| Approach | What it uses | Limitation here |
|---|---|---|
| Supervised fine-tuning | Expert rebuttals as next-token targets | Copies style, does not optimize “would this change a mind?” |
| RLHF | Human preference labels | Expensive; preferences ≠ persuasion success |
| AIRL / GAIL | Expert trajectories vs policy samples | Recovers a reward from *successful* arguments and keeps adapting as the policy improves |

AIRL is a good fit because:

- The environment (a debate thread) has no built-in reward.
- Successful CMV comments are **demonstrations of a high-reward policy**.
- A discriminator can keep pace with the generator, so the reward does not collapse to a static classifier.

This implementation is GAIL-style classification plus the **AIRL log-odds reward**, not a full state–action occupancy matching derivation.

---

## Pipeline

```text
winning-args-corpus (successful CMV comments)
        │
        ▼
   Dataset builder
   context = thread so far
   expert  = winning rebuttal
        │
        ├──────────────────────────────┐
        ▼                              ▼
 Policy generates rebuttal      Sample expert rebuttals
        │                              │
        └──────────┬───────────────────┘
                   ▼
         Discriminator D(x)
         expert = 1, generated = 0
                   │
                   ▼
         r = logit(D) = log D − log(1−D)
         normalize + clip
                   │
                   ▼
              PPO + GAE
         update Qwen + value head
```

Each training batch is one full adversarial step: generate, retrain `D`, score, PPO.

---

## Data

`src/data.py` loads Cornell Convokit’s [`winning-args-corpus`](https://convokit.cornell.edu/). On first run it downloads the corpus, filters, tokenizes, and caches `data/data.json`.

A sample is kept only if:

- The utterance is marked `success == 1` (the OP awarded a ∆ / the argument won).
- The speaker is **not** the original poster (OP last-word “successes” are dropped).

For each winning comment the code walks `reply_to` up the thread, labels speakers as `Original Poster` vs `Opponent`, concatenates the history as **context**, and uses the winning comment as the **expert response**. Text is cleaned (EDIT notes, CMV boilerplate, encoding fixes via `ftfy`).

Training currently uses the first **1000** cached examples, max sequence length **256**.

---

## Models

### Policy (actor) + value head (critic)

`src/policy.py` wraps `Qwen/Qwen2.5-0.5B`:

- Causal LM generates the rebuttal (`do_sample=True`, `top_k=50`, `top_p=0.95`).
- A linear **value head** reads the last-token hidden state and predicts `V(s)`.
- Separate Adam LRs: policy `1e-6`, value head `1e-5` at init; PPO uses one optimizer over both at `1e-5`.

Generation also stores per-token log-probs of the sampled continuation for the PPO ratio.

Optional 4-bit quantization lives in `src/quantize.py` (not on the default training path).

### Discriminator

`src/discriminator.py` is a 2-layer MiniBERT:

- Backbone: `google/bert_uncased_L-2_H-128_A-2`
- Head: `Linear(128 → 128) → ReLU → Linear(128 → 1) → Sigmoid`
- Labels: generated `0`, expert `1`
- Loss: binary cross-entropy
- Only the classification head is optimized (Adam `1e-4`); the BERT encoder is used as a frozen feature extractor in the current training setup

`D(x)` is the probability that text `x` is an **expert** winning argument.

---

## Reward

After each discriminator update, generated rebuttals are scored with the AIRL / GAIL occupancy-measure reward:

\[
r(x) = \log D(x) - \log(1 - D(x)) = \operatorname{logit}(D(x))
\]

Intuition:

- `D → 1` (looks expert) → large **positive** reward
- `D → 0.5` (indistinguishable) → reward near **0**
- `D → 0` (obviously generated) → large **negative** reward

Stabilization before PPO (CLIP-style):

1. Subtract batch mean, divide by batch std
2. Clamp to `[-4, 4]`

Rewards are **terminal**: each rebuttal is one episode (`done = 1`, `V(s') = 0`). There is no token-level environment reward.

---

## Adversarial training loop

`src/training.py` runs this for every minibatch:

1. **Rollout.** For each CMV context, the policy samples a rebuttal and records token log-probs.
2. **Expert batch.** Random expert responses are drawn from the dataset (same batch size).
3. **Discriminator step.** Shuffle policy text + expert text and train `D` for 6 BCE epochs.
4. **Reward.** Score only the *generated* texts with `logit(D)`, then normalize and clip.
5. **Values.** The critic estimates `V(context)` for GAE.
6. **PPO.** Several epochs of clipped policy and value updates.
7. **Logging.** Rewards and losses go to Weights & Biases (`ppo-training`).

Default outer loop: **2 epochs**, batch size **25**.

This is the adversarial part: as the policy gets better at fooling `D`, `D` is retrained, so the reward target moves. The policy is chasing a live estimate of “would this look like a winning CMV argument?”

---

## PPO details

`src/ppo.py` implements clipped PPO with GAE(λ).

| Hyperparameter | Value | Role |
|---|---|---|
| `γ` | 0.99 | Discount |
| `λ` | 0.95 | GAE bias–variance |
| `clip_ε` | 0.2 | Policy and value clipping |
| `vf_coef` | 0.5 | Value loss weight |
| `ent_coef` | 0.01 | Entropy bonus (currently a stub of 0) |
| PPO epochs | 4 | Reuses each batch of rollouts |
| Grad clip | 1.0 | Stable LM updates |

Per sample:

- Advantages \(A_t\) from GAE; returns \(R_t = V_t + A_t\)
- Advantages normalized over the batch
- Policy loss: \(-\mathbb{E}[\min(\rho A, \operatorname{clip}(\rho, 1-\varepsilon, 1+\varepsilon) A)]\), with \(\rho = \exp(\log\pi_\theta - \log\pi_{\text{old}})\)
- Value loss: clipped MSE
- NaN/Inf ratios replaced with 1 before clipping

Checkpoints:

- Policy + tokenizer + value head → `models/policy_model`
- Discriminator BERT/tokenizer → `models/discriminator`
- Curves → `data/training_metrics.json`

---

## Evaluation

`src/evaluation.py` compares three sources of rebuttals on the same threads:

- AIRL-trained policy
- Untuned `Qwen/Qwen2.5-0.5B`
- Held-out expert CMV responses

Each string is scored with the trained discriminator reward. Results are written to `evaluation_results.json`.

`src/evaluate.py` plots PPO losses, entropy, and rewards (including a moving average) from `data/training_metrics.json` into `plots/`.

---

## Project layout

```text
src/training.py          # AIRL + PPO loop
src/policy.py            # Qwen policy + value head
src/discriminator.py     # MiniBERT D(x) and AIRL reward
src/ppo.py               # GAE + clipped PPO
src/data.py              # CMV winning-args dataset
src/evaluation.py        # Trained vs baseline vs expert
src/evaluate.py          # Metric plots
src/quantize.py          # Optional 4-bit Qwen export
src/utils/utils_log.py   # Colored logging
```

---

## Setup

```bash
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

Hardware: CUDA if available, else Apple MPS, else CPU. The 0.5B policy is chosen so the loop can run on a single GPU or Apple Silicon.

W&B: `src/training.py` logs to entity `aboufarhatelio-university-of-toronto`, project `ppo-training`. Change that (or disable wandb) before running.

---

## Running

Train (downloads Convokit data on first run, then caches `data/data.json`):

```bash
python -m src.training
```

Plot training curves:

```bash
python -m src.evaluate
```

Compare trained policy vs baseline vs experts (update model paths if your checkpoints are under `models/` rather than `src/models/`):

```bash
python -m src.evaluation
```

---

## Implementation notes

- Episodes are single-turn: one context → one rebuttal. Multi-turn debate as an MDP is left for later.
- Discriminator training currently tokenizes the full shuffled list once per epoch and does not minibatch inside that step; watch memory if you raise `batch_size`.
- Entropy regularization is wired into the PPO loss but the entropy tensor is filled with zeros, so it does not yet encourage exploration.
- The discriminator `save()` writes BERT weights, not the extra MLP head; reload that head if you need exact eval-time rewards.
- Training hyperparameters are hardcoded in `src/training.py`.

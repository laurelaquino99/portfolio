# Tune Suggest — Spotify × Gemini

Connect your Spotify account, pick **Music** or **Podcasts** mode, and get AI-powered recommendations from Google Gemini based on your listening history.

## Stack

- **Next.js 14** (App Router) — single app with API routes so secrets stay on the server
- **Spotify Web API** — OAuth 2.0, recently played, top tracks, saved shows
- **Google Gemini** — `gemini-1.5-flash` for recommendations

## Setup

### 1. Spotify app

1. Go to [Spotify Developer Dashboard](https://developer.spotify.com/dashboard) and create an app.
2. In the app, add a **Redirect URI**:  
   `http://127.0.0.1:3000/api/auth/spotify/callback`  
   (Use your production URL when you deploy.)
3. Copy the **Client ID** and **Client Secret**.

### 2. Gemini API key

1. Go to [Google AI Studio](https://aistudio.google.com/apikey) and create an API key.
2. Copy the key.

### 3. Environment variables

In the project root, copy the example env file and fill in your values:

```bash
cp .env.local.example .env.local
```

Edit `.env.local`:

```env
SPOTIFY_CLIENT_ID=your_spotify_client_id
SPOTIFY_CLIENT_SECRET=your_spotify_client_secret
SPOTIFY_REDIRECT_URI=http://127.0.0.1:3000/api/auth/spotify/callback
GEMINI_API_KEY=your_gemini_api_key
```

`SPOTIFY_REDIRECT_URI` must match exactly what you set in the Spotify Dashboard.

### 4. Install and run

```bash
npm install
npm run dev
```

Open [http://127.0.0.1:3000](http://127.0.0.1:3000), click **Connect with Spotify**, then:

- Choose **Music** or **Podcasts**
- Click **Load my listening history**
- Click **Get recommendations (Gemini)** to see suggestions

## Modes

- **Music** — uses your recently played tracks and top tracks, then asks Gemini for song/artist recommendations.
- **Podcasts** — uses your saved shows and recently played episodes, then asks Gemini for podcast/show recommendations.

## Logout

Use **Log out** to clear the stored Spotify tokens (cookies). Reconnect with **Connect with Spotify** when you want to use the app again.

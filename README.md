# Yunzii B75 Pro Max — Spotify Now Playing Display

Displays your current Spotify track (album art, song name, artist) on the built-in screen of the **Yunzii B75 Pro Max** keyboard, running entirely in the browser using WebHID. No native app, no drivers, no backend.

![layout preview](https://i.imgur.com/placeholder.png)

---

## Requirements

- **Keyboard:** Yunzii B75 Pro Max (the HID protocol is specific to this model)
- **Browser:** Google Chrome (or any Chromium-based browser) — WebHID is not supported in Firefox or Safari
- **Spotify account** (free or premium)

---

## Setup

### 1. Create a Spotify Developer App

1. Go to [developer.spotify.com/dashboard](https://developer.spotify.com/dashboard)
2. Log in and click **Create app**
3. Fill in any name and description
4. Set **Redirect URI** to: `http://127.0.0.1:3000/yunzii_spotify.html`
   - Click **Add** then **Save**
5. Go to **Settings** and copy your **Client ID**

### 2. Add your Client ID to the file

Open `yunzii_spotify.html` in a text editor, find this line near the top of the `<script>` section:

```js
const CLIENT_ID = 'YOUR_SPOTIFY_CLIENT_ID';
```

Replace `YOUR_SPOTIFY_CLIENT_ID` with the Client ID you copied.

### 3. Serve the file locally

WebHID and Spotify OAuth both require a proper origin (not a `file://` URL). The easiest way is Python's built-in server:

```bash
cd /path/to/folder
python3 -m http.server 3000 --bind 127.0.0.1
```

Then open **[http://127.0.0.1:3000/yunzii_spotify.html](http://127.0.0.1:3000/yunzii_spotify.html)** in Chrome.

### 4. Use it

1. **Step 1 — Spotify:** Click **Login with Spotify** and authorise the app. The page will redirect back automatically.
2. **Step 2 — Keyboard:** Plug in your Yunzii B75 Pro Max via USB, then click **Connect Keyboard**. Select the keyboard from the browser prompt.
3. Done — the screen updates automatically every 2 seconds.

> **Order matters:** log in to Spotify first, then connect the keyboard. The OAuth redirect will disconnect any HID device, so doing it the other way around means you'd have to reconnect the keyboard.

---

## Options

| Control | Description |
|---|---|
| **Auto-update** | Toggle the refresh loop on/off |
| **Refresh interval** | 1s / 2s / 5s / 10s |
| **Layout** | Cycle between *Clean* (big art left), *Cover top* (art fills top), *Progress* (with progress bar and timestamps) |
| **Art fit** *(Cover top only)* | *Zoom* (crop sides to fill), *Blur sides* (blurred background with crisp centre), *Stretch* |
| **Rounded art corners** | Applies to all layouts |

---

## How it works

The keyboard exposes a 160×96 RGB565 framebuffer over WebHID. The protocol (reverse-engineered from the official yunzii-game.com web app) is:

1. **`0x40`** — init packet (tells the keyboard a frame transfer is starting)
2. **`0x41`** — metadata packet
3. **`0x41` × 549** — pixel data chunks (56 bytes of RGB565 each, with offset + checksum header)
4. **`0x42`** — commit (display the frame)

The app draws the current track onto a hidden `<canvas>`, converts it to RGB565, and sends the full sequence every 2 seconds. Album art is fetched via the Spotify Web API with CORS support.

---

## Privacy

- No data leaves your machine except to Spotify's API
- Spotify tokens are stored in `localStorage` (so you stay logged in across sessions) and never sent anywhere else
- No analytics, no tracking, no server

---

## Limitations

- Requires Chrome — WebHID is not available in Firefox or Safari
- The page must stay open and visible in a window (minimising to a small window in the corner works fine)
- The keyboard's screen reverts to its default display a few seconds after the last push, which is why the 2s auto-refresh is needed

---

## Acknowledgements

Protocol reverse-engineered from [yunzii-game.com](https://yunzii-game.com) using Chrome DevTools WebHID capture.

# Yunzii B75 Pro Max — Spotify Now Playing Display

Displays your current Spotify track (album art, song name, artist) on the built-in screen of the **Yunzii B75 Pro Max** keyboard, running entirely in the browser using WebHID. No native app, no drivers, no backend.

---

## Requirements

- **Keyboard:** Yunzii B75 Pro Max (the HID protocol is specific to this model)
- **Browser:** Google Chrome (or any Chromium-based browser) — WebHID is not supported in Firefox or Safari
- **Spotify account** (free or premium)

---

## Usage

### 1. Get a Spotify Client ID

You need your own Client ID — this is free and takes 2 minutes.

1. Go to [developer.spotify.com/dashboard](https://developer.spotify.com/dashboard)
2. Log in and click **Create app**
3. Fill in any name and description
4. Under **Redirect URIs**, add the URL of this page (shown on the page itself in Step 1) then click **Save**
5. Go to **Settings** and copy your **Client ID**

### 2. Open the app

Visit the hosted page, paste your Client ID into the input field in Step 1, then click **Login with Spotify**.

### 3. Connect your keyboard

After the Spotify login redirects back, click **Connect Keyboard** in Step 2 and select your Yunzii from the browser prompt.

That's it — the screen updates automatically.

> **Order matters:** log in to Spotify first, then connect the keyboard. The OAuth redirect disconnects any HID device, so doing it the other way around means reconnecting.

---

## Options

| Control | Description |
|---|---|
| **Auto-update** | Toggle the refresh loop on/off |
| **Refresh interval** | 1s / 2s / 5s / 10s |
| **Layout** | *Clean* (big art left) · *Cover top* (art fills top) · *Progress* (progress bar + timestamps) |
| **Art fit** *(Cover top only)* | *Zoom* (crop to fill) · *Blur sides* (blurred bg + crisp centre) · *Stretch* |
| **Rounded art corners** | Applies to all layouts |

---

## How it works

The keyboard exposes a 160×96 RGB565 framebuffer over WebHID. The protocol (reverse-engineered from the official yunzii-game.com web app) is:

1. **`0x40`** — init packet (signals start of frame transfer)
2. **`0x41`** — metadata packet  
3. **`0x41` × 549** — pixel data chunks (56 bytes of RGB565 each, with offset + checksum header)
4. **`0x42`** — commit (display the frame)

The app draws the current track onto a hidden `<canvas>`, converts it to RGB565, and sends the full sequence every 2 seconds. Album art is fetched via the Spotify Web API.

---

## Privacy

- Your Client ID is stored only in your browser's `localStorage` — it never leaves your machine
- Spotify tokens are also stored in `localStorage` and only used to call Spotify's API
- No analytics, no tracking, no server

---

## Limitations

- Chrome only — WebHID is not available in Firefox or Safari
- The page must stay open in a window (minimising to a small corner window works fine)
- The keyboard screen resets to its default after a few seconds without a push — hence the 2s auto-refresh

---

## Acknowledgements

Protocol reverse-engineered from [yunzii-game.com](https://yunzii-game.com) using Chrome DevTools WebHID capture.

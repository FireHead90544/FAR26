# FAR26

> *"I'm not really a stage person — so instead, I built something."*

A two-device WebRTC webapp built for my graduation farewell rampwalk. One page runs on the display/projector. The other runs on your phone as a silent remote. You stream a live camera feed to the screen, snap a group photo from your phone, and the image drops in as a polaroid with confetti and a farewell message — simultaneously posting to a Discord channel for permanent safekeeping.

No server. No build step. Two static HTML files.

> **A small note** — Though neither I had such tech-savvy audience, nor my college would have appreciated this. This is what I'd have loved to showcase at my farewell, had I got the chance & such environment. So, if you've got your farewell & you're deeply hinged into tech, go wild with creativity, feel free to take it to a whole new level & make sure to keep your day memorable! All the best, you're a graduate now 🎉

---

## How it works

```
 [controller.html]  ──── WebRTC (PeerJS) ────▶  [index.html]
   your phone                                    projector / display laptop
   · camera feed                                 · shows live feed full-screen
   · shutter button                              · flashes on capture
   · go-live toggle                              · polaroid drop animation
                                                 · confetti + farewell text
                                                         │
                                                         ▼
                                                   Discord webhook
                                                   (image saved permanently)
```

The two devices connect peer-to-peer via [PeerJS](https://peerjs.com/) — a free WebRTC signalling layer. After the initial handshake, all video and data travels directly between devices. No cloud, no backend, no database.

---

## Files

```
memory-capture/
├── index.html        ← display screen (open on laptop → projector)
└── controller.html   ← phone remote (open via saved URL)
```

---

## Setup

### 1. Configure the room code

In `index.html`, find the CONFIG block near the bottom `<script>`:

```js
const ROOM_ID      = 'FAR26';
const PEER_ID      = `farewell2026-${ROOM_ID}`;
const DISCORD_HOOK = 'https://discord.com/api/webhooks/...';
```

- **`ROOM_ID`** — any short string you want. This is your private session identifier. Keep it to yourself; whoever knows it can open `controller.html?room=FAR26` and connect.
- **`PEER_ID`** — leave the template as-is. It prefixes the room code with a namespace (`farewell2026-`) to avoid collisions on PeerJS's shared public server.
- **`DISCORD_HOOK`** — see below.

### 2. Set up the Discord webhook

1. Open the Discord server where you want the photo to land.
2. Go to a channel → **Edit Channel → Integrations → Webhooks → New Webhook**.
3. Give it a name, optionally set an avatar, and click **Copy Webhook URL**.
4. Paste that URL as the value of `DISCORD_HOOK` in `index.html`.

The photo posts automatically the moment you capture it — labelled `🎓 Farewell, Class of 2026 · DD Mon YYYY`. It's fire-and-forget; any failure is logged silently to the console and doesn't affect the on-screen experience.

### 3. Deploy

Fork this repository, update the CONFIG block and deploy it to any static hosting provider of your choice.

**Netlify (recommended — drag and drop, 60 seconds):**

1. Clone the repository.
```bash
git clone https://github.com/FireHead90544/FAR26
```
2. Go to [netlify.com](https://netlify.com) → drag the repository folder into the deploy zone.
3. You get the deployment URL. Done.

**Vercel:**

```bash
npm i -g vercel
vercel --yes   # run from inside the project folder
```

**GitHub Pages:**

1. Fork the repository.
2. Go to **Settings → Pages → Source → main branch / root**.
3. Your site will be at `https://<username>.github.io/FAR26/`.

> **HTTPS is required** for camera access — all three options above serve HTTPS by default, so you're covered.

---

## Usage

Assuming your deployment URL is `https://yoursite.com` (replace it with your actual URL)

### Before the event — test run

1. Open `https://yoursite.com` on your laptop → you'll see a dark terminal card: *"waiting for connection"*.
2. On your phone, navigate to `https://yoursite.com/controller.html?room=FAR26`.
3. The controller asks for camera permission → grant it. The display switches to the aurora standby screen once connected.
4. Tap **GO LIVE** → feed goes fullscreen on the display.
5. Tap the shutter → photo drops in as a polaroid, confetti fires, farewell text fades in.
6. Check your Discord channel — the image should arrive within a few seconds.

Do this test at least once on the actual venue network (or hotspot) before the event.

### On the day

| Step | You | Screen |
|------|-----|--------|
| Setup (before your turn) | Open `https://yoursite.com` on the display laptop. Load `https://yoursite.com/controller.html?room=FAR26` on your phone | Shows terminal card |
| Phone connects | — | Switches to aurora standby. Looks like art. |
| Your intro lines | Walk on stage, do your thing | Aurora glitch-text plays |
| Tap GO LIVE | — | Full-screen live camera feed |
| *"Everyone, say cheese..."* | Rotate phone to landscape, frame the crowd | Audience sees themselves |
| Tap shutter | — | Flash → polaroid snap → confetti → farewell text |
| Peace ✌️ | Walk off | Image stays on screen |

### Network note

Use your **personal hotspot** on the display laptop. Venue/college WiFi often blocks WebRTC STUN packets, which will prevent the connection from establishing. Your phone stays on its own data for the controller. The initial PeerJS handshake (~2 seconds) needs internet; after that it's direct device-to-device.

---

## Orientation / Landscape capture

The controller is designed to be used in **landscape mode** (phone rotated -90° / top pointing left). In this orientation, the camera stream fills the display screen wide, and the captured JPEG is automatically corrected to a proper landscape image before being sent to the display and Discord.

If you hold the phone portrait, the capture will also be portrait — which is fine for testing, but landscape looks better on a projector.

---

## Customisation

| What | Where |
|------|-------|
| Room code | `ROOM_ID` in `index.html` |
| Discord channel | `DISCORD_HOOK` in `index.html` |
| Farewell caption text | `polaroid-cap` text in `doCapture()` in `index.html` |
| Farewell heading | `<h1>` inside `#state-captured` in `index.html` |
| Batch year | Grep for `2026` — appears in the heading, caption, and Discord message |
| Confetti colours | `colors` array in `fireConfetti()` in `index.html` |
| Polaroid tilt angle | `rotate(-2.5deg)` in `.polaroid-wrap`'s `snap-in` keyframe |

---

## Tech stack

- **[PeerJS 1.5.4](https://peerjs.com/)** — WebRTC peer connection and video streaming
- **[canvas-confetti 1.9.2](https://github.com/catdad/canvas-confetti)** — the confetti burst
- **[JetBrains Mono](https://www.jetbrains.com/legalnotice/fonts/)** + **[Caveat](https://fonts.google.com/specimen/Caveat)** — typography
- **Discord Webhooks API** — permanent image storage
- Zero dependencies to install. No bundler. No framework.

---

## Browser support

Tested on:

- Chrome / Edge (display laptop) ✓
- Safari iOS (controller phone) ✓
- Chrome Android (controller phone) ✓

Both devices need a modern browser with WebRTC support (everything released after 2020 is fine).

---

## Security note

The room code in the controller URL (`?room=FAR26`) is the only access control. The display page shows no code or QR — just "waiting for connection" — so the audience can't connect. Keep your controller URL to yourself and change `ROOM_ID` to something less guessable if you're publishing this repo publicly before the event.

The Discord webhook URL is embedded in client-side JS, so it's visible to anyone who views source. For a one-time event this is acceptable; if you want to keep it private, remove it from the repo or add the file to `.gitignore` and substitute the webhook manually before deploying.

---

*Built for the farewell of the class of 2026. One click, one memory, permanent on the internet.*
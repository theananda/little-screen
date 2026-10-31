# Little Screen

A YouTube client where nothing plays unless a parent allows it. Parents set a PIN, then allow channels, block channels, approve single videos, block words, turn Shorts off, set a daily watch-time limit, and answer requests from the child.

It is a static web app: `index.html`, `manifest.json`, `icon.svg`. No server code, no accounts. All settings live in the browser on the child's device.

## How the rules work (in order)

1. **Blocked channel** → never plays, even from a direct link or an approved video.
2. **Approved single video** → plays (skips the word and Shorts rules).
3. **Blocked word** in title, description or tags → doesn't play.
4. **Shorts** (tagged #shorts, or 60 seconds or shorter) → doesn't play while Shorts blocking is on.
5. **Live streams** → don't play.
6. **Allowed channel** → plays.
7. Anything else → blocked. The child can tap **Ask a grown-up**; it appears under **Requests** with Approve video / Allow channel / Block channel / Dismiss.

Other protections: 3-minute idle auto-lock on the grown-up panel (and on leaving the tab), increasing lockout after 5 wrong PINs, the PIN stored only as a salted SHA-256 hash, YouTube's privacy-enhanced player (youtube-nocookie.com), click shields over the YouTube title bar and logo, our own pause and end screens so YouTube's "more videos" suggestions never show, and our own full-screen mode so the shields stay in place.

## 1. Host it

Any static host with HTTPS works (HTTPS is needed for the PIN hashing and the YouTube player):

- **Railway / any Docker host:** serve the folder with a static server, e.g. `npx serve .` or an nginx container.
- **GitHub Pages, Cloudflare Pages, Netlify:** upload the three files.
- **Testing on your own computer:** `npx serve .` then open `http://localhost:3000`.

Opening `index.html` straight from disk (`file://`) won't work well; the YouTube player needs a real web address.

## 2. Add a YouTube API key (recommended)

Without a key, allowed channels still work for shared links, and parents can approve single videos, but the home screen can't list channel videos or read video lengths.

1. Go to console.cloud.google.com → create a project.
2. **APIs & Services → Library** → enable **YouTube Data API v3**.
3. **Credentials → Create credentials → API key.**
4. Restrict it: *Application restrictions → Websites* → add your site address (e.g. `https://kids.example.org/*`). *API restrictions → YouTube Data API v3*.
5. Paste it in Little Screen → Grown-up mode → **Settings → YouTube API key**.

The free quota (10,000 units a day) is plenty: loading one channel costs about 2 units, refreshed at most every 6 hours.

## 3. Lock the device (important)

Any app-level lock can be walked around if the child can open another browser or the YouTube app. Little Screen should be the *only* way to reach YouTube on the child's device. Pair it with the device's own controls:

| Device | What to do |
|---|---|
| **iPad / iPhone** | Open the site in Safari → Share → *Add to Home Screen*. Then **Screen Time → Content & Privacy → Content Restrictions → Web Content**: *Allowed Websites Only*, add your Little Screen address, `youtube-nocookie.com`, `youtube.com`, `googlevideo.com`, `ytimg.com`, `googleapis.com`, `noembed.com`. Remove the YouTube app. For a single session, **Guided Access** keeps the child inside the app. |
| **Android** | Install from Chrome (*Add to Home screen* / *Install app*). With **Family Link**, block the YouTube app and other browsers, and in Chrome set *Only allow certain sites* with the same list as above. Or use **Screen pinning**. |
| **Windows / Mac / Chromebook** | Use a separate child account. Block other browsers and the YouTube app with Microsoft Family Safety, macOS Screen Time, or Family Link. Allow only the sites above. A Chrome kiosk shortcut (`--kiosk https://your-site`) works for a shared computer. |

Note: allowing `youtube.com` at the network level is needed for the player itself; the device lock is what stops the child from browsing youtube.com directly in another browser.

## Known limits

- Settings are per device. Use **Settings → Back up or move settings** to copy lists to another device (PIN and API key are not copied).
- Requests are seen on the same device (open Grown-up mode). There's no phone notification.
- The Shorts rule uses the #shorts tag and a 60-second cutoff. YouTube now allows Shorts up to 3 minutes, so some longer Shorts without the tag can get through. Block the channel or add a blocked word if that happens.
- Search covers videos already loaded from allowed channels (the latest ~40 per channel) plus approved videos. It does not search all of YouTube, by design.
- A determined child with access to browser developer tools could clear the site's storage. The device lock in step 3 prevents that on locked-down devices.

## Ideas for a next version

- A Burmese / English language switch for the child's screen.
- Sync between parent phone and child device (small backend, e.g. on Railway, with requests pushed to Telegram).
- Approved playlists, and per-child profiles.

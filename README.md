# Watchface Clock

A live analog clock whose dial is a photo of your own watch. It's a single-file PWA (index.html + manifest + service worker + icons), it works offline, and it has no build step or dependencies.

## Run
Any static web server works, e.g. `python3 -m http.server 8080` and then open http://localhost:8080.
Service worker / install / Wake Lock need **HTTPS** (or localhost), so host it on a static host to use it on a phone.

## Features
- Upload a watch photo (or drag & drop / paste). The dial is auto-detected and you can fine-tune it: drag to move, pinch or scroll to resize, twist or Shift+scroll to rotate, arrow keys to nudge, plus sliders for centre, dial radius, rotation, zoom, circle crop / whole photo and crop radius.
- Hand styles: dauphine, baton, sword, breguet, arrow, pencil. You can set colours, length and width, lume inlays, lume glow and shadows. Second hand: on/off, needle or lollipop style, smooth sweep, mechanical 8-beat or quartz tick.
- Clean-up of the photo's own printed hands: radial fill (with an angular median filter), blur, or flat colour. You can set its size, strength and feather.
- Date window: position presets, x/y/w/h, rounding, font, bold, colours, recessed look. It shows the day, weekday + day, or weekday, and updates at midnight.
- Themes are stored in IndexedDB with the image as an ArrayBuffer. You can switch, rename, calibrate, duplicate, delete, export/import (`.watchface.json` with the image embedded) and export all. The last-used theme is remembered, and you can swipe or use ←/→ to switch.
- Two built-in themes are drawn procedurally (original artwork): "Atelier Ivory" and "Abyss Diver".
- Accurate time: network offset (see below), time zone picker, optional digital readout (12/24 h, seconds, date).
- Display mode: fullscreen with the UI hidden and the screen kept awake (Wake Lock). Tap to show controls, double-tap or Esc to exit.

## Time sync
Sources, in order: `time.akamai.com/?ms` (ms precision, CORS), `timeapi.io` (ms, CORS), Cloudflare `cdn-cgi/trace` (1 s), and the hosting server's own `Date` header (1 s).
The app takes 4 samples and keeps the one with the shortest round trip. It sets offset = server − (t0 + rtt/2) and estimates accuracy as rtt/2 + precision/2.
It re-syncs every 15 min, when the tab becomes visible (if the last sync is more than 5 min old), when the device comes back online, and when it sees the wall clock jump against the monotonic clock. When offline it falls back to the device clock.

## Credits
The test photo used during development (not shipped): "Binkada Quartz Wristwatch with Blue Sunburst Dial and Faceted Crystal 01" by A S M Jobaer, CC BY-SA 4.0, Wikimedia Commons.

## Snap a watch (in-app camera)

Tap **Snap** in the bottom bar (or **📷 Snap a watch** in Themes, or press **C**).

- **Viewfinder:** uses the rear camera by default, with a round guide and the hint "Fill the circle with the dial, hold steady".
  - Tap to focus, where the phone supports it.
  - The torch button appears only if the camera has a torch.
  - The switch-camera button appears when there is more than one camera.
- **After the shot:**
  - The app finds the dial as an ellipse (edge points along rays, RANSAC conic fit, tried from several starting points).
  - If the dial looks tilted (minor/major axis ratio below 0.965), it straightens the ellipse back into a circle with an affine stretch around the dial centre.
  - It can also balance brightness and contrast, and reduce glare (specular highlights are blended out; this is skipped on light dials).
  - Each step has an on/off switch in the review screen. Hold the photo to compare it with the original.
- **Retake / Quick save / Use photo:**
  - **Use photo** opens the calibration editor already filled in. The default name is "Watch – <date>".
  - **Quick save** saves the theme right away.
- **Fallback:** if live camera access is unavailable or denied, the app uses the phone's own camera app (`<input type=file accept=image/* capture=environment>`), or lets you choose an existing photo. The same processing then runs.
- **Requires HTTPS:** live camera access only works over HTTPS (or on localhost).
- **Limits:**
  - Straightening is affine, so a strong close-up perspective can leave slight distortion.
  - The pivot can sit a little off when the case edge isn't concentric with the hands. Fine-tune it in the editor.

## 3D watch

The clock is drawn as layered depth in one canvas. Each layer gets a tilt projection plus parallax by height. From back to front:
- drop shadow
- case wall: an extruded cylinder that becomes visible when tilted
- bezel ring with rim lighting that follows the light
- dial photo
- recessed date wheel, seen through its aperture
- hour, minute and second hands and the centre cap, each a little higher, with soft shadows that move with tilt
- domed crystal with a moving specular highlight and sheen

**What moves the view:** Settings › 3D › "Perspective follows":
- **Auto** (the default): phone tilt on phones and tablets, the mouse or pen on computers, and a gentle idle float otherwise.
- **Phone tilt / Pointer / Still:** force one of these.

**How phone tilt behaves:**
- Readings are smoothed and soft-clamped to about ±22°.
- They are measured from how you normally hold the phone. That resting angle re-centres itself slowly (about 6 s), and you can also tap **Recentre 3D view** or press R.
- Landscape works too.

**iPhone/iPad (iOS 13+):** motion needs permission granted from a tap. A gold **✦ Enable 3D motion** button appears on the clock (and in Settings). After you allow it once, the app asks again silently on your first tap of each visit.

**Android:** works without a prompt over HTTPS.

**Reduced motion:** when the system asks for reduced motion, Auto keeps the watch still. You can still pick Phone tilt or Pointer yourself.

**Sliders:** Depth and Crystal glare. Calibrate › Dial › "3D look (this watch)" can turn 3D off for one theme or give it its own depth and glare. Display mode is supported.

## Background (per theme)

Set it under Calibrate › Dial › Background. It is also offered on the Snap review screen.
- **Black:** the watch head and dial circle, with a case rim, on pure black. This is the default for new snaps and uploads; the app then remembers your last choice.
- **Show strap:** the whole photo (case, lugs, strap or bracelet) fills the screen behind the clock, centred on the calibrated dial.
  - Use Zoom to show more or less strap, Rotation to run the strap top-to-bottom, and Quick fit › Fill screen / Whole photo.
  - The photo's edges fade to black.
  - In 3D, the whole photo tilts as one base layer, with the hands and crystal above it.
  - Snaps saved in this mode keep a much wider crop of the camera frame, so the strap is included.
- **Blurred:** the dial circle over a soft, darkened, slightly parallaxed copy of the photo.
- **App colour:** the previous look, following Settings › Background. Built-in themes use this.

## Straps (per theme)
Calibrate → **Strap** (also a *Strap* picker on the Snap review step):

- **None**: the dial only (the original behaviour).
- **Photo strap**: shows the strap from your photo (the same as Background › Show strap).
- **Drawn straps**: Oyster (3-link), Jubilee (5-link), President (rounded 3-link), Milanese mesh, Engineer (heavy link), Beads of rice, NATO (striped fabric with colour presets), Leather (smooth or croc, with stitching colour), Rubber, Tropic, FKM ridged.

Metal finishes are Steel brushed, Steel polished, Two-tone (steel with gold centre links), Yellow gold, Rose gold, Black PVD and Titanium. Fabric, leather and rubber straps have colour pickers.

Straps attach at the 12 and 6 o'clock lugs and run off-screen. Their width scales with the dial, and you can adjust it with the Width and Length sliders. The clasp or buckle is optional.

Straps are drawn as a 3D layer just below the case, so they shift with tilt and the metallic sheen moves as the watch tilts. They work on Black, Blurred and App-colour backgrounds and are saved with the theme.

## Digital · Rugged face
Open Themes → **⌚ New digital face**, or pick one of the built-in *Digital · Rugged* themes.

The face is a chunky resin case with bezel, screws, bumpers and four side buttons. It has a 7-segment LCD with faint "ghost" segments, a dot-matrix weekday, the date, PM/24H, signal and alarm indicators.

Tap the drawn buttons (or use the keyboard: **M** mode, **S** start, **X** reset, **J** adjust, **L** light):

| Mode | START (upper right) | RESET (lower right) | ADJUST (upper left) |
|---|---|---|---|
| Time | 12/24 h | positive/negative display | hourly chime on/off |
| World time | next city | previous city | next city |
| Stopwatch (1/100 s) | start/stop | split / clear split / reset | – |
| Timer | start/pause (+1 min while editing) | reset (−1 min while editing) | edit length |
| Alarm | on/off (+ while editing) | chime (− while editing) | edit hour → minute |

- **MODE** (lower left) cycles through the modes.
- **LIGHT** (front) or tapping the display turns the backlight on for about 2.6 s (blue-green or amber).
- The timer, alarm and hourly chime beep through WebAudio. This is best effort: it only works while the app is open and in the foreground. Browsers may stay silent until you've tapped the page once.

In the Face tab you can set positive or negative display, LCD tint, illuminator colour, 12/24 h, chime, case colour (Black, Matte olive, White, Red/black, Gold/black, Clear), world-time city, alarm and timer length. It uses the same synced time as the analog faces and takes part in 3D tilt: the case has depth and the LCD sits recessed under glass with glare. The default strap is resin/rubber, and every strap type is available.

## Updating
The service worker caches the app for offline use. When a new version is published, an open app picks it up in the background. The next time the app is opened or reloaded it shows the new version (close and reopen an installed PWA).

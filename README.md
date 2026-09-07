# Lift

A percentage-driven workout tracker built to the spec in `workout-app-spec.md`.
Runs as an installable web app on your Pixel. No accounts, no server, no cost.

## Files

| File | |
|---|---|
| `index.html` | The whole app — all logic, all styling |
| `manifest.webmanifest` | Lets Chrome install it to your home screen |
| `sw.js` | Offline support, so it works with no signal in the gym |
| `icon-192.png` `icon-512.png` `icon-maskable.png` | Home screen icons |

All six need to sit in the same folder.

---

## Getting it on your phone

It needs a real web address. Opening `index.html` straight from your Downloads
folder won't work — Chrome refuses to install or reliably save data from a
`file://` page. Both options below are free and take about five minutes once.

### Option A — GitHub Pages

1. Make a free account at github.com.
2. Click **+** → **New repository**. Name it `lift`. Set it to **Public**. Create.
3. On the empty repo page, click **uploading an existing file** and drag in all
   six files. Click **Commit changes**.
4. Go to **Settings** → **Pages** in the left sidebar.
5. Under Source pick **Deploy from a branch**, branch **main**, folder **/ (root)**. Save.
6. Wait about a minute, then reload. It shows your address:
   `https://YOURNAME.github.io/lift/`

### Option B — Netlify Drop

Go to app.netlify.com/drop and drag the folder containing all six files onto the
page. You get an address immediately. No account needed to start, though making
one keeps the address from expiring.

### Then install it

1. Open the address in **Chrome on your Pixel**.
2. Tap the **⋮** menu → **Add to Home screen** (or **Install app**).
3. Launch it from your home screen. It opens fullscreen with no browser bar and
   works offline from then on.

---

## Back up your log

Your workout history lives in Chrome's storage on your phone and nowhere else.
Clearing site data, uninstalling, or resetting the phone erases it.

**Settings → Export backup file** saves everything as a `.json` file. Do it every
few weeks and keep the files in Drive or anywhere off the phone. **Restore from a
backup** reads one back. The app reminds you if it's been more than two weeks.

---

## How it behaves

**Weights** are computed from the training max and always round **down** to the
nearest 5 lb. 75% of 420 is 315; 80% of 420 is 336, so you lift 335.

**Sets** are 5 / 5 / AMRAP. Every number is editable during the session —
overriding a weight doesn't change your TM or anything about the next session.

**Percentages climb within a session and across weeks.** The scaling percent does
both jobs. The mesocycle increase raises the starting point each new cycle.

**Deload week** is 3 sets at 50% of TM, and accessory set counts are halved,
rounded down. An accessory with 3 sets does 1; an accessory with only 1 set drops
out of that session entirely. It comes back at full size the following week —
deload sessions deliberately don't overwrite your saved accessory setup.

**Breaking your TM** happens on the AMRAP set, scored by Epley
(`weight × (1 + reps/30)`). The popup shows the raw estimate; if the 5% cap
applied you also get `(capped NNN)`. The new TM is staged and takes effect when
the next cycle starts. Best estimate of the cycle wins. A bad AMRAP does nothing —
TMs only go down if you edit them yourself.

**Percentages can exceed 100% of TM** if you go many cycles without a break.
That's intended. Fix it in **Settings → the lift → Position** by setting the
cycle and week by hand.

**Accessories** are remembered per core lift. Whatever you did last time is
pre-filled next time; edits stick going forward. Notes don't carry over — they
belong to the session they were written in and show up in that exercise's history.

**Skip** logs nothing and moves to the next week.

---

## Changing the app later

Edit `index.html` and re-upload it. Also bump the version in `sw.js`
(`const CACHE = 'lift-v1'` → `'lift-v2'`), or phones will keep serving the old
cached copy. Then close and reopen the app twice to pick up the change.

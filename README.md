# CRX Arm Scheduler

A shared weekly timetable for booking the CRX arm. Hourly slots, drag to select a
block, everyone sees changes live, and the whole board clears itself every Monday.

Static site — no server, no build step. Hosted on GitHub Pages, with
[Firebase Firestore](https://firebase.google.com/docs/firestore) doing the realtime sync.

---

## Setup

You need to do this once. Budget about ten minutes.

### 1. Create a Firebase project

1. Go to <https://console.firebase.google.com> and click **Add project**.
2. Name it anything (`crx-scheduler`). Turn **Google Analytics off** — it isn't needed.
3. In the left sidebar choose **Build → Firestore Database → Create database**.
4. Pick **Start in production mode** and any region (`us-west1` is closest to Berkeley).

### 2. Register a web app and copy the config

1. Project settings (gear icon) → scroll to **Your apps** → click the `</>` web icon.
2. Give it a nickname. **Do not** check "Firebase Hosting".
3. Firebase shows you a `firebaseConfig` object. Copy it.

### 3. Paste the config into `index.html`

Near the bottom of `index.html` there is a block marked `PASTE YOUR CONFIG BELOW`.
Replace the placeholder values with yours:

```js
const FIREBASE_CONFIG = {
  apiKey:            "AIzaSy...",
  authDomain:        "crx-scheduler.firebaseapp.com",
  projectId:         "crx-scheduler",
  storageBucket:     "crx-scheduler.appspot.com",
  messagingSenderId: "123456789012",
  appId:             "1:123456789012:web:abc123"
};
```

These values are **not secrets**. Firebase web configs are designed to ship inside
public client code. What actually protects the data is the security rules in step 4.

### 4. Set the security rules

Firestore Database → **Rules** tab → replace everything with the contents of
[`firestore.rules`](firestore.rules) in this repo → **Publish**.

Those rules allow anyone who loads the page to read and write the `weeks`
collection, and nothing else. That is deliberate: there are no accounts, so the
page cannot prove who anyone is. See [Security](#security) below for what that
means in practice.

### 5. Push and turn on Pages

```bash
git clone https://github.com/Joongeun/CRX_Arm_Scheduler.git
cd CRX_Arm_Scheduler
# copy index.html, firestore.rules, .nojekyll and this README in
git add .
git commit -m "Add CRX arm scheduler"
git push
```

Then in the repo: **Settings → Pages → Source: Deploy from a branch → `main` / `(root)` → Save.**

Give it a minute. The site lands at:

```
https://joongeun.github.io/CRX_Arm_Scheduler/
```

Share that link. Anyone who opens it can book — no account, no sign-in.

---

## Using it

- **Type your name** in the left sidebar. It's saved in your browser, so you type it once.
- **Drag across empty cells** to claim a block. The drag covers a rectangle, so you can
  grab Tue–Thu 2–5 PM in one motion.
- **Drag across your own slots** to release them.
- **Right-click any slot** to force-clear it, including someone else's. You'll get a
  confirmation prompt first.
- **Release my slots this week** in the sidebar wipes your bookings in the week shown.
- **‹ and ›** above the board move between weeks.

Each person gets a color derived from their name, so the board reads at a glance.

Hours that have already passed grey out and can't be booked.

### Weeks and past days

The arrows above the board move between weeks. You can go forward as far as you like,
but not back past the current week. The board opens on today, and days that have
already passed are not shown. At midnight Pacific, yesterday's column drops off on its
own, even in a tab that has been left open.

Each week is stored as one Firestore document, keyed by that week's Monday. A document
is created only when someone books a slot in that week, so browsing ahead costs
nothing. Week documents older than three weeks are deleted automatically.

---

## Changing the schedule

Near the top of the script in `index.html`:

```js
var START_HOUR = 8;    // first bookable hour, 24h clock
var END_HOUR   = 22;   // last row is END_HOUR-1, so 22 means the last slot is 9–10 PM
var DAY_COUNT  = 7;    // 7 = Mon–Sun, 5 = Mon–Fri
var TIMEZONE   = "America/Los_Angeles";
```

Edit, commit, push. Pages redeploys in about a minute.

---

## Security

There is no login, so the rules cannot restrict writes to specific people. Anyone
with the URL can book a slot, and anyone with the URL can clear someone else's.

For a club booking a shared robot arm, this is usually the right tradeoff — the
friction of accounts costs more than the occasional mistake. But be aware of it:

- The URL is effectively the password. Don't post it publicly.
- A malicious visitor could wipe the board. Nothing is permanently lost; people
  just re-book.
- The rules do cap document size and reject writes outside the `weeks` collection,
  so nobody can use your Firestore as free storage.

If you later want real accounts, the upgrade path is Firebase Anonymous Auth or
Google Sign-In plus a rule change — the data model already stores a per-person id.

## Free tier

Firestore's free tier allows 50,000 document reads and 20,000 writes per day. This
app uses one listener per open tab and one write per booking action, so a club of
twenty people will use a fraction of a percent of that.

## Files

| File | What it is |
|---|---|
| `index.html` | The entire app — markup, styles, logic, and the Firebase glue |
| `firestore.rules` | Paste into the Firebase console, Rules tab |
| `.nojekyll` | Tells GitHub Pages to serve the files as-is |

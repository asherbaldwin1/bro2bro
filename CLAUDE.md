# Bro 2 Bro Golf Club

A website where a small group of friends in Arizona track golf rounds, scores, handicaps and leaderboards. It's live on GitHub Pages and uses Firebase for login and data. Tagline: "Made by Bros, for Bros."

## About the owner (read this first)

- Asher is new to coding, VS Code and git. Use plain language, explain what you changed and why in a sentence or two, and don't assume he knows terms like "commit", "branch" or "deploy" without a quick explanation.
- **Always ask before you commit or push.** Pushing to `main` updates the live site within a minute or two, and real friends are using it.
- Keep it free: use free tools and free tiers only (GitHub Pages, Firebase Spark plan, EmailJS free plan) until he says otherwise.
- He dislikes text that wraps onto two lines in headers and buttons. Keep these to a single line on a phone.

## Files

| File | What it is | Notes |
|---|---|---|
| `index.html` | The whole app: HTML, CSS and JS in one file, plus the login screen and the Firebase bridge | Almost all changes happen here |
| `config.js` | Firebase project keys plus the admin account ID (`adminUids`) | **Never overwrite or regenerate this.** Edit only if asked |
| `firestore.rules` | Firestore security rules | Changes do NOTHING until pasted into Firebase Console → Firestore → Rules → Publish. Tell Asher whenever this is needed |
| `favicon.png`, `apple-touch-icon.png` | The B2B logo as the browser tab icon and iPhone home-screen icon | The `<link>` tags carry `?v=2`; bump it if the icon changes so browsers fetch the new one |
| `README.md` | Public description of the project shown on GitHub | Keep it accurate when features change |
| `LICENSE` | All rights reserved, Asher Baldwin | Don't replace with an open-source license unless asked |

Keep the app a single self-contained `index.html`. No build step, no bundler, no npm. Load libraries from a CDN with pinned versions.

## Architecture

- **Hosting:** GitHub Pages from `main`, at the repo root (`https://asherbaldwin1.github.io/bro2bro/`).
- **Backend:** Firebase (compat SDK v10.12.2): Auth plus Firestore. Offline persistence is on so scoring works with weak signal on the course.
- **Bridge shim:** the app was first built as a claude.ai artifact, so the app code calls `window.claude.use('db')` and `window.claude.use('user')`. A small shim in `index.html` maps those onto Firebase. `update()` is done as `set(..., {merge: true})` to keep deep-merge behavior. Keep using that API in app code rather than calling Firebase directly all over the place.
- **Sign-in:** Google (popup, falling back to redirect), email + password (sign in, create account, forgot password) and an emailed sign-in link. Spark plan limits: 5 email links/day and 150 password-reset emails/day. Google and password sign-in have no cap.
- **Data:**
  - Must be signed in to read anything. Every rule checks `active()` (signed in and not blocked; the admin can't be blocked).
  - `players/{uid}`: each user can edit only their own. Only `name`, `joined` and `pic` (a small JPEG data URL, about 10–20 KB, or `''`; there's no Firebase Storage on the free plan).
  - `rounds/{id}`: created with yourself as host. Only the host or an admin can delete.
  - `courseinfo/{id}`: only `par`, `pn`, `tees` and `added`. Pars and rating/slope added during a round are shared with the group.
  - `friends/{uidA~uidB}`: `{a, b, from, status, created, since}`. Writable only by those two users.
  - `blocked/{uid}`: `{at, by}`. Admin-only writes. A blocked player sees a "Your account has been blocked" screen.
  - `feedback/{id}`: `{by, name, kind ('bug'|'idea'), text, page, ua, scr, at, status ('open'|'prog'|'done'), notes {timestamp: text}, closed}`. Any active player can create one as themselves (status `open`, no notes). Only the admin can read, update or delete.
  - `emails/{uid}`: `{email, pw}`. Each player saves their own on load (must match their login email); only the admin can read them, for password resets.
  - `bags/{uid}`: `{clubs: [up to 14 club ids], dist: {clubId: {avg, shots, at}}}`. Private: only that user can read or write it.
  - The admin is matched by Firebase account ID (UID), not email, and that UID is hard-coded in both `config.js` and `firestore.rules`. Keep them in sync.
- **Feedback email (EmailJS):** after a ticket saves, `fbMail()` emails it to Asher. Free plan, 200 emails/month. The `EMAILJS` constant in `index.html` holds the Service ID, Template ID and Public Key. Template variables: `name`, `kind`, `title`, `message`, `page`, `device`, `time`. If an email fails, the ticket is still saved and shows in Admin → Feedback.
- **Course data:** about 12,700 real US courses from a one-time snapshot of github.com/lukhed/yardage-hub-data (about 350 fake/duplicate entries removed). Arizona is shown by default, other states are searchable.

## Security

- **Public by design:** the Firebase web `apiKey` in `config.js` and the EmailJS Public Key, Service ID and Template ID in `index.html`. GitHub secret scanning flags the Firebase key; that alert is expected. Real protection comes from the Firestore rules, Auth authorized domains and the API key's HTTP-referrer restriction.
- **Never** add to the repo: a Firebase service-account JSON (anything with `"private_key"`), the EmailJS Private Key, passwords or tokens.
- Changing data shapes or adding a field means updating `firestore.rules` too, or saves fail with "Couldn't save that change."
- The API key is locked (HTTP referrer) to `https://asherbaldwin1.github.io/*`, `https://bro2bro-4d153.firebaseapp.com/*` and `http://localhost/*`. Any new address must be added in two places: Google Cloud Console → Credentials → Browser key, and Firebase Auth → Authorized domains.

## Features: key decisions

Details are in the code; these are the choices and preferences behind them.

- **Navigation:** Home has the brand header, then Play (full width), then Rounds, Friends, Leaderboards and Range. Settings lives in the account menu (top right), not on Home. The account menu ends with Sign out in oxblood. Feedback is top left on every signed-in page.
- **Back pills:** every sub-page gets a "‹ …" pill at the top of its first card, made with `crumb(key,label)` (`CRUMB` says where each goes). Use it for new sub-pages instead of a back button at the bottom.
- **Popups over inline confirms** when an inline confirm would squash text (for example "Remove friend?").
- **Friends scope the app:** pickers, Rounds' Friends toggle, Live now and Leaderboards show only your circle. You don't appear in your own friends list.
- **Friend profiles** show stats, a scoring mix and recent rounds. **No graphs:** Asher removed the handicap graph on purpose and prefers just the number.
- **In-round layout is phone-first:** scorecard, hole strip, "Hole N" with par, one-tap score buttons, then inline hole navigation. Anyone in the round can score for anyone.
- **Leaderboards (first version):** Asher will give more detail; **don't redesign this without him.**
- **Range:** one choice page for "Add to my average" or "Start fresh", with the erase warning on that same page. Asher doesn't want an extra confirm step.
- **Admin** is reached from Settings or the account menu. It can't change a player's email or password directly (needs the paid Blaze plan), so it offers a password reset email instead. Removing a player deletes their profile but not their login.
- **View as (admin):** a player's Admin page has "View as …", which shows the app as they see it. It's read-only on purpose: real impersonation needs the paid plan. The Firebase bridge refuses every write while `window.B2B_VIEW` is set, an oxblood banner shows who you're viewing, and Exit reloads the page. Their Range stays hidden because bags are private.

## Design system: "Clubhouse"

Upscale, clean, simple, elegant, with some color. Rejected directions: bright fairway green (too casual) and red/white/black (too sporty).

- **Palette:**

  | Name | Hex |
  |---|---|
  | Racing green | `#0F3D2E` |
  | Green 2 | `#1E5A43` |
  | Brass | `#B8925A` / `#D9B77E` |
  | Cream | `#F7F3EA` |
  | Parchment | `#EFE6D3` |
  | Oxblood (danger) | `#7A1F2B` |
  | Green-black | `#0B1411` |

- **Light mode:** cream page and white cards. Racing green for primary actions, selections and the current hole. Brass for accents.
- **Dark mode:** green-black page, `#16211C` cards and cream text. Brass is the accent and the primary color.
- **Same sizes in both modes:** a dark-mode style must never change a size.
- **Rules:**
  - Flat colors: no gradients, glows or decorative circles.
  - Hairline borders instead of heavy shadows.
  - Plain icons and generous spacing.
- **Fonts:** Barlow Condensed for headings and Public Sans for body text. Asher likes these, so keep them.
- **Header:** centered. "Bro 2 Bro **Golf Club**", then "Made by Bros, for Bros.", then "EST. 2026" in brass between hairlines.
- **Logo:** a racing-green rounded tile with a cream "B2B", a brass "2" and a brass dot. Used as the favicon and on the login screen, not in the Home header.
- **Phone first:** big tap targets and 16px minimum on inputs (prevents iOS zoom).
- **Computer screens (960px and wider):** one `@media (min-width:960px)` block widens the page to 1080px. Home's four smaller cards sit in one row under Play. Rounds, Friends (and profiles), Leaderboards, Settings and Admin get `.cols`, and `pack()` stacks their cards into two columns, each card going to the shorter one. Play and Range stay one 720px column. Don't use CSS multi-column (`column-count`/`column-span`): Safari drew the Admin page twice with it.

## Workflow

Project skills in `.claude/skills/`:

- `/test-locally`: preview a change on a local copy with sample data, at phone and desktop width, light and dark.
- `/ship`: summarize, ask, flag rules changes, push, confirm the live site updated, remind Asher to hard-refresh.
- `/security-check quick` or `/security-check deep`: the security checklist (quick before pushes that touch logins, data or rules; deep about monthly).

User data lives in Firebase, so deploying a new `index.html` never wipes rounds or accounts.

### Tools on this machine

- git is installed and pushes to GitHub over HTTPS using the GitHub CLI's sign-in.
- The GitHub CLI is at `~/bin/gh` and is **not** on the PATH. Call it by full path (for example `~/bin/gh repo view`).
- No Node.js, npm or Java, so the Firebase emulator isn't available locally.

### Testing on the live site

- Asher's bro2bro account signs in with **email and password**. Don't use "Continue with Google" to test as Asher: it could create a second account or change how his account signs in. Ask Asher to sign in himself.
- Asher's account is the admin, so it can't prove the regular-player limits. Clean up any test data you create.

## Wishlist

- More detail for Leaderboards (waiting on Asher).
- A custom domain someday (about $12/year). Set it as the Pages custom domain, then add it in the two places listed under Security.

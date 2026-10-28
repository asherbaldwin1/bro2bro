# Bro 2 Bro Golf Club

A website where a small group of friends in Arizona track golf rounds, scores, handicaps and leaderboards. It's live on GitHub Pages and uses Firebase for login and data. Tagline: "Made by Bros, for Bros."

## About the owner (read this first)

- Asher is new to coding, VS Code and git. Use plain language, explain what you changed and why in a sentence or two, and don't assume he knows terms like "commit", "branch" or "deploy" without a quick explanation.
- **Always ask before you commit or push.** Pushing to `main` updates the live site within a minute or two, and real friends are using it.
- Keep it free: use free tools and free tiers only (GitHub Pages, Firebase Spark plan) until he says otherwise.
- He dislikes text that wraps onto two lines in headers and buttons. Keep these to a single line on a phone.

## Files

| File | What it is | Notes |
|---|---|---|
| `index.html` | The whole app: HTML, CSS and JS in one file, plus the login screen and the Firebase bridge | Almost all changes happen here |
| `config.js` | Firebase project keys plus the admin account ID (`adminUids`) | **Never overwrite or regenerate this.** Edit only if asked. The Firebase web `apiKey` is public by design (see Security) |
| `firestore.rules` | Firestore security rules | Changes do NOTHING until pasted into Firebase Console → Firestore → Rules → Publish. Tell Asher when this is needed |
| `README.md` | Public description of the project shown on GitHub | Keep it accurate when features change |
| `LICENSE` | All rights reserved, Asher Baldwin | Don't replace with an open-source license unless asked |

Keep the app a single self-contained `index.html`. No build step, no bundler, no npm. Load libraries from a CDN with pinned versions.

## Architecture

- **Hosting:** GitHub Pages from `main`, at the repo root (`https://asherbaldwin1.github.io/bro2bro/`). No custom domain yet. Adding one later means setting a Pages custom domain and adding it to Firebase Auth → Authorized domains.
- **Backend:** Firebase (compat SDK v10.12.2): Auth plus Firestore. Offline persistence is on so scoring works with weak signal on the course.
- **Bridge shim:** the app was first built as a claude.ai artifact, so the app code calls `window.claude.use('db')` and `window.claude.use('user')`. A small shim in `index.html` maps those onto Firebase. `update()` is done as `set(..., {merge: true})` to keep deep-merge behavior. Keep using that API in app code rather than calling Firebase directly all over the place.
- **Sign-in:** Google (popup, falling back to redirect), email + password (sign in, create account, forgot password) and an emailed sign-in link. Spark plan limits: 5 email links/day and 150 password-reset emails/day. Google and password sign-in have no cap.
- **Data:**
  - Player profiles: each user can edit only their own.
  - Rounds: created with yourself as host. Only the host or an admin can delete.
  - `friends/{uidA~uidB}`: `{a, b, from, status, created, since}`. Writable only by those two users.
  - `blocked/{uid}`: `{at, by}`. Admin-only writes. Every other rule checks `active()` (signed in and not blocked; the admin can't be blocked). A blocked player sees a "Your account has been blocked" screen.
  - `feedback/{id}`: `{by, name, kind ('bug'|'idea'), text, page, ua, scr, at, status ('open' New |'prog' In Progress |'done' Complete), notes {timestamp: text}, closed}`. Any active player can create one as themselves (status `open`, no notes). Only the admin can read, update or delete. After a ticket saves, `fbMail()` also emails it to Asher through EmailJS (see below).
  - `emails/{uid}`: `{email, pw}`. Each player saves their own on load (must match their login email); only the admin can read them, for password resets.
  - `bags/{uid}`: `{clubs: [up to 14 club ids], dist: {clubId: {avg, shots, at}}}`. Private: only that user can read or write it.
  - Must be signed in to read anything. The admin is matched by Firebase account ID (UID), not email, and that UID is hard-coded in both `config.js` and `firestore.rules`. Keep them in sync.
- **Feedback email (EmailJS):** free plan, 200 emails/month, connected to Asher's EmailJS account. The `EMAILJS` constant in `index.html` holds the Service ID, Template ID and Public Key, which are public by design. The EmailJS **Private Key** must never go in the repo. Template variables: `name`, `kind`, `title`, `message`, `page`, `device`, `time`. If an email fails, the feedback is still saved in Firestore and shows in Admin → Feedback.
- **Course data:** about 12,700 real US courses from a one-time snapshot of github.com/lukhed/yardage-hub-data (about 350 fake/duplicate entries removed). Arizona is shown by default, other states are searchable. Pars and rating/slope added during a round are shared with the group.

## Security

- GitHub secret scanning flags the Firebase `apiKey` in `config.js`. It's a public identifier, not a secret. Real protection comes from the Firestore rules, Auth authorized domains and an HTTP-referrer restriction on the key in Google Cloud Console.
- **Never** add a Firebase service-account JSON (anything with `"private_key"`), passwords or tokens to the repo.
- If you change data shapes, check that `firestore.rules` still allows the writes, and remind Asher to publish the rules.
- The rules restrict fields: `players` docs may only hold `name`, `joined` and `pic` (a profile photo stored as a small JPEG data URL, about 10–20 KB, or `''` when removed; there's no Firebase Storage on the free plan); `courseinfo` only `par`, `pn`, `tees` and `added`. Adding a new field means updating the rules too, or saves will fail with "Couldn't save that change."
- The API key is locked (HTTP referrer) to `https://asherbaldwin1.github.io/*`, `https://bro2bro-4d153.firebaseapp.com/*` and `http://localhost/*`. A new address (custom domain, another host) must be added to the key in Google Cloud Console → Credentials → Browser key, and to Firebase Auth → Authorized domains.

## Features (current)

- **Navigation:**
  - Home has the centered brand header, then Play (full width), then Rounds, Friends, Leaderboards and Range. A lone last card stretches full width.
  - Each page has a top bar with "‹ Home" plus the page title. Hash routes: `#play`, `#rounds`, `#people`, `#leaders`, `#range`, `#settings`, and the back button works.
  - Top-right on every page: a light/dark toggle (saved per device, light by default) and an account menu (photo or person icon) with Settings, Friends, My rounds and Invite friends, plus Admin for the admin, then Sign out (oxblood, below a hairline). Settings lives in that menu, not on Home.
  - Sub-pages get a "‹ …" back pill at the top of their first card, made with `crumb(key,label)`; `CRUMB` says where each goes: friend profile → All friends, admin player → All players, feedback ticket → Queue/Complete, Range club or bag edit → Your bag, rating form → Rounds, add a course → Courses. Use it for any new sub-page instead of a back button at the bottom.
  - Top-left on every signed-in page: a Feedback button ("Feedback" pill on Home, icon only next to "‹ Home" on other pages). It opens a bottom sheet: Something's broken / Idea, a text box, and it auto-attaches the page (including course and hole during a live round), device and screen size. The page title shrinks a little (`fitTtl`) on narrow phones so it never overlaps.
- **Friends:**
  - Every user has a friend code like `RXA-2WW`. Search by name or code, plus "People you've played with" suggestions.
  - Flow: Add → Requested → Accept/Decline. Sent requests can be cancelled and friends removed.
  - Profiles: tap a name in "Your friends" (which lists friends only, not you; each row has a red Remove pill that opens a centered "Remove friend?" popup, `rmPaint()` into `#popk`) to open their profile (`profUid`, a history step on `#people`, so back works). It shows the photo, "Member since" (from `players.joined`), friend code, stat tiles (handicap, rounds, best round, average, birdies, par or better %), a scoring mix bar, Around the club (rounds with you, favorite course, last played, friends) and the last 5 rounds. The handicap graph (score differentials) was removed on purpose: Asher prefers just the number. All-time stats only, built from existing data.
  - Friends scope the app: player pickers, the Rounds "Friends" toggle, Live now and Leaderboards show only your circle. Guests can be added to a round by name.
- **Rounds:**
  - My rounds by default, with a Friends toggle. Unfinished rounds sit on top with Resume and Delete (or Leave round).
  - Saved rounds have a trash icon with an inline confirm. Rounds with blank holes are tagged Incomplete.
- **In-round layout (phone-first):**
  - Top to bottom: the scorecard, the hole strip, "Hole N" with Par, then one-tap score buttons per player (1 through triple bogey plus "+ More"; tap again to clear).
  - Below the scores, inline hole navigation: "‹ Hole 1 · Hole 2 of 18 · Hole 3 ›". Swiping also changes holes.
  - The bottom bar has Save round and Discard round (danger color), or Leave round for non-starters.
  - Scoring works for solo or group rounds. Each player can score on their own phone, or one person can score for everyone.
- **Leaderboards (first version):** All time / season toggle and a "Club leader" banner. Boards: Handicap index, Best round, Scoring average, Birdies or better, Par or better %, Rounds played. Guests are excluded. **Asher will give more detail; don't redesign this without him.**
- **Range:** pick up to 14 clubs (Driver through Lob Wedge, plus Putter). Tap a club, enter 5 range shots in yards, and Save average stores the rounded average. A club that already has data first asks "Add to my average" (combines with saved shots, keeping the latest 100) or "Start fresh" (its erase warning is on the same page; Asher wants one choice page, no extra confirm step). Putter has no distance.
- **Admin (`#admin`, admin only):** reached from Settings → Manage players or the account menu. Lists every player (search by name or code). A player's page shows stats and lets the admin rename them, send a password reset email (password accounts only), block or unblock them, remove their photo, or remove their player profile (deletes `players/{uid}`; their login stays and must be deleted in Firebase Console → Authentication). It can't edit friend links or bags. Changing someone's email or password directly needs the Admin SDK (paid Blaze plan), so it isn't offered.
- **Feedback queue (Admin → Feedback tab):** Queue (New and In Progress) and Complete toggles with counts. A ticket shows the text, sender, time, page, device and screen, a New / In Progress / Complete status picker, private notes (add as many as needed) and Delete for good with an inline confirm. The New count shows in the account menu next to Admin and in Settings → Admin, and a red dot sits on the admin's profile picture while any ticket is New.
- **Settings:**
  - Account card, sign-in info with Sign out, invite instructions with Copy site link.
  - Device preferences: home state (AZ) and usual tees (Red, White, Blue, Black or Gold; White by default).
  - An About section explaining how the handicap is calculated.

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
- **Rules:**
  - Flat colors: no gradients, glows or decorative circles.
  - Hairline borders instead of heavy shadows.
  - Plain icons and generous spacing.
- **Fonts:** Barlow Condensed for headings and Public Sans for body text. Asher likes these, so keep them.
- **Header:**
  - Centered, every line on a single line.
  - Order: "Bro 2 Bro **Golf Club**", then "Made by Bros, for Bros.", then "EST. 2026" in brass between hairlines.
- **Logo:** a racing-green rounded tile with a cream "B2B", a brass "2" and a brass dot. Used as the favicon (`favicon.png`, `apple-touch-icon.png`) and on the login screen, not in the Home header.
- **Phone first:** big tap targets and 16px minimum on inputs (prevents iOS zoom). Check every change at phone width in both light and dark.
- **Computer screens (960px and wider):** one `@media (min-width:960px)` block widens the page to 1080px. Home puts Rounds, Friends, Leaderboards and Range in one row under Play. `render()` tags `#view` with `.cols` on Rounds, Friends (and profiles), Leaderboards, Settings and Admin, then `pack()` moves each run of cards into two `.mcol` columns, each card going to the shorter one (masonry, no gaps). Chip rows, notes, stat tiles, the profile header and Admin lists (`.admlist`, rows split in two inside one card) span the full width. Play and Range get `.narrow`, one 720px column. Don't use CSS multi-column (`column-count`/`column-span`): Safari drew the Admin page twice with it. Phones are untouched. Check new pages at desktop width too.

## Workflow

1. Make changes in `index.html`. Keep the style consistent with what's already there.
2. Test locally if useful (for example `python3 -m http.server`). Note that Google sign-in only works on authorized domains, so login may need the live site.
3. Summarize the change in plain words and ask Asher before committing and pushing to `main`.
4. After pushing, remind him to hard-refresh the live site (Pages can take a minute or two).
5. User data lives in Firebase, so deploying a new `index.html` never wipes rounds or accounts.

### Tools on this machine

- git is installed and pushes to GitHub over HTTPS using the GitHub CLI's sign-in.
- The GitHub CLI is at `~/bin/gh` and is **not** on the PATH. Call it by full path (for example `~/bin/gh repo view`).
- No Node.js, npm or Java, so the Firebase emulator isn't available locally.

### Testing on the live site

- Asher's bro2bro account signs in with **email and password**. Don't use "Continue with Google" to test as Asher: it could create a second account or change how his account signs in. Ask Asher to sign in himself.
- Asher's account is the admin, so it can't prove the regular-player limits. Clean up any test data you create.

## Wishlist

- More detail for Leaderboards (waiting on Asher).
- A custom domain someday (about $12/year).

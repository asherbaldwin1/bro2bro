# Bro 2 Bro Golf Club

*Made by Bros, for Bros.*

A website where a small group of friends tracks golf rounds, scores, handicaps and leaderboards.

**Live site:** https://asherbaldwin1.github.io/bro2bro/

## What you can do

- **Play a round:** score solo or with friends, hole by hole, live on your phone. One person can score for everyone, or each player can score on their own phone.
- **Rounds:** see your round history and your friends' rounds.
- **Friends:** add friends by name or friend code.
- **Leaderboards:** handicap index, best round, scoring average, birdies and more.
- **Courses:** about 12,700 US courses built in, with Arizona shown first.

On iPhone, open the site in Safari and tap **Share → Add to Home Screen** to get an app icon.

## How it's built

| Piece | What it does |
|---|---|
| `index.html` | The whole app in one file: pages, styling and code |
| `config.js` | Firebase project settings and the admin account ID |
| `firestore.rules` | Database security rules (a copy of what's published in Firebase) |
| [GitHub Pages](https://pages.github.com/) | Hosts the site for free from the `main` branch |
| [Firebase](https://firebase.google.com/) | Sign-in (Google, email and password, or email link) and the database for rounds, scores and friends. Free plan |

There's no build step: the files in this repo are exactly what the site serves.

## Updating the site

- Changing `index.html` and pushing to `main` updates the live site within a minute or two. Hard-refresh to see it.
- Rounds, scores and friends live in Firebase, so updating the site never erases them.
- Editing `firestore.rules` here does **not** change the live rules. Paste it into Firebase Console → Firestore → Rules and click **Publish**.

## Security

- The Firebase `apiKey` in `config.js` is meant to be public. It's locked to this site's address, and the data is protected by the database rules.
- Never add passwords, private keys or Firebase service-account files to this repo.

## License

Copyright © 2026 Asher Baldwin. All rights reserved. This code is shared for viewing only and may not be copied, reused or hosted elsewhere without permission. See [LICENSE](LICENSE).

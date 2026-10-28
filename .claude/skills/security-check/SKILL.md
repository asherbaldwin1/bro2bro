---
name: security-check
description: Run Bro 2 Bro's security check. "/security-check quick" (or "run the quick security check") checks the files, git history and GitHub without any logins. "/security-check deep" (or "run the deep security check") does the quick check, then walks Asher through read-only checks in Firebase Console and Google Cloud Console. Use before any push that touches logins, data or the rules, and about once a month.
argument-hint: quick | deep
---

# Security check

Run the level in `$ARGUMENTS`: `quick` or `deep`. If it's missing, ask which one. Suggest the **deep check about once a month**, and the **quick check before any push that touches logins, data or the rules**.

Report every item in plain language as **Pass / Warning / Problem**, with one sentence on what you found. **Don't fix anything without asking.** Asher makes any console changes himself.

## Quick check

Must not need Asher: no logins, no console visits. Use only the files, git and the GitHub CLI at `~/bin/gh` (not on the PATH). Check:

1. **No secrets** in the files or the git history (`git log -p --all`): passwords, tokens, anything with `"private_key"`, the EmailJS Private Key. The Firebase web `apiKey` and the EmailJS Public Key, Service ID and Template ID are public by design.
2. **Escaping:** all player-entered text (player names, feedback text, notes, course names and info, guest names) passes through `esc()` before it's put into the page.
3. **Rules coverage:** `firestore.rules` covers every collection and field the app writes, with owner-only writes, size limits, and private data (`emails`, `bags`, `feedback`) limited to the owner or the admin.
4. **Admin actions** (rename a player, remove a player or their photo, block or unblock, the feedback queue, reading `emails`) are enforced by the rules, not just hidden in the app.
5. **Admin UID** matches in `config.js` (`adminUids`) and `firestore.rules`.
6. **CDN libraries** use exact pinned versions (for example `firebase@10.12.2`, never a bare `@10` or `latest`).
7. **GitHub security alerts:** `~/bin/gh api repos/asherbaldwin1/bro2bro/secret-scanning/alerts`, plus Dependabot and code scanning if enabled. The Firebase web API key alert is expected and fine.
8. **Live site matches `main`:** compare a hash of `curl -s https://asherbaldwin1.github.io/bro2bro/index.html` with `git show origin/main:index.html` (run `git fetch` first).

## Deep check

Do everything in the quick check first. Then walk Asher through signing in to **Firebase Console** (console.firebase.google.com, project `bro2bro-4d153`) and **Google Cloud Console** in the browser pane **himself**: never type his password or click sign-in buttons for him. **Only read; never change a setting**, even if something looks wrong. Note it as a Problem and let him decide. Check:

1. The **published Firestore rules** (Firestore → Rules) match `firestore.rules` exactly.
2. **Authentication → Settings → Authorized domains** lists only our real addresses: `asherbaldwin1.github.io`, `localhost`, and the Firebase defaults (`bro2bro-4d153.firebaseapp.com`, `bro2bro-4d153.web.app`).
3. **Authentication → Sign-in method** has only the ones we use enabled: Google, Email/Password, and Email link (passwordless sign-in).
4. **Authentication → Users:** read the list to Asher and ask him to confirm he recognizes every account.
5. **Google Cloud → APIs & Services → Credentials:** the browser API key is still limited (HTTP referrers) to `https://asherbaldwin1.github.io/*`, `https://bro2bro-4d153.firebaseapp.com/*` and `http://localhost/*`.
6. **Firebase Usage** (Firestore usage and Authentication usage): no unusual spikes in reads, writes or sign-ins.
7. Ask Asher to confirm **two-step verification** is on for his GitHub and Google accounts.

End by reminding Asher to **sign out of the browser pane** (Firebase and Google Cloud).

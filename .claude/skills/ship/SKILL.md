---
name: ship
description: Commit and push Bro 2 Bro changes to the live site safely. Summarizes in plain words, asks Asher before committing, flags firestore.rules changes that need publishing, pushes to main, confirms the live site updated, and reminds him to hard-refresh. Use whenever Asher says to push, ship, or put a change live.
---

# Ship

Pushing to `main` updates the live site that real friends use, so follow every step in order.

## 1. Summarize and ask

1. Run `git status` and `git diff` to see exactly what will go out.
2. Tell Asher in plain words what changed and why, in a few short bullets. Don't use jargon without a quick explanation.
3. **Ask before committing.** Wait for a clear yes. A yes to an earlier push doesn't count for this one.

Only stage the files that belong to the change. Never commit:
- `.claude/launch.json` (it points at a temporary folder)
- scratch or test files
- anything secret (see Security in CLAUDE.md)

## 2. Flag rules changes

If `firestore.rules` is in the diff, say so clearly. **Pushing does not update the rules.** Asher has to paste the file into Firebase Console → Firestore → Rules → Publish. If the new app code needs the new rules, he should publish them **before** you push. Otherwise saves will fail with "Couldn't save that change."

If the change touches logins, data or the rules, suggest running `/security-check quick` first.

## 3. Commit and push

1. Commit with a short, clear message. End it with the attribution lines this session asks for.
2. Push: `git push origin main`.
3. Confirm `git status` is clean, apart from files that are deliberately left out.

## 4. Confirm the live site updated

1. Watch the Pages deploy: `~/bin/gh run list -R asherbaldwin1/bro2bro -L 1`. Wait (in the background, not by polling quickly) until `pages build and deployment` shows `completed success`. It usually takes about a minute.
2. Compare the live page with what you pushed. A hash of `curl -s https://asherbaldwin1.github.io/bro2bro/index.html` should match `git show HEAD:index.html`. If it doesn't match yet, wait a minute and check again (the CDN can lag).
3. Tell Asher it's live, or what's still pending.

## 5. Remind Asher to hard-refresh

Tell him to hard-refresh the live site (**Cmd + Shift + R** on the Mac, pull down to reload on a phone) so he sees the new version. Mention anything worth checking by hand, especially in Safari.

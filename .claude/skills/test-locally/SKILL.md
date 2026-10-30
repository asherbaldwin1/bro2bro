---
name: test-locally
description: Preview and check a Bro 2 Bro change locally before shipping. Serves a copy of the site from a temp folder, confirms the port, shows signed-in pages with sample data, and checks phone width (light and dark) plus desktop width. Use after any change to index.html that shows up on screen.
---

# Test locally

## 1. Serve a copy from a temp folder

macOS blocks the test server from reading the Documents folder, so serve a **copy**:

1. Copy `index.html`, `config.js`, `favicon.png` and `apple-touch-icon.png` into `<scratchpad>/site/` (this session's scratchpad directory).
2. Make sure `.claude/launch.json` has a `site` configuration pointing at that folder. The path changes every session, so rewrite it if needed:
   ```json
   {"name":"site","runtimeExecutable":"bash","runtimeArgs":["-c","cd <scratchpad>/site && exec python3 -m http.server ${PORT:-8000}"],"port":8000}
   ```
3. Start it with `preview_start {name:"site"}`.
4. **After every edit, copy the files again** and reload the page. The server only sees the copy.

## 2. Confirm which port actually responds

The preview may say it moved to another port when 8000 is busy, and an old server from an earlier session may still be holding 8000 and serving stale files.

- Check with `lsof -nP -iTCP:8000 -iTCP:8001 -sTCP:LISTEN`.
- Load the page and confirm it's the new copy (for example, look for the change you just made).
- If the wrong server answers, `preview_stop` and start again. `${PORT:-8000}` in the command lets the server follow the port the preview assigns.

## 3. View signed-in pages with sample data

Google sign-in doesn't work on localhost, and you must never sign in as Asher. Instead, in `javascript_tool` after the page loads (wait about 1.5 seconds):

```js
document.getElementById('gate').hidden=true;          // hide the login screen
me='u1';state='ok';canWrite=true;meName='Asher';meEmail='test@example.com';
owner=true;                                          // only when checking Admin
D['players/u1']={name:'Asher',joined:Date.parse('2026-09-02')};
D['players/u2']={name:'Tyler Johnson',joined:Date.parse('2026-10-05')};
D['friends/u1~u2']={a:'u1',b:'u2',status:'accepted',from:'u1'};
// rounds/{id}: {cn,date,n:18,par:[…],status:'done',rating,slope,host,players:{u1:{uid:'u1',o:1}},scores:{u1:[…]}}
// also feedback/{id} and bags/{uid} when a page needs them (see Data in CLAUDE.md)
real.players=true;
go('people');                                        // or openProf('u2'), go('admin'), etc.
```

Rules for sample data:
- It lives only in this browser tab's memory. **Never click anything that saves** (Save, Remove friend, Delete, Block, Send feedback). Those would try to write to the real database.
- The preview can show real club data if the browser pane still has a Firebase session. Treat it as read-only and don't mention players' details beyond what's needed.
- To switch the theme from code: `theme='dark';document.documentElement.dataset.theme='dark';lastNav=null;render()`.

## 4. Check sizes and themes

1. **Phone:** `resize_window {preset:"mobile"}`. Screenshot in light, then in dark. Check:
   - headers and buttons stay on one line
   - nothing is cut off or squashed
   - there are no console errors (`read_console_messages`)
2. **Desktop:** `resize_window {width:1280,height:800}`. Screenshot the changed pages in light and dark. Two-column pages should pack cleanly with no big gaps.
3. **Every page at phone (375), tablet (820) and desktop (1280) width**, not just the one you changed. With sample data loaded, run a quick check on each page for:
   - anything wider than the screen
   - buttons, chips, page titles and card titles that wrap onto two lines
   - text cut off inside its box
   - the tee dot centered in its box

   Ignore what's two lines on purpose: score buttons (number over label), list rows with a description under the title, and screen-reader-only labels (`.sr`).
4. **Mac Safari draws form controls its own way.** This browser can't show that. Dropdowns are styled with `appearance:none` and a fixed height so they look the same everywhere; keep it that way, and never rely on the browser's built-in look for `select`, `input` or buttons.
5. **Same sizes in both modes:** for layout changes, compare every element's size in light vs dark on each affected page. They must match.
6. **Reset the size** when done: `resize_window {preset:"desktop"}`.

Share a screenshot as proof, and say plainly what you couldn't check (for example Safari, real sign-in, or a live round).

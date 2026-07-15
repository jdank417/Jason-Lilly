# Jason-Lilly

A drag-and-drop weekend planner for the anniversary trip (July 31 – August 2, 2026).
Open `index.html` in a browser — no build step, no install.

## Features

- Drag ideas from the Idea Locker onto Friday/Saturday/Sunday
- Filter ideas by weather, category, or ★ Favorites
- Click ✎ on any idea to edit its name, place, time, or note
- Click ★ to favorite an idea
- Load a starter plan, or add your own ideas
- "Copy plan as text" to paste the board anywhere
- Saves automatically to your browser (localStorage)

## Live sync with your partner (optional)

By default the board only saves to your own browser. To make it update live
between two people (e.g. you and Lilly, on two different phones/computers):

1. Go to [console.firebase.google.com](https://console.firebase.google.com) →
   **Add project** (the free tier is plenty — skip Analytics if you want).
2. In the project, go to **Build → Realtime Database → Create Database**,
   and start in **test mode** (fine for a private, low-stakes planning board
   between two people — just note that test-mode rules expire after 30 days
   and will need renewing, or you can open them permanently).
3. Click the gear icon → **Project settings** → scroll to "Your apps" →
   add a **Web app** → copy the `firebaseConfig` object it shows you.
4. Open `index.html`, find `FIREBASE_CONFIG` near the top of the `<script>`
   block, and paste in your `apiKey`, `authDomain`, `databaseURL`, and
   `projectId`.
5. Reload the page on both devices — you should see "Synced live" near the
   top-left once both are connected. Changes either of you makes (dragging
   ideas, adding notes, favoriting) show up on the other's screen.

Without this step the app still works fully, it just won't sync between
devices.

### How sync actually works

Rather than overwriting the entire board on every change, the app syncs each
piece independently: each day's list, each idea's favorite state, and each
idea's edits are stored and written separately. That means two people can
drag different ideas, favorite different things, or edit different notes at
the exact same moment with zero conflict — nothing gets clobbered.

Every synced piece also carries a **revision number**. A device only accepts
incoming data that is *newer* than what it already has, and if it sees the
server holding something *older* than its own copy, it pushes its copy back
up (self-healing). So the winner is always the most recent edit — never
"whichever message happened to arrive last," which is what previously caused
the board to flash correct data and then revert on refresh.

Two implementation notes, for the curious:

- Lists are stored JSON-encoded because Realtime Database silently deletes
  empty arrays — stored raw, "I cleared Friday" would never sync and the
  other device would resurrect the old list.
- The data lives at a versioned path (`boards/anniversary-2026-v3`), so a
  stale cached copy of the page writing the old data shape physically can't
  corrupt the current board. The sync badge also shows a device count
  ("Synced live · 2 devices") so you can see both ends are connected.

The one case that still resolves as "newest wins" rather than merging is two
people reordering the *same day's list* within the same second — rare enough
in practice not to matter.

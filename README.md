# Jason & Lilly's Anniversary Chart

A drag-and-drop weekend planner for Jason & Lilly's one-year anniversary trip
(July 31 – August 2, 2026). Open `index.html` in a browser — no build step,
no install.

## Features

- Drag ideas from the Idea Locker onto Friday/Saturday/Sunday
- Filter ideas by weather, category, or ★ Favorites
- Click ✎ on any idea to edit its name, place, time, or note
- Click ★ to favorite an idea
- 7 starter plans (Harbor Classic, Cape Ann Loop, Ipswich & Essex, Rainy-Day
  Rescue, Salem Staycation, Salem History Walk, Foodie & Brew Trail), or add
  your own ideas
- Phone number and website links right on any idea that names a specific
  business or venue
- "Copy plan as text" to paste the board anywhere
- Saves automatically to your browser (localStorage)

## A note on the idea data

Every phone number and website in the locker was looked up, not guessed —
guessing a phone number for a real business is exactly the kind of thing that
sends someone to the wrong place. Two entries that shipped earlier ("Friday
bingo at St. John the Baptist" and "Saturday bingo at Saint Mary Italian
Church") turned out to be stale: St. John's bingo ended in 2016, and Saint
Mary's Italian Church stopped operating as a parish in 2003. Both were
removed and replaced with verified alternatives. Hours, prices, and seasonal
schedules (marked in the notes where relevant, e.g. the Schooner Fame and
Salem Ferry both run May–October) can still change after the fact — that's
what the phone numbers are for.

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

Every synced piece also carries a **revision** — a counter plus the id of the
device that wrote it, never a wall-clock timestamp. (An earlier version of
this used `Date.now()` as the revision, which meant two phones with slightly
different clocks could make the *older* edit always win — the fix is a
[logical clock](https://en.wikipedia.org/wiki/Logical_clock), not a wall
clock.) A device only accepts incoming data ordered *after* its own copy; if
it sees the server holding something ordered *before* its own copy, it
pushes its copy back up (self-healing). Two devices that both write the same
revision number at the same moment still resolve identically on both ends,
by comparing device ids as a tiebreaker. So the winner is always well-defined
— never "whichever message happened to arrive last," which is what
previously caused the board to flash correct data and then revert.

A few implementation notes, for the curious:

- Lists are stored JSON-encoded because Realtime Database silently deletes
  empty arrays — stored raw, "I cleared Friday" would never sync and the
  other device would resurrect the old list.
- The data lives at a versioned path (`boards/anniversary-2026-v4`), so a
  stale cached copy of the page writing an old data shape physically can't
  corrupt the current board. The footer shows a `BUILD` number — if your
  phone and Lilly's ever show different numbers, one of you is on a cached
  copy and needs a hard refresh.
- The sync badge shows a live device count ("Synced live · 2 devices"),
  counted by distinct device id rather than by connection, so two tabs open
  on the same phone still count as one device.

The one case that still resolves as "newest wins" rather than merging is two
people reordering the *same day's list* within the same second — rare enough
in practice not to matter.

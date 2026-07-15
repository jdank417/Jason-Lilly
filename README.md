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
devices. Note: if you're both editing the exact same idea's note at the same
moment, the last save wins — there's no merge, just a straightforward
whole-board sync.

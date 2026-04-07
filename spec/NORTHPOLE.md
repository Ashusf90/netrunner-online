# NORTHPOLE — Netrunner Online End-State Vision

## What This Is

A zero-install, browser-based tabletop for playing Android: Netrunner with a friend. Not a rules engine — a faithful
digital table where two humans play the game, make the calls, and keep each other honest, just like cardboard.

The design principle: **if you can do it with physical cards and pennies, you can do it here — and nothing more.**

## Core Experience

1. **Peer-to-peer, no server.** Two players connect via WebRTC. No accounts, no matchmaking, no backend. Share a link,
   play. Solo mode stays for testing decks alone.
1. **Deck import from text.** Paste a NetrunnerDB-format list, hit load. Cards appear shuffled in a deck object.
   Identity is placed automatically.
1. **Free-form board.** Cards and tokens are DOM elements you drag anywhere. Grid-snap keeps things tidy. No enforced
   zones — players arrange servers, rig, and score area however they like.
1. **Card lifecycle.** Draw (click deck) → hand (hidden from opponent) → board (install/play) → archives/heap (discard).
   Every transition is a drag or a context-menu action. Cards flip, rotate, and tuck under other cards.
1. **Tokens as physical objects.** Credits, advancements, viruses, tags, bad pub, brain damage — spawn from the resource
   panel, stack on cards, double-click to flip dual-purpose tokens, middle-click to duplicate. Token bin to clean up.

## What "Done" Looks Like

### Fully Wired Interactions

- Context menu on cards: flip, rotate, put on top of deck, shuffle into deck, put on bottom, move to archives/heap.
- Context menu on tokens: duplicate, flip, remove.
- Context menu on decks: shuffle, search (open a scrollable overlay of all cards in the deck, pick one to draw).
- Middle-click shortcuts stay as fast alternatives to the menu.

### Deck Search & Browsing

- Clicking "search deck" opens a filtered, scrollable card list for that deck.
- Player picks a card, it moves to hand. Deck reshuffles automatically.
- A global card browser (separate from deck search) lets players look up any card by name for reference — no dragging
  from it, just reading.

### Board Quality-of-Life

- **Keyboard shortcuts:** R to rotate, F to flip, Delete to discard, Escape to deselect.
- **Undo stack:** Ctrl+Z reverts the last move/flip/draw. Local only, not synced — each player manages their own.
- **Snap zones:** Optional soft-snap regions for common layouts (HQ, R&D, Archives, remote servers / stack, grip, heap).
  Togglable, never mandatory.
- **Counter overlays:** Click a card to see/edit hosted counters inline (advancement tokens, virus counters) without
  manually placing token objects.
- **Screen resolution handling:** Responsive layout that works from 1280x720 to ultrawide. Panels scale, card size
  adapts, hand areas resize.

### Multiplayer Polish

- **In-game chat.** Text box pinned to a corner, messages synced over the data channel. No history beyond the session.
- **Connection resilience.** Auto-reconnect on drop. Detect stale peers and show a clear "opponent disconnected" state
  instead of silently breaking.
- **Spectator mode.** A third peer can watch (read-only data channel). Board mirrors the Corp player's perspective.

### Visual & Audio

- Card placement, flip, and shuffle have subtle sound effects. Master volume slider, default 30%, mutable.
- Current hearts + SVG background stay. Add a dark theme toggle (dark board, light card borders).
- Opponent's cursor is faintly visible on your board so you can see what they're pointing at during discussion.

### Deck Building (Stretch)

- A built-in deck editor that queries NetrunnerDB, enforces faction and influence rules, and exports a shareable list.
- This is a nice-to-have. The app is complete without it — paste-import covers the need.

## What This Is NOT

- **Not a rules engine.** No agenda point tracking, no credit pool enforcement, no click counter, no ice-encounter
  resolution. Players know the rules. The table doesn't babysit.
- **Not a platform.** No user profiles, no ELO, no replay storage, no tournament brackets. Play and close the tab.
- **Not a mobile app.** Tablet might work with responsive layout. Phone is out of scope — the board needs real estate.

## Technical Boundaries

- Vanilla JS, no framework. This is a feature, not a compromise — keep the bundle tiny and the code obvious.
- Vite for dev/build. Vitest for tests.
- All card data from NetrunnerDB API. No local card database, no scraping, no bundled images.
- PeerJS for signaling. TURN relay for NAT traversal. No custom server beyond what PeerJS provides.
- State lives in the DOM. No redux, no state machine, no model layer. The board IS the state.

## Priority Order

1. Wire all context-menu actions (deck ops, token ops) — table is incomplete without them.
1. Deck search overlay.
1. Keyboard shortcuts.
1. Connection resilience and reconnect.
1. Responsive layout.
1. Undo stack.
1. In-game chat.
1. Sound effects.
1. Spectator mode.
1. Dark theme.
1. Deck builder (if ever).

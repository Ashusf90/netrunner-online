# NORTHPOLE — Netrunner Online End-State Vision

## What This Is

A zero-install, browser-based tabletop for playing Android: Netrunner with a friend. Not a rules engine — a faithful
digital table where two humans play the game, make the calls, and keep each other honest, just like cardboard.

**Design principle: if you can do it with physical cards and pennies, you can do it here — and nothing more.**

## A Typical Session

Player A opens the page and hosts a game. A code appears. They send it to Player B over Discord, text, whatever. Player
B joins with the code. Both players paste a deck list, pick a side, and load. Identity hits the board, the rest becomes
a shuffled deck. They play — dragging cards, spawning tokens, flipping and rotating — exactly as they would on a
physical table. Game ends, tab closes, nothing persists.

Solo mode is the same table with no connection — one player, two decks, for testing builds alone.

## The Table

The board is a free-form canvas. Cards and tokens are objects you place anywhere. There are no enforced zones, no
automatic layouts, no invisible rails. Grid snap keeps things tidy. Players arrange servers, rig, and score area however
they like — the same way you'd spread cards across a kitchen table.

The board has a fixed size and aspect ratio. On screens that do not match, it scales proportionally and black bars fill
the edges — the same way a console game fits a TV. Both players always see the same board at the same proportions.

Hand areas sit at the top and bottom edges. Your hand is visible to you; your opponent sees card backs. The play area is
the open space between.

## Cards

Cards move through a natural lifecycle: deck to hand, hand to board, board to archives or heap — and sometimes back.
Every transition is something the player does. The system never moves a card on its own.

You can always see the front of your own cards, except in your deck. You see an opponent's card only when it is face-up.
"Rezzing" means the Corp player flips a card face-up. Archives can hold both face-up and face-down cards. The player
controls the state — the table does not.

Cards flip, rotate, tuck under other cards, return to the deck, shuffle back in. Everything you can do with a physical
card, you can do here.

Cards have a holographic sheen — a light-reactive rainbow effect that shifts as the cursor moves across the card face,
the way a foil trading card catches the light when you tilt it in your hand.

## Tokens

Tokens are physical objects. Credits, advancement counters, viruses, tags, bad pub, brain damage — you spawn them from a
panel, place them on cards, stack them, duplicate them, toss them in a bin when done. Dual-purpose tokens flip between
faces the same way a real Netrunner token does.

No inline counters, no automatic tracking. Tokens *are* the counters — same as pennies on cardboard.

## Decks

Paste a standard deck list, hit load. Cards are matched against the NetrunnerDB card pool. If something doesn't match,
you're told what failed and the rest still loads.

You can shuffle your deck, search through it (pick a card, deck reshuffles), or draw from the top. Your opponent can see
that you're searching but not what you're looking at — same as physical play.

## Multiplayer

Two players connect directly, peer-to-peer. No accounts, no matchmaking, no backend that sees game state. Share a link,
play.

The connection must survive the length of a game — 30 minutes or more. Drops reconnect automatically. The board resyncs.
A clear banner tells you when your opponent is gone, not a silently frozen screen.

Players need to talk. In-game chat is not a feature — it is part of the table. Netrunner is a conversation: "I rez
this," "run on HQ," "take two credits." Without chat the table is incomplete.

## What Finished Looks Like

The product is done when two players can sit down, load decks, and play a full game of Netrunner without ever feeling
like the tool is in their way. Every card action works. Every token behaves. The connection holds. The board is
responsive from a laptop to an ultrawide monitor. Keyboard shortcuts make common actions fast. An undo catches the
inevitable misclick.

Polish beyond that — sound effects, spectators, opponent cursor ghosts, dark themes — is welcome but not the finish
line.

## What This Is NOT

- **Not a rules engine.** No agenda tracking, no credit enforcement, no click counter, no ice-encounter resolution.
  Players know the rules. The table does not babysit.
- **Not a platform.** No user profiles, no ELO, no replays, no tournament brackets. Play and close the tab.
- **Not a mobile app.** Tablet may work. Phone is out of scope — the board needs real estate.
- **Not a deck builder.** NetrunnerDB exists. This app imports lists; it does not author them.

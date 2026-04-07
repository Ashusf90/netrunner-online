# TECHNICAL_VISION — How This Codebase Works When It Grows Up

## What This Is

The technical north star for Netrunner Online. NORTHPOLE describes what the product feels like to players. This document
describes what the codebase feels like to work on — the architecture, the boundaries, and the standards that let us ship
features without the floor creaking.

**Principle: a contributor reads one module, understands it fully, changes it, and trusts the tests to catch what
breaks.**

## Code Paradigm

Data-oriented first. Game state is plain data — typed objects and arrays, not class hierarchies. Cards, tokens, board
positions, and P2P messages are inert structures that functions operate on. The store holds data. Functions transform
it. The DOM renders it.

Object-oriented where ownership is natural. A peer connection owns its lifecycle. A drag interaction owns its event
listeners and cleanup. These are objects with state and methods — not because of a pattern fetish, but because the thing
genuinely owns resources that need setup and teardown.

Functional where it keeps things honest. Deck parsing, visibility resolution, message serialization, grid snapping —
pure functions that take input and return output with no side effects. Easy to test, easy to reason about, impossible to
break by calling in the wrong order.

No class for what a plain object does. No pure function for what needs to hold state across calls. Pick the tool that
fits the problem, not the one that fits a paradigm label.

## TypeScript

The codebase is TypeScript. Not JSDoc-annotated JavaScript, not gradual migration with `// @ts-check` — TypeScript, end
to end. Vite handles it natively. Card data has a defined shape. Message payloads have a defined shape. Function
signatures tell you what goes in and what comes out. The type checker catches the bugs that unit tests miss.

Every `interface` and `type` that crosses a module boundary lives in a shared `types.ts` file. Internal types stay
local.

`tsconfig.json` starts permissive and tightens over time. `strict` is the destination — `noImplicitAny`,
`strictNullChecks`, `noUncheckedIndexedAccess`, all of it. Each flag gets enabled when the codebase is ready for it, not
all at once on day one. The ratchet turns one way.

## State Ownership

State lives in a single, observable store. Modules subscribe to what they need. No module mutates state it does not own.
The store is the only source of truth — the DOM reflects it, never the other way around.

- Player side, connection status, board contents, hand contents — all in the store.
- The P2P layer reads outgoing actions from the store and writes incoming actions into it.
- The DOM layer renders from the store. It never queries the DOM to discover game state.
- `window` holds nothing. Globals are dead.

The store is a lightweight custom implementation — no Redux, no Zustand. A plain object behind a proxy with
`subscribe(key, callback)`. This project does not need a state management library. It needs one file that does one thing
well.

## Domain Structure

Code is split by domain, not by technical role. A domain owns its data, its logic, and its types. Nothing outside the
domain reaches into its internals.

The game has clear bounded contexts:

- **Cards** — card data, lifecycle transitions, visibility rules, deck parsing. Knows what a card is and what can happen
  to it. Knows nothing about how a card is rendered or transmitted.
- **Tokens** — token types, hosting, stacking, duplication. Same boundary — data and rules, no DOM, no network.
- **Board** — spatial concerns. Grid snapping, positioning, z-ordering, hand area membership. Operates on coordinates,
  not on cards or tokens directly.
- **P2P** — connection lifecycle, message protocol, serialization, reconnection. Speaks in messages, not in game
  objects. Translates between the wire format and the store.
- **UI** — rendering, panels, modals, chat, context menus. Reads from the store, writes to the DOM. Never computes game
  logic.

Each domain exposes a narrow public surface — types and functions that other domains are allowed to use. Internal
helpers stay internal. If two domains need to coordinate, they do it through the store, not by importing each other's
internals.

A file that touches cards, tokens, and the DOM is a file that belongs to three domains and owns none of them. Split it.
Code that changes together lives together. Code that changes for different reasons lives apart.

## Module Boundaries

A clean dependency tree with no cycles. Each module has one job and a narrow public surface.

```
game.ts (entry point — wires everything, owns nothing)
  ├── store.ts       (state — the single source of truth)
  ├── p2p.ts         (networking — reads/writes the store, knows nothing about DOM)
  ├── board.ts       (rendering — reads the store, writes to DOM, knows nothing about P2P)
  ├── input.ts       (user interaction — drag, click, keys — dispatches actions to the store)
  ├── deck.ts        (deck parsing — pure function, no side effects)
  └── ui/            (panels, modals, chat — thin wrappers over DOM, driven by store)
```

The entry point composes. Everything else is composable. If you cannot draw the dependency tree on a napkin, it is too
tangled.

## DOM Coupling

DOM references are resolved once, at startup, and passed as dependencies. No module reaches into the document on its
own. Element references live near the HTML they describe, not in business logic three files away. Selectors that must
exist at runtime are validated at startup — fail loud and early, not halfway through a game.

A `dom.ts` module exports typed references to every significant element. Other modules import from `dom.ts`, never from
`document`. If the HTML changes, exactly one file breaks and the TypeScript compiler tells you where.

## P2P Protocol

Messages have a schema defined in `types.ts`. Every message is validated on receipt with a type guard. Malformed
messages are dropped and logged, never silently applied.

Every message carries a protocol version number. When a client receives a message with a mismatched version, it surfaces
a clear "client version mismatch" warning to both players. No silent board drift.

On reconnect, the host sends a full board snapshot — every card position, face state, token placement, hand contents.
The joining client replaces its local state entirely. The game picks up where it left off.

A heartbeat ping fires every 5 seconds. Three missed pings triggers the disconnect banner. This distinguishes "opponent
is thinking" from "opponent's browser crashed."

Last-write-wins for conflicts. CRDTs and OT are overkill for a two-player table.

## Error Handling

Every failure the user can hit has a visible, recoverable state. Card API down — a banner says so and offers retry. Deck
list has bad entries — the error names each one. Connection drops — the disconnect banner appears instantly. No
operation leaves the app in a state where the user does not know what happened.

Internally: promises reject with useful context. Event listeners clean up after themselves. Functions that take external
input validate it at the boundary and trust it downstream. No defensive coding in internal modules — the boundary does
the work so the core stays clean.

## Testing

Integration tests are the default. Tests exercise real modules wired together — store, domain logic, DOM updates — not
isolated units behind mocks. A test that loads a deck should go through the real parser, hit the real store, and verify
the real DOM output. If a test needs to mock a collaborator to work, that is a signal the code is too coupled, not a
reason to add a mock.

[MSW](https://mswjs.io/) intercepts network requests at the service worker level. Card API fetches, PeerJS signaling,
TURN token requests — all stubbed through MSW handlers, not hand-rolled fetch mocks. The application code never knows it
is being tested. MSW handlers live in a shared `test/mocks/` directory and mirror the real API shapes from `types.ts`.

Playwright E2E tests cover the critical path — two browser contexts connect via PeerJS, load decks, draw cards, flip,
disconnect, reconnect. This is the test that gives confidence to ship.

Isolated unit tests are the exception, not the rule. Pure functions with no dependencies (grid snapping, visibility
resolution, message serialization) earn unit tests because there is nothing to integrate with. Everything else is tested
through integration.

Vitest runs all non-E2E tests. The test suite runs in CI on every push. A red build blocks the deploy. No exceptions.

Coverage thresholds start low and ratchet up. Every feature that lands raises the floor — the threshold is bumped to
match the new baseline. Coverage never goes down. The goal is not a vanity number; it is a guarantee that the parts of
the codebase that are tested stay tested, and the untested surface shrinks with every PR.

## Code Quality Tooling

Biome for linting and formatting in a single tool, enforced in CI. A contributor runs `biome check` and gets lints and
formatting in one pass. No ESLint-Prettier interplay, no plugin chains, no config sprawl — one binary, one config file,
instant results.

Rules start lenient and tighten over time. New code meets the current bar; existing code gets cleaned up as it is
touched. The config grows stricter with each pass — never looser. The goal is a codebase where Biome runs clean with the
strictest reasonable settings, reached incrementally, not by a formatting blitz.

## CSS

[LiteWind](https://litewindcss.com/) for styling — a static, utility-first CSS framework with Tailwind-compatible class
names and zero build step. No PostCSS, no JIT compiler, no purge pass. Drop in the stylesheet and use the classes
directly in HTML.

Custom properties in a single `:root` block define project-specific values — card dimensions, grid size, z-index layers,
board colors. LiteWind handles spacing, layout, typography, and responsive utilities. Hand-written CSS handles only what
utility classes cannot: card flip transforms, drag state, tooltip positioning, and the holographic card effect.

Magic pixel values are gone. Card scale, grid snap distance, hand area height — all derived from custom properties or
LiteWind's spacing scale.

The board renders at a fixed internal resolution and scales proportionally to fit the viewport. Black bars (letterbox or
pillarbox) fill the remaining space. The internal coordinate system is identical for every player regardless of screen
size — a card at position (400, 300) is at (400, 300) on both sides. This eliminates an entire class of P2P desync bugs:
no rounding differences, no layout reflow, no viewport-dependent position drift. The board is a canvas with a fixed
aspect ratio, not a responsive page.

## Accessibility

Cards are focusable. Tab moves between interactive elements. Arrow keys navigate the board. Enter and Space activate the
focused element (flip, play, draw). Modals trap focus. Context menus are keyboard-navigable. Every interactive element
has an ARIA label.

This is not about compliance checklists. It is about not locking out players who navigate differently.

## What This Is NOT

- **Not a rewrite plan.** The codebase works. These changes land incrementally, carried by the features being built. No
  big-bang refactor, no feature freeze for infrastructure.
- **Not negotiable on direction.** The decisions above are made. TypeScript, not JSDoc. Custom store, not a library.
  Self-hosted signaling, not third-party. Playwright, not Cypress. Biome, not ESLint+Prettier. The stack is chosen.
  Build with it.

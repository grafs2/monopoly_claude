# Monopoly — Single-File Implementation

## Overview

A fully playable German-language Monopoly game delivered as a single self-contained HTML file (`index.html`). No build tools, no external JavaScript libraries, no backend. The only external resource is the Google Fonts CDN (Roboto, Oswald). The entire game — structure, styles, and logic — lives in one ~3 800-line file.

**Key capabilities:**
- 2–4 players (any mix of human and AI)
- Full rule set: rent, tax, chance/community-chest cards, jail, houses, hotels, mortgages, trades, auctions, and bankruptcy
- AI opponents with buy/build/auction/raise-money heuristics
- Auto-save to `localStorage` with resume on next visit
- Responsive layout that scales to smaller screens

---

## File Structure

```
monopoly_claude/
├── index.html        ← entire game (HTML + CSS + JS, ~3 800 lines)
└── promptmonopoly.md ← original German-language specification
```

`index.html` is divided into three logical regions:

| Region | Lines (approx.) | Content |
|--------|-----------------|---------|
| `<head>` + CSS | 1 – 1 441 | Meta, Google Fonts import, all styles |
| HTML body | 1 443 – 1 813 | Setup screen, game screen, all modals |
| `<script>` | 1 814 – end | Data, game state, all game logic |

---

## CSS Architecture

### Custom Properties

Defined on `:root`:

```css
/* Board dimensions */
--board-size:   720px
--corner-size:   90px   /* corner fields */
--field-size:    60px   /* regular fields */

/* Board colours */
--felt-green, --felt-dark

/* UI colours */
--card-bg, --panel-bg, --panel-solid, --panel-border
--text-light, --text-dark, --gold, --modal-bg

/* Property colour groups */
--color-brown, --color-lightblue, --color-pink, --color-orange
--color-red,   --color-yellow,    --color-green, --color-darkblue
```

### Page Layout

The game screen uses a CSS Grid:

```
grid-template-areas:
  "header header header"
  "lpanel board  rpanel"
  "footer footer footer"

columns: 260px  1fr  280px
```

- **Header** (`#game-header`): logo, active-player banner, utility buttons (Verwalten / Speichern / Neu)
- **Left panel** (`#left-panel`): player cards for all seats (0 – 3)
- **Board area** (`#board-area`): contains the 11 × 11 board grid
- **Right panel** (`#log-panel`): scrollable game log
- **Footer** (`#game-footer`): all action buttons (Roll, Build, Trade, Buy Request, Manage, End Turn)

### Board Grid

The board is an 11 × 11 CSS Grid:

```
grid-template-columns: var(--corner-size) repeat(9, var(--field-size)) var(--corner-size)
grid-template-rows:    same pattern
```

Field positions on the grid:

| Side | Grid row | Grid columns | Fields |
|------|----------|-------------|--------|
| Bottom | 11 | 11 → 1 | 0 – 10 |
| Left | 10 → 2 | 1 | 11 – 19 |
| Top | 1 | 1 → 11 | 20 – 30 |
| Right | 2 → 10 | 11 | 31 – 39 |

Corner fields (0, 10, 20, 30) occupy the full 90 × 90 px cell. Regular fields are 60 × 60 px.

Each field element receives an orientation class (`bot`, `top`, `lft`, `rgt`) which rotates the internal layout so the colour stripe and property name always face the board centre.

### Color Group Classes

Used on both field colour stripes and property card headers:

```
.cg-brown      .cg-lightblue  .cg-pink     .cg-orange
.cg-red        .cg-yellow     .cg-green    .cg-darkblue
.cg-railroad   .cg-utility
```

### Modal System

All modals share `.modal-overlay` (fixed, full-viewport) with `backdrop-filter: blur(3px)`. The buy modal overrides this with `backdrop-filter: none` so the board stays visible while inspecting a property group. Modal box sizes:

| Class | Max-width |
|-------|-----------|
| `.modal-box` (default) | 480 px |
| `.modal-box.wide` | 620 px |
| `.modal-box.wider` (buy modal) | 760 px |
| `.modal-box.narrow` | 360 px |

### Game Log Colours

| Class | Colour | Used for |
|-------|--------|---------|
| `.ll-player` | Bright green | Player names |
| `.ll-money` | `#5dde8a` | Currency amounts |
| `.ll-property` | Blue italic | Property names |
| `.ll-system` | Gold | System announcements |
| `.ll-jail` | Orange | Jail events |
| `.ll-danger` | Red | Bankruptcy |

---

## HTML Structure

### Setup Screen (`#setup-screen`)

Shown on first visit (no saved game). Contains:

- `#setup-total` — selector for total player count (2–4)
- `#setup-ai` — selector for how many of those are AI
- `#setup-player-list` — dynamically rendered rows (name input + colour picker) for each human seat
- `#setup-error` — inline validation message
- Start button → `startGame()`

### Game Screen (`#game-screen`)

Hidden until game starts. Sub-elements:

**Header**
- `#hdr-dot` — coloured dot of active player
- `#hdr-name` — active player's name
- `#hdr-label` — phase label ("ist am Zug" / "Zug beenden" / "Spiel beendet")
- Header utility buttons: Verwalten → `openManageModal()`, Speichern → `saveGame()`, Neu → `confirmNewGame()`

**Board centre (`#board-center`)**
- `#board-player-name` — current player name displayed above the logo (coloured)
- `.center-logo` — "MONOPOLY" heading
- `#die1`, `#die2` — individual die faces (Unicode)
- `#center-status` — sum or "Pasch!" display
- `.center-decks` — visual Chance and Community Chest deck cards

**Player cards** — `#pc-0` through `#pc-3`; all four in the left panel.

**Footer action buttons**

| ID | Label | Condition |
|----|-------|-----------|
| `#btn-roll` | Würfeln | roll phase (human) |
| `#btn-build` | Bauen | endturn + has buildable monopoly |
| `#btn-trade` | Handeln | endturn + ≥ 2 human players |
| `#btn-buy-req` | Kaufanfrage | endturn + another player owns at least one property |
| `#btn-manage` | Verwalten | always (human) |
| `#btn-end` | Zug beenden | endturn phase |

**Log** — `#log-entries` in `#log-panel` (right panel, scrollable, newest entries added at top)

### Modals

| ID | Purpose |
|----|---------|
| `#modal-buy` | Buy or decline a property; shows full colour group |
| `#modal-auction` | Round-robin auction bidding |
| `#modal-card` | Display drawn Chance / Community Chest card |
| `#modal-jail` | Jail options (roll, pay, use free card) |
| `#modal-build` | Build / sell houses and hotels |
| `#modal-manage` | Mortgage / unmortgage properties |
| `#modal-trade` | Compose a trade offer |
| `#modal-trade-accept` | Counter-party reviews and accepts / rejects trade |
| `#modal-bankrupt` | Emergency liquidation before confirming bankruptcy |
| `#modal-buy-request` | Buyer selects a property owned by another player to request |
| `#modal-buy-respond` | Seller sees the request and can decline or name a price |
| `#modal-buy-accept` | Buyer sees the seller's offered price and accepts or declines |
| `#modal-win` | Victory screen |

---

## JavaScript — Data

### `PLAYER_COLORS`

Array of 8 colour objects: `{ id, label (German), hex }`.
IDs: `red`, `blue`, `lime`, `yellow`, `purple`, `orange`, `pink`, `teal`.

### `COLOR_GROUPS`

Object keyed by group name. Each entry: `{ label, css, count }` where `css` is the `.cg-*` class and `count` is 2 or 3 streets in the group.

Groups: `brown`, `lightblue`, `pink`, `orange`, `red`, `yellow`, `green`, `darkblue`.

### `BOARD_FIELDS_DEF`

Array of 40 field definition objects. Common properties:

```js
{
  index,       // 0–39
  type,        // 'street'|'railroad'|'utility'|'tax'|'chance'|'chest'|'go'|'jail'|'freeparking'|'gotojail'
  name,        // German display name (may contain \n for line breaks)
  gridRow,     // CSS grid row (1–11)
  gridCol,     // CSS grid column (1–11)
  side,        // 'bot'|'top'|'lft'|'rgt'|'corner'
}
```

Street-specific additional properties:

```js
{
  colorGroup,  // key into COLOR_GROUPS
  price,       // purchase price
  mortgage,    // mortgage value (= price / 2)
  houseCost,   // cost per house/hotel
  rents: { base, monopoly, h1, h2, h3, h4, hotel }
}
```

#### All 40 Fields

| # | Name | Type | Price | Group |
|---|------|------|-------|-------|
| 0 | LOS | go | — | — |
| 1 | Badstraße | street | 60 | brown |
| 2 | Gemeinschaftsfeld | chest | — | — |
| 3 | Turmstraße | street | 60 | brown |
| 4 | Einkommensteuer | tax | 200 | — |
| 5 | Südbahnhof | railroad | 200 | — |
| 6 | Chaussestraße | street | 100 | lightblue |
| 7 | Ereignisfeld | chance | — | — |
| 8 | Elisenstraße | street | 100 | lightblue |
| 9 | Poststraße | street | 120 | lightblue |
| 10 | Gefängnis (Besuch) | jail | — | — |
| 11 | Seestraße | street | 140 | pink |
| 12 | Elektrizitätswerk | utility | 150 | — |
| 13 | Hafenstraße | street | 140 | pink |
| 14 | Neue Straße | street | 160 | pink |
| 15 | Westbahnhof | railroad | 200 | — |
| 16 | Münchener Straße | street | 180 | orange |
| 17 | Gemeinschaftsfeld | chest | — | — |
| 18 | Wiener Straße | street | 180 | orange |
| 19 | Berliner Straße | street | 200 | orange |
| 20 | Frei-Parken | freeparking | — | — |
| 21 | Theaterstraße | street | 220 | red |
| 22 | Ereignisfeld | chance | — | — |
| 23 | Museumstraße | street | 220 | red |
| 24 | Opernplatz | street | 240 | red |
| 25 | Nordbahnhof | railroad | 200 | — |
| 26 | Lessingstraße | street | 260 | yellow |
| 27 | Schillerstraße | street | 260 | yellow |
| 28 | Wasserwerk | utility | 150 | — |
| 29 | Goetheplatz | street | 280 | yellow |
| 30 | Gehen Sie in das Gefängnis | gotojail | — | — |
| 31 | Rathausplatz | street | 300 | green |
| 32 | Hauptstraße | street | 300 | green |
| 33 | Gemeinschaftsfeld | chest | — | — |
| 34 | Bahnhofstraße | street | 320 | green |
| 35 | Ostbahnhof | railroad | 200 | — |
| 36 | Ereignisfeld | chance | — | — |
| 37 | Parkstraße | street | 350 | darkblue |
| 38 | Zusatzsteuer | tax | 100 | — |
| 39 | Schlossallee | street | 400 | darkblue |

### `CHANCE_CARDS_DEF` / `CHEST_CARDS_DEF`

Each deck has 16 card objects:

```js
{ id, deckType: 'chance'|'chest', text, actionType, param }
```

**Action types and their `param` shapes:**

| actionType | param | Effect |
|------------|-------|--------|
| `gain` | `{ amount }` | Player receives amount from bank |
| `pay` | `{ amount }` | Player pays amount to bank |
| `moveTo` | `{ target, collectLos? }` | Move to board position; collect 200 if passing GO |
| `moveBack` | `{ steps }` | Move backwards N spaces |
| `jail` | — | Go directly to jail |
| `getOutOfJail` | — | Receive a Get Out of Jail Free card |
| `repairs` | `{ house, hotel }` | Pay per house/hotel owned |
| `birthday` | `{ amount }` | Collect amount from every other player |
| `nearestRailroad` | — | Move to nearest railroad, pay double rent |
| `nearestUtility` | — | Move to nearest utility, pay 10× dice |

### `makePlayer(id, name, colorId, isAI)`

Returns a fresh player object:

```js
{
  id,           // integer index
  name,         // display name
  colorId,      // key into PLAYER_COLORS
  color,        // hex string
  isAI,         // boolean
  cash: 1500,
  position: 0,
  inJail: false,
  jailTurns: 0, // increments each turn in jail; forced out after 3
  jailFreeCards: 0,
  bankrupt: false,
  properties: [] // array of field indices
}
```

### `makeEmptyGameState()`

Returns a complete game state object (`gs`):

```js
{
  players,          // array of player objects
  fields,           // deep clone of BOARD_FIELDS_DEF + runtime state
  chanceDeck,       // shuffled card array
  chestDeck,
  chanceIdx,        // draw pointer (mod length)
  chestIdx,
  currentPlayerIdx,
  phase,            // see Phase State Machine below
  doublesCount,     // consecutive doubles this turn (0–2; 3 → jail)
  lastDice: [d1,d2],
  log,              // array of HTML strings for game log
  pendingCard,      // card being resolved
  pendingTrade,     // trade offer awaiting acceptance
  pendingBankrupt,  // bankruptcy context { playerId, creditorId, amount }
  pendingAuction,   // auction context object
  pendingBuy,       // { fieldIdx, playerId }
  turnCount,
  gameOver: false,
  winnerId: null
}
```

Fields in `gs.fields` accumulate runtime state beyond the static definition:

```js
{
  ...staticDef,
  ownerId,     // player id or null
  mortgaged,   // boolean
  houses,      // 0–4
  hotel        // boolean
}
```

---

## JavaScript — Functions

### Setup & Initialization

| Function | Description |
|----------|-------------|
| `DOMContentLoaded` listener | Check localStorage; show resume banner or setup screen |
| `showResumeBanner()` | Display the saved-game banner |
| `showSetupScreen()` | Hide game, show setup |
| `showGameScreen()` | Hide setup, show game |
| `dismissResume()` | Delete save, show setup |
| `resumeGame()` | Deserialize save, render full game state |
| `onSetupChange()` | Sync AI count options when total player count changes |
| `renderSetupPlayers()` | Build name-input + colour-picker rows for human seats |
| `updateTokenPreview(i)` | Update colour dot preview on row i |
| `startGame()` | Validate inputs, call `makeEmptyGameState()`, render, start |
| `confirmNewGame()` | Prompt for confirmation before discarding current game |

### Persistence

| Function | Description |
|----------|-------------|
| `saveGame()` | `JSON.stringify(gs)` → `localStorage['monopoly-singlefile-save']` |
| `loadGame()` | Parse and return saved state, or `null` |

### Rendering

| Function | Description |
|----------|-------------|
| `renderBoard()` | Remove all field cells, recreate 40 from `gs.fields`: grid position, orientation class, colour stripe, name/price label, ownership bar (player colour), building pips, mortgage overlay, player piece tokens |
| `renderAllPlayerCards()` | Show/hide and refresh `#pc-0` – `#pc-3` |
| `renderPlayerCard(idx)` | Build player card HTML: token, name, badge (human / KI / Gefängnis / Bankrott), cash, net worth, property count summary, property chips, jail-free card count |
| `updateHeader()` | Set `#hdr-dot` colour, `#hdr-name` text, `#hdr-label` phase text, and `#board-player-name` |
| `updateActionButtons()` | Enable/disable the six action buttons based on `gs.phase` and player capabilities; auto-end turn if no meaningful actions remain |

### Game Flow

```
handleRoll()
  └─ executeRoll(playerId)
       ├─ rollDice() + showDice()
       ├─ doubles × 3 → sendToJail()
       └─ movePlayer(playerId, steps)
            └─ processLanding(playerId, fieldIdx, diceTotal)
                 ├─ street/railroad/utility → openBuyModal() or payRent()
                 ├─ tax → payTax()
                 ├─ chance/chest → drawChanceCard() / drawChestCard()
                 ├─ gotojail → sendToJail()
                 └─ afterLanding()
                      └─ endTurn()  (or next roll if doubles)
```

| Function | Description |
|----------|-------------|
| `handleRoll()` | Guard (phase must be `'roll'`); show jail modal if jailed, else `executeRoll` |
| `executeRoll(playerId)` | Roll dice; on doubles increment counter (jail at 3); animate; call `movePlayer` |
| `movePlayer(playerId, steps, cb)` | Compute new position mod 40; award 200 € for passing GO; invoke callback |
| `processLanding(playerId, fieldIdx, diceTotal, dblRent)` | Dispatch on field type |
| `payRent(fromId, toId, amount, fieldIdx)` | Transfer cash or call `triggerBankruptcy` |
| `payTax(playerId, amount)` | Pay bank or call `triggerBankruptcy` |
| `afterLanding(playerId, skipPhase)` | Set phase to `'roll'` (doubles) or `'endturn'`; call `checkWinCondition` |
| `endTurn()` | Advance `currentPlayerIdx` to next active player; reset doubles; set phase `'roll'`; log turn start; trigger AI if needed |

### Rule Helpers

| Function | Returns | Description |
|----------|---------|-------------|
| `getMonopolyOwner(colorGroup)` | player id \| null | Owner id if all streets in group are owned by same player |
| `hasMonopoly(playerId, colorGroup)` | boolean | Shorthand |
| `getStreetRent(fieldIdx)` | number | Rent based on houses/hotel; doubled for unimproved monopoly |
| `getRailroadRent(ownerId)` | number | 25/50/100/200 for 1/2/3/4 railroads |
| `getUtilityRent(ownerId, diceTotal)` | number | 4× dice (1 utility) or 10× dice (both utilities) |
| `computeNetWorth(player)` | number | Cash + unmortgaged property values + 50% of building investment |
| `getNextActivePlayer()` | index | Next non-bankrupt player index |
| `canAnyBuild(player)` | boolean | True if player has at least one buildable slot (monopoly, not mortgaged, not hotel, even-build OK) |
| `liquidationValue(player)` | number | Total cash obtainable by selling all buildings and mortgaging all properties |

### Dice

| Symbol / Function | Description |
|-------------------|-------------|
| `DICE_FACES` | `['','⚀','⚁','⚂','⚃','⚄','⚅']` — Unicode die faces |
| `rollDice()` | Returns `[1-6, 1-6]` |
| `showDice(d1, d2)` | Applies `.rolling` CSS animation; updates `#die1`, `#die2`, `#center-status` |

### Jail

| Function | Description |
|----------|-------------|
| `sendToJail(playerId)` | Set `position = 10`, `inJail = true`, `jailTurns = 0` |
| `releaseFromJail(playerId)` | Clear `inJail`, `jailTurns` |
| `showJailModal()` | Present three options: roll for doubles, pay 50 €, use free card |
| `jailRollDice(mustLeave)` | Roll; on doubles → release and move; on 3rd turn → force pay and release |

### Cards

| Function | Description |
|----------|-------------|
| `drawChanceCard(playerId, diceTotal)` | Advance `chanceIdx`; log and show card modal |
| `drawChestCard(playerId, diceTotal)` | Same for community chest |
| `showCardModal(card)` | Display card text in `#modal-card` |
| `dismissCard()` | Close modal; call `resolveCard` |
| `resolveCard(card, playerId, diceTotal)` | Execute card effect by `actionType` |

### Buy & Auction

| Function | Description |
|----------|-------------|
| `openBuyModal(fieldIdx, playerId)` | AI: auto-buy if cash allows, else auction. Human: open modal |
| `buildPropCardHTML(fi)` | Return HTML for a single property card (with full rent table) |
| `buildGroupDisplayHTML(fi, buyerPid)` | Return HTML for a row of all group properties; each card has a coloured status badge: *Zu kaufen* (orange), *Dein Besitz* (green), *Besitz: [name]* (red), *Verfügbar* (grey) |
| `confirmPurchase()` | Deduct price, set `ownerId`, update board |
| `declinePurchase()` | Call `triggerAuction(fieldIdx)` |
| `triggerAuction(fieldIdx)` | Build `pendingAuction` with all non-bankrupt players as bidders; call `runAuctionStep` |
| `runAuctionStep()` | Cycle bidders; human → show modal; AI → auto-bid via `aiAuctionBudget` or pass |
| `renderAuctionModal()` | Show current highest bid, leader name, and bidder list |
| `placeBid()` | Validate amount > current bid and ≤ player cash; advance to next bidder |
| `passAuction()` | Mark bidder as passed; advance |
| `resolveAuction()` | Award property to highest bidder (at bid price); if no bids, property stays with bank |
| `aiAuctionBudget(pid, fi)` | Max bid: 90% of street price, 115% railroad, 75% utility — each minus a 200 € safety buffer |

### Build & Sell

| Function | Description |
|----------|-------------|
| `openBuildModal()` | Show `#modal-build` |
| `closeBuildModal()` | Hide modal |
| `refreshBuildModal()` | Render a row per buildable colour group; show current houses on each street; + / − buttons |
| `doBuildHouse(fi)` | Even-build check (must be at group minimum); deduct `houseCost`; increment `houses` or set `hotel`; re-render |
| `doSellHouse(fi)` | Even-sell check (must be at group maximum); refund 50% of `houseCost`; decrement |

**Even-building rule:** A house can only be built on a street if its current count equals the group minimum. A house can only be sold if its current count equals the group maximum.

### Manage (Mortgage)

| Function | Description |
|----------|-------------|
| `openManageModal()` | Show `#modal-manage` |
| `closeManageModal()` | Hide |
| `refreshManageModal()` | List all player properties; Mortgage button if unmortgaged & no buildings; Unmortgage button if mortgaged |
| `mortgageField(idx)` | Set `mortgaged = true`; add `mortgage` value to cash |
| `unmortgageField(idx)` | Set `mortgaged = false`; deduct `Math.ceil(mortgage × 1.1)` (10% interest) |

### Trade

`tradeState` module-level object: `{ offerProps, offerCash, wantProps, wantCash, targetId }`.

| Function | Description |
|----------|-------------|
| `openTradeModal()` | Initialise `tradeState`; select first non-AI partner; render |
| `closeTradeModal()` | Clear `tradeState`; hide modal |
| `renderTradeModal()` | Build partner dropdown, two property lists (offer / want), two cash inputs |
| `onTradePartnerChange(val)` | Switch `targetId`; re-render |
| `toggleTradeProp(side, fi)` | Add/remove field index from `offerProps` or `wantProps` |
| `submitTrade()` | Create `pendingTrade`; show acceptance modal |
| `renderTradeAcceptModal()` | Show offer/want summary to target player |
| `acceptTrade()` | Transfer properties and cash both ways; update `ownerId` on fields; re-render |
| `rejectTrade()` | Clear `pendingTrade`; return to trade modal |

### Buy Request

`buyRequestState` module-level object: `{ fieldIdx, buyerId, sellerId, offeredPrice? }`. Not persisted in `gs`.

A three-step negotiation initiated by the current human player to purchase a property from another player:

1. **Buyer** opens `#modal-buy-request`, browses all properties owned by other players grouped by owner, selects one, and submits the request.
2. **Seller** sees `#modal-buy-respond` and can decline outright or click "Preis nennen" to enter a selling price (pre-filled with a suggested value: face value, or +50% if the buyer would gain a monopoly).
3. **Buyer** sees `#modal-buy-accept` with the offered price and can accept (cash is transferred, ownership updated) or decline.

| Function | Description |
|----------|-------------|
| `openBuyRequestModal()` | Filter all purchasable properties owned by others; open modal |
| `closeBuyRequestModal()` | Hide modal; clear `buyRequestState` |
| `renderBuyRequestModal()` | Render properties grouped by owner; highlight selected property; enable submit button when a property is selected |
| `selectBuyRequestProp(fieldIdx)` | Set `buyRequestState` with the chosen field; re-render |
| `submitBuyRequest()` | Hide buyer modal; show seller respond modal (`#modal-buy-respond`) |
| `showBuyRespondPriceInput()` | Reveal price input in seller modal with a suggested price |
| `submitBuyRespondPrice()` | Store `offeredPrice` in `buyRequestState`; show buyer accept modal (`#modal-buy-accept`) |
| `declineBuyRespond()` | Seller declines; clear `buyRequestState`; log the refusal |
| `acceptBuyPrice()` | Validate buyer cash; transfer cash and ownership; update board and player cards; save |
| `declineBuyPrice()` | Buyer rejects the offered price; clear `buyRequestState` |

### Bankruptcy

| Function | Description |
|----------|-------------|
| `triggerBankruptcy(playerId, creditorId, amount)` | If `liquidationValue ≥ amount`: AI auto-raises, human opens modal. Else immediate bankruptcy. |
| `openBankruptModal(playerId, creditorId, amount)` | Show emergency sell/mortgage interface |
| `refreshBankruptBtn()` | Update confirm button: "Zahlung leisten" if now solvent, "Bankrott erklären" if not |
| `eSellH(idx)` | Emergency sell one house (50% return) |
| `eMortgage(idx)` | Emergency mortgage a property |
| `confirmBankruptcy()` | If now solvent: pay debt. Else: transfer all properties to creditor (or bank), mark `bankrupt = true`, check win |

### Win Condition

| Function | Description |
|----------|-------------|
| `checkWinCondition()` | Count non-bankrupt players; if only one remains → `gs.gameOver = true`, set `winnerId`, call `openWinModal` |
| `openWinModal(wid)` | Show winner name (in player colour), cash, net worth, property count, and total turn count |

### AI Logic

| Function | Description |
|----------|-------------|
| `runAITurn()` | Entry point called after `endTurn` hands control to an AI player. Phase `'roll'`: handle jail or `executeRoll`. Phase `'endturn'`: `aiBuildHouses`, then `endTurn`. |
| `aiHandleJail(p)` | Priority: use free card → pay 50 € → roll for doubles. Pays if cash > 200 €. |
| `aiBuildHouses(pid)` | Loop: find the monopoly group with lowest house count; build on the field at minimum — repeat while cash allows (keeping 200 € reserve) |
| `aiRaiseMoney(pid, amount)` | Sequential liquidation: sell hotels (50% refund) → sell houses → mortgage properties, until target reached or nothing left |

---

## Phase State Machine

```
         startGame()
              │
              ▼
         ┌─────────┐
    ┌───►│  roll   │◄──────────────────────────────┐
    │    └────┬────┘                               │
    │         │ executeRoll()                       │
    │         ▼                                     │
    │    ┌─────────┐  buy/auction resolved          │
    │    │ action  ├──────────────────────────────► │  (afterLanding → endturn)
    │    └────┬────┘                               │
    │         │ afterLanding()                      │
    │         ▼                                     │
    │    ┌──────────┐                               │
    │    │ endturn  │  endTurn() ────────────────── ┘
    │    └────┬─────┘  (advance to next player)
    │ doubles │
    └─────────┘

         any point: checkWinCondition()
              └──► gameover
```

Doubles allow the same player to roll again (up to 3 times; the third double sends them to jail).

---

## localStorage Persistence

**Key:** `monopoly-singlefile-save`

**Format:** Full `JSON.stringify(gs)` written after every state-changing action (`saveGame()`). On page load, if the key exists the resume banner is shown; the player can resume or start fresh (which deletes the save).

The serialized object includes all runtime field state (`ownerId`, `mortgaged`, `houses`, `hotel`), the full shuffled decks with draw pointers, all pending interactions, and the complete game log.

---

## Design Decisions & Patterns

### Single `gs` Object
All mutable game state lives in one global `gs`. Mutations are always followed by targeted re-renders (`renderBoard()`, `renderAllPlayerCards()`, etc.) and `saveGame()`. The object is fully JSON-serializable, making persistence trivial.

### Pending State Pattern
Complex multi-step interactions (buy, auction, trade, bankruptcy, card resolution) are mediated through `gs.pending*` objects. The modal opens, the player interacts, and the resolve function reads the pending object before clearing it. This makes every modal idempotent — closing and re-opening returns the same state.

### Phase-Gated Actions
The five action buttons are enabled/disabled entirely based on `gs.phase` and player capabilities. Code that triggers game-advancing actions (e.g. `handleRoll`) guards with `if (gs.phase !== 'roll') return` so duplicate calls are safe.

### Auto-End Turn
`updateActionButtons()` checks whether a human player in phase `'endturn'` has no meaningful actions (cannot build, no human trade partners). If so, a 600 ms `setTimeout` automatically calls `endTurn()`, with guards for open modals and phase changes.

### AI Uses Same Code Paths
AI players call `openBuyModal`, `executeRoll`, `triggerBankruptcy`, etc. — the same functions used for humans. The AI branch is the early-return path at the top of each function (`if (p.isAI) { ... return; }`), keeping all rule enforcement centralised.

### Function-Declaration Hoisting
The JavaScript is written in four sequential parts appended to the same `<script>` block. Forward-declared stub functions (`function foo() {}`) in earlier parts are overwritten by the real declarations in later parts, relying on JS function-declaration hoisting so cross-part calls work without any module system.

### Even-Building Enforcement
`doBuildHouse` checks that the target field has `houses === groupMinimum` before allowing a build. `doSellHouse` checks `houses === groupMaximum`. This enforces the official even-building rule at the data layer rather than just the UI layer.

### German Localisation
All UI text, board names, card texts, and log messages are in German. Currency is formatted with `toLocaleString('de-DE')` (thousands separator as full stop). The `fmt(n)` helper wraps this and appends ` €`.

---

## Rent Tables (Streets)

| Group | Price | H-Cost | Base | Monopoly | 1H | 2H | 3H | 4H | Hotel |
|-------|-------|--------|------|----------|----|----|----|-----|-------|
| Brown | 60 | 50 | 2/4 | 4/8 | 10/20 | 30/60 | 90/180 | 160/320 | 250/450 |
| Light Blue | 100/120 | 50 | 6/8 | 12/16 | 30/40 | 90/100 | 270/300 | 400/450 | 550/600 |
| Pink | 140/160 | 100 | 10/12 | 20/24 | 50/60 | 150/180 | 450/500 | 625/700 | 750/900 |
| Orange | 180/200 | 100 | 14/16 | 28/32 | 70/80 | 200/220 | 550/600 | 750/800 | 950/1000 |
| Red | 220/240 | 150 | 18/20 | 36/40 | 90/100 | 250/300 | 700/750 | 875/925 | 1050/1100 |
| Yellow | 260/280 | 150 | 22/24 | 44/48 | 110/120 | 330/360 | 800/850 | 975/1025 | 1150/1200 |
| Green | 300/320 | 200 | 26/28 | 52/56 | 130/150 | 390/450 | 900/1000 | 1100/1200 | 1275/1400 |
| Dark Blue | 350/400 | 200 | 35/50 | 70/100 | 175/200 | 500/600 | 1100/1400 | 1300/1700 | 1500/2000 |

*Two values separated by `/` indicate the cheaper / more expensive street in the group.*

### Railroads

| Railroads owned | Rent |
|----------------|------|
| 1 | 25 € |
| 2 | 50 € |
| 3 | 100 € |
| 4 | 200 € |

### Utilities

| Utilities owned | Rent |
|----------------|------|
| 1 | 4 × dice total |
| 2 | 10 × dice total |

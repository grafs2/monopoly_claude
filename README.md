# Monopoly — Single-File Web Implementation

A fully playable, feature-complete Monopoly game implemented as a single, self-contained HTML file. This project follows the classic German "Monopoly Classic" rules and is designed to run locally in any modern web browser without the need for a web server, build tools, or external dependencies (aside from Google Fonts).

## 🚀 Features

- **Single File Architecture**: Everything (HTML, CSS, and JavaScript) is contained within `index.html`.
- **2–4 Players**: Support for any combination of human and AI players (at least 2 players total).
- **Smart AI Opponents**: AI players utilize heuristics for buying properties, building houses, participating in auctions, and managing finances.
- **Full Rule Set**:
    - Property trading and auctions.
    - Building houses and hotels (with even-building enforcement).
    - Mortgages and unmortgaging.
    - Jail mechanics (doubles, paying, or cards).
    - Chance and Community Chest cards.
    - Bankruptcy and debt liquidation.
- **Save & Resume**: Automatically saves game state to `localStorage` after every turn, allowing players to resume games after a browser refresh.
- **Responsive UI**: A modern CSS Grid-based game board (11x11) that scales for desktop use with a professional "felt-green" aesthetic.
- **German Localization**: All board names, cards, and UI elements are in German.

## 🛠 Technical Stack

- **HTML5**: Semantic structure and modal systems.
- **CSS3**: Advanced layouts using CSS Grid and Flexbox, custom properties for themes, and animations for dice/movement.
- **Vanilla JavaScript**: Pure ES6 logic with no external frameworks.
    - **State Management**: A centralized `gameState` object ensures data integrity.
    - **Phase Controller**: A state-machine-driven turn logic (Roll -> Action -> End Turn).
    - **Persistence**: JSON serialization for `localStorage`.

## 🎮 How to Play

1.  **Run the game**: Simply open `index.html` in any modern web browser (Chrome, Firefox, Edge, Safari).
2.  **Setup**: Select the number of players and AI. Assign names and colors.
3.  **Gameplay**:
    - Click **"Würfeln"** to move your token.
    - Follow the prompts to buy properties or pay rent.
    - Use the **"Bauen"**, **"Handeln"**, and **"Verwalten"** buttons in the footer to manage your empire.
4.  **Winning**: The last player remaining after all others have gone bankrupt is declared the winner.

## 🏗 Project Structure

The code is organized into logical regions within the single file:

| Section | Description |
| :--- | :--- |
| **CSS Variables** | Definition of colors, board sizes, and field dimensions. |
| **Board Grid** | 11x11 Grid layout logic for the classic square board. |
| **Modals** | UI components for auctions, trading, and property management. |
| **Data Definitions** | Hardcoded arrays for fields, cards, and property rents. |
| **Game Logic** | Core functions for movement, rent calculation, and AI heuristics. |
| **UI Sync** | DOM update functions that reflect the `gameState` to the user. |

## ⚖️ Rules Implemented

- **Rent**: Base rent, doubled for monopolies, and scaled for houses/hotels.
- **Railroads & Utilities**: Variable rent based on ownership and dice rolls.
- **Trading**: Human-to-human property and cash exchange.
- **Auctions**: Triggered when a player declines to buy a property.
- **Building**: Only allowed on full monopolies; houses must be built and sold evenly across the group.
- **Mortgages**: 10% interest charged when lifting a mortgage.

## 📝 Development Notes

- The project uses **Forward-declared stub functions** and hoisting to organize logic sequentially without a module system.
- **Event Delegation** is used sparingly, preferring direct state-to-UI binding.
- The **AI logic** prioritizes maintaining a 200€ cash reserve to avoid immediate bankruptcy.

---
*Note: This is an educational implementation of the classic board game logic.*
---

## R.O.T.O.R. – Retro-Höhlenrettung (`rotor/index.html`)

Ein eigenständiges Single-File-Spiel im Stil der C64-Höhlenrettungs-Klassiker von 1984 (Rotor-Rucksack, Helm-Laser, Dynamit, Power-Leiste). Figur, Titel, Level und Musik sind eigene Entwürfe.

- 320×200 virtueller Bildschirm, 16-Farben-C64-Palette, Multicolor-Pixel (2×1)
- 8 handgebaute Level mit Bildschirmwechsel (Flip-Screen), danach PRO-Durchgang mit schnellerem Energieverbrauch
- Spinnen, Fledermäuse, Motten, Wandschlangen, Tentakel, Magma, Wasser, Laternen (Dunkelheit), sprengbare Wände
- Punkte: Gegner 50, Wand 75, Rettung 1000, Rest-Power und Rest-Dynamit als Bonus, Extraleben alle 20.000
- Steuerung: Pfeile/WASD, Leertaste = Laser, Pfeil runter = Dynamit, P = Pause, M = Ton; Gamepad und Touch-Steuerung
- Sound per WebAudio-Synth, keine externen Bibliotheken

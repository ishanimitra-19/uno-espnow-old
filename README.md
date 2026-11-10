# UNO over ESP-NOW

A remix of the Module 3 Button Mash Racer starter code: 2–3 LILYGO T-Display ESP32s play UNO against each other. Each board holds a private hand; the pile (the "communal deck") is shared wirelessly with ESP-NOW. No breadboard or extra hardware.

<!-- TODO: add a GIF of the game here -->

## Files

| File | What it is |
| --- | --- |
| `uno_espnow.ino` | Back end: UNO rules, hands, buttons, ESP-NOW sync |
| `ui_display.ino` | Front end: everything drawn on the screen (placeholder — replace with your own design) |

Both files must stay together in a folder called `uno_espnow`.

## Setup

1. Install the ESP32 board package and the `TFT_eSPI` library, set up for the T-Display, exactly as for the starter code.
2. Open `uno_espnow/uno_espnow.ino` in the Arduino IDE.
3. Set `TEAM_ID` (same on every board) and `PLAYER_ID` (0, 1 or 2 — different on every board).
4. Upload to each board.

## How to play

Hold the board upright with the two buttons at the bottom.

| Button | In the lobby / after a win | During a game |
| --- | --- | --- |
| Left | Deal | Sort through your cards |
| Right | Deal | Put the selected card in the pot |
| Both, held 1.5 s | | Deal a brand new game |

- The top of the screen shows the communal deck; the bottom shows your cards.
- A card can be played if it matches the pile's color or number/symbol, or if it is a Wild.
- Nothing to play? Scroll to the `DRAW` slot at the end of your hand and press the right button: you take one card and your turn passes.
- Wild cards ask for a color: left button changes it, right button confirms.
- `X` = Skip, `R` = Reverse, `+2` = Draw 2, `W` = Wild, `+4` = Wild Draw 4. Draw cards skip the player who draws; they do not stack.
- First player to empty their hand wins.

## How it works

There is no host board. All boards share one small `GameState` struct (top card, whose turn, direction, any draw penalty, winner) that carries a version number: `(round, nonce, seq)`.

- Only the player whose turn it is may change the state, and every move adds 1 to `seq`.
- Every board re-broadcasts the newest state it knows four times a second.
- A board that receives a newer state than its own adopts it, so a lost packet is fixed by the next heartbeat.
- A new deal adds 1 to `round` and picks a random `nonce`, which settles the case where two people press deal at the same moment.
- Hands are never sent. Each board draws its own random cards (with real UNO deck odds) and only reports how many it is holding.

## Writing your own UI

`ui_display.ino` lists everything the back end exposes. The back end calls only `uiSetup()`, `uiError()` and `drawScreen()`; the UI only reads variables such as `game`, `hand[]`, `handCount`, `selected`, `pickingColor` and `handCounts[]`.

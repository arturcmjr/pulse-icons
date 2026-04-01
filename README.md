# Pulse Icon Asset Pipeline

A build tool for the **Pulse** smart home system that acts as the single source of truth for all 24×24 monochromatic UI icons.

It takes a 2D sprite-sheet grid (`input/master_icons.png`) and automatically generates:

| Output file | Purpose |
|---|---|
| `output/icons.h` | C++ PROGMEM header for ESP32 / TFT_eSPI |
| `output/icons.scss` | SCSS file using CSS mask-position for an Angular frontend |

Empty / fully-transparent grid cells at the **end** of the grid are allowed (unused trailing slots). However, the engine **rejects** any sprite sheet that has an empty cell in the middle of the sequence — ensuring the icon index is always a dense, contiguous range starting at `0`.

---

## How the Grid Processing Works

The engine reads the PNG and divides it into a uniform grid of 24×24-pixel cells, scanning left-to-right, top-to-bottom.

Icons must be packed **continuously** from the top-left corner. Each cell is scanned for alpha values — a cell with every pixel at alpha `0` marks the end of the icon set (trailing padding). Any empty cell that is followed by a non-empty cell is treated as a **gap** and causes the build to exit with an error.

```
sprite sheet grid (cols × rows)
┌────┬────┬────┬────┐
│  0 │  1 │  2 │  3 │   ← all filled, valid
├────┼────┼────┼────┤
│  4 │  5 │    │    │   ← trailing empty cells, OK
└────┴────┴────┴────┘
Icons exported: 0 1 2 3 4 5  (2 trailing empty cells ignored)

┌────┬────┬────┬────┐
│  0 │    │  1 │  2 │   ← ERROR: gap at (col 1, row 0)
└────┴────┴────┴────┘
Build fails with an error message.
```

---

## Usage

### Prerequisites

- Node.js ≥ 18
- npm

### Install dependencies

```bash
npm install
```

### Prepare input

Place your sprite sheet at:

```
input/master_icons.png
```

The image **must** have both a width and height that are exact multiples of 24. Icons must be packed continuously left-to-right, top-to-bottom — trailing transparent cells are allowed as padding, but gaps in the middle will cause an error.

### Run the build

```bash
npm run build
```

The generated files will appear in the `output/` directory.

---

## Output Examples

### C++ – ESP32 / TFT_eSPI (`output/icons.h`)

Each non-empty icon is packed into 32 bytes (256 bits, MSB first, 1 = solid pixel).

```cpp
#include "icons.h"
#include <TFT_eSPI.h>

TFT_eSPI tft;

void setup() {
  tft.init();
  tft.fillScreen(TFT_BLACK);

  // Draw icon at index 1 at position (10, 20)
  tft.drawBitmap(10, 20, all_icons[1], 24, 24, TFT_WHITE);
}
```

### Angular HTML – CSS masking (`output/icons.scss`)

Import `icons.scss` into your Angular project and place `master_icons.png` under `src/assets/`.

```html
<!-- Display icon at index 1 -->
<div class="icon icon-1"></div>

<!-- Display icon at index 5 -->
<div class="icon icon-5"></div>
```

The `.icon` base class sets `mask-image` to the sprite sheet, and each `.icon-N` class sets the correct `mask-position` to reveal only that icon's cell.

---

## Project Structure

```
pulse-icons/
├── input/
│   └── master_icons.png   ← place your sprite sheet here
├── output/
│   ├── icons.h            ← generated C++ header
│   └── icons.scss         ← generated SCSS
├── src/
│   └── generate.js        ← the Smart Grid Engine
├── package.json
└── README.md
```

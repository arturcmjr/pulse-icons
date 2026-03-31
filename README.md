# Pulse Icon Asset Pipeline

A build tool for the **Pulse** smart home system that acts as the single source of truth for all 16×16 monochromatic UI icons.

It takes a 2D sprite-sheet grid (`input/master_icons.png`) and automatically generates:

| Output file | Purpose |
|---|---|
| `output/icons.h` | C++ PROGMEM header for ESP32 / TFT_eSPI |
| `output/icons.scss` | SCSS file using CSS mask-position for an Angular frontend |

Empty / fully-transparent grid cells are detected and **automatically skipped**, so no flash memory or CSS rules are wasted on blank slots.

---

## How the Grid Processing Works

The engine reads the PNG and divides it into a uniform grid of 16×16-pixel cells, scanning left-to-right, top-to-bottom.

Before processing each cell it inspects **all 256 pixels**.  
If every pixel has an alpha value of `0` (fully transparent), the cell is considered an _empty slot_ and is silently skipped.  
Only non-empty cells receive a sequential `validIconIndex` and produce output — this index is contiguous and always starts at `0`.

```
sprite sheet grid (cols × rows)
┌────┬────┬────┬────┐
│  0 │  1 │    │  2 │   ← cell 2 is empty → skipped
├────┼────┼────┼────┤
│  3 │    │  4 │  5 │   ← cell 5 is empty → skipped
└────┴────┴────┴────┘
Valid icons: 0 1 2 3 4  (5 icons from 8 cells)
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

The image **must** have both a width and height that are exact multiples of 16.

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
  tft.drawBitmap(10, 20, all_icons[1], 16, 16, TFT_WHITE);
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

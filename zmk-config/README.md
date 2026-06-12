# Split KB — ZMK Firmware Config

Custom split keyboard firmware for **nice!nano v2** + Kailh Choc v1 switches
with rotary encoders. Uses [ZMK Firmware](https://zmk.dev).

---

## ⚠️ Before You Build — Verify Pin Assignments

The PDF text extraction of your KiCad schematic does not show wire routing,
so pin assignments are based on the standard nice!nano Lily58 configuration.
**You must cross-check these in KiCad before flashing:**

| Signal    | Pin  | nice!nano pad | In overlay file              |
|-----------|------|---------------|------------------------------|
| ROW0      | P0.31| 17            | `<&gpio0 31 ...>`            |
| ROW1      | P0.29| 18            | `<&gpio0 29 ...>`            |
| ROW2      | P0.02| 19            | `<&gpio0  2 ...>`            |
| ROW3      | P1.15| 20            | `<&gpio1 15 ...>`            |
| ROW4      | P1.13| 21            | `<&gpio1 13 ...>`            |
| COL0      | P0.22| 7             | `<&gpio0 22 ...>`            |
| COL1      | P0.24| 8             | `<&gpio0 24 ...>`            |
| COL2      | P1.00| 9             | `<&gpio1  0 ...>`            |
| COL3      | P0.11| 10            | `<&gpio0 11 ...>`            |
| COL4      | P1.04| 11            | `<&gpio1  4 ...>`            |
| COL5      | P1.06| 12            | `<&gpio1  6 ...>`            |
| Encoder A | P1.11| 22            | `a-gpios = <&gpio1 11 ...>`  |
| Encoder B | P0.09| 24            | `b-gpios = <&gpio0  9 ...>`  |

If keys register wrong or are shifted, you may need to:
- Swap `col2row` ↔ `row2col` in the overlay (and swap col/row gpio entries)
- Reorder the `col-gpios` or `row-gpios` lists to match your trace layout

---

## How to Build (GitHub Actions — no toolchain needed)

1. **Fork this repo** to your GitHub account

2. **Push any change** (or go to Actions → Build ZMK Firmware → Run workflow)

3. GitHub builds both halves automatically. When it finishes, go to:
   **Actions → latest run → Artifacts**
   and download `firmware.zip`

4. Inside you'll find:
   - `nice_nano_v2-split_kb_left-zmk.uf2`  ← flash to LEFT  half
   - `nice_nano_v2-split_kb_right-zmk.uf2` ← flash to RIGHT half

---

## How to Flash

1. Double-tap the **RST** button on the nice!nano — it appears as a USB drive
   called `NICENANO`
2. Drag the corresponding `.uf2` file onto the drive
3. The drive disappears and the firmware is flashed — done
4. Flash the left half first, then the right half

**First pairing after flash:**
- Left half connects to your PC/phone via BLE as "Split KB"
- Right half pairs to the left half automatically (give it ~30 seconds)
- If they don't pair, hold LOWER+RAISE to enter ADJUST layer → tap the
  `BT_CLR` key (row 3, col 5 on left) to clear stored bonds, then re-pair

---

## Keymap Layers

| Layer  | Activated by        | Contents                              |
|--------|---------------------|---------------------------------------|
| QWERTY | Default             | Standard QWERTY                       |
| LOWER  | Hold left thumb     | F1–F10, number row on home, symbols   |
| RAISE  | Hold right thumb    | F11/F12, arrow keys, brackets         |
| ADJUST | LOWER + RAISE       | BLE profiles (BT_SEL 0–4), BT_CLR, bootloader, reset |

### Encoder bindings

| Layer  | Left encoder (SW57) | Right encoder (SW58)  |
|--------|---------------------|-----------------------|
| QWERTY | Vol Up / Vol Dn     | Page Up / Page Dn     |
| LOWER  | Brightness +/−      | Next / Prev track     |
| RAISE  | Vol Up / Vol Dn     | Next / Prev track     |
| ADJUST | Vol Up / Vol Dn     | Page Up / Page Dn     |

Encoder **click** (push): Mute (left) / Play-Pause (right) on the default layer.
These are regular matrix keys — remap them in `split_kb.keymap` as you like.

---

## Customising the Keymap

Edit `config/split_kb.keymap`. Key bindings use ZMK's `&kp KEY` syntax.
Full key code reference: https://zmk.dev/docs/codes

To remap the encoder rotation, change the `sensor-bindings` lines in each layer:
```
sensor-bindings = <&inc_dec_kp CLOCKWISE_KEY COUNTER_CW_KEY
                    &inc_dec_kp RIGHT_CW_KEY  RIGHT_CCW_KEY>;
```

---

## File Structure

```
zmk-config/
├── build.yaml                          ← which board+shield pairs to build
├── .github/workflows/build.yml         ← GitHub Actions workflow
└── config/
    ├── west.yml                        ← ZMK source manifest
    ├── split_kb.keymap                 ← key bindings (edit this)
    ├── split_kb_left.conf              ← left half Kconfig
    ├── split_kb_right.conf             ← right half Kconfig
    └── boards/shields/split_kb/
        ├── Kconfig.shield              ← shield declaration
        ├── Kconfig.defconfig           ← default config values
        ├── split_kb_left.overlay       ← left  hardware (pins, matrix, encoder)
        └── split_kb_right.overlay      ← right hardware (pins, matrix, encoder)
```

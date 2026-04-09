# Hold-Tap Behavior Reference

This document is a detailed reference for the hold-tap behaviors used in this keymap. It supplements the ZMK docs with repo-specific values and reasoning so that timing and positioning decisions can be made without re-deriving from first principles.

Source: [ZMK hold-tap docs](https://v0-3-branch.zmk.dev/docs/keymaps/behaviors/hold-tap)

---

## Core Parameters

| Parameter | What it controls | This repo's values |
|---|---|---|
| `tapping-term-ms` | How long the key must be held before the hold action triggers (all else equal) | `hml`/`hmr`: 250ms · `hml_shift`/`hmr_shift`: 200ms · `tm`/`tm_magic_shift`: 300ms |
| `flavor` | How an interrupting keypress affects the hold/tap decision — see section below | `balanced` on hml/hmr/tm/tm_magic_shift · `tap-preferred` on hml_shift/hmr_shift |
| `hold-trigger-key-positions` | Array of key indices; hold only triggers if the interrupting key's position is in this list | Opposite-hand keys for hml/hmr; all non-thumb keys for tm — see section below |
| `hold-trigger-on-release` | Delays `hold-trigger-key-positions` evaluation until the interrupting key is *released* rather than pressed | Enabled on all behaviors — required for reliable multi-modifier chording |
| `require-prior-idle-ms` | Forces tap resolution if the hold-tap is pressed within this window after another non-modifier key | `hml`/`hmr`: 70ms · not set on shift variants or thumb behaviors |
| `quick-tap-ms` | If the hold-tap is pressed again within this window after a tap, always resolves as tap (enables key repeat) | `hml`/`hmr`: 180ms · `tm`: 260ms · `tm_magic_shift`: 1500ms (intentionally long) |

---

## Flavor Decision Logic

Flavor determines what happens when another key is pressed *while* the hold-tap key is held but before `tapping-term-ms` expires.

### `balanced` (used by `hml`, `hmr`, `tm`, `tm_magic_shift`)

Hold triggers if:
- `tapping-term-ms` expires, **or**
- An interrupting key is pressed **and released** before the hold-tap is released

This prevents accidental modifiers during fast typing (a quick roll where you release both keys fast resolves as tap) while still allowing intentional holds where you type a full key while holding a modifier.

### `tap-preferred` (used by `hml_shift`, `hmr_shift`)

Hold triggers **only** when `tapping-term-ms` expires. Interrupting key presses alone never trigger hold.

Shift uses this because same-hand rolls into shift are extremely common (e.g., rolling `F`→`E` to get `E`, not `Shift+E`). The stricter flavor means you must hold shift for the full tapping term before it activates, which is fine for intentional shift but prevents false positives on fast typing.

### `hold-preferred` (ZMK default for `&mt` — not used here)

Hold triggers if `tapping-term-ms` expires **or** any key is pressed. Too aggressive for homerow use.

### `tap-unless-interrupted` (not used here)

Hold triggers **only** if another key is pressed before `tapping-term-ms` expires. Opposite of hold-preferred.

---

## `hold-trigger-key-positions` in This Repo

Key indices are assigned sequentially across the keymap matrix. The point of this parameter is to ensure the hold behavior only activates for "valid" combinations — typically, cross-hand for homerow mods.

### `hml` (left homerow: A S D F)

Positions = all right-hand finger keys + all thumb keys.

Effect: A left-hand homerow mod will only activate if the interrupting key is on the right hand or a thumb. Prevents left-hand same-side typing rolls from accidentally firing modifiers.

### `hmr` (right homerow: J K L ;)

Positions = all left-hand finger keys + all thumb keys.

Effect: Same principle, mirrored. Right-hand mods only activate for left-hand or thumb interrupts.

### `hml_shift` / `hmr_shift` (F key left, J key right)

Positions = a narrower subset — specific opposite-hand keys and thumb positions.

Effect: More permissive than the standard mods — shift must be able to combine with same-hand keys for capitalisation (e.g., `Shift+E`, `Shift+W`). The narrower list still prevents the most common false triggers while allowing intentional shift combos.

### `tm` (thumb layer-tap: backspace/space)

Positions = all 50 finger keys (indices 0–49, everything except thumbs).

Effect: Any finger keypress can trigger the layer hold. Excludes other thumb keys to avoid thumb-chord conflicts (pressing two thumbs simultaneously shouldn't accidentally activate a layer through the wrong thumb).

### `tm_magic_shift` (thumb shift)

Positions = all finger keys (same as `tm`), balanced flavor.

---

## Behavior-by-Behavior Summary

| Behavior | Hold action | Tap action | Flavor | Tapping term | `require-prior-idle-ms` | `quick-tap-ms` | Notes |
|---|---|---|---|---|---|---|---|
| `hml` | `&kp MOD` | `&kp KEY` | balanced | 250ms | 70ms | 180ms | Left homerow mods (LCTRL/LALT/LGUI on A/S/D) |
| `hmr` | `&kp MOD` | `&kp KEY` | balanced | 250ms | 70ms | 180ms | Right homerow mods (RCTRL/RALT/RGUI on ;/L/K) |
| `hml_shift` | `&kp LSHIFT` | `&kp KEY` | tap-preferred | 200ms | — | 180ms | Left shift homerow mod (F key) |
| `hmr_shift` | `&kp RSHIFT` | `&kp KEY` | tap-preferred | 200ms | — | 180ms | Right shift homerow mod (J key) |
| `tm` | `&mo LAYER` | `&kp KEY` | balanced | 300ms | — | 260ms | Thumb layer-tap (backspace→layer 2, space→layer 1) |
| `tm_magic_shift` | `&kp MOD` | `&shift_or_caps_word` | balanced | 300ms | — | 1500ms | Tap = shift/caps-word tap-dance; long quick-tap enables deliberate double-tap to caps-word |

---

## Tuning Guide

| Symptom | Likely cause | Fix |
|---|---|---|
| Accidental modifier fires during fast typing | `require-prior-idle-ms` too low, or flavor too aggressive | Increase `require-prior-idle-ms` on the affected behavior, or switch flavor to `tap-preferred` |
| Intentional hold feels delayed / laggy | `tapping-term-ms` too high | Decrease `tapping-term-ms` |
| Same-hand chord accidentally triggers hold | `hold-trigger-key-positions` too broad (includes same-hand keys) | Remove same-hand key indices from the positions list |
| Multi-modifier chording unreliable (e.g., Ctrl+Shift) | `hold-trigger-on-release` not set, or chording keys not in each other's positions lists | Ensure `hold-trigger-on-release` is set; verify both modifier keys are in each other's `hold-trigger-key-positions` |
| Key repeat not working on a hold-tap key | `quick-tap-ms` missing or too high | Add or decrease `quick-tap-ms` |
| Caps-word double-tap on `tm_magic_shift` not triggering | `quick-tap-ms` on that behavior prevents re-tap | Intentional — the 1500ms window is meant to force a deliberate double-tap |

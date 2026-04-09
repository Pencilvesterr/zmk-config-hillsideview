# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

This is a ZMK firmware configuration repository for the **Hillside View** keyboard — a split ergonomic 46-key keyboard with nice!nano v2 microcontrollers and nice!view gem OLED displays. ZMK (Zephyr Mechanical Keyboard) is an open-source keyboard firmware based on the Zephyr RTOS.

## Building Firmware

Firmware is built automatically via GitHub Actions on every push. To trigger a build, simply push changes. Download the compiled `.uf2` artifacts from the Actions tab.

**Build matrix** (`build.yaml`) compiles four targets:
- `nice_nano_v2` + `hillside_view_left nice_view_gem`
- `nice_nano_v2` + `hillside_view_right nice_view_gem`
- `nice_nano_v2` + `settings_reset` (utility to clear BLE bonds)

To flash: copy the downloaded `.uf2` file to the bootloader drive that appears when double-tapping the reset button.

## Repository Structure

```
config/
├── hillside_view.keymap           # Primary keymap — custom behaviors and layers
├── hillside_view.conf             # Main firmware config (BT, display, sleep, input)
├── hillside_view.json             # Key position geometry for keymap-editor
├── west.yml                       # Pinned ZMK and nice-view-gem dependency commits
└── boards/shields/hillside_view/
    ├── hillside_view.zmk.yml      # Shield definition (features, siblings)
    ├── hillside_view.conf         # Shield-level config (I2C, Pinnacle trackpad)
    ├── hillside_view.keymap       # Simpler alternative/example keymap (not primary)
    └── hillside_view_{left,right}.conf  # Per-side overrides
```

The **primary keymap** is `config/hillside_view.keymap`. The one in `boards/shields/` is a simpler fallback/example.

## Keymap Architecture

The keymap uses three layers:

1. **Default** — QWERTY with homerow mods (`hml`/`hmr` behaviors)
2. **Layer 1** — Symbols and numbers (accessed via thumb hold or shift+enter)
3. **Layer 2** — Navigation, function keys, Bluetooth control (accessed via shift+backspace)

### Custom Behaviors

All behaviors are defined in `config/hillside_view.keymap` under the `behaviors` block. See [`docs/hold-tap-reference.md`](docs/hold-tap-reference.md) for a detailed parameter reference, flavor decision logic, and tuning guide specific to this repo.

- **`hml` / `hmr`** — Homerow modifiers (balanced flavor, 250ms tapping term). Hold positions are configured so only the opposite hand can trigger hold, preventing accidental modifier activation while typing.
- **`hml_shift` / `hmr_shift`** — Shift-specific homerow mod with tap-preferred flavor.
- **`tm`** — Thumb layer-tap (300ms tapping term, balanced flavor).
- **`tm_magic_shift`** — Shift with caps-word on hold; uses combo for quick-tap.
- **`backspace_or_delete`** — Mod-morph: backspace alone, delete with shift held.
- **`skq`** — Sticky key with quick-release (1000ms).
- **`shift_or_caps_word`** — Tap-dance: tap = shift, double-tap = caps word.

### Key Tuning Parameters

Defined in `hillside_view.conf`:
- Debounce: `10ms` press and release
- Idle timeout: `30000ms` (30s)
- Sleep timeout: `600000ms` (10min)
- Bluetooth TX power: `+8dBm`
- ZMK Studio: enabled (`CONFIG_ZMK_STUDIO=y`)

## ZMK Concepts Reference

- **Behaviors** — Actions bound to keys (hold-tap, mod-morph, tap-dance, etc.)
- **Hold-trigger positions** — In `hml`/`hmr`, these restrict which key positions can trigger the "hold" action, preventing accidental modifier activation.
- **`balanced` flavor** — Hold-tap flavor that decides tap vs hold based on whether the next key is pressed and released before the tapping term.
- **`tap-preferred` flavor** — Prefers tap unless held longer than the tapping term.
- **Shields** — ZMK term for the keyboard hardware definition module.
- **west** — Zephyr's package manager; `config/west.yml` pins dependency commits.

## Editing Layouts

The key positions and layout are visualized using [keymap-editor](https://nickcoutsos.github.io/keymap-editor/). The `config/hillside_view.json` file defines the physical key geometry used by that tool.

When modifying behaviors (especially homerow mods), the `hold-trigger-key-positions` arrays must list the key indices for the opposite hand. These indices correspond to the key matrix positions in the shield definition.

# zmk-config: Lily58

ZMK firmware config for my wireless Lily58 split keyboard. GitHub Actions builds the firmware. Nothing gets compiled locally. Agents: also read `AGENTS.md`.

## Hardware
- Lily58 split, a 6x4 grid plus 4 thumb keys and 1 inner key per half (58 keys)
- 2x nice!nano v2 controllers, one per half. The left half is the central and talks to the host.
- 2x nice!view displays (`nice_view_adapter nice_view` shields)
- Kailh Choc Pink switches

## Files
| File | What it does |
|---|---|
| `config/lily58.keymap` | Layers and key bindings. Most changes go here. |
| `config/lily58.conf` | Kconfig flags: display on, mouse keys (pointing) on, encoder off |
| `config/west.yml` | Which ZMK gets pulled in (upstream `main`) |
| `build.yaml` | Build matrix, left and right halves |
| `.github/workflows/build.yml` | CI that calls ZMK's reusable `build-user-config.yml` |

### Upstream ZMK
`west.yml` tracks `zmkfirmware/zmk` at `main`, and the CI workflow uses upstream's `build-user-config.yml@main`. Both move together, so the official docs at zmk.dev apply. Mouse keys use upstream's pointing support (`CONFIG_ZMK_POINTING`, `dt-bindings/zmk/pointing.h`, `ZMK_POINTING_DEFAULT_SCRL_VAL`). Scroll is set to 40, versus upstream's default of 10.

The catch is that ZMK's `main` can change under you. If a build breaks with no change to this repo, check the ZMK blog for migration notes. The 25-12 Zephyr 4.1 update, for example, renamed the board from `nice_nano_v2` to `nice_nano//zmk`.

(Until 26-10 this config used petejohanson's `feat/pointers-move-scroll` fork for mouse support.)

## Update the keymap
1. Edit `config/lily58.keymap`. Each layer has an ASCII diagram comment above its `bindings`.
2. Commit and push to `master`. Every push triggers a build (pull requests and manual `workflow_dispatch` runs do too).
3. Wait for the build and download the firmware:
   ```
   gh run watch                                                   # pick the latest run
   rm -rf ~/Downloads/lily58 && gh run download -n firmware -D ~/Downloads/lily58
   ```
   Or go to GitHub, then **Actions**, then the run, and download the `firmware` artifact.
4. There's one `.uf2` per half, named after its shield (`lily58_left ...` and `lily58_right ...`).

If a build fails, `gh run view --log-failed` shows the error. Look at the `West Build` step. Devicetree errors there usually point to a line in the keymap.

## Flash the keyboard
Flash each half on its own:
1. Plug the half into the computer over USB.
2. Double-tap its reset button. It mounts as a USB drive (`NICENANO`).
3. Copy that half's `.uf2` to the drive. The drive unmounts and the half reboots once the copy finishes. A "drive ejected unexpectedly" warning is normal.
4. Repeat for the other half.

After a keymap-only change, flashing just the **left** half is usually enough, because the central holds the keymap. Flash both halves whenever ZMK, modules, or `.conf` change.

### When things go wrong
- **Halves don't connect, or pairing is stuck:** flash a settings-reset build to both halves, then flash the normal firmware back onto both. To get the reset build, add a build entry to `build.yaml` with `shield: settings_reset` on `nice_nano_v2`. After that, remove the host's old Bluetooth pairing and pair the keyboard again.
- **Re-pair one device:** on `BLTH`, select the profile (1–5), press Q (`BT_CLR`) to clear it, then pair from the host. Only that profile is affected.
- **Keyboard plugged in but typing over Bluetooth (or vice versa):** `BLTH` + W toggles USB/BLE output.

## Layout

| # | Layer | How to reach it | What's on it |
|---|---|---|---|
| 0 | `DEFAULT` | always | QWERTY. Number row, `` ` `` top-left, `\` on the right of the Tab row, `-` and `=` on the inner keys |
| 1 | `OHND` | hold left Esc thumb (tap = Esc), or hold right outer thumb | F1–F12, mouse buttons and scroll, PgUp/PgDn, Backspace, Ctrl+Backspace. Mirrored on both hands for one-handed use |
| 4 | `MODL` | hold left inner thumb | Left home-row mods (GUI/Alt/Shift/Ctrl on A/S/D/F). **T** gives `SYMB`, **G** gives `NAVL`, **B** gives `NUMP`. Caps Lock on the outer bottom keys |
| 5 | `MODR` | hold right inner thumb | Right home-row mods (Ctrl/Shift/Alt/GUI on J/K/L/;). **H** gives `NAVR` |
| 2 | `NUMP` | `MODL` + B | Left mods, right-hand numpad and brackets |
| 3 | `SYMB` | `MODL` + T | Left mods, right-hand symbols |
| 6 | `NAVL` | `MODL` + G | Left mods, right-hand arrows (on HJKL), Home/End/Del/Ins, F-row |
| 7 | `NAVR` | `MODR` + H | Mirror of `NAVL`, with arrows on the left hand and mods on the right |
| 8 | `BLTH` | hold both inner thumbs | BT profiles 1–5 (keys 1–5), clear current BT profile (Q), USB/BLE output toggle (W), external power toggle (`` ` `` key), media play/vol (6/7/8), `GAME` toggle (G) |
| 9 | `GAME` | `BLTH` + G toggles it. Press G again to turn it off | WASD on the left, keypad digits on the right |

On every layer, `sensor-bindings` maps an encoder to volume, but the encoder is off (`CONFIG_EC11` is commented out in `lily58.conf`).

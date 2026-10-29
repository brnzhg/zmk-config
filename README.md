# zmk-config: Lily58

ZMK firmware config for my wireless Lily58 split keyboard. GitHub Actions builds the firmware. Nothing gets compiled locally.

## Hardware
- Lily58 split, a 6x4 grid plus 4 thumb keys and 1 inner key per half (58 keys)
- 2x nice!nano v2 controllers, one per half. The left half is the central and talks to the host.
- 2x nice!view displays (`nice_view_adapter nice_view` shields)
- Kailh Choc Pink switches

## Files
| File | What it does |
|---|---|
| `config/lily58.keymap` | Layers and key bindings. Most changes go here. |
| `config/lily58.conf` | Kconfig flags: display on, mouse on, encoder off |
| `config/west.yml` | Which ZMK and which modules get pulled in |
| `build.yaml` | Build matrix, left and right halves |
| `.github/workflows/build.yml` | CI that calls ZMK's reusable `build-user-config.yml` |

### Not upstream ZMK
`west.yml` pins ZMK to **petejohanson's `feat/pointers-move-scroll` branch**, not `zmkfirmware/zmk`. That branch is where the mouse move/scroll support (`&mkp`, `&msc`, `ZMK_MOUSE_DEFAULT_SCRL_VAL`, `CONFIG_ZMK_MOUSE`) came from. Keep this in mind:
- Upstream ZMK docs might not match this branch. When a feature doesn't work, check whether the branch actually has it.
- The CI workflow uses `build-user-config.yml@main` from *upstream* while the build uses the fork. If upstream changes that workflow in a way that breaks the old branch, pin the `uses:` line to an older ref.
- Switching to upstream ZMK is possible, since pointer support has since been merged there. Expect renames, though, such as `CONFIG_ZMK_POINTING` and `dt-bindings/zmk/pointing.h`. Check the current upstream docs before trying it.

## Update the keymap
1. Edit `config/lily58.keymap`. Each layer has an ASCII diagram comment above its `bindings`.
2. Commit and push to `master`. Every push triggers a build (pull requests and manual `workflow_dispatch` runs do too).
3. Go to GitHub, then **Actions**, then the latest run, and download the `firmware` artifact (a zip).
4. Unzip it. There's one `.uf2` per half: `lily58_left nice_view_adapter nice_view-nice_nano_v2-zmk.uf2` and the matching right file.

If a build fails, open the failed job's log and look at the `West Build` step. Devicetree errors there usually point to a line in the keymap.

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

## Notes for agents editing this repo
- **Keep the ASCII comments in sync with `bindings`.** When you edit a layer, update its diagram too. If the two ever disagree, `bindings` is what's on the keyboard.
- Each layer's `bindings` must hold exactly 58 entries, laid out as 6 + 6 / 6 + 6 / 6 + 6 / 6 + 1 + 1 + 6 / 4 + 4 thumbs. The two inner keys sit at the end of row 4, between `B` and `N`.
- The layer `#define`s at the top must match the order of the layer nodes inside `keymap`.
- Layers that should pass through use `&trans`. The mod layers rely on that, since they're mostly transparent.
- There's no local build setup on this machine. Validate a change by pushing and checking the Actions run. `podman` is installed if a local ZMK build container is ever wanted.

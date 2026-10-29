ZMK config for my Lily58. Read `README.md` first for the hardware, the layer layout, and how building and flashing work.

# Editing the keymap
- **Keep the ASCII comments in sync with `bindings`.** When you edit a layer, update its diagram too. If the two ever disagree, `bindings` is what's on the keyboard.
- Each layer's `bindings` must hold exactly 58 entries, laid out as 6 + 6 / 6 + 6 / 6 + 6 / 6 + 1 + 1 + 6 / 4 + 4 thumbs. The two inner keys sit at the end of row 4, between `B` and `N`.
- The layer `#define`s at the top must match the order of the layer nodes inside `keymap`.
- Layers that should pass through use `&trans`. The mod layers rely on that, since they're mostly transparent.
- When a change adds or moves a key, update the layout table in `README.md`.
- Check zmk.dev for behavior and Kconfig names. This tracks upstream `main`, so older blog posts and forum answers may use outdated names (`mouse.h`, `CONFIG_ZMK_MOUSE`, `nice_nano_v2`).

# Building
- There's no local build setup on this machine. Every push to `master` runs a GitHub Actions build, so commit and push only when I ask.
- After pushing, run `gh run watch` on the new run. On failure, run `gh run view --log-failed`. A failure with no change to this repo usually means upstream ZMK changed, so check the ZMK blog.
- On success, download the firmware with `rm -rf ~/Downloads/lily58 && gh run download <run-id> -n firmware -D ~/Downloads/lily58`. Then tell me which halves to flash: left only for keymap-only changes, both for `.conf`, `west.yml`, or ZMK changes.
- `podman` is installed if a local ZMK build container is ever wanted.

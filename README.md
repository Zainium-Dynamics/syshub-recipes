# syshub-recipes

Package recipes for the base OS (syshub) of Zainium OS, an immutable Linux distribution built on musl.

Recipes use the same `ZEXBUILD` format and CI as [userland-recipes](https://github.com/Zainium-Dynamics/userland-recipes), which documents the format in full.

## How it differs from userland-recipes

- `manifest.toml` sets `_syshub = true` and installs under `/overlayer/syshub/`.
- Toolchain packages that are already linked for the target set `native = true` so packing skips the interpreter and RPATH patching.
- The core team maintains this repo. Public package contributions go to userland-recipes.
- A `-dev` subpackage of a syshub package is published to userland. Give it its own `<name>-dev.manifest.toml` with `_syshub = false`.

## Layout

Most packages install into the usual `bin`, `lib`, `include` and `share` directories under `/overlayer/syshub/`. `lib64` is a symlink to `lib`.

Some packages install elsewhere:

- Cross toolchain (`musl`, `gcc-musl-cross`, `binutils`): `/overlayer/syshub/x86_64-zainium-linux-musl/`
- Firmware: `/overlayer/syshub/drivers/hardware/firmwares/`
- Kernel modules: `/overlayer/syshub/drivers/modules/<kernel-version>/`
- Service definitions: `/overlayer/syshub/engine/services/`
- Quantra (PID 1): `/overlayer/syshub/engine/`

## CI

GitHub Actions builds each changed package in an Alpine container. Pull requests are build-only. Every push to `master` builds, verifies the musl interpreter, packs with [substrate](https://crates.io/crates/substrate-zainium) and publishes to `core/syshub/x86_64/`.

## License

The recipes in this repository are licensed under GPL-3.0-only. See `LICENSE`. Upstream software keeps its own license.

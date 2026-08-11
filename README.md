# syshub-recipes

Recipes for Zainium OS's syshub (base OS) packages — the immutable core, not the userland fork/MR model.

Same `ZEXBUILD` format and CI mechanics as [`userland-recipes`](https://github.com/Zainium-Dynamics/userland-recipes) — see that repo's `README.md` for the full recipe format spec. The differences here:

- `manifest.toml`'s `[install]` table sets `_syshub = true` and installs under `/overlayer/syshub/...`, not `/overlayer/zexlib/union/...`.
- Self-hosting toolchain packages (e.g. a native `gcc-musl-cross` build already correctly linked for the target) set `native = true` in `manifest.toml`'s `[package]` table so `substrate pack` skips its interpreter/RPATH patching pass.
- Curated by the core team, not open fork/PR from the public — this repo isn't the "anyone can add a package" surface `userland-recipes` is.
- A `-dev` subpackage of a syshub package (e.g. `openssl-dev` off `openssl`) still goes to **userland**, not syshub — give it its own `<sub>.manifest.toml` with `_syshub = false` and userland install paths (`/overlayer/zexlib/union/...`), same shape as any standalone userland-recipes package. `ci/build.sh` packs and publishes each subpackage as its own `.zex` from its own manifest, so `zex-ports publish` routes the main package and its `-dev` subpackage to two different places automatically — nothing extra to configure.

Publishing target: `core/syshub/x86_64/` (packages + `syshub.toml`), not `userland/x86_64/` — `zex-ports publish` routes there automatically based on `manifest.toml`'s `_syshub` flag, no separate tool or command. The syshub ledger's own shape is different from the userland one (`[core_info]` not `[ledger_info]`, no description/license/maintainer/tags — see `zex-ports`' `SyshubLedger`/`CoreInfo`/`SyshubEntry` types) and its `build_version` (e.g. `2026v0.1`) auto-increments on every publish — minor rolls into major at 10 (`2026v0.9` → `2026v1`, not semver).

## Real syshub layout (`/overlayer/syshub/`, confirmed against the reference zairoot)

Most packages are the standard `bin/sbin/lib/libexec/include/share/etc/var` set under `_syshub = true` — same install-map convention as any userland recipe, just pointed at `/overlayer/syshub/...` instead of `/overlayer/zexlib/union/...`. `lib64` is a symlink to `lib`, not a real directory. A few package families need non-standard `[install]` keys instead (substrate's install map is a flat, arbitrary `key = dest` table — any key name works, not just the standard ones):

| Content | Real path | `[install]` key example |
|---|---|---|
| musl cross toolchain itself (`musl`, `gcc-musl-cross`, `binutils`) | `/overlayer/syshub/x86_64-zainium-linux-musl/{bin,include,lib,share}` | `bin = ".../x86_64-zainium-linux-musl/bin"`, etc. |
| Firmware blobs | `/overlayer/syshub/drivers/hardware/firmwares/` | `firmwares = "/overlayer/syshub/drivers/hardware/firmwares"` |
| Kernel modules | `/overlayer/syshub/drivers/modules/<kernel-version>/` (standard `kernel/{arch,crypto,drivers,fs,kernel,lib,mm,net,security,sound,virt}` tree + `modules.*`) | `modules = "/overlayer/syshub/drivers/modules/<kernel-version>"` |
| Service definitions (`syshub.toml` itself, `dbus.toml`, `zex_services.toml`) | `/overlayer/syshub/engine/services/` | `services = "/overlayer/syshub/engine/services"` |
| Quantra (PID 1) | `/overlayer/syshub/engine/` | — |

Driver/firmware packages typically carry patches too, same flat-file-next-to-`ZEXBUILD` convention as `userland-recipes` (`*.patch`, applied explicitly inside `ZEXBUILD`) — just staged into these non-standard destinations instead of the usual `bin/lib/share`.

## CI

Runs on GitHub Actions (`.github/workflows/ci.yml`), branch `master` — direct port of `userland-recipes`' pipeline (Alpine container for real musl builds, `ci/changed-packages.sh` + `ci/build.sh` unchanged from that repo verbatim). `check` on every PR (build-only), `release` on every push to `master` (build, verify musl/interpreter, pack, publish).

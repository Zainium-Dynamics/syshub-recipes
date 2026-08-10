# syshub-recipes

Recipes for Zainium OS's syshub (base OS) packages — the immutable core, not the userland fork/MR model.

Same `ZEXBUILD` format and CI mechanics as [`userland-recipes`](https://github.com/Zainium-Dynamics/userland-recipes) — see that repo's `README.md` for the full recipe format spec. The differences here:

- `manifest.toml`'s `[install]` table sets `_syshub = true` and installs under `/overlayer/syshub/...`, not `/overlayer/zexlib/union/...`.
- Self-hosting toolchain packages (e.g. a native `gcc-musl-cross` build already correctly linked for the target) set `native = true` in `manifest.toml`'s `[package]` table so `substrate pack` skips its interpreter/RPATH patching pass.
- Curated by the core team, not open fork/PR from the public — this repo isn't the "anyone can add a package" surface `userland-recipes` is.

Publishing target: `core/syshub/x86_64/` (packages + `syshub.toml`), not `userland/x86_64/`.

Scaffolding only for now — CI wiring follows the same shape as `userland-recipes`' `.github/workflows/ci.yml`/`ci/build.sh` once needed.

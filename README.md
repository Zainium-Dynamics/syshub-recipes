# syshub-recipes

Recipes for Zainium OS's syshub (base OS) packages — the immutable core, not the userland fork/MR model.

Same `ZEXBUILD` format and CI mechanics as [`userland-recipes`](https://github.com/Zainium-Dynamics/userland-recipes) — see that repo's `README.md` for the full recipe format spec. The differences here:

- `manifest.toml`'s `[install]` table sets `_syshub = true` and installs under `/overlayer/syshub/...`, not `/overlayer/zexlib/union/...`.
- Self-hosting toolchain packages (e.g. a native `gcc-musl-cross` build already correctly linked for the target) set `native = true` in `manifest.toml`'s `[package]` table so `substrate pack` skips its interpreter/RPATH patching pass.
- Curated by the core team, not open fork/PR from the public — this repo isn't the "anyone can add a package" surface `userland-recipes` is.

Publishing target: `core/syshub/x86_64/` (packages + `syshub.toml`), not `userland/x86_64/` — `zex-ports publish` routes there automatically based on `manifest.toml`'s `_syshub` flag, no separate tool or command.

## CI

Runs on GitHub Actions (`.github/workflows/ci.yml`), branch `master` — direct port of `userland-recipes`' pipeline (Alpine container for real musl builds, `ci/changed-packages.sh` + `ci/build.sh` unchanged from that repo verbatim). `check` on every PR (build-only), `release` on every push to `master` (build, verify musl/interpreter, pack, publish).

### Repository secrets / variables (repo settings → Secrets and variables → Actions)

Same as `userland-recipes`:

| Name | Kind | Meaning |
|---|---|---|
| `R2_ENDPOINT` | secret | `https://<account_id>.r2.cloudflarestorage.com` |
| `R2_BUCKET` | secret | The R2 bucket packages publish to. |
| `R2_ACCESS_KEY_ID` / `R2_SECRET_ACCESS_KEY` | secret | R2 API token (S3-compatible credentials) — publish-only scope. |
| `REQUIRES_SYSHUB` | variable | Passed straight to `substrate pack --requires-syshub`. |

These need to be set on **this** repo separately — GitHub Actions secrets don't carry over from `userland-recipes`.

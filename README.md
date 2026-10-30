Zainium OS base recipes
=======================

These are the recipes for syshub, the base of Zainium OS: the C library,
the toolchain, systemd, the desktop and everything else the system ships.
Each directory is one package: a ZEXBUILD build script, a manifest.toml
and any patches it needs.

The core team maintains this repository. Software you would like users to
be able to install belongs in userland-recipes, which also explains the
ZEXBUILD and manifest.toml formats in full:

    https://github.com/Zainium-Dynamics/userland-recipes

Where packages install
----------------------

Base packages install under /overlayer/syshub (bin, lib, include, share,
etc). A few have places of their own:

    musl and the cross toolchain   /overlayer/syshub/x86_64-zainium-linux-musl
    firmware                       /overlayer/syshub/drivers/hardware/firmwares
    package lists and mirrors      /overlayer/syshub/engine/services

The kernel and its modules come from kernel-recipes.

A base package's manifest.toml sets _syshub = true. A -dev subpackage of a
base package goes to the userland: it has its own <name>-dev.manifest.toml
with _syshub = false. Toolchain packages that are already linked for the
target set native = true, so packing leaves their loader and library path
as they are.

Building
--------

GitHub Actions builds every changed package in an Alpine container. A pull
request is only built. A push to master is built, checked, packed with
substrate and published to the base archive; installed systems get it with
"elevate zex update".

License
-------

The recipes are GPL-3.0-only (see LICENSE). The software they build keeps
its own license.

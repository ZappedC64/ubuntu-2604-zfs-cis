# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repository is

Not a software project — it's documentation plus a working Ubuntu Server 26.04
`autoinstall` config (cloud-init NoCloud datasource) for an unattended install
with ZFS root, extra ZFS datasets, and quotas. There is no build system,
linter, or test suite; the only "build" step is packaging the seed ISO below.

## Files

- `README.md` — the full writeup: rationale, VM setup, troubleshooting notes,
  and a copy of the `user-data`/`meta-data` config as documentation.
- `user-data` — the actual autoinstall config used to build the seed ISO.
  Currently has real values filled in (`identity.username: rwurttem`,
  `late-commands` chsh target `rwurttem`) but the `identity.password` hash is
  still the placeholder `$6$CYOURPASSWORD_HASH` — must be replaced with a real
  `openssl passwd -6` hash before the ISO is built and used.
- `meta-data` — minimal NoCloud metadata, required by the datasource format
  even though it's nearly empty.

When editing the autoinstall logic, **keep `user-data` and the config block
embedded in `README.md` in sync** — the README's copy is meant to double as
documentation and a sanitized template (`youruser`/`REPLACE_WITH_YOUR_HASH`),
while `user-data` is the real, ready-to-build file.

## Building the seed ISO

```bash
genisoimage -output seed.iso -volid cidata -joliet -rock user-data meta-data
```

The volume label must be exactly `cidata` — that's what the NoCloud datasource
scans for at boot.

## Key architecture / non-obvious facts

- Ubuntu's guided ZFS autoinstall layout (`storage.layout.name: zfs`) is the
  only ZFS root layout `autoinstall` exposes; it's not available in the
  interactive Server installer UI at all. It creates `bpool` (`/boot`),
  `rpool/ROOT/ubuntu_<id>` (`/`), `rpool/home` (`/home`), plus a swap
  partition and zsys-style children of `var` — but **not** `var/tmp`,
  `var/log/audit`, or `tmp`.
- The `late-commands` block in `user-data` creates those three missing
  datasets and sets quotas (`var`=8G, `tmp`=6G) via a `curtin in-target --
  bash -c '...'` script. That script is the load-bearing piece of this repo —
  see the "Lessons learned" section of `README.md` before modifying it,
  especially:
  - Find the root dataset with `zfs list -H -o name -t filesystem | grep -E
    "^rpool/ROOT/[^/]+$"` — `zpool get bootfs rpool` returns `-` and is not
    reliable here.
  - `zfs set mountpoint=<path> <dataset>` mounts as a side effect; a
    subsequent explicit `zfs mount` fails under `set -e`.
  - The single-quoted `bash -c '...'` block can't contain nested single
    quotes — use double quotes inside it (e.g. in `grep -E` patterns).
  - Network config must match interfaces by pattern (`match: name: "en*"`),
    not a hardcoded name like `eth0`, since VirtIO/modern kernel interface
    names vary.

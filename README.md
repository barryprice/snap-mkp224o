# mkp224o (snap)

A Snap package of [mkp224o](https://github.com/cathugger/mkp224o) — a vanity address
generator for Tor v3 (ed25519) onion services, available from the Snap Store here:

https://snapcraft.io/mkp224o

> **Community / unofficial.** This is an unofficial, community-maintained snap.

## What's in the snap

A single binary, `bin/mkp224o`, exposed as the `mkp224o` command.

It is built from a [pinned upstream release](https://github.com/cathugger/mkp224o/releases)
(currently `v1.7.0`). The pinned tag lives in `snap/snapcraft.yaml`; Renovate bumps it via a
pull request whenever cathugger/mkp224o publishes a new release, so upgrades are reviewed
rather than picked up silently on the next build.

### Build options

| Architecture | ed25519 implementation |
| ------------ | ---------------------- |
| amd64        | `amd64-51-30k` (SUPERCOP hand-written assembly) |
| everything else | `donna` (portable C) |

Additionally:

- `--enable-intfilter` is on, per upstream's `OPTIMISATION.txt`. Filtering uses 64-bit
  integers instead of binary strings, which is faster but **limits filters to 12
  characters**. A 12-character prefix needs on the order of 1.2e18 candidate keys, so this
  is not a practical restriction.
- `CFLAGS` is pinned to `-O3 -fomit-frame-pointer`. Upstream's `configure` otherwise adds
  `-march=native`, which would compile the published snap for whichever CPU the build farm
  happened to use.
- The binary is linked with `-Wl,-z,noexecstack`. Upstream's assembly files carry no
  `.note.GNU-stack` section, so without this the stack would be marked executable. (Earlier
  core22 builds of this snap used the `execstack` tool for the same purpose; it was dropped
  from the Ubuntu archive after 24.04, and a linker flag is the correct replacement — it also
  works on arm64 and riscv64, which `execstack` never supported.)

Every build runs upstream's own test vectors (`test_base16`, `test_base32`, `test_base64`,
`test_ed25519`) against the selected ed25519 implementation, then generates one throwaway
key end-to-end, before the snap is packed.

## Installing

```sh
snap install mkp224o
```

From the `edge` track:

```sh
snap install mkp224o --edge
```

## Usage

```sh
mkp224o -d mykeys neko
```

This writes one directory per discovered address (`hostname`,
`hs_ed25519_public_key`, `hs_ed25519_secret_key`) under `mykeys/`. With no `-d`, keys land
in the current directory. Run `mkp224o -h` for all options.

### Interfaces / filesystem access

The snap runs under **strict confinement** and declares:

- `home` — read/write access to `$HOME`. **Auto-connected** on install; this is what lets
  mkp224o write key directories under your home directory.
- `removable-media` — access to `/media` and `/mnt`. **Not** auto-connected:

  ```sh
  sudo snap connect mkp224o:removable-media
  ```

> **Note on paths.** Strict confinement limits access to paths outside `$HOME`, the snap's
> own mount points, and `/media`/`/mnt`. Run mkp224o from — or point `-d` at — a directory
> under one of those, or key generation will fail to write its output.

## Building locally

Requires `snapcraft` (install with `sudo snap install snapcraft --classic`):

```sh
snapcraft
```

This builds `mkp224o_<version>_<arch>.snap` in the project root. You can also run a local
lint:

```sh
snapcraft lint
```

## Continuous integration

[`.github/workflows/snap.yml`](.github/workflows/snap.yml) is CI-only:

- Builds the snap on every push / pull request.
- Lints the built snap with `snapcraft lint` in the same job (a hard gate: lint failures
  fail the build).

This workflow is a **merge gate only** — it never publishes. Building **and** publishing to
`latest/edge` is handled by snapcraft.io, which schedules its own builds of this snap and
pushes them to the `edge` track automatically. Promoting to a `stable` track remains a
manual decision (via the Snap Store dashboard). This repo is intentionally never tagged —
the upstream `mkp224o` repo is where the release tags live.

## License

The packaged software (mkp224o) is dedicated to the public domain under CC0 1.0, as
recorded in `snap/snapcraft.yaml` and reproduced in [`LICENSE`](LICENSE).

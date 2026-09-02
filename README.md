# hermes-setup

[![release](https://img.shields.io/github/v/release/MakFly/hermes-setup-releases?label=release&color=0b7285)](https://github.com/MakFly/hermes-setup-releases/releases/latest)
[![downloads](https://img.shields.io/github/downloads/MakFly/hermes-setup-releases/total?label=downloads&color=495057)](https://github.com/MakFly/hermes-setup-releases/releases)
[![platforms](https://img.shields.io/badge/linux-amd64%20%7C%20arm64-1864ab)](#what-a-release-contains)
[![Debian](https://img.shields.io/badge/Debian-13-a80030)](#platforms)
[![signature](https://img.shields.io/badge/releases-minisign%20signed-2b8a3e)](#verify-it-yourself)
[![checks](https://img.shields.io/badge/CI-gofmt%20%C2%B7%20vet%20%C2%B7%20staticcheck%20%C2%B7%20govulncheck-5f3dc4)](#how-these-binaries-are-built)
[![license](https://img.shields.io/badge/license-proprietary-868e96)](#license)

**One binary you run on your own server. It installs a pinned
[Hermes](https://github.com/NousResearch/hermes-agent) runtime, hardens the
host, and links it to OHB without opening a single inbound port.**

This repository holds published artefacts only: the static binaries, their
`SHA256SUMS`, the minisign signature of that file, and the installer. The
source code lives elsewhere, privately.

## Install

The installer checks the signature before writing anything, so `minisign` is
required:

```bash
sudo apt install minisign
curl -fsSL https://github.com/MakFly/hermes-setup-releases/releases/latest/download/install.sh | sudo sh
```

```text
signature: valid
hermes-setup_linux_amd64: OK
sha256: e63c6eedbed6a76df1d308cbc2707ee9187f1b249f0281a0949a7745d9d33c87
hermes-setup v0.1.0
installed /usr/local/sbin/hermes-setup, next: sudo hermes-setup detect
```

| Variable | Default | What it does |
|---|---|---|
| `HERMES_SETUP_VERSION` | `latest` | Install a specific version (`v0.1.0`). |
| `HERMES_SETUP_DEST` | `/usr/local/sbin/hermes-setup` | Where the binary lands. |
| `HERMES_SETUP_PUBKEY` | the key below | Verify against a different release key. |

## Verify it yourself

Nothing forces you to trust the installer. The same verification is five
lines:

```bash
base=https://github.com/MakFly/hermes-setup-releases/releases/latest/download
curl -fsSLO "$base/hermes-setup_linux_amd64"
curl -fsSLO "$base/SHA256SUMS"
curl -fsSLO "$base/SHA256SUMS.minisig"
minisign -Vm SHA256SUMS -P 'RWTz4o0G4nQll5Zew5U4RiENKKaF2u0OBQ4sX02PI9XMq/WIg42fg3G3'
grep ' hermes-setup_linux_amd64$' SHA256SUMS | sha256sum -c -
sudo install -m 0755 hermes-setup_linux_amd64 /usr/local/sbin/hermes-setup
```

The release public key, the very one embedded in every binary:

```text
untrusted comment: minisign public key 972574E2068DE2F3
RWTz4o0G4nQll5Zew5U4RiENKKaF2u0OBQ4sX02PI9XMq/WIg42fg3G3
```

The private half is offline and has never touched a build server.

An installed binary checks itself, no root needed:

```bash
$ hermes-setup self-verify
binary   /usr/local/sbin/hermes-setup
sha256   e63c6eedbed6a76df1d308cbc2707ee9187f1b249f0281a0949a7745d9d33c87
release  listed in SHA256SUMS, signature valid (embedded key)
```

## First steps

```bash
sudo hermes-setup detect     # host facts, recommended mode, preflight
sudo hermes-setup plan       # what "up" would do, writing nothing
sudo hermes-setup up --mode connector --admin-key /root/admin.pub --console-confirmed --yes
```

> [!IMPORTANT]
> `up` hardens the host under a **600 second rollback**. From a *second*
> terminal, before that window closes: `ssh hermes-admin@<ip>`, then
> `hermes-setup verify-access`. Without that proof of access everything is
> reverted automatically. Never close your current session before it passes.

`connector` is the default mode: the host opens an outbound connection, no
inbound port is exposed, and the token never leaves the machine.
`hermes-setup help` lists every verb; `audit --strict` judges the posture;
`reset --yes` removes everything.

## Updating

```bash
hermes-setup upgrade --yes
```

Downloads the latest release, verifies the signature and the digest against
the embedded key, replaces the binary, restarts the units. A binary signed by
any other key is refused.

## What a release contains

| File | |
|---|---|
| `hermes-setup_linux_amd64` | static binary, no dependency |
| `hermes-setup_linux_arm64` | same, 64 bit ARM |
| `SHA256SUMS` | digests of both binaries |
| `SHA256SUMS.minisig` | minisign signature of `SHA256SUMS` |
| `install.sh` | the installer above, verification included |

## Platforms

Debian 13, amd64 or arm64. The tool refuses a host it cannot handle rather
than leaving it half configured: run `detect` first, it says so before
anything is written.

## How these binaries are built

Every release passes `gofmt`, `go vet`, `staticcheck` and `govulncheck` in
strict mode, where one known reachable vulnerability in the standard library
is enough to block publication. It is then cross built for both architectures
and signed offline. The checks badge is static: the pipeline lives in the
source repository, which is private.

## License

Binaries are distributed as is, all rights reserved. The source code is not
public.

Something broken, or a question: [open an issue](https://github.com/MakFly/hermes-setup-releases/issues).

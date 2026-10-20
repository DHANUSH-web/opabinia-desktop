# Contributing to Opabinia

Thanks for your contributions. This repo is the **macOS-only** native AppKit application developed with Objective-C++. Scanning, safety, and trashing live in **[opalib](https://github.com/DHANUSH-web/opalib)** (`extras/opalib`).

Please read [CODE_OF_CONDUCT.md](CODE_OF_CONDUCT.md) and [SECURITY.md](SECURITY.md). Engine-only changes belong in opalib, not here.

## Branches

| Branch | Role |
|--------|------|
| `beta` | Default for PRs and new work |
| `main` | Stable releases — do not open feature PRs against `main` |

The engine lives at `extras/opalib` (submodule of `git@github.com:DHANUSH-web/opalib.git`, **`main`** branch). The engine repo is private; clone and `make init` use SSH. Contributors do not clone opalib as a second repo.

After an engine release is on **opalib `main`**:

```bash
make init    # fetch origin/main into extras/opalib
```

## Setup

macOS 13+, CMake 3.21+, Ninja, Apple Clang, Xcode Command Line Tools.

```bash
git clone --recurse-submodules https://github.com/DHANUSH-web/opabinia-desktop.git
cd opabinia-desktop
git checkout beta
make init                 # if the submodule is missing
make build release
make test release
make run release
```

If you cloned without `--recurse-submodules`, `make init` clones `extras/opalib` from `git@github.com:DHANUSH-web/opalib.git` (`main`).

## Pull requests

1. Fork and branch from **`beta`**.
2. One feature or one fix per PR.
3. Run `make test release`.
4. UI changes: screenshot or a short note of what you clicked.
5. Open the PR **against `beta`**.

Commit subjects: `FEAT: …` or `FIX: …`.

## Rules

- Native AppKit / Human Interface Guidelines. No custom dark palettes or painted chrome.
- Do not use `sidebarWithViewController:` (Tahoe liquid-glass sidebar).
- Do not reimplement scan/trash logic in `.mm` files — call `opalib`.
- Deletes stay opt-in, with Cancel as the default confirm. Report bytes freed.
- Headers in `include/`, implementations in `src/`, third-party and the engine in `extras/`.

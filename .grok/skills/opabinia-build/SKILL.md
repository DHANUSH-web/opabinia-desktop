---
name: opabinia-build
description: >
  Configure, build, test, and run the AppKit Opabinia macOS app with LLVM Clang,
  Ninja, and the extras/opalib submodule. Use when the user says build, test,
  relaunch, make init, ctest, or runs /opabinia-build.
---

# Build Opabinia

Run these commands from the repo root (`opabinia-mac`, the directory that contains `CMakeLists.txt`).

## Steps

1. If `extras/opalib/CMakeLists.txt` is missing, run `make init`. That clones `git@github.com:DHANUSH-web/opalib.git` and checks out engine `main`. Do not use a recursive submodule update.
2. Debug build and test: `make` then `make test`. Release: `make build release` then `make test release`.
3. One test: `ctest --preset debug -R <gtest-name> --output-on-failure` after the preset is configured.
4. Visible UI change: `make relaunch` (rebuild, quit `Opabinia`, open `build/debug/Opabinia.app`). Release app path is `build/release/Opabinia.app`.

The app target is `Opabinia`. Tests are `opabinia-tests`. The engine library target is `dcmm`. Toolchain file is `cmake/llvm-clang.cmake`.

Do not commit `build/` or `compile_commands.json`.

# Opabinia desktop — agent instructions

This file is the source of truth for agents working in **this** repo. The SwiftUI app under `../Opabinia` is a different project. Do not apply these instructions there, and do not port SwiftUI code into this tree.

## Priority

1. Correct behavior on the existing APIs.
2. Performance and efficiency. A scan, hash, directory walk, or process wait never runs on the main thread.
3. The smallest change that uses code already in the tree.

If an existing function is slower than it should be, fix that function. Do not add a second walker, hasher, or catalog beside it.

## Reuse before you write

Search in this order. Stop when you find a fit.

1. `include/` and `src/` — screens, widgets, settings, scan glue.
2. `extras/opalib/include/dcmm/` — the engine the app links.
3. `extras/opalib/src/mac/` then `src/common/` — only if you must change engine behavior.
4. `tests/` — a test may already describe the behavior.

Search commands that are enough:

```bash
rg -n "symbol_or_phrase" include src extras/opalib/include tests
```

Rules:

- If `dcmm::Engine` or a `dcmm::` free function already does it, call that. Do not reimplement it in a `.mm` file.
- If a `DC*` widget in `include/ui/Theme.h` already draws it, use that widget. Do not invent a new button, table, or empty state.
- If a catalog entry already covers a path, extend that entry. Do not add a second scan of the same directory.
- Mac-only engine helpers stay in `extras/opalib/include/dcmm/mac/` and `extras/opalib/src/mac/`. Do not put them in `common/` or behind a cross-platform `#ifdef` in the app.
- The sibling checkout `../opalib` may use namespace `opa` and different headers. **This app compiles `extras/opalib`, namespace `dcmm`, include path `dcmm/`.** Do not include `opa/` from the desktop target.

A new API is allowed only when the search shows nothing that can do the job, or the existing one cannot meet the behavior without a breaking change. Say which search you ran and what you found before adding the symbol.

## What lives where

```
include/                 app headers (Modules.h, AppFeatures.hpp, AppSettings.hpp, ui/)
src/main.mm              process entry
src/AppDelegate.mm       launch, menus, about
src/SystemInfo.cpp       Mac model / host facts for My Mac
src/ui/Widgets.mm        DC* controls (implementation of include/ui/Theme.h)
src/ui/*View.mm          one screen each
extras/opalib            engine submodule, CMake target dcmm
tests/                   GoogleTest for the app's glue (links dcmm)
resources/               Info.plist.in, Icon Composer icon, DMG art
```

Screen map:

| Sidebar title | `ui::Module` | View |
|---|---|---|
| My Mac | `MyMac` | `DashboardView` |
| Smart Scan | `SmartScan` | `ResultsView` via `ui::runScan` |
| Deep Scan | `DeepScan` | `ResultsView` via `ui::runScan` |
| Large Files | `LargeFiles` | `LargeFilesView` |
| Duplicates | `Duplicates` | `DuplicatesView` |
| Uninstaller | `Uninstaller` | `UninstallerView` |
| Privacy | `Privacy` | `ResultsView` via `ui::runScan` |
| Aperture | `SpaceLens` | `SpaceLensView` |
| Dev Corner | `DevCorner` | `DevCornerView` |
| Maintenance | `Maintenance` | `MaintenanceView` |
| Settings | `Settings` | `SettingsView` |

`include/Modules.h` owns order, titles, and subtitles. `include/AppFeatures.hpp` owns `runScan` and Deep Scan's post-pass `regroupNativeSystemItems`.

Window: width 760, default height 700, minimum height 520, not resizable horizontally (`MainWindowController.mm`). Do not change that unless the user asks.

## Engine API

Header: `extras/opalib/include/dcmm/common/engine.hpp`. Types: `extras/opalib/include/dcmm/common/types.hpp`. One `dcmm::Engine` per screen. The same instance serializes its work on a recursive mutex. `cancel()` does not take that lock. Do not call two scans on one instance at the same time and expect overlap.

| Need | Call |
|---|---|
| Recommended clean | `scan_smart` |
| Item-by-item junk | `deep_scan` |
| Browser traces | `scan_privacy` |
| Move paths to Trash | `trash_paths` |
| Trash the last scan's selected rows | `trash_selected` |
| Large files | `find_large_files` |
| SHA-256 duplicates | `find_duplicates` |
| Directory sizes | `space_lens`, `space_lens_children` |
| Installed apps and leftovers | `list_apps`, `attach_left_overs` |
| VS Code / Cursor / Antigravity / Codex / Rust | the matching `list_*` / `uninstall_*` |
| Disk and memory | `disk`, `memory` |
| Maintenance tasks | `maintenance_tasks`, `preview_maintenance`, `run_maintenance` |
| Stop an in-flight scan | `cancel` |

Catalogs (`smart_catalog`, `junk_catalog`, `privacy_catalog`) are the path lists. On Mac they are implemented in `extras/opalib/src/mac/catalog.cpp`. Deep Scan uses `junk_catalog`. Each existing directory is one group; each child is one row; rows start unchecked. `runScan` then moves any hit whose name contains `com.apple` into a **Native System Items** group (`reviewFirst`, unchecked). That group is not a filesystem root.

Protected paths and "is this safe to trash" belong in `extras/opalib/src/mac/safety.cpp` and `src/common`. Do not re-check safety only in the UI.

Helio (installed app path, site URL) is Mac-only: `dcmm/mac/helio.hpp`, `helio_installed`, `helio_app_path`, `helio_site`.

C ABI is `extras/opalib/include/dcmm/common/dcmm.h` and `src/common/c_abi.cpp`. The app calls C++ directly. Do not add a C wrapper for a call that `Engine` already exposes.

## UI API

`include/ui/Theme.h` and `src/ui/Widgets.mm` are the control kit. Use them.

- Labels: `DCLabel`, `DCSecondaryLabel`, `DCCaptionLabel`, `DCTitleLabel`, `DCLabelSeparator`
- Buttons: `DCPushButton`, `DCDefaultButton`, `DCDestructiveButton`, `DCSetButtonPalette`, `DCGlowButton`
- Palette: `DCPalette` plus `DCPaletteFill` / `DCPaletteTitle`. Secondary washes are the palette, not a new hardcoded alpha.
- Tables: `DCStyleTable`, `DCWrapTable`, `DCCenteredTextCell`, `DCCenteredCheckCell`, `DCCenteredFillCell`, `DCCenteredIconTextCell`
- Empty and info states: `DCStartScreen`, `DCLegendView`
- Strings from the engine: `DCNS`
- Appearance: `DCSetAppearancePref` in `Widgets.mm`. System sets `NSApp.appearance = nil`. Light sets Aqua. Dark sets Dark Aqua. Do not add a second theme path or a SwiftUI `preferredColorScheme`.

Sidebar order and symbols come from `AppFeatures.hpp` / `Modules.h`. A new module needs an enum value, title, subtitle, sidebar row, a page registered in `MainWindowController`, and a test in `tests/test_modules.cpp` and `tests/test_navigation.cpp`.

## Threading

Engine work runs off the main queue at user-initiated QoS, then UI updates hop back:

```objc
dispatch_async(dispatch_get_global_queue(QOS_CLASS_USER_INITIATED, 0), ^{
  // dcmm::Engine calls only
  dispatch_async(dispatch_get_main_queue(), ^{
    // AppKit only
  });
});
```

Do not `wait` on that work from the main thread. Progress callbacks that touch views must hop to the main queue first. Copy paths and plain data into the background block. Do not touch `NSView` there.

## Build and test

LLVM Clang and Ninja. Deployment target is 13.0. The engine submodule is required.

```bash
make init          # extras/opalib from git@github.com:DHANUSH-web/opalib.git, branch main
make               # debug preset, build/debug/Opabinia.app
make test          # ctest for that preset
make relaunch      # rebuild, quit Opabinia, open the app
make build release
make test release
```

`make init` checks out **engine** `main`. Do not point the desktop submodule at a random engine commit unless the user asks. CI must clone the submodule with the SSH deploy key. Do not use a recursive submodule checkout. That rewrite turns `git@` into HTTPS and the private clone fails.

Presets live in `CMakePresets.json`. The app target is `Opabinia`. The test target is `opabinia-tests`. The engine target is `dcmm`.

Run the tests that cover the module you touched. `ctest --preset debug -R <name> --output-on-failure` is enough when the suite is already configured.

## Feature checklist

1. Read this file and open the screen's `.mm` plus the engine header you expect to call.
2. Search as in [Reuse before you write](#reuse-before-you-write).
3. Implement the missing piece only.
4. Keep deletion as Trash through `trash_paths` / `trash_selected`. No `rm` of user files from the UI.
5. Build with `make`. Run the related gtest. `make relaunch` when the change is visible.
6. Do not reformat unrelated files. `make format` is clang-format on `include`, `src`, and `tests` when you are already touching those lines.

## Style

Match the file you are editing. Engine C++ uses Allman braces and `snake_case`. AppKit types use the `DC` prefix and Objective-C names already in that file. Include `dcmm/...` headers. Do not add a new namespace.

Comments only for a constraint the next reader cannot see from the code.

## Git

Desktop commits land on **`beta`**. Commit only when the user asks. Subject: `FEAT:` or `FIX:` in uppercase, then an imperative line.

Engine behavior changes are commits on opalib **`dev`**, then a submodule pointer update in this repo. Do not copy engine sources into `src/`.

Do not commit `build/`, `compile_commands.json`, or generated `Assets.car`.

## Out of scope unless the user asks

- Rewriting a screen in SwiftUI.
- A new scan root that duplicates `junk_catalog`, `smart_catalog`, or `privacy_catalog`.
- Custom drawing for tabs, digit transitions, or a second appearance engine.
- Pushing, tagging, or editing the release workflow.

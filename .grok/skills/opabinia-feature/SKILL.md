---
name: opabinia-feature
description: >
  Add or change a feature in the AppKit Opabinia macOS app (opabinia-desktop).
  Search existing dcmm engine APIs and DC* UI widgets before writing new ones.
  Performance first: scans and deletes stay off the main thread and go through opalib.
  Use when the user adds a screen, scan, cleaner action, setting, or button, or runs /opabinia-feature.
---

# Add a feature

Read [INSTRUCTIONS.md](../../../INSTRUCTIONS.md) at the repo root. It owns the API map, screen map, and threading rules. Follow it. Do not restate a second copy.

## Steps

1. Write down the visible behavior in one sentence.
2. Search before creating any symbol:

   ```bash
   rg -n "<phrase>" include src extras/opalib/include extras/opalib/src/mac tests
   ```

3. Call the existing `dcmm::Engine` method or `DC*` widget if the search hit can do the job. Mac-only engine helpers already live under `extras/opalib/include/dcmm/mac/`.
4. Add a function only when that search found nothing usable. Put engine code in `extras/opalib`, not in a `.mm` file. Put UI code in the existing screen's `.mm`, using `include/ui/Theme.h`.
5. Run engine work on `QOS_CLASS_USER_INITIATED`. Update AppKit on the main queue. Trash files only through `trash_paths` or `trash_selected`.
6. Build and run the tests named in INSTRUCTIONS.md for the module you touched.

The desktop target includes `dcmm/`, namespace `dcmm`, from `extras/opalib`. Do not include the sibling `opa` tree.

A new sidebar module also needs `ui::Module`, a page in `MainWindowController`, and the navigation tests. Titles stay in `include/Modules.h`.

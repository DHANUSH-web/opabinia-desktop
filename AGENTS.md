# Agent entry

This repository is **opabinia-desktop**: the macOS AppKit / Objective-C++ app. Scanning, hashing, safety, and Trash live in the **opalib** submodule at `extras/opalib`. Do not put engine logic in the app.

Before any work, read **[INSTRUCTIONS.md](INSTRUCTIONS.md)**.

Performance is the highest requirement. Search for an existing API and call it. Add code only when nothing that already exists can do the job.

Skills, in `.grok/skills/`:

- `opabinia-feature` — add or change a feature without a second implementation
- `opabinia-build` — configure, build, test, and run

Work on **`beta`**. **`main`** is stable. Engine changes go in opalib on **`dev`**, then the submodule pointer is updated here. Do not commit or push unless the user asks.

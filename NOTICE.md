# Third-Party Code & Licensing Notice

This repository packages two distinct things:

1. **`game/`** — an unmodified (for now) clone of [pmotschmann/Evolve](https://github.com/pmotschmann/Evolve),
   Copyright © Peter Motschmann and contributors, licensed under the
   **Mozilla Public License 2.0** (see `game/LICENSE`).
2. **Everything else** (`android/`, `capacitor.config.json`, root `package.json`, build scripts,
   this wrapper's own code) — the native Android/Capacitor shell written for this project.

## What MPL-2.0 requires of us

MPL-2.0 is a *file-level* copyleft license, not a whole-project one like GPL:

- We can combine the MPL-covered `game/` code with our own separately-licensed wrapper code
  (native Android/Kotlin/Java, Capacitor config, build scripts) in one "Larger Work" without
  that wrapper code being forced under MPL. Our Android glue code can stay MIT/proprietary/whatever
  we choose.
- Any file that **originates from Evolve and that we modify** stays under MPL-2.0, and we must make
  the source of *that modified file* available to anyone we distribute the app to (this repo being
  public satisfies that).
- We must keep copyright/license notices in the original files intact — don't strip headers.
- We must include a copy of the MPL-2.0 text (already present at `game/LICENSE`) and this notice
  pointing to it.
- No trademark rights are granted — we should not present the app as official or use the original
  project's name/branding in a way that implies endorsement. Pick our own app name/icon.
- The code is provided "as is", no warranty.

## Practical takeaway

- Publishing this as a free **or paid** Android app is allowed under MPL-2.0 — there is no
  commercial-use restriction.
- If we only ever *use* `game/` unmodified as a WebView payload, our obligations are just:
  keep the license file, keep notices, don't claim it's our own game, credit the original author
  (e.g. in an in-app "About" screen or this README).
- If we start patching `game/src/*.js` to add background-mode hooks, save bridging, etc., those
  patched files must remain MPL-2.0 and their source must stay available here.

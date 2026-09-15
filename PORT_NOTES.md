# Port notes (read this first)

Internal tracking doc for whoever (or whatever Claude session) picks this
port up next - not user-facing, not linked from the README on purpose.

## Sync base

This port's shared code (`app/`, `tools/`, `main.py`, `assets/`,
`requirements.txt`, `.spec` files) was copied from the Windows repo
[Jayconius/OhFudgeMyBatteryChat](https://github.com/Jayconius/OhFudgeMyBatteryChat)
at:

- **Commit**: `7e04a70` (tag `v1.2.1`)
- **Date**: 2026-09-15

**There is no git relationship between the two repos** (no fork, no shared
remote, no submodule) - this was a one-time manual copy. Syncing later means
diffing the Windows repo's current state against this commit, then
re-applying whatever's new *on top of* the Linux-specific patches below, not
overwriting files wholesale (that would silently reintroduce the bugs those
patches fixed). Update the commit/date above once a sync is done, so the
next one has an accurate anchor.

## Linux-specific patches applied on top of the shared code

Preserve all of these across any future sync - none of them exist in the
Windows repo, and blindly overwriting a file wholesale from Windows would
lose them:

- **`app/theme.py`**: `import winreg` was unconditional at module level and
  crashed on import outside Windows. Now gated behind
  `sys.platform == "win32"`; `detect_windows_dark_mode()` returns `False`
  immediately when `winreg is None`. Dark title bar (`_apply_dark_titlebar`)
  already had its own `sys.platform` guard upstream - untouched.
- **`app/paths.py`**: `app_data_dir()` branches on `sys.platform != "win32"`
  to use `$XDG_DATA_HOME` (or `~/.local/share`)`/OhFudgeMyBatteryChat`
  instead of the Windows "Data folder next to the exe" convention. Windows
  branch is byte-for-byte the original logic, untouched.
- **`app/server.py` + `app/gui.py`**: added `SoundTestState` (mirrors
  `PreviewState`) so the sound-row "Test" button plays through the browser
  overlay (`/media/_sound_test` route + `sound_test_nonce` in `/api/status`
  + `checkSoundTest()`/`playAlertSound()` client JS) instead of a local OS
  audio call. This replaced `winsound.PlaySound`/`os.startfile` entirely -
  **this change is not Windows-vs-Linux, it's a real improvement that
  removed platform-specific code from `_test_sound()`.** If the Windows
  repo hasn't picked up the same change, it's worth suggesting there too
  rather than treating it as Linux-only debt (ask the user first - don't
  touch the Windows repo unprompted).
- **`app/gui.py`**: `DEFAULT_FONT_FAMILY = "Segoe UI"` replaced with
  `_default_font_family()`, a small resolver that probes
  `tkinter.font.families()` against an ordered candidate list
  (`_FONT_FAMILY_CANDIDATES`) and caches the first match. Every direct
  `"Segoe UI"` chrome reference was switched to call this instead (search
  for `_default_font_family()` to find all call sites). The user-configurable
  overlay caption font default (`... or "Segoe UI"` in three places, for the
  *browser-rendered* caption text, not app chrome) was deliberately left
  alone - that's a per-item user choice with its own combobox, and the
  browser already degrades gracefully if the named font isn't installed.
- **`tools/fake_vr_signal_simulator.py`**: same font-resolver pattern,
  duplicated locally (not imported from `app.gui`) to keep the Simulator
  lightweight - this part IS Linux-port-specific. Its
  `from app.device_ids import STEAMVR_SERVICE_SERIAL` import (not
  `app.vr_monitor`) is **not** a Linux-only fix, though - `app/device_ids.py`
  already exists in the Windows repo too, fixing the exact same "Simulator
  exe crashes because importing vr_monitor pulls in openvr, which the
  Simulator's PyInstaller spec never bundles" bug that was caught there
  first, during the original v1.2.1 SteamVR Service work. Do not change this
  import - just don't list `device_ids.py` as Linux-only when syncing (see
  correction below).
- **`app/i18n.py`**: `"tlh"` (Klingon) removed from the `LANGUAGES` dict.
  The `STRINGS` dict still has every `"tlh"` translation - left in as inert
  data, not stripped, since `piqad.py`'s font-loading is Windows-only and
  there was no value in a larger diff to remove them. `set_language()`
  already falls back to English for any unrecognized key, so nothing
  breaks if a shared config somehow has `"language": "tlh"` in it.

## Linux-only additions (don't exist in the Windows repo at all)

**Correction (2026-09-15): `app/device_ids.py` was wrongly listed here
originally - it already exists in the Windows repo** (added there first,
during the v1.2.1 SteamVR Service feature work, for the same reason
described above). It's part of the shared code, not a Linux-specific
patch - don't skip it or treat it as something to remove/replace when
syncing from Windows.

- `.github/workflows/build.yml` - builds on `ubuntu-22.04`, smoke-tests both
  binaries under Xvfb (launches them, confirms still running after 5s, not
  just that they compiled), packages two AppImages + one combined tarball,
  publishes a prerelease on `v*` tag pushes.
- `assets/app_icon.png` - generated via Pillow (same visual style as
  `default_assets.py`'s icon generator) specifically because AppImage
  tooling requires an icon file to build; there's no equivalent asset in
  the Windows repo since it's never needed one (Tk's default icon there).
- `README.md`, `PORT_NOTES.md` (this file) - both Linux-repo-specific.

## Known untested area

Nothing in `vr_monitor.py` was changed - it's written against the standard
OpenVR API with nothing Windows-specific in it, so it should work against
real SteamVR-for-Linux, but this has **not been verified against real
hardware**. If a Linux user reports OpenVR behaving differently, that's the
first place to look.

## Release state (last updated 2026-09-15)

- `v0.0.1-beta` published as a prerelease, all 3 artifacts live and
  verified (sizes sane, AppImages are valid ELF binaries, tarball contains
  both apps). Repo `About` sidebar description/topics mirror the Windows
  repo's, adapted for Linux/beta.
- First CI run passed clean on the first attempt - no build issues hit yet.

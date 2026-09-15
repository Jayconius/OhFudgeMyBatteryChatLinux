<div align="center">

# Oh Fudge, My Battery Chat - Linux

[![Version](https://img.shields.io/badge/version-v0.0.1--beta-orange)](https://github.com/Jayconius/OhFudgeMyBatteryChatLinux/releases/latest)
[![Platform](https://img.shields.io/badge/platform-Linux-FCC624)](#)
[![Requires](https://img.shields.io/badge/requires-SteamVR-orange)](#)
[![License](https://img.shields.io/badge/license-MIT-lightgrey)](LICENSE)

*The unofficial Linux port of [Oh Fudge, My Battery Chat!](https://github.com/Jayconius/OhFudgeMyBatteryChat)*

</div>

> [!IMPORTANT]
> **This is a low-priority, best-effort port, not the main project.** I
> primarily develop and use this on Windows - the
> [Windows version](https://github.com/Jayconius/OhFudgeMyBatteryChat) is
> where active development happens. This repo gets updated randomly, or on
> request, to stay in sync with it - not on any regular schedule.
>
> The Linux build is intended to be **functionally identical to Windows
> v1.2.1** and should theoretically work the same way, but **it has not
> been fully tested on real Linux hardware/SteamVR by me**. If you try it
> and something's broken, [opening an issue](https://github.com/Jayconius/OhFudgeMyBatteryChatLinux/issues)
> with what you saw (distro, desktop environment, SteamVR version, what you
> expected vs. what happened) is genuinely the most useful thing you can do
> for this port - feedback from people actually running it on Linux is what
> keeps it moving.

## What this is

Shows your headset, controllers, and trackers' battery live on stream, with
pop-in alerts when something's running low - as a normal OBS Browser
Source. Same feature set as the Windows version: Nudge groups, Overlay
Effects (including a fake "SteamVR Service" device for a disconnect/crash
alert), the Simulator companion tool for testing without SteamVR running,
dark mode, multi-language UI. See the
[main repo's README](https://github.com/Jayconius/OhFudgeMyBatteryChat#readme)
for the full feature walkthrough and screenshots - this repo only documents
what's different about the Linux build itself.

## Download

Grab the latest from [Releases](https://github.com/Jayconius/OhFudgeMyBatteryChatLinux/releases/latest).
Two ways to run it, pick whichever's easier:

- **AppImage** (`OhFudgeMyBatteryChat-x86_64.AppImage`) - download, mark it
  executable (`chmod +x` or via your file manager's Properties), double-click
  to run. No installation, nothing else to download.
  - If it refuses to launch with an error mentioning `libfuse.so.2`, your
    distro (Ubuntu 22.04+ and newer, notably) may have dropped `libfuse2`
    from the default install - `sudo apt install libfuse2` (or your
    distro's equivalent) fixes it. This is a known AppImage-wide issue, not
    specific to this app.
- **Tarball** (`OhFudgeMyBatteryChatLinux-x86_64.tar.gz`) - extract it and
  run the binaries directly. No FUSE dependency at all, useful if the
  AppImage route gives you trouble. Contains both the main app and the
  Simulator together, same as the Windows download.

Either way, run it with **SteamVR** already open, same as the Windows
version.

## What's different from Windows

- **Data folder**: instead of a portable `Data` folder next to the exe, this
  build stores its config/media at `~/.local/share/OhFudgeMyBatteryChat`
  (respecting `$XDG_DATA_HOME` if you have it set) - the normal place Linux
  apps keep their data. The main app and Simulator share this automatically
  without needing to sit in the same folder.
- **Dark mode**: the manual Light/Dark toggle in About works the same as
  Windows, but there's no OS dark-mode auto-detection or dark title bar on
  Linux - there's no single equivalent setting to read across different
  desktop environments, and window decorations are owned by your window
  manager, not this app.
- **Klingon** isn't included in this build - it depended on a Windows-only
  font-loading API.
- **Sound preview**: the "Test" button on a sound picker plays through the
  browser overlay (the same place your actual alerts play) instead of
  locally in the app - this was actually changed for both platforms, not
  Linux-specific, so it's identical either way.

## Known unknowns

This build has not been verified against a real SteamVR-for-Linux
install - it's written against the standard OpenVR API with nothing
Windows-specific in the VR-polling code, so it should work, but "should"
and "verified" aren't the same thing. If OpenVR behaves differently on
Linux in some way that needs code changes, that's a bridge to cross once
someone (hopefully) reports it - see the note at the top about opening an
issue.

## Building it yourself

```
pip install -r requirements.txt
pyinstaller OhFudgeMyBatteryChat.spec --noconfirm
pyinstaller OhFudgeMyBatteryChatSimulator.spec --noconfirm
```

Needs `python3-tk` installed via your distro's package manager first (not
something pip installs for you). The actual release builds are produced by
this repo's own [GitHub Actions workflow](.github/workflows/build.yml) on a
clean Ubuntu runner, so that's the authoritative build process if something
here goes stale.

## License

MIT, same as the [main project](https://github.com/Jayconius/OhFudgeMyBatteryChat) - see [LICENSE](LICENSE).

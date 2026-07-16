# DetectPad

DetectPad is a minimal Windows desktop notepad for jotting sensitive notes that should not
leak through screen sharing, screen recording, or screenshots. It excludes its own window from
screen capture, has a global panic hotkey to hide it instantly, and autosaves your note as you
type.

This repository distributes the DetectPad binary only. The source code is not public.

## Download

Latest release: <https://github.com/gengen1255/detectpad-releases/releases/latest>

Download `DetectPad.exe` from the latest release and run it directly. No installer and no
separate .NET runtime install is needed - the exe is self-contained.

It is a compressed single-file executable, so the first launch is slower than usual while it
unpacks itself; later launches are fast.

## System requirements

- Windows 10 or 11, x64.
- Windows 10 version 2004 (build 19041) or later is required for the screen-capture-exclusion
  feature to take effect. On older Windows versions DetectPad still runs, but the window is not
  protected from capture.

## Windows SmartScreen

The exe is unsigned, so Windows SmartScreen will likely show "Windows protected your PC" the
first time you run it. Click **More info**, then **Run anyway** to continue. This is expected
for an unsigned indie binary and is not a sign of tampering.

## Antivirus

Because DetectPad excludes its own window from screen capture, some antivirus software may
occasionally flag it as a false positive. If yours blocks the download or the exe, you may need
to allow it manually.

## Source availability

This repository holds only the published binary and its license. The DetectPad source code is
not public.

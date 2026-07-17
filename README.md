<p align="center">
  <img src="assets/DetectPad-logo-transparent.svg" width="128" alt="DetectPad logo">
</p>

<h1 align="center">DetectPad</h1>

<p align="center"><em>The notepad your screen share can’t see.</em></p>

DetectPad is a minimal Windows desktop notepad for jotting sensitive notes that should never
leak through screen sharing, screen recording, or screenshots. It is a single window with one
plain-text, word-wrapped note area, nothing else.

**Upgrading from an earlier release?** See the backup note under [Download](#download) before
you install - it takes a minute, and it does not apply if this is your first install.

This repository distributes the DetectPad binary only; the source code is not public.

## What it does

- Excludes its own window from screen capture, so it does not appear in screenshots, screen
  recordings, or live screen shares (Zoom/Teams/Meet/etc.).
- Provides a global panic hotkey, **Ctrl+Alt+H**, that instantly hides the window - even when
  DetectPad does not have keyboard focus - and restores it to its prior position, size, and
  state when pressed again. Any unsaved text is saved before the window hides.
- Autosaves your note as you type, so you never have to remember to save.
- Follows the Windows system light/dark app theme by default, with a compact toggle to
  override it; your choice is remembered across restarts.
- Encrypts your note at rest automatically, with no setup required: the encryption key is
  protected by your Windows account by default, and you can optionally set a passphrase instead,
  for protection that no longer depends on your Windows login. See "Privacy model - read this"
  below for exactly what this does and does not protect against.
- Shows a slim status bar with privacy-mode status, autosave state, and the currently
  registered panic hotkey.

## Privacy model - read this

DetectPad has three independent protection layers. Read what each one actually protects against,
and what it explicitly does not, before you trust it with anything sensitive.

### 1. Capture exclusion and panic hide

**What it protects against:** the note window appearing in screenshots, screen recordings, or
live screen shares (Zoom/Teams/Meet/etc.). The panic hotkey (**Ctrl+Alt+H** by default)
additionally hides the window instantly for shoulder-surfing situations.

**What it does NOT protect against:** anyone physically looking at your screen while the window
is visible, or a capture method that does not go through the operating system's normal capture
path (for example, a camera pointed at the physical monitor). If your version of Windows does not
support capture exclusion (see Requirements below), DetectPad shows a one-time warning and runs
unprotected for this layer only; the status bar reflects this for the rest of the session.

### 2. Encryption at rest, default mode (no passphrase)

With no setup, every save is AES-256-GCM encrypted, and the encryption key itself is protected by
Windows DPAPI. The real lock here is your Windows account password, not this particular piece of
hardware: the same key material that unwraps the note lives in your Windows profile
(`%APPDATA%`), right alongside the note itself, and none of it is sealed to any specific machine,
chip, or TPM.

What this protects against, and how strongly, depends on the account and the PC:

- **A local Windows account (no Microsoft sign-in):** your account password is the whole lock. A
  copy of your `%APPDATA%` folder - a cloud backup, a sync tool, a stolen drive image - carries
  both the note and the key material that unwraps it, so that copy plus your Windows password is
  enough to decrypt the note offline, on any machine, without an attacker ever touching this PC.
  Do not treat an `%APPDATA%` copy as a safe backup by itself; your password is what protects it.
- **Signed in with a Microsoft account:** generally somewhat stronger. Windows keeps part of the
  key material in a machine-local component outside the plain `%APPDATA%` tree, so a bare copy of
  `%APPDATA%` plus your Microsoft account sign-in password is generally not, by itself, enough to
  decrypt the note off this PC. This is a hedge, not a guarantee: it is not a claim that Microsoft
  can recover the note for you, and it is not a claim that off-PC decryption is impossible.
- **A local administrator on the same PC:** not reliably blocked. An administrator can read every
  profile on the machine regardless of ownership and, if you are currently signed in with an
  unlocked session, can act within that session to read the note as you, with no password needed.
  Without a live session, an administrator's access reduces to the same password-derived
  protection described above - so a strong Windows password still matters.
- **A work/school (domain or Microsoft Entra) IT administrator, on a managed PC:** by design, not
  blocked. Managed accounts have their key material recoverable by IT specifically so a forgotten
  password does not mean permanent data loss - which means a domain or Entra administrator can
  recover your note without ever knowing your password. This applies only to a managed PC, not to
  a personal PC signed in with a local account or a personal Microsoft account.

**What none of the above changes:** anything already running as you, in your own unlocked
Windows session, can read the note the same way DetectPad itself does, silently and with no
prompt, because this mode needs no passphrase. That includes malware or any other program running
as your user - default mode protects the note at rest, not from your own already-unlocked
session.

**Resetting vs. changing your Windows password:** an ordinary password change (you know the old
password) preserves your existing key material, and your note stays readable. Having your
password *reset* instead (without knowing the old one) breaks that key rather than replacing it -
this can permanently lock you out of a device-mode note, with no recovery.

**A separate fact that does not contradict anything above: DetectPad itself will not open the
note elsewhere.** Run normally - just the app, no special tools - DetectPad cannot open a
device-mode `note.dat` on a different PC, under a different Windows account (even on the same
PC), after a profile rebuild, or after certain account or credential changes on the original
machine. In every one of these cases the file itself is left completely intact - nothing is
deleted or overwritten - it simply cannot be opened, and DetectPad tells you so rather than
failing silently. This is a genuine data-loss risk worth planning around; it is not proof that a
copy of the file is safe from the scenarios described above.

**If you need to move a note to another machine on purpose:** switch to passphrase mode first
(below), or copy the plain text of your note by hand. Do not copy `note.dat` itself expecting it
to open elsewhere - by design, it will not.

### 3. Encryption at rest, optional mode (passphrase)

You may optionally set a passphrase from the status bar's encryption indicator. This replaces
the DPAPI key wrap with a key derived from your passphrase using PBKDF2-HMAC-SHA256, so
protection of the note no longer depends on your Windows account at all.

**What it protects against:** everything device mode protects against, plus - unlike device mode
- anything running as you under your own Windows session that does not already know your
passphrase, for a note DetectPad has not yet unlocked in the current run.

**What it does NOT protect against, and the single most important thing to understand:** a
forgotten passphrase means **permanent, total loss of the note, with no recovery, no reset, no
escrow, and no backdoor of any kind, by design**. Nobody - including the developer - can recover
a note whose passphrase is forgotten. This mode also does not protect the note once you have
unlocked it in the current session (see the disclosures below), and it does not protect against
a keylogger or similar capturing the passphrase as you type it.

**On passphrase strength:** DetectPad derives the key from your passphrase using
PBKDF2-HMAC-SHA256 (600,000 iterations, matching the OWASP-stated floor), a well-established
method, rather than a more GPU/ASIC-resistant algorithm such as Argon2id. This is a deliberate,
disclosed trade-off, made to keep DetectPad free of external runtime dependencies. As a direct
consequence, passphrase **length matters more than complexity** here: prefer a long passphrase
(a short sentence, or several unrelated words) over a short one stuffed with symbols and digits.

**Tamper detection:** the authenticated file header binds the key-derivation parameters and the
salt, so an attacker who can write to the note file cannot lower the iteration count or swap the
salt to make an offline cracking attempt cheaper. This is the only integrity property the format
provides; it is not a broader integrity, rollback, or anti-replay guarantee.

### Disclosures that apply regardless of layer or mode

- Once a note is decrypted for display, its plaintext, your typed passphrase (if you use one),
  and the session's encryption key all exist in this process's memory, unscrubbed, for as long as
  the app stays open. A crash dump, the Windows pagefile, or a hibernation file created during
  that time could contain any of them.
- DetectPad has no secure-delete anywhere. Every file deletion it performs is an ordinary
  operating-system delete, not a secure wipe. In particular, migrating an existing plain-text
  note to the encrypted format deletes the old plain-text file with a normal delete; the disk
  blocks that held that plaintext may remain recoverable from the physical medium afterward -
  this is a general property of most filesystems and storage media, not something specific to
  DetectPad, and DetectPad does not attempt to overwrite or erase those blocks.
- DetectPad does not provide a "panic wipe" or any other secure-delete feature, and does not
  provide any passphrase recovery mechanism. If either of these matters to your threat model,
  treat their absence as a hard limit of this tool.
- If a note cannot be decrypted for any reason, DetectPad enters a locked state that refuses to
  write to disk rather than risk overwriting a note it cannot recover, and it tells you rather
  than failing silently.

## Default hotkey

**Ctrl+Alt+H** toggles hide/show of the main window. This is a fixed default in v1; there is no
remapping UI. If another application has already registered this combination, DetectPad cannot
register it either, and the status bar will indicate that the hotkey is unavailable.

## Storage location

The note is saved encrypted, at a fixed path:

```
%APPDATA%\DetectPad\note.dat
```

`note.dat` is a small versioned, authenticated binary format - it is not a plain-text file, and
opening it in a text editor shows ciphertext, not your note. The directory and file are created
automatically on first save if they do not already exist. There is a single implicit document:
each save re-encrypts and overwrites the file (no versioning, no backups, no multi-note support).

If DetectPad finds an existing plain-text `%APPDATA%\DetectPad\note.txt` from a version prior to
1.5.0, and no `note.dat` yet exists, it migrates that note automatically on the next startup: the
plain-text content is encrypted into a new `note.dat`; the write is verified by reading it back
and decrypting it; and only then is the old `note.txt` deleted. This migration is **one-way** -
DetectPad has no built-in "decrypt back to plain text" or export feature. If migration is
interrupted (power loss, forced quit) at any point, you are left with either the intact original
`note.txt` or a valid, verified `note.dat` - never neither, and never a corrupted, half-written
file. If the old plain-text file is encrypted successfully but cannot be removed afterward (for
example, because another program is holding it open), DetectPad tells you this and names where
the file still is, rather than reporting plain success - your note is protected either way.

DetectPad also carries a one-time bridge that moves a note saved under the product's former name
into the current location as a plain-text file, ahead of the encryption migration described
above. In an unusual situation - an already-encrypted profile where a stale, old-named
plain-text note is reintroduced later, typically by restoring a backup or an old profile - this
bridge can write a plain-text copy of that stale note to disk. If that note is never unlocked in
the same session (the unlock prompt is cancelled, or the passphrase entered is wrong), the
plain-text copy can persist on disk until a later successful unlock removes it. This does not
affect, or expose the content of, your current already-encrypted note; it is a narrow edge case
around a stale file being reintroduced from elsewhere, disclosed here for completeness.

The selected theme mode is persisted alongside the note at `%APPDATA%\DetectPad\settings.json`.
This file holds only UI preferences such as the theme choice - it never contains your
passphrase, any derived or wrapped key, or any other secret material.

## Download

> **Upgrading from a v1.0.0 install?** Before you run the new version for the first time, copy
> your existing `%APPDATA%\DetectPad\note.txt` to a safe location **outside**
> `%APPDATA%\DetectPad` - for example, your Documents folder or removable media. On first launch,
> DetectPad converts that note into the new encrypted `note.dat` format, and your old `note.txt`
> disappears once that finishes - this is expected, not data loss, and your note text will be
> exactly as you left it. Your original file is not touched until the new encrypted copy has
> been written and verified, so the process is designed to be safe to interrupt. Do not restore
> your backup copy into `%APPDATA%\DetectPad` afterward: once DetectPad has an encrypted note, a
> `note.txt` placed there is not read as your note, and is cleaned up automatically.
>
> If this is your first time installing DetectPad, none of this applies to you.

Direct download: <https://github.com/gengen1255/detectpad-releases/releases/latest/download/DetectPad.exe>

Latest release page: <https://github.com/gengen1255/detectpad-releases/releases/latest>

No installer and no separate .NET runtime install is needed - the exe is self-contained. It is
about 62 MB, a compressed single-file executable, so the first launch is a little slower than
usual while it unpacks itself; later launches are fast.

## Requirements

- Windows 10 or 11, x64.
- Windows 10 version 2004 (build 19041) or later is required for capture exclusion to take
  effect. On older Windows versions DetectPad still runs, but the window is not protected from
  capture; encryption at rest still applies regardless of Windows version.

## Windows SmartScreen

The exe is unsigned, so Windows SmartScreen will likely show "Windows protected your PC" the
first time you run it. Click **More info**, then **Run anyway** to continue. This is expected
for an unsigned indie binary and is not a sign of tampering.

## Antivirus

Because DetectPad excludes its own window from screen capture, some antivirus software may
occasionally flag it as a false positive. If yours blocks the download or the exe, you may need
to allow it manually.

## Non-goals (v1)

No secure-delete or "panic wipe" of the note file or the disk blocks a prior plain-text note
occupied, no passphrase recovery or reset mechanism of any kind, no multi-note/tabs, no tray
icon, no find/replace, no settings UI beyond the theme toggle and the passphrase
set/change/remove flow, no high-contrast theme, no font settings, no installer/packaging (the
Releases download is a single standalone exe, not an MSI/MSIX/winget package), and no
macOS/Linux support.

<div align="center">

<img src="brand/besop-logo-512-transparent.png" width="140" alt="Bêşop">

# Bêşop

**Erases without a trace**

Overwrites a file's contents 50 times with random data, then deletes it for good,
so that recovery tools cannot bring it back.

Android · Windows · macOS · Linux
Kurmanji · Sorani · Arabic · Turkish · English

[Türkçe](README.md)

</div>

---

## Why

When you delete a file, the operating system does not destroy its contents. It
only marks the space as free, and the bytes stay on the disk. Recovery tools
(DiskDigger, Recuva, PhotoRec) look for exactly that.

Bêşop destroys the contents **before** deleting the file.

## How it works

The order matters. Each step removes the trace the previous one left:

1. The contents are overwritten **50 times with random data**
2. `fsync` after every pass, so the data is really written to disk instead of
   sitting in a cache
3. The file is truncated to **0 bytes**
4. It is **permanently deleted**, not moved to the trash
5. The deletion is **verified**; if it failed, the app says so clearly

On desktop, the file name is also randomized three times before deletion, so
the old name in the directory entry is scrambled too.

### Test evidence

A recognizable signature was written into a 30 MB file and Bêşop was run on it.
The first 16 bytes were watched from outside during the whole process:

| Stage | First bytes of the file |
|---|---|
| Before | `GIZLIVERI-CYBERG` |
| During | `9e 87 4c 14 74 8a fc f0` |
| After | `No such file or directory` |

## Honest limits

Flash storage (eMMC / UFS / SSD) uses **wear leveling**: the controller writes
new data to a different physical block instead of the old one. The copy in the
old block stays inside the chip, where software cannot reach it.

**So 50 passes are not safer than 1 pass.** The app says this too. Against
recovery software, though, it is enough: the contents really are destroyed at
the file-system level, and that is the app's goal.

If you need an absolute, lab-grade guarantee, the only way is full-disk
encryption followed by destroying the key.

> **Note:** When you delete a gallery photo, apps like Google Photos or Files may
> keep a separate copy in their own trash. Empty that from the app itself.

## Screenshots

<div align="center">
<img src="docs/screenshots/01-karanlik.png" width="240" alt="Dark theme">
<img src="docs/screenshots/02-aydinlik.png" width="240" alt="Light theme">
<img src="docs/screenshots/03-hakkinda.png" width="240" alt="About screen">
</div>

## Features

- **Three buttons**: pick a file, pick a folder, destroy. No other settings.
- **Five languages**, Kurmanji first. Right-to-left languages have a real RTL layout.
- **Light / dark / system** theme.
- **No internet permission**: even if it wanted to, the app could not send data anywhere.
- No ads, no tracking, no analytics, no accounts.
- **Disk mode** (desktop): fully formats a USB stick / SD card / external disk
  and overwrites it with 3 passes. The system disk is locked.
- Work continues in the background with progress in a notification (Android).

## Install

See the [Releases](https://github.com/cybergah8-lang/besop/releases/latest) page for ready-made builds.

| Platform | File |
|---|---|
| Android 8.0+ | `Besop-Android-*.apk` |
| Windows 10/11 | `Besop-Windows-Kurulum-*.msi` (installer) or `*-Portable-*.zip` (no install) |
| macOS | `Besop-macOS-*.dmg` |

> The macOS app is unsigned, so it may be blocked on first launch:
> right-click → **Open**, or System Settings → Privacy & Security → **Open Anyway**.

## Build from source

Requires **JDK 17**, plus the **Android SDK** (compileSdk 34) for Android.

```bash
./gradlew :app:assembleRelease          # Android APK
./gradlew :desktop:packageMsi           # Windows installer (.msi)
./gradlew :desktop:packageDmg           # macOS (.dmg), on a Mac only
./gradlew :desktop:createDistributable  # portable folder
./gradlew :desktop:run                  # run the desktop app directly
./gradlew :desktop:selftest             # verify the wipe engine
```

`jpackage` cannot cross-compile, so each platform's package is built on its own
OS. GitHub Actions builds all three.

### Signed builds

The signing key is not stored in the repository. To produce a signed APK:

```bash
cp keystore.properties.example keystore.properties
# fill in the values with your own key
```

Without `keystore.properties` the build still works and is signed with the debug key.

## Project layout

```
shared/src/    code SHARED by Android and desktop
  Strings.kt       all texts for 5 languages, single source
  Model.kt         data model, progress state
  Theme.kt         color palette, theme mode
  WipeUi.kt        the whole main screen
  WipeAnimation.kt wipe animation
  Logo.kt          logo and CyberGah stamp
  Splash.kt        splash animation
  About.kt         about screen

app/           Android only
  WipeEngine.kt    SAF-based wipe engine
  WipeService.kt   foreground service + notification
  Picker.kt        file/folder picker

desktop/       desktop only
  DesktopEngine.kt file-system wipe engine
  DiskWipe.kt      disk wipe (using OS tools)
  DiskScreen.kt    disk selection screen

brand/         logo files (SVG + PNG)
docs/          screenshots
```

The UI and translations come from a single source. To fix a language, editing
`shared/src/com/cybergah/securewipe/Strings.kt` is enough, and both platforms
pick it up.

## Contributing

The Kurmanji and Sorani translations are open for review. A fix is usually a
one-line change in `Strings.kt`: open an issue or send a PR.

## License

[MIT](LICENSE)

---

<div align="center">
<sub>C Y B E R G A H</sub>
</div>

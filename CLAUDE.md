# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Android Device Owner utility app for Sonim XP5800/XP3800 devices (Android 8.1). Enables APK/XAPK installation without constant ADB connection by leveraging Device Owner/Device Policy Manager permissions to bypass installation restrictions on enterprise devices.

## Build Commands

Build the APK (requires Termux environment with Android SDK tools):
```bash
./build.sh
```

**Required tools:** `aapt2`, `ecj`, `d8`, `zip`, `apksigner`
**Required files:** `~/.android/android.jar`, `~/.android/debug.keystore` (password: "android")

Output: `build/DeviceAdminApp.apk`

**Install and configure:**
```bash
adb install -t -r build/DeviceAdminApp.apk
adb shell dpm set-device-owner com.example.deviceownerapp/.DeviceAdmin
```

No unit tests exist - the app is tested manually on physical devices.

## Architecture

**Build System:** Custom bash script (no Gradle/Maven) designed for Termux on-device compilation.

**Key Components:**

- `MainActivity.java` - Main hub: lists installed apps, triggers file picker for installation, handles update checks via GitHub API, feedback via Formspree webhook
- `InstallActivity.java` - Handles APK/XAPK file installation; detects file type and either installs directly (APK) or extracts and streams all contained APKs (XAPK/ZIP bundles)
- `AppDetailActivity.java` - Per-app management: show/hide apps, grant/deny runtime permissions (Device Owner only)
- `ProgressActivity.java` - Installation progress UI with spinner; receives broadcasts from InstallResultReceiver
- `InstallResultReceiver.java` - BroadcastReceiver for PackageInstaller results (success, pending user action, failure)
- `DeviceAdmin.java` - DeviceAdminReceiver for Device Owner callbacks
- `SimpleFileProvider.java` - Custom ContentProvider for serving APK files (replaces AndroidX FileProvider)
- `Logger.java` - File-based error logging to app-specific external storage

**Design Decisions:**
- No external dependencies including AndroidX - direct Android SDK usage only
- Manual JSON parsing for GitHub API (no JSON library)
- AsyncTask for background operations (targets Android 8.1)
- Package name: `com.example.deviceownerapp` (hardcoded throughout)

## External Services

- GitHub API: `https://api.github.com/repos/flipphoneguy/DeviceOwnerProject/releases/latest` (update checking)
- Formspree: `https://formspree.io/mnnwyprr` (feedback submission)

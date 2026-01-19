# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Android Device Owner utility app that enables APK/XAPK installation on devices with restricted installation policies. Leverages Device Owner/Device Policy Manager permissions to bypass installation restrictions on enterprise or locked-down devices.

## Build Commands

Build the APK (requires Termux environment with Android SDK tools):
```bash
./build.sh
```

**Required tools:** `aapt2`, `ecj` (or `javac`), `d8`, `zip`, `apksigner`
**Required files:** `~/.android/android.jar`, `~/.android/debug.keystore` (password: "android")

Output: `build/DeviceAdminApp.apk`

**Install and configure (Native Device Owner mode):**
```bash
adb install -t -r build/DeviceAdminApp.apk
adb shell dpm set-device-owner com.example.deviceownerapp/.DeviceAdmin
```

**Alternative: Dhizuku mode** - Install the app normally and connect to Dhizuku (which must be set as Device Owner) through the app's UI.

No unit tests exist - the app is tested manually on physical devices.

## Architecture

**Build System:** Custom bash script (no Gradle/Maven) designed for Termux on-device compilation. External JARs in `libs/` are included in classpath and dexed.

### Dual-Mode Device Owner Support

The app supports two modes for obtaining Device Owner privileges:

1. **Native Device Owner** (`Mode.NATIVE_OWNER`) - App set as DO via `adb shell dpm set-device-owner`
2. **Dhizuku Mode** (`Mode.DHIZUKU`) - App requests DO privileges from Dhizuku at runtime

**DpmHelper.java** - Central abstraction layer that routes all DevicePolicyManager and PackageInstaller operations through the appropriate mode. Key methods:
- `getActiveMode()` - Detects current privilege mode
- `setApplicationHidden()`, `isApplicationHidden()` - App visibility control
- `getPermissionGrantState()`, `setPermissionGrantState()` - Runtime permission management
- `installApkThroughDhizuku()` - Silent APK installation via Dhizuku's wrapped binders

### Dhizuku Integration

For Dhizuku mode, the app wraps system binders through `Dhizuku.binderWrapper()` to execute operations with Device Owner privileges:
- Wraps IPackageManager → IPackageInstaller → IPackageInstallerSession
- Creates PackageInstaller sessions owned by Dhizuku's UID for silent installation
- Uses reflection to access hidden APIs (IPackageManager, IPackageInstaller, etc.)

**libs/dhizuku-api.jar** - Dhizuku API library (built from `Dhizuku-API/` submodule)
**src/androidx/core/os/BundleCompat.java** - Shim class providing AndroidX BundleCompat methods required by Dhizuku API

### Key Components

- `MainActivity.java` - Main hub: lists installed apps, status display (Native DO/Dhizuku/None), Dhizuku connect button, file picker, update checks
- `InstallActivity.java` - Routes to `DhizukuBinderInstallTask` or `NativeInstallTask` based on mode; handles APK/XAPK files
- `AppDetailActivity.java` - Per-app management: show/hide apps, grant/deny runtime permissions
- `ProgressActivity.java` - Installation progress UI; receives results from InstallResultReceiver
- `InstallResultReceiver.java` - BroadcastReceiver for PackageInstaller results
- `DeviceAdmin.java` - DeviceAdminReceiver for Device Owner callbacks
- `Logger.java` - File-based error logging to app-specific external storage

### Design Decisions

- No external dependencies except Dhizuku API - direct Android SDK usage
- Manual JSON parsing for GitHub API (no JSON library)
- AsyncTask for background operations
- Package name: `com.example.deviceownerapp` (hardcoded throughout)
- Reflection used extensively to access hidden Android APIs for binder wrapping

## External Services

- GitHub API: `https://api.github.com/repos/flipphoneguy/DeviceOwnerProject/releases/latest` (update checking)
- Formspree: `https://formspree.io/mnnwyprr` (feedback submission)

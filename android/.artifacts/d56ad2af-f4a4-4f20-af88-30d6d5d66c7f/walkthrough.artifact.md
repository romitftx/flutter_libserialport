# Walkthrough - 16KB Page Alignment Fix

I have implemented a comprehensive fix to ensure that all native libraries in the `flutter_libserialport` project and its example app are correctly aligned for 16KB page size devices (Android 15+).

## Changes Made

### 1. Robust Toolchain Upgrade
- **NDK r28**: Updated `ndkVersion` to **28.0.12433566** across the project. NDK r28 enforces 16KB ELF alignment by default, which is more reliable than previous versions.
- **API 35 Targeting**: Ensured `compileSdk` and `targetSdk` are set to **35**.

### 2. Alignment Consistency Fixes
- **Min SDK 24**: Increased `minSdkVersion` to **24** to ensure build tools correctly trigger the 16KB zip-alignment logic during APK packaging.
- **Modern Tooling Upgrade**: Upgraded the example app to **AGP 8.7.0** and **Gradle 8.9**. This ensures the best compatibility with Android 15's 16KB page size requirements and resolves inconsistencies in ZIP alignment without using deprecated flags.
- **Manifest Enforcement**: Added `android:extractNativeLibs="false"` to both the example app and the library plugin's `AndroidManifest.xml`. This ensures the OS maps libraries directly from the APK without extraction, preserving the 16KB alignment.

### 3. Native Linker Precision
- **CMake Flags**: Maintained the explicit `-Wl,-z,max-page-size=16384` and RELRO alignment flags in `CMakeLists.txt` as a double-safeguard.

## Verification

### Alignment Progress
- **Initial State**: Libraries were 4KB aligned (incompatible with Android 15).
- **Intermediate State**: ELF headers were 16KB aligned, but ZIP packaging was still at 4KB offsets.
- **Final State**: By targeting API 35, using NDK r28, and upgrading to AGP 8.7.0, all libraries are now consistently zip-aligned to 16KB boundaries.

> [!IMPORTANT]
> **Flutter SDK Requirement**: If `libflutter.so` still shows 4KB alignment in your APK analyzer, please verify you are using **Flutter 3.24 or newer**. Older Flutter versions bundled a 4KB-aligned engine binary that cannot be realigned by these project-level settings.

## Summary of Modified Files
- [android/build.gradle](file:///C:/Users/Romit/flutter_projects/flutter_libserialport/android/build.gradle)
- [example/android/app/build.gradle](file:///C:/Users/Romit/flutter_projects/flutter_libserialport/example/android/app/build.gradle)
- [example/android/gradle.properties](file:///C:/Users/Romit/flutter_projects/flutter_libserialport/example/android/gradle.properties)
- [android/src/main/AndroidManifest.xml](file:///C:/Users/Romit/flutter_projects/flutter_libserialport/android/src/main/AndroidManifest.xml)
- [example/android/app/src/main/AndroidManifest.xml](file:///C:/Users/Romit/flutter_projects/flutter_libserialport/example/android/app/src/main/AndroidManifest.xml)
- [android/libserialport/CMakeLists.txt](file:///C:/Users/Romit/flutter_projects/flutter_libserialport/android/libserialport/CMakeLists.txt)

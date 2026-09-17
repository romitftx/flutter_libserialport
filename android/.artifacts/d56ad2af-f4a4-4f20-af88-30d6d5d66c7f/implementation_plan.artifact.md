# Implementation Plan - Fix 16KB Page Alignment for Android 15+

The goal is to resolve the "relro 16KB alignment issue" by ensuring that the native library (`libserialport.so`) is correctly built and packaged with 16KB page alignment, as required by Android 15 (API 35) and later.

## Background
Android 15 introduced support for devices with 16KB page sizes. To support these devices, shared libraries (.so files) must have their ELF segments aligned to 16KB boundaries and be zip-aligned to 16KB within the APK. Inconsistent alignment (where some libs are aligned and others are not) typically indicates a failure in the automatic zip-alignment process during APK packaging.

## User Review Required
> [!IMPORTANT]
> This fix now requires **NDK r28**, **minSdkVersion 24**, and **AGP 8.7.0**.

## Proposed Changes

### Build Tooling Upgrade
#### [MODIFY] [settings.gradle](file:///C:/Users/Romit/flutter_projects/flutter_libserialport/example/android/settings.gradle)
- Upgrade AGP to `8.7.0` to ensure stable 16KB alignment support.
- Upgrade Kotlin plugin to `2.0.0` for consistency.

### Configuration Cleanup
#### [MODIFY] [gradle.properties](file:///C:/Users/Romit/flutter_projects/flutter_libserialport/example/android/gradle.properties)
- Remove the deprecated `android.useNewApkCreator=false`.

### Native Library Configuration
#### [MODIFY] [build.gradle](file:///C:/Users/Romit/flutter_projects/flutter_libserialport/android/build.gradle)
- Set `minSdkVersion` to `24`.
- Set `ndkVersion` to `28.0.12433566`.

### Example App Configuration
#### [MODIFY] [gradle.properties](file:///C:/Users/Romit/flutter_projects/flutter_libserialport/example/android/gradle.properties)
- Remove `android.bundle.enableUncompressedNativeLibs=false`.

#### [MODIFY] [app/build.gradle](file:///C:/Users/Romit/flutter_projects/flutter_libserialport/example/android/app/build.gradle)
- Add `packaging` options to disable native library compression for 16KB support.

## Why these changes?
1. **`ANDROID_SUPPORT_FLEXIBLE_PAGE_SIZES=ON`**: This is the standard AGP flag to enable 16KB page support. It tells the NDK toolchain to align ELF segments to 16KB.
2. **`ndkVersion "27.x"`**: 16KB support was officially added and matured in NDK r27. Older versions may not handle the alignment correctly even with the flags.
3. **`useLegacyPackaging = false`**: When set to false, AGP stores `.so` files uncompressed in the APK/AAR, which is required for the system to map them directly into memory with the correct alignment.
4. **`max-page-size=16384`**: Explicitly forces the linker to align all ELF segments (including RELRO) to 16KB boundaries.

## Verification Plan
### Automated Tests
- Run `gradlew assembleDebug` to ensure the project builds successfully with the new NDK and flags.
- (Manual) Verify alignment using `readelf -l` on the generated `.so` file if a shell is available, or check for "16KB" in the ELF headers.

### Manual Verification
- Deploy the app to an Android 15 device (or emulator with 16KB page size support) and verify that the library loads without `relro` alignment errors.

# Summary of 16KB Page Alignment Changes

This document summarizes the changes made to ensure `flutter_libserialport` and its example app are compatible with Android 15 (API 35) devices using 16KB memory page sizes.

## 1. Plugin Project (`android/`)

### Build Tooling & SDKs
- **NDK Upgrade**: Set `ndkVersion` to `28.0.12433566` (r28). NDK r28 is required for reliable 16KB page support.
- **SDK Versions**: 
    - `compileSdk` updated to `35`.
    - `minSdkVersion` increased to `24`.
- **Repository**: Migrated from deprecated `jcenter()` to `mavenCentral()`.

### Native Library Build (`CMake`)
- **ELF Alignment**: Added `target_link_options` to `libserialport/CMakeLists.txt` to force 16KB alignment:
    - `-Wl,-z,max-page-size=16384`
    - `-Wl,-z,relro` (Relocation Read-Only)
    - `-Wl,-z,now` (Immediate binding)
- **NDK Flags**: Enabled `-DANDROID_SUPPORT_FLEXIBLE_PAGE_SIZES=ON` in `build.gradle`.

### Packaging & Manifest
- **Uncompressed Libraries**: Set `useLegacyPackaging = false` in `build.gradle`.
- **Manifest Enforcement**: Added `android:extractNativeLibs="false"` to `AndroidManifest.xml` to ensure the OS maps libraries directly from the APK, preserving alignment.

---

## 2. Example App (`example/android/`)

### Build Infrastructure
- **AGP Upgrade**: Updated Android Gradle Plugin to `8.7.0` in `settings.gradle`.
- **Gradle Wrapper**: Updated to Gradle `8.9` in `gradle-wrapper.properties`.
- **Kotlin Upgrade**: Updated Kotlin plugin to `2.0.0`.

### App Configuration
- **SDK Versions**:
    - `compileSdk` and `targetSdk` set to `35`.
    - `minSdk` set to `24`.
    - `ndkVersion` set to `28.0.12433566`.
- **Packaging**: Set `useLegacyPackaging = false` in `app/build.gradle`.
- **Manifest**: Added `android:extractNativeLibs="false"` to `app/src/main/AndroidManifest.xml`.
- **Properties**: Removed the deprecated `android.bundle.enableUncompressedNativeLibs` flag from `gradle.properties`.

---

## Verification Checklist
- [ ] Run `flutter clean`
- [ ] Run `flutter build apk --release`
- [ ] Verify in **APK Analyzer**:
    - `libserialport.so` should show **16 KB** in the Alignment column.
    - `libapp.so` should show **16 KB**.
    - `libflutter.so` should show **16 KB** (requires Flutter 3.24+).

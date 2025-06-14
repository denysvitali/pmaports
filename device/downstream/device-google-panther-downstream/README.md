# Google Pixel 7 (panther) port for postmarketOS

This is a downstream kernel port of the Google Pixel 7 for postmarketOS, using the AOSP Pantah 6.1 kernel source.

## Current Status

The basic structure is now configured to use the correct AOSP kernel repositories:
- **Kernel Source**: `android.googlesource.com/kernel/common` (branch: `android-gs-pantah-6.1-android16`)
- **Device Config**: `android.googlesource.com/kernel/devices/google/pantah`
- **SoC Config**: `android.googlesource.com/kernel/devices/google/gs201`

This is a modern 6.1 kernel, much more recent than LineageOS alternatives.

## How to Complete the Port

### 1. Get the Actual Commit Hashes

The APKBUILD currently uses `HEAD` for commits. You need to get the actual commit hashes:

```bash
# Check the latest commits for each repo
curl -s "https://android.googlesource.com/kernel/common/+log/android-gs-pantah-6.1-android16" | head -20
curl -s "https://android.googlesource.com/kernel/devices/google/pantah/+log/android-gs-pantah-6.1-android16" | head -20  
curl -s "https://android.googlesource.com/kernel/devices/google/gs201/+log/android-gs-pantah-6.1-android16" | head -20
```

Then update the `_commit`, `_pantah_commit`, and `_gs201_commit` variables in the APKBUILD.

### 2. Get the Proper Kernel Configuration

**Option A: Build from AOSP source (recommended)**
```bash
# Set up AOSP kernel build
repo init -u https://android.googlesource.com/kernel/manifest -b android-gs-pantah-6.1-android16
repo sync

# Build with pantah defconfig
build/build_pantah.sh

# Extract the final config
cp out/pantah/private/devices/google/pantah/.config /path/to/pmaports/device/downstream/linux-google-panther-downstream/config-google-panther-downstream.aarch64
```

**Option B: Extract from running device**
```bash
adb pull /proc/config.gz
gunzip config.gz
mv config /path/to/pmaports/device/downstream/linux-google-panther-downstream/config-google-panther-downstream.aarch64
```

**Option C: Use defconfig from source**
```bash
# After setting up AOSP source
make O=out ARCH=arm64 gki_defconfig pantah_gki_defconfig
cp out/.config /path/to/pmaports/device/downstream/linux-google-panther-downstream/config-google-panther-downstream.aarch64
```

### 3. Update the SHA512 Checksums

After getting proper commit hashes:

```bash
cd /path/to/pmaports/device/downstream/linux-google-panther-downstream/
# Download the sources with proper commit hashes
# Calculate checksums and update APKBUILD
```

### 4. Add Required Build Dependencies

The AOSP kernel might need additional dependencies. Update `makedepends` in APKBUILD if needed:
- `bazel` (for Kleaf build system)
- `python3`
- Additional toolchain components

### 5. Build the Package

```bash
pmbootstrap build linux-google-panther-downstream
pmbootstrap build device-google-panther-downstream
```

### 6. Use the Downstream Kernel

```bash
pmbootstrap install --kernel google-panther-downstream
```

## Notes

- This uses the official AOSP kernel (6.1) which is actively maintained by Google
- The AOSP kernel uses the Kleaf build system (Bazel-based), but this port adapts it for traditional make
- You'll likely need to disable some Android security features (SELinux, etc.) for postmarketOS compatibility
- The kernel includes all Tensor GS201-specific drivers and optimizations
- Consider extracting the vendor modules separately if needed for full hardware support 

# libmpv Build Guide for Linux

This guide provides detailed instructions for building libmpv for Linux x64 and ARM64 architectures.

## Table of Contents

- [Overview](#overview)
- [Prerequisites](#prerequisites)
- [Build Methods](#build-methods)
  - [Automated GitHub Actions](#automated-github-actions)
  - [Manual Build on Linux](#manual-build-on-linux)
  - [Cross-compilation for ARM64](#cross-compilation-for-arm64)
- [Configuration Options](#configuration-options)
- [Integration with Unreal Engine](#integration-with-unreal-engine)
- [Troubleshooting](#troubleshooting)

## Overview

libmpv is a client library for the mpv media player that can be embedded into applications. This guide uses the [mpv-build](https://github.com/mpv-player/mpv-build) repository, which provides helper scripts to compile mpv, ffmpeg, and libass together.

### What Gets Built

- **libmpv.so** - Main shared library
- **libmpv.so.2** - Versioned symlink
- **Headers** - C API headers for integration

### Features Enabled

- ✅ Hardware-accelerated video decoding (VDPAU, VAAPI)
- ✅ Lua scripting support
- ✅ Audio output (ALSA, PulseAudio)
- ✅ Video output (X11, OpenGL)
- ✅ Multiple video codec support (H.264, H.265, VP8, VP9, etc.)

## Prerequisites

### For x64 Build

**Ubuntu/Debian:**
```bash
sudo apt-get update
sudo apt-get install -y \
  build-essential \
  git \
  yasm \
  nasm \
  cmake \
  ninja-build \
  autoconf \
  automake \
  libtool \
  pkg-config \
  libx11-dev \
  libxext-dev \
  libxrandr-dev \
  libxinerama-dev \
  libxcursor-dev \
  libxi-dev \
  libxss-dev \
  libxv-dev \
  libvdpau-dev \
  libva-dev \
  libgl1-mesa-dev \
  libasound2-dev \
  libpulse-dev \
  libfribidi-dev \
  libfreetype6-dev \
  libfontconfig1-dev \
  libharfbuzz-dev \
  libjpeg-dev \
  libssl-dev \
  libbluray-dev \
  libdvdread-dev \
  libdvdnav-dev \
  libuchardet-dev \
  zlib1g-dev \
  liblcms2-dev \
  libarchive-dev \
  python3 \
  python3-pip

# Upgrade meson (Ubuntu 22.04 has 0.61.2, but libplacebo requires >= 0.63)
sudo pip3 install --upgrade meson
```

**Note:** Meson >= 0.63 is required for libplacebo. Ubuntu 22.04 ships with 0.61.2, so you must upgrade it via pip.

### For ARM64 Build

- Docker
- QEMU user-static (for cross-compilation)

```bash
# Install Docker
sudo apt-get install docker.io

# Set up QEMU
docker run --rm --privileged multiarch/qemu-user-static --reset -p yes
```

## Build Methods

### Automated GitHub Actions

The easiest method is to use the GitHub Actions workflow:

1. Navigate to your repository on GitHub
2. Go to **Actions** → **Build libmpv for Linux**
3. Click **Run workflow**
4. Configure options:
   - **MPV version**: `master` (bleeding edge) or `release` (stable) or custom tag
   - **FFmpeg version**: `master` or `release` or custom tag
5. Wait for the build to complete (~15-30 minutes)
6. Download artifacts from the workflow run

The workflow produces:
- `libmpv-linux-x64` - x64 build
- `libmpv-linux-arm64` - ARM64 build
- `libmpv-linux-all-architectures` - Combined package

### Manual Build on Linux

#### Step 1: Clone mpv-build

```bash
git clone https://github.com/mpv-player/mpv-build.git
cd mpv-build
```

#### Step 2: Select Versions

**For latest development versions:**
```bash
./use-mpv-master
./use-ffmpeg-master
./use-libass-master
```

**For stable releases:**
```bash
./use-mpv-release
./use-ffmpeg-release
./use-libass-master
```

**For specific versions:**
```bash
./use-mpv-custom v0.38.0
./use-ffmpeg-custom n6.1
```

#### Step 3: Configure Build Options

**FFmpeg options** (create `ffmpeg_options` file):
```bash
# Hardware acceleration for video decoding
printf "%s\n" --enable-vdpau >> ffmpeg_options
printf "%s\n" --enable-vaapi >> ffmpeg_options
```

**MPV options** (create `mpv_options` file):
```bash
# Enable libmpv as shared library
printf "%s\n" -Dlibmpv=true >> mpv_options

# Enable Lua scripting
printf "%s\n" -Dlua=enabled >> mpv_options

# Disable JavaScript (optional)
printf "%s\n" -Djavascript=disabled >> mpv_options
```

#### Step 4: Build

```bash
# Full rebuild (recommended)
./rebuild -j$(nproc)

# Or incremental build (faster but may fail)
./build -j$(nproc)
```

#### Step 5: Locate Built Libraries

After a successful build:
```bash
# Find libmpv
find . -name "libmpv.so*"

# Typical location:
# mpv/build/libmpv.so
# mpv/build/libmpv.so.2
# mpv/build/libmpv.so.2.X.Y
```

### Cross-compilation for ARM64

Using Docker with QEMU emulation:

```bash
docker run --rm --platform linux/arm64 \
  -v $(pwd):/workspace \
  -w /build \
  arm64v8/ubuntu:22.04 \
  bash -c '
    set -e
    
    # Install dependencies
    apt-get update
    DEBIAN_FRONTEND=noninteractive apt-get install -y \
      build-essential git yasm nasm cmake meson ninja-build \
      autoconf automake libtool pkg-config \
      libx11-dev libxext-dev libfribidi-dev \
      libfreetype6-dev libfontconfig1-dev libharfbuzz-dev \
      libjpeg-dev libssl-dev libvdpau-dev libva-dev \
      libasound2-dev libpulse-dev python3 python3-pip
    
    # Clone and build
    git clone https://github.com/mpv-player/mpv-build.git
    cd mpv-build
    
    # Configure
    ./use-mpv-master
    ./use-ffmpeg-master
    ./use-libass-master
    
    # Set options
    printf "%s\n" -Dlibmpv=true >> mpv_options
    printf "%s\n" --enable-vdpau >> ffmpeg_options
    printf "%s\n" --enable-vaapi >> ffmpeg_options
    
    # Build
    ./rebuild -j$(nproc)
    
    # Copy to workspace
    mkdir -p /workspace/Linux/arm64/lib
    find . -name "libmpv.so*" -exec cp -P {} /workspace/Linux/arm64/lib/ \;
  '
```

## Configuration Options

### Common FFmpeg Options

**Hardware Acceleration (Decoding):**
```bash
--enable-vdpau          # NVIDIA VDPAU (video decode)
--enable-vaapi          # Intel/AMD VAAPI (video decode)
--enable-nvdec          # NVIDIA hardware decoding
--enable-cuda           # CUDA acceleration
```

**Optional Encoders (if you need encoding features):**
```bash
--enable-libx264        # H.264 encoder
--enable-libx265        # H.265 encoder
--enable-libvpx         # VP8/VP9 encoder
--enable-libmp3lame     # MP3 encoder
--enable-gpl            # Required for x264/x265
```

**Protocols:**
```bash
--enable-openssl        # HTTPS support
--enable-gnutls         # Alternative to OpenSSL
```

### Common MPV Options

```bash
# Library options
-Dlibmpv=true           # Build libmpv (default: true)
-Dcplayer=false         # Don't build mpv binary

# Scripting
-Dlua=enabled           # Lua scripting
-Djavascript=enabled    # JavaScript scripting

# Features
-Dmanpage-build=disabled  # Skip man page generation
-Dpdf-build=disabled      # Skip PDF documentation
-Dzimg=enabled            # High quality video scaling
```

## Integration with Unreal Engine

### Directory Structure

Place the built libraries in your Unreal Engine plugin:

```
YourPlugin/
├── Source/
│   └── ThirdParty/
│       └── libmpv/
│           ├── include/
│           │   └── mpv/
│           │       └── client.h
│           └── lib/
│               ├── Linux/
│               │   ├── x64/
│               │   │   ├── libmpv.so
│               │   │   ├── libmpv.so.2
│               │   │   └── libmpv.so.2.X.Y
│               │   └── arm64/
│               │       ├── libmpv.so
│               │       ├── libmpv.so.2
│               │       └── libmpv.so.2.X.Y
```

### Build.cs Configuration

```csharp
// In your .Build.cs file
if (Target.Platform == UnrealTargetPlatform.Linux)
{
    string LibraryPath = Path.Combine(ModuleDirectory, "ThirdParty", "libmpv", "lib", "Linux", "x64");
    string IncludePath = Path.Combine(ModuleDirectory, "ThirdParty", "libmpv", "include");
    
    PublicIncludePaths.Add(IncludePath);
    PublicAdditionalLibraries.Add(Path.Combine(LibraryPath, "libmpv.so"));
    RuntimeDependencies.Add(Path.Combine(LibraryPath, "libmpv.so.2"));
}
```

### Basic Usage Example

```cpp
#include "mpv/client.h"

// Initialize mpv
mpv_handle* mpv = mpv_create();
if (!mpv)
{
    UE_LOG(LogTemp, Error, TEXT("Failed to create mpv context"));
    return;
}

// Configure
mpv_set_option_string(mpv, "vo", "gpu");
mpv_set_option_string(mpv, "hwdec", "auto");

// Initialize
if (mpv_initialize(mpv) < 0)
{
    UE_LOG(LogTemp, Error, TEXT("Failed to initialize mpv"));
    mpv_terminate_destroy(mpv);
    return;
}

// Load file
const char* cmd[] = {"loadfile", "video.mp4", NULL};
mpv_command(mpv, cmd);

// Cleanup
mpv_terminate_destroy(mpv);
```

## Troubleshooting

### Build Errors

**"Meson version is X but project requires >=0.63":**
```bash
# Ubuntu 22.04 ships with meson 0.61.2, but libplacebo requires >= 0.63
# Upgrade meson via pip:
sudo pip3 install --upgrade meson

# Verify version:
meson --version  # Should show >= 0.63
```

**"meson not found":**
```bash
# Install via pip:
sudo pip3 install meson

# Or install via apt (older version):
sudo apt-get install meson
```

**"yasm/nasm not found":**
```bash
sudo apt-get install yasm nasm
```

**Missing dependencies:**
- Check `config.log` in the ffmpeg or mpv build directories
- Install missing `-dev` packages for your distribution

### Runtime Errors

**"libmpv.so.2: cannot open shared object file":**
- Ensure the library is in your LD_LIBRARY_PATH
- Use `ldd` to check dependencies: `ldd libmpv.so`

**Hardware acceleration not working:**
```bash
# Check VDPAU support
vainfo

# Check VAAPI support  
vdpauinfo

# Test with mpv directly
mpv --hwdec=auto video.mp4
```

**Playback issues:**
- Check logs: `mpv --msg-level=all=trace video.mp4`
- Verify codec support: `mpv --list-codecs`
- Try software decoding: `mpv --hwdec=no video.mp4`

### Verification

**Check architecture:**
```bash
file libmpv.so
# Should show: ELF 64-bit LSB shared object, x86-64
# Or: ELF 64-bit LSB shared object, ARM aarch64
```

**Check dependencies:**
```bash
ldd libmpv.so
# Should show all dependencies are found
```

**Check symbols:**
```bash
nm -D libmpv.so | grep mpv_
# Should show exported mpv_* functions
```

## Additional Resources

- [mpv-build GitHub Repository](https://github.com/mpv-player/mpv-build)
- [mpv Documentation](https://mpv.io/manual/master/)
- [libmpv Client API Documentation](https://mpv.io/manual/master/#client-api)
- [FFmpeg Documentation](https://ffmpeg.org/documentation.html)

## Version Information

The workflow builds with:
- **mpv**: Latest master or specified release
- **FFmpeg**: Latest master or specified release  
- **libass**: Latest master
- **Compiler**: GCC from Ubuntu 22.04

For reproducible builds, specify exact commit hashes or tags when running the workflow.


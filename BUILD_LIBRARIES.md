# Building Third-Party Libraries for Runtime Video Recorder

This document describes how to build the third-party libraries (x264, lsmash, mp4v2) for various platforms.

## Table of Contents

- [Overview](#overview)
- [Quick Start](#quick-start)
- [Building MP4v2](#building-mp4v2)
  - [Linux ARM64](#linux-arm64)
  - [Linux x64](#linux-x64)
  - [macOS](#macos)
  - [Windows](#windows)
- [Building x264](#building-x264)
- [Building lsmash](#building-lsmash)
- [Using GitHub Actions](#using-github-actions)
- [Integration with Unreal Engine](#integration-with-unreal-engine)
- [Troubleshooting](#troubleshooting)

---

## Overview

The Runtime Video Recorder plugin depends on several third-party libraries for video encoding and MP4 muxing:

- **x264** - H.264 video encoder
- **lsmash** - MP4 muxer/demuxer library
- **mp4v2** - Alternative MP4 library

Pre-built libraries are included in this repository for:
- Windows (x64)
- Linux (x64, ARM64)
- macOS (Universal)
- Android (armv8-a, x86_64)

---

## Quick Start

### Using GitHub Actions (Recommended)

The easiest way to build libraries is using the provided GitHub workflows:

1. **Fork or clone this repository**
2. **Navigate to Actions tab**
3. **Select the appropriate workflow:**
   - `Build MP4v2 for Linux ARM64` - Autotools-based build
   - `Build MP4v2 (CMake) for Multiple Platforms` - CMake-based multi-platform
   - `Build MP4v2 (Native ARM64 Runners)` - For GitHub Team/Enterprise
4. **Click "Run workflow"**
5. **Download artifacts** from the completed workflow

### Manual Build (Linux/macOS)

```bash
# Install dependencies
sudo apt-get install build-essential cmake git autoconf automake libtool

# Clone mp4v2
git clone https://github.com/enzo1982/mp4v2.git
cd mp4v2

# Build with CMake
mkdir build && cd build
cmake .. -DCMAKE_BUILD_TYPE=Release -DBUILD_SHARED_LIBS=OFF
make -j$(nproc)

# Output: libmp4v2.a
```

---

## Building MP4v2

MP4v2 is a C/C++ library for creating and modifying MP4 files according to ISO-IEC:14496-1:2001.

**Repository:** https://github.com/enzo1982/mp4v2

### Linux ARM64

#### Method 1: Using Docker with QEMU (Cross-compilation)

```bash
# Set up QEMU (one-time setup)
docker run --rm --privileged multiarch/qemu-user-static --reset -p yes

# Build in ARM64 container
docker run --rm --platform linux/arm64 \
  -v $(pwd):/workspace \
  -w /build \
  arm64v8/ubuntu:22.04 \
  bash -c "
    # Install dependencies
    apt-get update && \
    apt-get install -y build-essential cmake git autoconf automake libtool pkg-config && \
    
    # Clone repository
    git clone https://github.com/enzo1982/mp4v2.git && \
    cd mp4v2 && \
    
    # Build with CMake
    mkdir build && cd build && \
    cmake .. -DCMAKE_BUILD_TYPE=Release -DBUILD_SHARED_LIBS=OFF && \
    make -j\$(nproc) && \
    
    # Copy to workspace
    mkdir -p /workspace/Linux/arm64 && \
    find . -name 'libmp4v2.a' -exec cp {} /workspace/Linux/arm64/ \; && \
    
    # Verify
    file /workspace/Linux/arm64/libmp4v2.a
  "
```

#### Method 2: Using Autotools

```bash
docker run --rm --platform linux/arm64 \
  -v $(pwd):/workspace \
  arm64v8/ubuntu:22.04 \
  bash -c "
    apt-get update && \
    apt-get install -y build-essential autoconf automake libtool git && \
    git clone https://github.com/enzo1982/mp4v2.git && \
    cd mp4v2 && \
    autoreconf -i && \
    ./configure --prefix=/usr/local --enable-static --disable-shared && \
    make -j\$(nproc) && \
    mkdir -p /workspace/Linux/arm64 && \
    cp .libs/libmp4v2.a /workspace/Linux/arm64/
  "
```

#### Method 3: On Native ARM64 Hardware

```bash
# On Raspberry Pi, ARM Mac with Linux VM, or ARM64 cloud instance
sudo apt-get update
sudo apt-get install -y build-essential cmake git

git clone https://github.com/enzo1982/mp4v2.git
cd mp4v2
mkdir build && cd build
cmake .. -DCMAKE_BUILD_TYPE=Release -DBUILD_SHARED_LIBS=OFF
make -j$(nproc)

# Output: libmp4v2.a
cp libmp4v2.a ~/RuntimeVideoRecorderDocs/Linux/arm64/
```

### Linux x64

```bash
sudo apt-get install -y build-essential cmake git

git clone https://github.com/enzo1982/mp4v2.git
cd mp4v2
mkdir build && cd build
cmake .. -DCMAKE_BUILD_TYPE=Release -DBUILD_SHARED_LIBS=OFF
make -j$(nproc)

# Output: libmp4v2.a
```

### macOS

#### Using CMake

```bash
# Install dependencies
brew install cmake

# Clone and build
git clone https://github.com/enzo1982/mp4v2.git
cd mp4v2
mkdir build && cd build

# Universal binary (Apple Silicon + Intel)
cmake .. \
  -DCMAKE_BUILD_TYPE=Release \
  -DBUILD_SHARED_LIBS=OFF \
  -DCMAKE_OSX_ARCHITECTURES="arm64;x86_64"
  
make -j$(sysctl -n hw.ncpu)

# Output: libmp4v2.a (universal binary)
```

#### Using Autotools

```bash
brew install autoconf automake libtool

git clone https://github.com/enzo1982/mp4v2.git
cd mp4v2
autoreconf -i

# Universal binary
./configure --enable-static --disable-shared \
  CFLAGS="-arch arm64 -arch x86_64" \
  CXXFLAGS="-arch arm64 -arch x86_64"
  
make -j$(sysctl -n hw.ncpu)

# Output: .libs/libmp4v2.a
```

### Windows

#### Using Visual Studio

```powershell
# Install dependencies: Visual Studio 2019/2022 with C++ workload
# Install Git for Windows

# Clone repository
git clone https://github.com/enzo1982/mp4v2.git
cd mp4v2

# Open Visual Studio solution
start vstudio\mp4v2.sln

# Or build from command line:
msbuild vstudio\mp4v2.sln /p:Configuration=Release /p:Platform=x64

# Output: vstudio\x64\Release\libmp4v2.lib
```

#### Using CMake

```powershell
# Using Visual Studio command prompt
git clone https://github.com/enzo1982/mp4v2.git
cd mp4v2
mkdir build
cd build

cmake .. -G "Visual Studio 17 2022" -A x64 -DBUILD_SHARED_LIBS=OFF
cmake --build . --config Release

# Output: Release\mp4v2.lib
```

### Android

```bash
# Set Android NDK path
export NDK_ROOT=/path/to/android-ndk

# Build for ARM64
mkdir build-android-arm64 && cd build-android-arm64

cmake .. \
  -DCMAKE_TOOLCHAIN_FILE=$NDK_ROOT/build/cmake/android.toolchain.cmake \
  -DANDROID_ABI=arm64-v8a \
  -DANDROID_PLATFORM=android-21 \
  -DBUILD_SHARED_LIBS=OFF \
  -DCMAKE_BUILD_TYPE=Release

make -j$(nproc)

# Output: libmp4v2.a
```

---

## Building x264

x264 is a free software library for encoding video streams into the H.264/MPEG-4 AVC format.

**Repository:** https://code.videolan.org/videolan/x264

### macOS Example

See the included `build_x264_macos.sh` script:

```bash
#!/bin/bash
git clone https://code.videolan.org/videolan/x264.git
cd x264

./configure \
  --enable-static \
  --disable-cli \
  --extra-cflags="-arch arm64 -arch x86_64" \
  --extra-ldflags="-arch arm64 -arch x86_64"

make -j$(sysctl -n hw.ncpu)

# Output: libx264.a
```

### Linux ARM64

```bash
git clone https://code.videolan.org/videolan/x264.git
cd x264
./configure --enable-static --disable-cli --host=aarch64-linux
make -j$(nproc)
```

---

## Building lsmash

L-SMASH is a MP4 muxer/demuxer library.

**Repository:** https://github.com/l-smash/l-smash

### macOS Example

See the included `build_lsmash_macos.sh` script:

```bash
#!/bin/bash
git clone https://github.com/l-smash/l-smash.git
cd l-smash

./configure \
  --enable-static \
  --extra-cflags="-arch arm64 -arch x86_64" \
  --extra-ldflags="-arch arm64 -arch x86_64"

make -j$(sysctl -n hw.ncpu)

# Output: liblsmash.a
```

### Linux ARM64

```bash
git clone https://github.com/l-smash/l-smash.git
cd l-smash
./configure --enable-static
make -j$(nproc)
```

---

## Using GitHub Actions

This repository includes several GitHub Actions workflows for automated builds:

### Workflow Options

| Workflow | Build System | Platforms | Auto-commit |
|----------|--------------|-----------|-------------|
| `build-mp4v2-linux-arm64.yml` | Autotools | Linux ARM64 | Yes |
| `build-mp4v2-cmake.yml` | CMake | Linux ARM64/x64 | No |
| `build-mp4v2-native-arm64.yml` | Both | Linux ARM64 | Optional |

### Running a Workflow

1. **Navigate to your GitHub repository**
2. **Click on "Actions" tab**
3. **Select a workflow from the left sidebar**
4. **Click "Run workflow" button**
5. **Configure options (if available):**
   - Platform selection
   - Commit options
   - Build configuration
6. **Click "Run workflow" to start**

### Downloading Artifacts

After the workflow completes:

1. **Open the completed workflow run**
2. **Scroll to "Artifacts" section at the bottom**
3. **Click on the artifact name to download** (e.g., `mp4v2-linux-arm64.zip`)
4. **Extract the ZIP file**
5. **Copy files to your project:**
   ```bash
   unzip mp4v2-linux-arm64.zip
   cp -r Linux/arm64/* RuntimeVideoRecorderDocs/Linux/arm64/
   cp -r include/mp4v2/* RuntimeVideoRecorderDocs/include/mp4v2/
   ```

### Workflow Configuration

To modify build options, edit the workflow YAML file:

```yaml
# Example: Adding compiler flags
cmake .. \
  -DCMAKE_BUILD_TYPE=Release \
  -DBUILD_SHARED_LIBS=OFF \
  -DCMAKE_CXX_FLAGS="-O3 -march=native"
```

---

## Integration with Unreal Engine

### Directory Structure

Place compiled libraries in the following structure:

```
YourPlugin/
├── Source/
│   └── ThirdParty/
│       ├── Win/
│       │   └── x64/
│       │       ├── libx264.lib
│       │       └── lsmash.lib
│       ├── Linux/
│       │   ├── x64/
│       │   │   ├── libx264.a
│       │   │   └── lsmash.a
│       │   └── arm64/
│       │       ├── libx264.a
│       │       ├── lsmash.a
│       │       └── libmp4v2.a
│       ├── Mac/
│       │   ├── libx264.a
│       │   └── liblsmash.a
│       └── include/
│           ├── x264.h
│           ├── x264_config.h
│           ├── lsmash.h
│           └── mp4v2/
```

### Build.cs Configuration

Update your plugin's `Build.cs` file:

```csharp
public class YourPlugin : ModuleRules
{
    public YourPlugin(ReadOnlyTargetRules Target) : base(Target)
    {
        string ThirdPartyPath = Path.Combine(ModuleDirectory, "ThirdParty");
        string IncludePath = Path.Combine(ThirdPartyPath, "include");
        
        PublicIncludePaths.Add(IncludePath);
        
        if (Target.Platform == UnrealTargetPlatform.Linux)
        {
            string LibPath = Target.Architecture.bIsX64
                ? Path.Combine(ThirdPartyPath, "Linux", "x64")
                : Path.Combine(ThirdPartyPath, "Linux", "arm64");
                
            PublicAdditionalLibraries.Add(Path.Combine(LibPath, "libx264.a"));
            PublicAdditionalLibraries.Add(Path.Combine(LibPath, "lsmash.a"));
            PublicAdditionalLibraries.Add(Path.Combine(LibPath, "libmp4v2.a"));
        }
        else if (Target.Platform == UnrealTargetPlatform.Win64)
        {
            string LibPath = Path.Combine(ThirdPartyPath, "Win", "x64");
            PublicAdditionalLibraries.Add(Path.Combine(LibPath, "libx264.lib"));
            PublicAdditionalLibraries.Add(Path.Combine(LibPath, "lsmash.lib"));
        }
        else if (Target.Platform == UnrealTargetPlatform.Mac)
        {
            string LibPath = Path.Combine(ThirdPartyPath, "Mac");
            PublicAdditionalLibraries.Add(Path.Combine(LibPath, "libx264.a"));
            PublicAdditionalLibraries.Add(Path.Combine(LibPath, "liblsmash.a"));
        }
    }
}
```

---

## Troubleshooting

### Build Errors

#### "configure: command not found"
```bash
# Install autotools
sudo apt-get install autoconf automake libtool  # Linux
brew install autoconf automake libtool          # macOS
```

#### "CMake version too old"
```bash
# Update CMake
sudo apt-get install cmake  # Linux (may need PPA for latest)
brew upgrade cmake          # macOS
# Or download from https://cmake.org/download/
```

#### "No rule to make target"
```bash
# Clean and rebuild
make clean
rm -rf build/
# Start build process again
```

### QEMU Issues

#### "qemu-aarch64-static: not found"
```bash
# Install QEMU
sudo apt-get install qemu-user-static
docker run --rm --privileged multiarch/qemu-user-static --reset -p yes
```

#### "exec format error"
```bash
# Verify QEMU is registered
docker run --rm --platform linux/arm64 arm64v8/ubuntu:22.04 uname -m
# Should output: aarch64
```

### Architecture Verification

```bash
# Verify library architecture
file libmp4v2.a

# Expected outputs:
# ARM64: "current ar archive" with aarch64/ARM
# x64: "current ar archive" with x86-64/x86_64
# Universal (macOS): "Mach-O universal binary with 2 architectures"

# List object files in archive
ar -t libmp4v2.a

# Check symbols
nm libmp4v2.a | grep MP4Create
```

### GitHub Actions Issues

#### Workflow not visible
- Ensure YAML files are in `.github/workflows/`
- Check YAML syntax with a validator
- Verify file has `.yml` or `.yaml` extension

#### Artifact not generated
- Check workflow logs for build errors
- Verify artifact path exists before upload
- Ensure files were actually built

#### Permission denied on commit
- Check repository settings → Actions → Workflow permissions
- Enable "Read and write permissions"

---

## Additional Resources

- **MP4v2 Documentation:** http://mp4v2.org/
- **x264 Documentation:** https://www.videolan.org/developers/x264.html
- **L-SMASH:** https://github.com/l-smash/l-smash
- **GitHub Actions:** https://docs.github.com/en/actions
- **Unreal Engine Build Configuration:** https://docs.unrealengine.com/en-US/ProductionPipelines/BuildTools/

---

## License

Each library has its own license:
- **mp4v2:** Mozilla Public License (MPL) 1.1
- **x264:** GNU GPL v2
- **lsmash:** ISC License

Ensure compliance with these licenses in your project.


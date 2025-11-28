Runtime Video Recorder (RVR)

![Version](https://img.shields.io/badge/version-2.0-blue.svg)
![UE5](https://img.shields.io/badge/Unreal%20Engine-5.3%2B-informational.svg)

**High-performance video recording plugin for Unreal Engine**

[Features](#features) • [Installation](#installation) • [Quick Start](#quick-start) • [API Reference](#api-reference) • [Support](#support)

---

## Documentation

Check this [page](https://unrealsolutions.com/docs/products/runtime-video-recorder/)

## Overview

Runtime Video Recorder (RVR) is a production-ready Unreal Engine plugin that enables **MP4 video recording at runtime without game or render thread hitches**. Built with performance in mind, it supports hardware-accelerated encoding on Windows (WMF), MacOS (AVFoundation), Oculus/Android (MediaCodec), Linux (OpenH264), and includes specialized features for VR and multi-camera setups.

### Key Features

✨ **Zero Performance Impact**
- Asynchronous encoding on dedicated thread
- No game thread blocking
- Hardware acceleration support (Windows, Mac, Android, Oculus)

🎥 **Flexible Recording Sources**
- Game/Editor viewport recording
- Render target recording
- Single camera recording (Camera & CineCamera)
- Multi-camera grid recording
- 360-degree video support (experimental)

🎵 **Audio Recording**
- Full in-game audio capture
- Selective SoundSubmix recording
- Synchronized audio/video encoding

⚡ **Advanced Features**
- Frame-rate independent recording
- Manual frame capture mode
- Circular buffer (record last N seconds)
- Deferred encoding option
- Real-time camera preview
- Screenshot capture

### Platform Support

| Platform | Status | Hardware Acceleration | Encoder |
|----------|--------|----------------------|---------|
| Windows (Win64) | ✅ Full Support | ✅ Yes (WMF) | H.264 |
| MacOS | ✅ Full Support | ✅ Yes (AVFoundation) | H.264 |
| Android | ✅ Full Support | ✅ Yes (MediaCodec) | H.264 |
| Oculus | ✅ Full Support | ✅ Yes (MediaCodec) | H.264 |
| Linux | ✅ Full Support | ❌ Software | OpenH264 |

---

## Installation
1. Download from [Google Disk](https://drive.google.com/drive/folders/1vv2RanJ07J2719C7mW-6G6-fQ_oTK391?usp=drive_link)
2. Follow [README](https://docs.google.com/document/d/1f9RaY4R70p-AkX3P68co_NG3qvLvSPHDur4j6os_gGY/edit?usp=drive_link)
3. Regenerate project files (right-click `.uproject` → Generate Visual Studio project files)
4. Compile the project
5. Enable the plugin in the editor (Edit → Plugins)

### Dependencies

The plugin automatically enables these required plugins:
- **AudioCapture** - For audio recording functionality
- **ElectraPlayer** - For video playback support

### Building Third-Party Libraries

This repository includes pre-compiled libraries (x264, lsmash, mp4v2, libmpv) for various platforms.

#### Building libmpv for Linux

The repository includes a GitHub Actions workflow to build libmpv for both Linux x64 and ARM64 architectures using the [mpv-build](https://github.com/mpv-player/mpv-build) repository.

##### Using GitHub Actions (Recommended)
1. Go to the **Actions** tab in your GitHub repository
2. Select the **Build libmpv for Linux** workflow
3. Click **Run workflow**
4. (Optional) Specify MPV and FFmpeg versions:
   - `master` - Latest development version (default)
   - `release` - Latest stable release
   - Custom tag/branch/commit (e.g., `v0.38.0`)
5. Download the artifacts once the build completes

The workflow will produce:
- `libmpv-linux-x64` - x64 libraries and headers
- `libmpv-linux-arm64` - ARM64 libraries and headers  
- `libmpv-linux-all-architectures` - Combined package with build info

##### Manual Build for Linux

**For x64:**
```bash
# Install dependencies (Ubuntu/Debian)
sudo apt-get update
sudo apt-get install -y \
  build-essential git yasm nasm cmake ninja-build \
  autoconf automake libtool pkg-config \
  libx11-dev libxext-dev libxrandr-dev libxinerama-dev \
  libxcursor-dev libxi-dev libxss-dev libxv-dev \
  libvdpau-dev libva-dev libgl1-mesa-dev \
  libasound2-dev libpulse-dev \
  libfribidi-dev libfreetype6-dev libfontconfig1-dev \
  libharfbuzz-dev libjpeg-dev libssl-dev \
  python3 python3-pip

# Upgrade meson (Ubuntu 22.04 has 0.61.2, but libplacebo requires >= 0.63)
sudo pip3 install --upgrade meson

# Clone and build
git clone https://github.com/mpv-player/mpv-build.git
cd mpv-build

# Configure versions (optional)
./use-mpv-master    # or ./use-mpv-release
./use-ffmpeg-master # or ./use-ffmpeg-release

# Configure options (hardware acceleration only)
printf "%s\n" -Dlibmpv=true >> mpv_options
printf "%s\n" --enable-vdpau >> ffmpeg_options
printf "%s\n" --enable-vaapi >> ffmpeg_options

# Build
./rebuild -j$(nproc)

# Libraries will be in mpv-build/mpv/build/
```

**For ARM64 (using Docker with QEMU):**
```bash
# Set up QEMU for ARM64 emulation
docker run --rm --privileged multiarch/qemu-user-static --reset -p yes

# Build in ARM64 container
docker run --rm --platform linux/arm64 \
  -v $(pwd):/workspace \
  -w /build \
  arm64v8/ubuntu:22.04 \
  bash -c "
    apt-get update && \
    apt-get install -y build-essential git yasm nasm cmake \
      ninja-build autoconf automake libtool pkg-config \
      libx11-dev libxext-dev libfribidi-dev libfreetype6-dev \
      libfontconfig1-dev libharfbuzz-dev libjpeg-dev libssl-dev \
      python3 python3-pip && \
    pip3 install --upgrade meson && \
    git clone https://github.com/mpv-player/mpv-build.git && \
    cd mpv-build && \
    ./use-mpv-master && \
    printf '%s\n' -Dlibmpv=true >> mpv_options && \
    printf '%s\n' --enable-vdpau >> ffmpeg_options && \
    printf '%s\n' --enable-vaapi >> ffmpeg_options && \
    ./rebuild -j\$(nproc) && \
    mkdir -p /workspace/Linux/arm64/lib && \
    find . -name 'libmpv.so*' -exec cp -P {} /workspace/Linux/arm64/lib/ \;
  "
```

#### Building MP4v2 for Linux ARM64

##### Using Docker with QEMU for cross-compilation
```bash
docker run --rm --platform linux/arm64 \
  -v $(pwd):/workspace \
  -w /build \
  arm64v8/ubuntu:22.04 \
  bash -c "
    apt-get update && \
    apt-get install -y build-essential cmake git autoconf automake libtool && \
    git clone https://github.com/enzo1982/mp4v2.git && \
    cd mp4v2 && \
    mkdir build && cd build && \
    cmake .. -DCMAKE_BUILD_TYPE=Release -DBUILD_SHARED_LIBS=OFF && \
    make -j\$(nproc) && \
    cp libmp4v2.a /workspace/Linux/arm64/
  "
```

The GitHub workflows support:
- **libmpv** - Linux x64 and ARM64 with hardware acceleration (VDPAU, VAAPI) for video playback
- **MP4v2** - Linux ARM64 using QEMU-based cross-compilation

---

## Troubleshooting

### Recording Not Starting

**Check:**
- Is another recording already in progress? Call `IsRecordingInProgress()`
- Does the output directory exist? Plugin won't create directories
- Are required plugins enabled? (ElectraPlayer, AudioCapture)

### Poor Video Quality

**Solutions:**
- Increase `VideoBitrate` (try 30000000 for 30 Mbps)
- Use `Profile_High` encoder profile
- Set `RCMode` to `RC_Quality`
- Increase `TargetQuality` to 75-90

### Performance Issues

**Solutions:**
- Enable hardware acceleration in Project Settings
- Lower video resolution
- Reduce `TargetFPS`
- Use `bPostponeEncoding = false` (default on-the-fly encoding)
- Disable `bDebugDumpEachFrame`

### Audio Sync Issues

**Check:**
- `bFrameRateIndependent` must be `false` for audio recording
- Ensure game is running at stable framerate
- Try increasing audio buffer size in project audio settings

### Hardware Acceleration Not Working

**Windows:**
- Check GPU drivers are up to date
- Try setting `bForceHardwareAccelerationForAllVendors = true` for AMD/Intel
- Some older GPUs may not support H.264 encoding

**Android:**
- Verify device supports MediaCodec H.264 encoding
- Some emulators don't support hardware encoding

---

## Sample Content

The plugin includes sample content in the `Content` folder:

- **Level_RuntimeVideoRecorder.umap** - Demo level with recording UI (UE 5.2 and below)
- **Level_RuntimeVideoRecorder_UE53Plus.umap** - Demo level for UE 5.3+
- **BP_RVR_GameMode** - Example game mode with recording controls
- **BP_RVR_VideoPlayer** - Video playback widget
- **Widget_AndroidRecording** - Mobile-friendly recording UI

## Support & Resources

- **Support Discord:** [Join Community](https://discord.com/invite/pBDSCBcdgv)
- **Developer:** [Unreal Solutions](https://unrealsolutions.com)

---

<div align="center">

**Made with ❤️ for the Unreal Engine Community**

⭐ Star this repo if you find it useful!

</div>

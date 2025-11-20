Runtime Video Recorder (RVR)

![Version](https://img.shields.io/badge/version-2.0-blue.svg)
![UE5](https://img.shields.io/badge/Unreal%20Engine-5.3%2B-informational.svg)

**High-performance video recording plugin for Unreal Engine**

[Features](#features) • [Installation](#installation) • [Quick Start](#quick-start) • [API Reference](#api-reference) • [Support](#support)

---

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

This repository includes pre-compiled libraries (x264, lsmash, mp4v2) for various platforms. To build MP4v2 for Linux ARM64:

#### Using GitHub Actions (Manual Trigger)
1. Go to the **Actions** tab in your GitHub repository
2. Select a workflow (LinuxARM64)
3. Click **Run workflow**
5. Download the artifacts once the build completes

#### Manual Build for Linux ARM64
```bash
# Using Docker with QEMU for cross-compilation
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
- **Linux ARM64** - Using QEMU-based cross-compilation
- **Linux x64** - Native compilation
- Both **Autotools** and **CMake** build systems

---

## Documentation

Check this [page](https://unrealsolutions.com/docs/products/runtime-video-recorder/)

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

<div align="center">
# Runtime Video Recorder (RVR)

![Version](https://img.shields.io/badge/version-2.0-blue.svg)
![UE5](https://img.shields.io/badge/Unreal%20Engine-5.3%2B-informational.svg)

**High-performance video recording plugin for Unreal Engine**

[Features](#features) • [Installation](#installation) • [Quick Start](#quick-start) • [API Reference](#api-reference) • [Support](#support)

</div>

---

## Overview

Runtime Video Recorder (RVR) is a production-ready Unreal Engine plugin that enables **MP4 video recording at runtime without game or render thread hitches**. Built with performance in mind, it supports hardware-accelerated encoding on Windows (WMF), macOS (AVFoundation), Linux/Android (OpenH264), and includes specialized features for VR and multi-camera setups.

### Key Features

✨ **Zero Performance Impact**
- Asynchronous encoding on dedicated thread
- No game thread blocking
- Hardware acceleration support (Windows, Android, Oculus)

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
| macOS | ✅ Full Support | ✅ Yes (AVFoundation) | H.264 |
| Linux | ✅ Full Support | ❌ Software | OpenH264 |
| Android | ✅ Full Support | ✅ Yes (MediaCodec) | H.264 |
| Oculus | ✅ Full Support | ✅ Yes | H.264 |

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

---

## Quick Start

### Blueprint Usage

#### Basic Viewport Recording

```cpp
// Get the recorder subsystem
URuntimeVideoRecorder* Recorder = GEngine->GetEngineSubsystem<URuntimeVideoRecorder>();

// Start recording
Recorder->StartRecording(
    "%auto%",        // Output filename (auto-generates timestamp)
    30,              // Target FPS
    1920,            // Width
    1080,            // Height
    FRuntimeEncoderSettings(),
    true,            // Record UI
    true             // Enable audio
);

// Stop recording (Blueprint with latent action)
Recorder->StopRecording(LatentInfo);
```

#### Camera Recording

```cpp
// Get your camera component
UCameraComponent* MyCamera = GetWorld()->GetFirstPlayerController()->PlayerCameraManager->GetCameraComponent();

// Start recording from camera
Recorder->StartRecordingCamera(
    MyCamera,
    "E:/MyVideos/camera_recording.mp4",
    60,              // 60 FPS
    1920, 1080,
    FRuntimeEncoderSettings(),
    true             // Enable audio
);
```

#### Render Target Recording

```cpp
// Create or get render target
UTextureRenderTarget2D* RenderTarget = NewObject<UTextureRenderTarget2D>();
RenderTarget->InitAutoFormat(1920, 1080);

// Start recording
Recorder->StartRecordingRenderTarget(
    RenderTarget,
    "%auto%",
    30, -1, -1,      // Use render target dimensions
    FRuntimeEncoderSettings()
);
```

### C++ Usage

```cpp
#include "RuntimeVideoRecorder.h"

void AMyActor::StartRecording()
{
    URuntimeVideoRecorder* Recorder = GEngine->GetEngineSubsystem<URuntimeVideoRecorder>();
    
    if (Recorder && !Recorder->IsRecordingInProgress())
    {
        FRuntimeEncoderSettings EncoderSettings;
        EncoderSettings.VideoBitrate = 20000000;  // 20 Mbps
        EncoderSettings.Profile = ERuntimeEncoderProfile::Profile_High;
        EncoderSettings.RCMode = ERuntimeEncoderRCMode::RC_Quality;
        EncoderSettings.TargetQuality = 75;
        
        bool bSuccess = Recorder->StartRecording(
            TEXT("E:/MyVideos/gameplay.mp4"),
            60,    // Target 60 FPS
            -1,    // Use viewport width
            -1,    // Use viewport height
            EncoderSettings,
            true,  // Record UI
            true,  // Enable audio
            false, // Frame rate dependent
            false, // Auto capture frames
            -1.0f  // No circular buffer
        );
        
        if (bSuccess)
        {
            UE_LOG(LogTemp, Log, TEXT("Recording started successfully"));
        }
    }
}

void AMyActor::StopRecording()
{
    URuntimeVideoRecorder* Recorder = GEngine->GetEngineSubsystem<URuntimeVideoRecorder>();
    
    if (Recorder && Recorder->IsRecordingInProgress())
    {
        // Use native API for synchronous stop (no latent action)
        Recorder->StopRecording_NativeAPI();
        
        // Get the output filepath
        FString OutputPath = Recorder->GetLastRecordingFilepath();
        UE_LOG(LogTemp, Log, TEXT("Recording saved to: %s"), *OutputPath);
    }
}
```

---

## API Reference

### Core Recording Methods

#### `StartRecording`
Records from the game viewport.

**Parameters:**
- `OutFilename` (FString) - Output path. Use `"%auto%"` to auto-generate filename in `ProjectDir/Saved/`
- `TargetFPS` (int32) - Target frames per second. Use `-1` for default (30 FPS)
- `Width` (int32) - Video width. Use `-1` for viewport width
- `Height` (int32) - Video height. Use `-1` for viewport height
- `EncoderSettings` (FRuntimeEncoderSettings) - Encoder configuration
- `bRecordUI` (bool) - Include UI/widgets in recording
- `bEnableAudioRecording` (bool) - Enable audio capture
- `bFrameRateIndependent` (bool) - Record at exact FPS regardless of game performance
- `bAllowManualCaptureOnly` (bool) - Only capture frames when `CaptureSingleFrame()` is called
- `LastSecondsToRecord` (float) - Circular buffer size in seconds (max 10 mins)
- `bPostponeEncoding` (bool) - Delay encoding until `StopRecording()`
- `InSubmix` (USoundSubmix*) - Record audio from specific submix only

**Returns:** `bool` - True if recording started successfully

---

#### `StartRecordingRenderTarget`
Records from a texture render target.

**Parameters:**
- `RenderTarget` (UTextureRenderTarget2D*) - Source render target
- *(Same parameters as StartRecording)*

**Returns:** `bool` - True if recording started successfully

---

#### `StartRecordingCamera`
Records from a camera component's perspective.

**Parameters:**
- `Camera` (UCameraComponent*) - Camera to record from
- `ScreenPercentage` (float) - Screen percentage for rendering (default 100.0)
- *(Other parameters same as StartRecording)*

**Returns:** `bool` - True if recording started successfully

---

#### `StartRecordingCineCamera`
Records from a cine camera component with cinematic features.

**Parameters:**
- `Camera` (UCineCameraComponent*) - Cine camera to record from
- *(Same parameters as StartRecordingCamera)*

**Returns:** `bool` - True if recording started successfully

---

#### `StartRecordingMultipleCameras`
Records multiple cameras in a grid layout.

**Parameters:**
- `Cameras` (TArray<UCameraComponent*>) - Array of cameras to record
- *(Other parameters same as StartRecording)*

**Returns:** `bool` - True if recording started successfully

**Example:** 4 cameras will be arranged in a 2×2 grid

---

#### `StopRecording`
Stops recording (Latent Blueprint action).

**Parameters:**
- `LatentInfo` (FLatentActionInfo) - Latent execution info
- `WorldContextObject` (UObject*) - World context

---

#### `StopRecording_NativeAPI`
Stops recording immediately (synchronous, C++ friendly).

---

### Status & Control Methods

#### `IsRecordingInProgress`
**Returns:** `bool` - True if currently recording

---

#### `CaptureSingleFrame`
Captures a single frame (only works when `bAllowManualCaptureOnly = true`).

**Returns:** `bool` - True if frame was captured

---

#### `GetLastRecordingFilepath`
**Returns:** `FString` - Full path to the last recorded video

---

### Camera Preview

#### `StartCameraPreview`
Creates a real-time preview of a camera.

**Parameters:**
- `InCamera` (UCameraComponent*) - Camera to preview
- `TargetFPS` (int32) - Preview update rate
- `Width` (int32) - Preview width
- `Height` (int32) - Preview height

**Returns:** `UTextureRenderTarget2D*` - Render target containing preview

---

#### `StopCameraPreview`
Stops the camera preview.

---

#### `GetCameraPreviewRenderTarget`
**Returns:** `UTextureRenderTarget2D*` - Current preview render target

---

### Utility Methods

#### `MakeScreenshot`
Captures a screenshot.

**Parameters:**
- `OutFilename` (FString) - Output filepath
- `bShowUI` (bool) - Include UI in screenshot

---

#### `EncodeCircularBufferToVideo`
Encodes circular buffer contents (emergency crash recovery).

**Parameters:**
- `OutputBasePath` (FString) - Base output path (auto-appends `_crash_recovery.mp4`)

**Returns:** `bool` - True if encoding succeeded

---

#### `GetPluginResourcesDirectory`
**Returns:** `FString` - Path to plugin's Resources directory

---

### Events (Delegates)

#### `OnRecordingStarted`
Fired when recording begins.

#### `OnRecordingFinished`
Fired when recording completes.

#### `OnPreviewStarted`
Fired when camera preview starts.

#### `OnPreviewStopped`
Fired when camera preview stops.

---

## Encoder Settings

### `FRuntimeEncoderSettings`

Configure video encoding parameters:

```cpp
FRuntimeEncoderSettings Settings;
Settings.VideoBitrate = 20000000;              // 20 Mbps
Settings.Profile = ERuntimeEncoderProfile::Profile_High;
Settings.RCMode = ERuntimeEncoderRCMode::RC_Quality;
Settings.TargetQuality = 50;                    // 0-100 (higher = better quality)
Settings.WIN_QualityVsSpeed = 50;              // Windows only: 0=fast, 100=quality
Settings.WIN_bUseLowLatencyEncoding = false;   // Windows only: Low latency mode
Settings.KEYFRAME_INTERVAL = 30;               // Keyframe every N frames
```

### Encoder Profiles

- **Profile_Baseline** - Maximum compatibility, lower quality
- **Profile_Main** - Good balance (default)
- **Profile_High** - Best quality, may have compatibility issues on older devices

### Rate Control Modes

- **RC_Bitrate** - Constant bitrate mode (predictable file sizes)
- **RC_Quality** - Variable bitrate mode (better quality per file size)

---

## Project Settings

Configure global plugin behavior in **Edit → Project Settings → Plugins → RuntimeVideoRecorder**:

| Setting | Description | Default |
|---------|-------------|---------|
| `bUseHardwareAcceleration` | Enable hardware-accelerated encoding | `true` |
| `bForceHardwareAccelerationForAllVendors` | Force HA for AMD/Intel on Windows | `false` |
| `bDebugDumpEachFrame` | Save raw frames to disk for debugging | `false` |
| `MaxLastSecondsToRecord` | Maximum circular buffer duration | `600s` (10 min) |
| `bShowRecordingsInMediaFolders` | Auto-save to device media folder (Android/Oculus) | `true` |

---

## Advanced Use Cases

### Frame-Rate Independent Recording

Record at exact FPS regardless of game performance (audio disabled):

```cpp
Recorder->StartRecording(
    "%auto%",
    60,              // Always output 60 FPS
    1920, 1080,
    FRuntimeEncoderSettings(),
    true,
    false,           // Audio disabled in this mode
    true             // bFrameRateIndependent = true
);
```

### Circular Buffer (Last N Seconds)

Record continuously but only save the last X seconds:

```cpp
Recorder->StartRecording(
    "%auto%",
    30, 1920, 1080,
    FRuntimeEncoderSettings(),
    true, true,
    false, false,
    30.0f            // Keep last 30 seconds
);
```

Use case: Highlight/replay systems, crash recovery

### Manual Frame Capture

Full control over when frames are captured:

```cpp
// Start recording in manual mode
Recorder->StartRecording(
    "%auto%", 30, 1920, 1080,
    FRuntimeEncoderSettings(),
    true, true, false,
    true             // bAllowManualCaptureOnly = true
);

// In your tick or event
Recorder->CaptureSingleFrame();
```

Use case: Time-lapse, frame-by-frame capture

### Deferred Encoding

Capture frames in memory, encode after stopping (minimizes runtime overhead):

```cpp
Recorder->StartRecording(
    "%auto%", 60, 1920, 1080,
    FRuntimeEncoderSettings(),
    true, true, false, false, -1.0f,
    true             // bPostponeEncoding = true
);

// All frames stored in memory
// Encoding happens when you call StopRecording()
```

⚠️ **Warning:** High memory usage! Only use for short recordings.

### Multi-Camera Grid Recording

```cpp
TArray<UCameraComponent*> Cameras;
Cameras.Add(FrontCamera);
Cameras.Add(BackCamera);
Cameras.Add(LeftCamera);
Cameras.Add(RightCamera);

Recorder->StartRecordingMultipleCameras(
    Cameras,
    "E:/Videos/multicam.mp4",
    30, 1920, 1080
);
```

### Selective Audio Recording (Submix)

Record only specific audio sources:

```cpp
// Assume you have a submix for music only
USoundSubmix* MusicSubmix = LoadObject<USoundSubmix>(nullptr, TEXT("/Game/Audio/MusicSubmix"));

Recorder->StartRecording(
    "%auto%", 30, 1920, 1080,
    FRuntimeEncoderSettings(),
    true, true, false, false, -1.0f, false,
    MusicSubmix      // Only record this submix
);
```

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

---

## Performance Characteristics

### Memory Usage

| Feature | Memory Impact |
|---------|---------------|
| Basic recording | ~50-100 MB (frame buffers) |
| Circular buffer (30s @ 1080p60) | ~1-2 GB |
| Deferred encoding (1 min @ 1080p60) | ~3-5 GB |

### CPU Usage

- **Hardware encoding:** <5% CPU overhead
- **Software encoding (OpenH264):** 10-30% CPU overhead
- **Audio capture:** <2% CPU overhead

---

## Best Practices

1. **Use hardware acceleration** whenever possible (enabled by default)
2. **Test output path** before starting long recordings
3. **Monitor disk space** for long recordings (1 min @ 1080p60 @ 20Mbps ≈ 150 MB)
4. **Use circular buffer** for replay/highlight systems instead of continuous recording
5. **Profile in packaged builds** - editor performance may differ
6. **Clean up old recordings** - plugin doesn't auto-delete files

---

## Known Limitations

- Maximum circular buffer: 10 minutes (configurable)
- Frame-rate independent mode disables audio recording
- 360-degree recording is experimental (currently commented out)
- No support for UE4 (use UE5.0+)
- Render target recording doesn't support UI capture

---

## Changelog

### Version 2.0
- Added multi-camera grid recording
- Added frame-rate independent recording mode
- Added circular buffer support (last N seconds)
- Added deferred encoding option
- Performance improvements for hardware acceleration
- Android MediaStore integration

### Version 1.0
- Initial release
- Basic viewport, camera, and render target recording
- Hardware acceleration support (Windows, Android)
- Audio recording with submix selection

---

## Support & Resources

- **Support Discord:** [Join Community](https://discord.gg/wptvWkhtGm)
- **Marketplace Page:** [Get Plugin](com.epicgames.launcher://ue/Fab/product/a883f9ac-b253-487b-b5ab-d612b660e41b)
- **Developer:** [Unreal Solutions](https://unrealsolutions.com)
- **Creator:** Petr Leontev

### Getting Help

1. Check this README and troubleshooting section
2. Search existing discussions on Discord
3. Post detailed questions with:
   - Unreal Engine version
   - Platform (Windows/Mac/Linux/Android)
   - Code snippet or Blueprint screenshot
   - Error messages or logs

---

## License

MIT License - Copyright (c) 2023-2025 Unreal Solutions

See [LICENSE](LICENSE) file for full details.

---

## Credits

**Third-Party Libraries:**
- **OpenH264** - Cisco Systems (BSD-2-Clause)
- **Windows Media Foundation** - Microsoft
- **AVFoundation** - Apple Inc.
- **MediaCodec** - Google/Android

---

<div align="center">

**Made with ❤️ for the Unreal Engine Community**

⭐ Star this plugin if you find it useful!

</div>

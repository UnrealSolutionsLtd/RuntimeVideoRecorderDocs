# GitHub Workflows for Building MP4v2

This directory contains GitHub Actions workflows for building the MP4v2 library for various platforms.

## Available Workflows

### 1. `build-mp4v2-linux-arm64.yml`
**Purpose:** Build MP4v2 for Linux ARM64 using Autotools

**Features:**
- Uses QEMU for ARM64 emulation on x86_64 runners
- Builds with GNU Autotools (./configure && make)
- Automatically commits built libraries to the repository
- Uploads artifacts for download

**Triggers:**
- Manual workflow dispatch only

**Output:**
- Static library: `Linux/arm64/libmp4v2.a`
- Shared libraries: `Linux/arm64/libmp4v2.so*` (if built)
- Headers: `include/mp4v2/`

**Usage:**
```bash
# Triggered automatically on push, or run manually:
# 1. Go to Actions tab
# 2. Select "Build MP4v2 for Linux ARM64"
# 3. Click "Run workflow"
```

---

### 2. `build-mp4v2-cmake.yml`
**Purpose:** Build MP4v2 for multiple platforms using CMake

**Features:**
- Supports Linux ARM64 and Linux x64
- Uses CMake build system for more flexibility
- Configurable platform selection via workflow inputs
- Longer artifact retention (90 days)
- Separate jobs for each platform

**Triggers:**
- Manual workflow dispatch with platform selection

**Output:**
- Linux ARM64: `Linux/arm64/libmp4v2.a`
- Linux x64: `Linux/x64/libmp4v2.a` (manual trigger only)
- Headers: `include/mp4v2/`

**Usage:**
```bash
# Manual trigger with options:
# 1. Go to Actions tab
# 2. Select "Build MP4v2 (CMake) for Multiple Platforms"
# 3. Click "Run workflow"
# 4. Enter platforms: "linux-arm64", "linux-x64", or "all"
```

---

## Workflow Comparison

| Feature | build-mp4v2-linux-arm64.yml | build-mp4v2-cmake.yml |
|---------|-----------------------------|-----------------------|
| Build System | Autotools | CMake |
| Platforms | Linux ARM64 | Linux ARM64, x64 |
| Auto Commit | Yes | No |
| Artifact Retention | 30 days | 90 days |
| Platform Selection | No | Yes (workflow_dispatch) |
| Trigger | Manual only | Manual only |

---

## Common Tasks

### Building for Linux ARM64
Both workflows can build for Linux ARM64. The differences:
- **Autotools workflow**: More traditional Unix build approach, auto-commits
- **CMake workflow**: Modern build system, more control, no auto-commit

### Downloading Artifacts
After a workflow completes:
1. Go to the workflow run page
2. Scroll to the **Artifacts** section
3. Download the ZIP file (e.g., `mp4v2-linux-arm64.zip`)
4. Extract and copy files to your project

### Modifying Build Options

**For Autotools workflow**, edit the `./configure` line:
```bash
./configure --prefix=/build/install --enable-static --disable-shared
```

**For CMake workflow**, edit the `cmake` command:
```bash
cmake .. \
  -DCMAKE_BUILD_TYPE=Release \
  -DBUILD_SHARED_LIBS=OFF \
  -DCMAKE_CXX_FLAGS="-O3"
```

---

## Technical Details

### QEMU Setup
Both workflows use QEMU for ARM64 emulation:
```yaml
- name: Set up QEMU
  uses: docker/setup-qemu-action@v3
  with:
    platforms: arm64
```

This allows building ARM64 binaries on x86_64 GitHub runners.

### Docker Container
ARM64 builds run in an `arm64v8/ubuntu:22.04` container:
```bash
docker run --rm --platform linux/arm64 \
  -v ${{ github.workspace }}:/workspace \
  arm64v8/ubuntu:22.04 bash -c "..."
```

### Verification
After building, workflows verify the architecture:
```bash
file Linux/arm64/libmp4v2.a
# Output: Linux/arm64/libmp4v2.a: current ar archive
```

---

## Troubleshooting

### Build Fails with "No such file or directory"
- Check that the git clone succeeded
- Verify the repository URL is correct
- Ensure network connectivity during workflow run

### Wrong Architecture Built
- Verify `--platform linux/arm64` is set in docker run command
- Check QEMU setup completed successfully
- Run `file` command on the artifact to verify architecture

### Artifact Not Found
- Check the output path matches the artifact path in the workflow
- Verify the build completed successfully
- Check that `make install` or copy commands succeeded

### Auto-commit Fails
- Ensure the workflow has write permissions to the repository
- Check that git config is set correctly
- Verify `[skip ci]` is in the commit message to prevent infinite loops

---

## Adding New Platforms

To add support for a new platform (e.g., Windows ARM64):

1. Create a new job in the workflow:
```yaml
build-windows-arm64:
  runs-on: windows-latest
  steps:
    - name: Checkout
      uses: actions/checkout@v4
    # Add Windows-specific build steps
```

2. Use appropriate tools for that platform:
   - Windows: Visual Studio, vcpkg
   - macOS: Xcode command line tools
   - Android: NDK

3. Update artifact paths to match your repository structure

---

## Resources

- **MP4v2 Repository:** https://github.com/enzo1982/mp4v2
- **GitHub Actions Documentation:** https://docs.github.com/en/actions
- **Docker QEMU Setup:** https://github.com/docker/setup-qemu-action
- **Upload Artifacts:** https://github.com/actions/upload-artifact


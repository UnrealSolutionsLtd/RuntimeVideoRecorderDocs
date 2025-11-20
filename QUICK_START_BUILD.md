# Quick Start: Building MP4v2 for Linux ARM64

This is a quick reference guide. For detailed documentation, see [BUILD_LIBRARIES.md](BUILD_LIBRARIES.md).

## 🚀 Fastest Method: GitHub Actions

1. **Go to:** `Actions` tab in GitHub
2. **Select:** "Build MP4v2 (CMake) for Multiple Platforms"
3. **Click:** "Run workflow" → Enter `linux-arm64` → Run
4. **Wait:** ~5-10 minutes
5. **Download:** Artifact from completed workflow

**Done!** ✅

---

## 💻 Local Build: Using Docker (Recommended)

Copy and paste this single command:

```bash
docker run --rm --platform linux/arm64 \
  -v $(pwd):/workspace \
  arm64v8/ubuntu:22.04 \
  bash -c "
    apt-get update && apt-get install -y build-essential cmake git file && \
    git clone --depth 1 https://github.com/enzo1982/mp4v2.git && \
    cd mp4v2 && mkdir build && cd build && \
    cmake .. -DCMAKE_BUILD_TYPE=Release -DBUILD_SHARED_LIBS=ON -DCMAKE_INSTALL_PREFIX=/tmp/install && \
    make -j\$(nproc) && make install && \
    mkdir -p /workspace/Linux/arm64 && \
    cp -P /tmp/install/lib/libmp4v2.so* /workspace/Linux/arm64/
  "
```

**Output:** `Linux/arm64/libmp4v2.so*` (shared libraries with version symlinks)

---

## 🐧 Local Build: On ARM64 Linux Machine

If you're already on ARM64 Linux (Raspberry Pi, ARM server, etc.):

```bash
# Install tools
sudo apt-get update
sudo apt-get install -y build-essential cmake git

# Clone and build
git clone https://github.com/enzo1982/mp4v2.git
cd mp4v2
mkdir build && cd build
cmake .. -DCMAKE_BUILD_TYPE=Release -DBUILD_SHARED_LIBS=ON -DCMAKE_INSTALL_PREFIX=$PWD/install
make -j$(nproc)
make install

# Output: install/lib/libmp4v2.so* (shared libraries)
```

---

## 🍎 macOS: Universal Binary

Build for both Apple Silicon and Intel:

```bash
git clone https://github.com/enzo1982/mp4v2.git
cd mp4v2
mkdir build && cd build
cmake .. \
  -DCMAKE_BUILD_TYPE=Release \
  -DBUILD_SHARED_LIBS=ON \
  -DCMAKE_OSX_ARCHITECTURES="arm64;x86_64" \
  -DCMAKE_INSTALL_PREFIX=$PWD/install
make -j$(sysctl -n hw.ncpu)
make install

# Output: install/lib/libmp4v2.dylib (universal shared library)
```

---

## 🪟 Windows: Visual Studio

```powershell
git clone https://github.com/enzo1982/mp4v2.git
cd mp4v2
mkdir build
cd build
cmake .. -G "Visual Studio 17 2022" -A x64 -DBUILD_SHARED_LIBS=ON -DCMAKE_INSTALL_PREFIX=install
cmake --build . --config Release
cmake --install . --config Release

# Output: install\bin\mp4v2.dll and install\lib\mp4v2.lib
```

---

## ✅ Verify Build

```bash
# Check architecture
file Linux/arm64/libmp4v2.so*
# Expected: "ELF 64-bit LSB shared object, ARM aarch64"

# Check size and symlinks
ls -lh Linux/arm64/
# Expected: libmp4v2.so.2.1.3, libmp4v2.so.2 -> libmp4v2.so.2.1.3, libmp4v2.so -> libmp4v2.so.2

# Check dependencies
readelf -d Linux/arm64/libmp4v2.so.2.1.3 | grep NEEDED
# Shows required system libraries

# Check symbols
nm -D Linux/arm64/libmp4v2.so.2.1.3 | grep MP4Create
```

---

## 📦 Available Workflows

| Workflow | Best For | Speed |
|----------|----------|-------|
| `build-mp4v2-cmake.yml` | Most users, flexible | Fast (QEMU) |
| `build-mp4v2-linux-arm64.yml` | Autotools, auto-commit | Fast (QEMU) |
| `build-mp4v2-native-arm64.yml` | Enterprise users | Very Fast (Native) |

---

## 🆘 Quick Troubleshooting

**"docker: not found"**
```bash
# Install Docker
curl -fsSL https://get.docker.com | sh
```

**"permission denied" (Docker)**
```bash
sudo usermod -aG docker $USER
# Log out and back in
```

**"exec format error"**
```bash
# Enable QEMU
docker run --rm --privileged multiarch/qemu-user-static --reset -p yes
```

**Wrong architecture built**
```bash
# Verify with:
file libmp4v2.a
# Should contain "aarch64" or "ARM" for ARM64
```

---

## 📁 File Locations

After building, place files here:

```
YourProject/
├── Linux/
│   └── arm64/
│       ├── libmp4v2.so.2.1.3   ← Actual library
│       ├── libmp4v2.so.2       ← Symlink to .2.1.3
│       └── libmp4v2.so         ← Symlink to .2
└── include/
    └── mp4v2/
        └── *.h                 ← Headers here
```

**Note:** Make sure to preserve symlinks when copying (use `cp -P` or `cp -a`)

---

## 📚 More Information

- **Detailed Guide:** [BUILD_LIBRARIES.md](BUILD_LIBRARIES.md)
- **Workflow Docs:** [.github/workflows/README.md](.github/workflows/README.md)
- **MP4v2 Project:** https://github.com/enzo1982/mp4v2

---

## ⏱️ Estimated Build Times

| Method | Time |
|--------|------|
| GitHub Actions (QEMU) | 5-10 min |
| Local Docker (QEMU) | 3-8 min |
| Native ARM64 Hardware | 1-3 min |
| GitHub Native ARM64 | 1-2 min |

---

**Need help?** See the [Troubleshooting](BUILD_LIBRARIES.md#troubleshooting) section in the full documentation.


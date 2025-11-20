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
    apt-get update && apt-get install -y build-essential cmake git && \
    git clone --depth 1 https://github.com/enzo1982/mp4v2.git && \
    cd mp4v2 && mkdir build && cd build && \
    cmake .. -DCMAKE_BUILD_TYPE=Release -DBUILD_SHARED_LIBS=OFF && \
    make -j\$(nproc) && \
    mkdir -p /workspace/Linux/arm64 && \
    find . -name 'libmp4v2.a' -exec cp {} /workspace/Linux/arm64/ \;
  "
```

**Output:** `Linux/arm64/libmp4v2.a`

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
cmake .. -DCMAKE_BUILD_TYPE=Release -DBUILD_SHARED_LIBS=OFF
make -j$(nproc)

# Output: libmp4v2.a in current directory
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
  -DBUILD_SHARED_LIBS=OFF \
  -DCMAKE_OSX_ARCHITECTURES="arm64;x86_64"
make -j$(sysctl -n hw.ncpu)
```

---

## 🪟 Windows: Visual Studio

```powershell
git clone https://github.com/enzo1982/mp4v2.git
cd mp4v2
mkdir build
cd build
cmake .. -G "Visual Studio 17 2022" -A x64 -DBUILD_SHARED_LIBS=OFF
cmake --build . --config Release
```

---

## ✅ Verify Build

```bash
# Check architecture
file Linux/arm64/libmp4v2.a
# Expected: "current ar archive" (ARM aarch64)

# Check size
ls -lh Linux/arm64/libmp4v2.a
# Expected: ~2-4 MB

# List contents
ar -t Linux/arm64/libmp4v2.a | head
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
│       └── libmp4v2.a          ← Here
└── include/
    └── mp4v2/
        └── *.h                 ← Headers here
```

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


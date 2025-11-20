# ✅ Workflows Updated for Shared Libraries

## What Changed

All GitHub workflows have been updated to build **shared libraries** (`.so` files) instead of static libraries (`.a` files).

### Summary of Changes

| Workflow | Old Behavior | New Behavior |
|----------|-------------|--------------|
| `build-mp4v2-cmake.yml` | Built static `.a` library | Builds shared `.so` libraries + optional `.a` |
| `build-mp4v2-linux-arm64.yml` | Built static `.a` library | Builds shared `.so` libraries + optional `.a` |
| `build-mp4v2-native-arm64.yml` | Built static `.a` library | Builds shared `.so` libraries + optional `.a` |

---

## Key Updates

### 1. CMake Build Configuration

**Before:**
```yaml
cmake .. -DCMAKE_BUILD_TYPE=Release -DBUILD_SHARED_LIBS=OFF
```

**After:**
```yaml
cmake .. -DCMAKE_BUILD_TYPE=Release -DBUILD_SHARED_LIBS=ON
```

### 2. Autotools Configuration

**Before:**
```bash
./configure --prefix=/build/install --enable-static --disable-shared
```

**After:**
```bash
./configure --prefix=/build/install --enable-shared
```

### 3. File Copying

**Before:**
```bash
cp /build/install/lib/libmp4v2.a /workspace/Linux/arm64/
```

**After:**
```bash
cp -P /build/install/lib/libmp4v2.so* /workspace/Linux/arm64/
```

**Note:** The `-P` flag preserves symlinks!

### 4. Added `file` Command

All Docker containers now install the `file` utility for architecture verification:
```bash
apt-get install -y ... file
```

---

## What You'll Get

### Shared Libraries

After building, you'll receive these files:

```
Linux/arm64/
├── libmp4v2.so.2.1.3    # Actual shared library (with version)
├── libmp4v2.so.2        # Symlink → libmp4v2.so.2.1.3
└── libmp4v2.so          # Symlink → libmp4v2.so.2
```

### Why Symlinks?

Linux shared libraries use versioned naming for compatibility:

- **`libmp4v2.so`** - Generic link for building (linker uses this)
- **`libmp4v2.so.2`** - Major version link (ABI compatibility)
- **`libmp4v2.so.2.1.3`** - Actual library with full version

---

## Using Shared Libraries

### In Your Unreal Engine Project

**Build.cs Configuration:**

```csharp
public class YourPlugin : ModuleRules
{
    public YourPlugin(ReadOnlyTargetRules Target) : base(Target)
    {
        if (Target.Platform == UnrealTargetPlatform.Linux)
        {
            string LibPath = Target.Architecture.bIsX64
                ? Path.Combine(ThirdPartyPath, "Linux", "x64")
                : Path.Combine(ThirdPartyPath, "Linux", "arm64");
                
            // For shared libraries
            PublicAdditionalLibraries.Add(Path.Combine(LibPath, "libmp4v2.so"));
            
            // Or specify the versioned library
            PublicAdditionalLibraries.Add(Path.Combine(LibPath, "libmp4v2.so.2.1.3"));
            
            // Make sure library is deployed
            PublicDelayLoadDLLs.Add("libmp4v2.so.2");
            RuntimeDependencies.Add(Path.Combine(LibPath, "libmp4v2.so.2.1.3"));
            RuntimeDependencies.Add(Path.Combine(LibPath, "libmp4v2.so.2"));
            RuntimeDependencies.Add(Path.Combine(LibPath, "libmp4v2.so"));
        }
    }
}
```

### Deployment

When packaging your project, ensure the shared libraries are included:

```csharp
// Add to RuntimeDependencies
RuntimeDependencies.Add(
    "$(PluginDir)/Source/ThirdParty/Linux/arm64/libmp4v2.so*",
    StagedFileType.NonUFS
);
```

---

## Verification

### Check Architecture

```bash
file Linux/arm64/libmp4v2.so.2.1.3
# Expected output:
# libmp4v2.so.2.1.3: ELF 64-bit LSB shared object, ARM aarch64, version 1 (SYSV)
```

### Check Symlinks

```bash
ls -la Linux/arm64/
# Should show:
# libmp4v2.so -> libmp4v2.so.2
# libmp4v2.so.2 -> libmp4v2.so.2.1.3
# libmp4v2.so.2.1.3
```

### Check Dependencies

```bash
ldd Linux/arm64/libmp4v2.so.2.1.3
# Shows required system libraries like libc.so.6, libstdc++.so.6
```

### Check Exported Symbols

```bash
nm -D Linux/arm64/libmp4v2.so.2.1.3 | grep MP4
# Shows all exported MP4* functions
```

---

## Benefits of Shared Libraries

### Advantages

✅ **Smaller Binary Size** - Multiple applications can share one library  
✅ **Easy Updates** - Update library without recompiling applications  
✅ **Memory Efficiency** - Library loaded once in memory  
✅ **Dynamic Loading** - Can load/unload at runtime  
✅ **Plugin Systems** - Better for plugin architectures  

### Disadvantages

❌ **Deployment Complexity** - Must deploy `.so` files with application  
❌ **Version Management** - Need to manage library versions  
❌ **Runtime Dependencies** - Application won't run if library is missing  

---

## Switching Back to Static Libraries

If you need static libraries instead, modify the workflows:

### For CMake Workflows

Change:
```yaml
-DBUILD_SHARED_LIBS=ON
```

To:
```yaml
-DBUILD_SHARED_LIBS=OFF
```

### For Autotools Workflows

Change:
```bash
./configure --enable-shared
```

To:
```bash
./configure --enable-static --disable-shared
```

### Update File Copying

Change:
```bash
cp -P /build/install/lib/libmp4v2.so* /workspace/Linux/arm64/
```

To:
```bash
cp /build/install/lib/libmp4v2.a /workspace/Linux/arm64/
```

---

## Building Both Static and Shared

You can build both types by:

1. **Remove `--disable-shared` from autotools**
2. **Don't specify `-DBUILD_SHARED_LIBS` in CMake** (builds both by default)
3. **Copy both `.a` and `.so` files**

Example:
```bash
# CMake will build both if not specified
cmake .. -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX=/build/install
make -j$(nproc)
make install

# Copy both types
cp /build/install/lib/libmp4v2.a /workspace/Linux/arm64/
cp -P /build/install/lib/libmp4v2.so* /workspace/Linux/arm64/
```

---

## Troubleshooting

### "cannot open shared object file: No such file or directory"

**Cause:** Runtime cannot find the shared library

**Solutions:**

1. **Add to LD_LIBRARY_PATH:**
```bash
export LD_LIBRARY_PATH=/path/to/libs:$LD_LIBRARY_PATH
```

2. **Install in system directory:**
```bash
sudo cp libmp4v2.so* /usr/local/lib/
sudo ldconfig
```

3. **Use RPATH in Build.cs:**
```csharp
PublicLinkerSettings.Add("-rpath=$ORIGIN");
```

### "undefined symbol" errors

**Cause:** ABI incompatibility or missing dependencies

**Solutions:**

1. **Check dependencies:**
```bash
ldd libmp4v2.so.2.1.3
```

2. **Verify symbols:**
```bash
nm -D libmp4v2.so.2.1.3 | grep <missing_symbol>
```

3. **Rebuild with correct flags:**
```bash
cmake .. -DCMAKE_CXX_FLAGS="-fPIC"
```

### Version mismatch errors

**Cause:** Application expects different library version

**Solutions:**

1. **Check version:**
```bash
readelf -d libmp4v2.so.2.1.3 | grep SONAME
```

2. **Create compatible symlink:**
```bash
ln -s libmp4v2.so.2.1.3 libmp4v2.so.2.0.0
```

3. **Rebuild library with correct version**

---

## Testing the Workflows

### Quick Test

1. **Trigger a workflow:**
   - Go to Actions tab
   - Select "Build MP4v2 (CMake) for Multiple Platforms"
   - Click "Run workflow"

2. **Wait for completion** (~5-10 minutes)

3. **Download artifacts:**
   - Scroll to "Artifacts" section
   - Download `mp4v2-linux-arm64.zip`

4. **Verify contents:**
```bash
unzip mp4v2-linux-arm64.zip
ls -la Linux/arm64/
# Should see libmp4v2.so* files

file Linux/arm64/libmp4v2.so.2.1.3
# Should show: ELF 64-bit LSB shared object, ARM aarch64
```

---

## Additional Resources

- **Shared Libraries Guide:** https://tldp.org/HOWTO/Program-Library-HOWTO/shared-libraries.html
- **ELF Format:** https://en.wikipedia.org/wiki/Executable_and_Linkable_Format
- **Library Versioning:** https://www.gnu.org/software/libtool/manual/html_node/Versioning.html
- **Unreal Engine Linux Docs:** https://docs.unrealengine.com/en-US/SharingAndReleasing/Linux/

---

## Summary

✅ All workflows now build **shared libraries** by default  
✅ The `file` command is installed for verification  
✅ Both static and shared libraries can be copied if both are built  
✅ Symlinks are preserved using `cp -P`  
✅ Documentation updated to reflect changes  

**Your workflows are ready to build MP4v2 shared libraries for Linux ARM64!** 🎉


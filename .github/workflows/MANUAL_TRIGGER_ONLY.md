# ✅ Workflows Set to Manual Trigger Only

## What Changed

All GitHub workflows have been updated to run **manually only** instead of automatically triggering on push/pull requests.

---

## Why This Change?

**Problem:** Two workflows (`build-mp4v2-cmake.yml` and `build-mp4v2-linux-arm64.yml`) were both configured to run automatically on:
- Push to `main` or `master` branches
- Pull requests

This caused **both workflows to run simultaneously**, wasting GitHub Actions minutes and potentially causing conflicts.

**Solution:** Changed all workflows to **manual trigger only** using `workflow_dispatch`.

---

## Updated Workflows

### Before

```yaml
on:
  push:
    branches: [ main, master ]
  pull_request:
    branches: [ main, master ]
  workflow_dispatch:
```

### After

```yaml
on:
  workflow_dispatch:
```

---

## Affected Workflows

| Workflow | Status |
|----------|--------|
| `build-mp4v2-cmake.yml` | ✅ Manual trigger only |
| `build-mp4v2-linux-arm64.yml` | ✅ Manual trigger only |
| `build-mp4v2-native-arm64.yml` | ✅ Already was manual only |

---

## How to Run Workflows Now

### Step-by-Step

1. **Navigate to your GitHub repository**
2. **Click on the "Actions" tab**
3. **Select a workflow from the left sidebar:**
   - Build MP4v2 (CMake) for Multiple Platforms
   - Build MP4v2 for Linux ARM64 (Autotools)
   - Build MP4v2 (Native ARM64 Runners)
4. **Click the "Run workflow" button** (top right)
5. **Configure options** (if available):
   - For CMake workflow: Select platform (`linux-arm64`, `linux-x64`, `all`)
   - For Native ARM64: Choose whether to commit results
6. **Click "Run workflow"** to start the build

### Visual Guide

```
GitHub Repo → Actions Tab → Select Workflow → Run workflow button → Configure → Run
```

---

## Benefits of Manual Trigger

### ✅ Advantages

1. **No Wasted Resources**
   - Workflows run only when you need them
   - Saves GitHub Actions minutes
   - Reduces unnecessary builds

2. **No Conflicts**
   - Only one workflow runs at a time (unless you start multiple manually)
   - Prevents race conditions when committing artifacts

3. **Better Control**
   - You decide when to build
   - Can test changes before triggering builds
   - Useful for rate-limited resources

4. **Cleaner Git History**
   - No automatic commits from CI on every push
   - Better control over when artifacts are updated

---

## When to Use Each Workflow

### 🎯 Recommended: `build-mp4v2-cmake.yml`

**Best for:** Most users, flexible platform selection

**Features:**
- Build for Linux ARM64, x64, or both
- CMake build system (modern, widely supported)
- 90-day artifact retention
- No automatic commits (cleaner git history)

**Use when:**
- You need to build for multiple platforms
- You want a modern CMake-based build
- You prefer downloading artifacts over auto-commits

---

### 🔧 Alternative: `build-mp4v2-linux-arm64.yml`

**Best for:** Users who prefer Autotools, want auto-commits

**Features:**
- GNU Autotools build (traditional Unix approach)
- Automatically commits results to repository
- 30-day artifact retention

**Use when:**
- You prefer Autotools over CMake
- You want artifacts committed automatically
- You're familiar with traditional Unix builds

---

### ⚡ Advanced: `build-mp4v2-native-arm64.yml`

**Best for:** GitHub Team/Enterprise users with ARM64 runners

**Features:**
- Uses native ARM64 runners (fastest)
- Falls back to QEMU if native unavailable
- Optional commit functionality
- Builds with both Autotools and CMake

**Use when:**
- You have access to native ARM64 runners
- You need the fastest build times
- You want to compare Autotools vs CMake builds

---

## FAQ

### Q: Will workflows run automatically anymore?

**A:** No. All workflows now require manual trigger via the Actions tab.

### Q: Can I make them automatic again?

**A:** Yes, add back the triggers:

```yaml
on:
  push:
    branches: [ main, master ]
  workflow_dispatch:
```

But be aware this will cause multiple workflows to run on each push.

### Q: How do I schedule builds?

**A:** Add a cron schedule:

```yaml
on:
  schedule:
    - cron: '0 0 * * 0'  # Weekly on Sunday at midnight
  workflow_dispatch:
```

### Q: Can I trigger workflows via API?

**A:** Yes, using GitHub's REST API:

```bash
curl -X POST \
  -H "Authorization: token YOUR_TOKEN" \
  -H "Accept: application/vnd.github.v3+json" \
  https://api.github.com/repos/OWNER/REPO/actions/workflows/WORKFLOW_ID/dispatches \
  -d '{"ref":"main","inputs":{"platforms":"linux-arm64"}}'
```

### Q: What if I forget to run a workflow?

**A:** No problem! The workflows don't need to run regularly. Only run them when:
- You need fresh builds
- MP4v2 library is updated
- You're setting up a new platform

---

## Troubleshooting

### "Run workflow" button is grayed out

**Cause:** You don't have write permissions to the repository

**Solution:**
- Make sure you're logged in to GitHub
- Check that you have write access to the repository
- Try refreshing the page

### Can't find the workflow in Actions tab

**Cause:** Workflow file has syntax errors or is not in `.github/workflows/`

**Solution:**
```bash
# Verify file location
ls -la .github/workflows/*.yml

# Validate YAML syntax
# Use https://www.yamllint.com/
```

### Workflow doesn't start after clicking "Run workflow"

**Cause:** GitHub Actions might be delayed or rate-limited

**Solution:**
- Wait a few seconds and refresh the page
- Check GitHub Actions status: https://www.githubstatus.com/
- Verify your repository has Actions enabled in Settings

---

## Comparison: Auto vs Manual

| Feature | Automatic Trigger | Manual Trigger |
|---------|------------------|----------------|
| Convenience | ✅ Runs automatically | ❌ Requires manual action |
| Control | ❌ Can't prevent runs | ✅ Full control |
| Resource Usage | ❌ May waste minutes | ✅ Efficient |
| Git History | ❌ Many auto-commits | ✅ Clean history |
| Testing | ❌ Runs on every push | ✅ Run when ready |
| CI/CD | ✅ Good for testing | ❌ Not for automated testing |

---

## Best Practices

### 1. Run Workflows Periodically

Even though they're manual, run them occasionally to:
- Keep libraries up to date
- Test that workflows still work
- Update to new library versions

### 2. Document When You Run Them

Add a note when you manually build:
```bash
git commit -m "Update MP4v2 Linux ARM64 libraries to v2.1.3"
```

### 3. Version Your Artifacts

Consider adding version info to artifact names:
```yaml
- name: Upload artifacts
  uses: actions/upload-artifact@v4
  with:
    name: mp4v2-linux-arm64-v2.1.3-${{ github.run_number }}
```

### 4. Test Before Production

Download artifacts and test them locally before deploying to production:
```bash
# Download artifact
unzip mp4v2-linux-arm64.zip

# Test
file Linux/arm64/libmp4v2.so.2.1.3
ldd Linux/arm64/libmp4v2.so.2.1.3

# If good, commit to production branch
```

---

## Reverting to Automatic

If you need automatic builds again, choose **one** workflow to run automatically:

### Option 1: Auto-run CMake workflow only

```yaml
# .github/workflows/build-mp4v2-cmake.yml
on:
  push:
    branches: [ main ]
  workflow_dispatch:
    inputs:
      platforms:
        description: 'Platforms to build'
        default: 'linux-arm64'
```

### Option 2: Run on release tags only

```yaml
on:
  push:
    tags:
      - 'v*'  # Runs on version tags like v1.0.0
  workflow_dispatch:
```

### Option 3: Run on specific paths only

```yaml
on:
  push:
    branches: [ main ]
    paths:
      - '.github/workflows/build-mp4v2-*.yml'
      - 'build_scripts/**'
  workflow_dispatch:
```

---

## Summary

✅ All workflows now require **manual trigger**  
✅ No more simultaneous workflow runs  
✅ Better control over when builds happen  
✅ Saves GitHub Actions minutes  
✅ Cleaner git history with fewer auto-commits  
✅ Documentation updated across all files  

**To build libraries: Go to Actions → Select workflow → Run workflow** 🚀

---

## Related Documentation

- [.github/workflows/README.md](README.md) - Workflow details
- [SETUP_COMPLETE.md](SETUP_COMPLETE.md) - Setup guide
- [../../BUILD_LIBRARIES.md](../../BUILD_LIBRARIES.md) - Building instructions
- [../../QUICK_START_BUILD.md](../../QUICK_START_BUILD.md) - Quick reference


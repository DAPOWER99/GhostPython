
# Building Ghost Python from Source

This document describes how to build the **Ghost Python** portable distribution from official Python 3.14 source code on Windows.

> **Prerequisite knowledge**: Familiarity with Windows command line, Visual Studio, and Python's build system.

---

## Build Requirements

| Tool | Version | Notes |
|------|---------|-------|
| **Windows SDK** | 10.0.20348.0 or later | Includes Windows headers and libraries |
| **Visual Studio** | 2022 (17.8+) | Desktop development with C++ workload |
| **Python source** | 3.14.x | Official tarball from python.org |
| **Tcl/Tk** | 8.6.13 | Bundled for tkinter support |
| **Git** | Latest | For cloning and patch management |
| **7-Zip** | Latest | Optional: compression for release archives |

---

## Directory Structure (Build Environment)

```
C:\build\
├── python-src\              # Python 3.14 source code
├── ghost-patches\           # Custom patches for Ghost Python
├── tcltk\                   # Precompiled Tcl/Tk 8.6 binaries
├── ghost-output\            # Final portable distribution
└── tools\                   # Build scripts
```

---

## Step 1: Obtain Python Source Code

```cmd
cd C:\build
curl -L -o python-3.14.0.tar.xz https://www.python.org/ftp/python/3.14.0/Python-3.14.0.tar.xz
tar -xf python-3.14.0.tar.xz
ren Python-3.14.0 python-src
```

Or clone from the official CPython repository:

```cmd
git clone --branch 3.14 https://github.com/python/cpython.git python-src
```

---

## Step 2: Apply Ghost Python Patches

Ghost Python requires several modifications to the standard Python build:

| Patch | Purpose |
|-------|---------|
| `no-registry.patch` | Disables all Windows Registry reads/writes |
| `relative-paths.patch` | Forces relative path resolution (no PYTHONHOME) |
| `silent-banner.patch` | Adds interactive/script detection logic |
| `sitecustomize-inject.patch` | Injects custom commands via builtins |
| `settings-config.patch` | Adds settings.config loader |

Apply patches:

```cmd
cd C:\build\python-src
git apply ..\ghost-patches\*.patch
```

> **Note**: Patch files are available in the `ghost-patches/` directory of the Ghost Python repository.

---

## Step 3: Prepare Tcl/Tk Binaries

Ghost Python ships with surgically aligned Tcl/Tk 8.6.13 to eliminate `tk.tcl` errors.

### Option A: Download Precompiled Binaries (Recommended)

```cmd
cd C:\build
curl -L -o tcltk.zip https://github.com/DAPOWER99/tcltk-bundled/releases/download/8.6.13/tcltk-win64.zip
tar -xf tcltk.zip
ren tcltk-win64 tcltk
```

### Option B: Build Tcl/Tk from Source (Advanced)

```cmd
cd C:\build
git clone https://github.com/tcltk/tcl.git
git clone https://github.com/tcltk/tk.git
cd tcl\win
nmake -f makefile.vc
cd ..\..\tk\win
nmake -f makefile.vc TCLDIR=C:\build\tcl
```

The resulting DLLs and `tcl`/`tk` folders should be placed in `C:\build\tcltk\`.

---

## Step 4: Configure the Build

Ghost Python uses a custom `PCbuild/profile.props` to control feature toggles.

Create `C:\build\python-src\PCbuild\ghost.props`:

```xml
<?xml version="1.0" encoding="utf-8"?>
<Project ToolsVersion="4.0" xmlns="http://schemas.microsoft.com/developer/msbuild/2003">
  <PropertyGroup>
    <!-- Disable registry usage -->
    <DefineConstants>$(DefineConstants);NO_REGISTRY</DefineConstants>

    <!-- Force relative DLL loading -->
    <DefineConstants>$(DefineConstants);PYTHONHOME_RELATIVE</DefineConstants>

    <!-- Custom sitecustomize path -->
    <SiteCustomizePath>$(PySourcePath)Ghost\sitecustomize.py</SiteCustomizePath>
  </PropertyGroup>

  <!-- Tcl/Tk paths -->
  <PropertyGroup Condition="'$(Platform)' == 'x64'">
    <TclRoot>C:\build\tcltk</TclRoot>
    <TkRoot>C:\build\tcltk</TkRoot>
  </PropertyGroup>
</Project>
```

Run the configuration script:

```cmd
cd C:\build\python-src\PCbuild
..\configure.bat --enable-shared --disable-posix-shm --with-tcltk=C:\build\tcltk
```

---

## Step 5: Compile Python

### Build the core interpreter and DLL:

```cmd
cd C:\build\python-src\PCbuild
msbuild python.vcxproj /p:Configuration=Release /p:Platform=x64 /p:UseGhostProps=true
```

### Build additional modules (tkinter, etc.):

```cmd
msbuild _tkinter.vcxproj /p:Configuration=Release /p:Platform=x64
msbuild _ssl.vcxproj /p:Configuration=Release /p:Platform=x64
msbuild _hashlib.vcxproj /p:Configuration=Release /p:Platform=x64
```

### Build the full distribution:

```cmd
msbuild build.proj /p:Configuration=Release /p:Platform=x64 /t:Build
```

Expected outputs (Release/x64):
- `python.exe`
- `python3.dll`
- `python314.dll`
- `Lib/` (standard library)
- `DLLs/` (extension modules)

---

## Step 6: Create the Portable Distribution

Run the assembly script to collect, prune, and package Ghost Python:

```cmd
cd C:\build
python tools\assemble_ghost.py --source python-src\PCbuild\amd64 --output ghost-output
```

The `assemble_ghost.py` script performs the following:

1. Copies `python.exe`, DLLs, `Lib/`, `DLLs/`
2. Copies Tcl/Tk from `C:\build\tcltk\` into `tcl/`
3. Injects `sitecustomize.py` and `settings.config`
4. Prunes unnecessary files (`test/`, `__pycache__/`, `*.pyc`, `*.pdb`)
5. Optionally strips debug symbols

### Manual assembly (if script is unavailable):

```cmd
mkdir ghost-output
xcopy /E python-src\PCbuild\amd64 ghost-output\
xcopy /E tcltk ghost-output\tcl\
copy ghost-patches\sitecustomize.py ghost-output\
copy ghost-patches\settings.config ghost-output\
rmdir /S /Q ghost-output\Lib\test
rmdir /S /Q ghost-output\Lib\__pycache__
del /S /Q ghost-output\*.pdb
```

---

## Step 7: Test the Build

### Basic sanity check:

```cmd
cd ghost-output
.\python.exe --version
# Expected: Python 3.14.0

.\python.exe -c "import tkinter; print('tkinter OK')"
# Expected: tkinter OK (no Tcl errors)
```

### Interactive test (banner should appear):

```cmd
.\python.exe
# Expected: Ghost banner with custom commands
```

### Script test (banner must be silent):

```cmd
echo print("hello") > test.py
.\python.exe test.py
# Expected: hello (no banner)
```

### Portability test:

```cmd
move ghost-output C:\temp\ghost-test
cd C:\temp\ghost-test
.\python.exe -c "import sys; print(sys.prefix)"
# Expected: C:\temp\ghost-test (not a registry-derived path)
```

---

## Step 8: Package for Release

Create a compressed archive for distribution:

```cmd
cd C:\build
7z a -tzip GhostPython-3.14.0-win64.zip .\ghost-output\*
```

Optional: Generate SHA256 checksum:

```cmd
certutil -hashfile GhostPython-3.14.0-win64.zip SHA256 > checksum.txt
```

---

## Build Variants

| Variant | Configuration | Size (approx) | Use Case |
|---------|--------------|---------------|----------|
| **Full** | All features enabled | ~35 MB (zipped) | General purpose, USB drive |
| **Headless** | `include_tcl = no` | ~27 MB (zipped) | Server, CI runners |
| **Micro** | `include_tcl = no`, `include_tools = no` | ~24 MB (zipped) | Embedded, minimal environments |
| **Debug** | `Configuration=Debug` | ~120 MB | Development, troubleshooting |

To build a headless variant:

```cmd
# After assembly, edit settings.config:
echo [Features] > ghost-output\settings.config
echo include_Lib = yes >> ghost-output\settings.config
echo include_DLLs = yes >> ghost-output\settings.config
echo include_tcl = no >> ghost-output\settings.config
echo include_tools = no >> ghost-output\settings.config
```

---

## Troubleshooting

| Error | Likely Cause | Solution |
|-------|--------------|----------|
| `Unable to find vcvarsall.bat` | Visual Studio not installed | Install VS2022 with C++ workload |
| `tk.tcl: no such file or directory` | Tcl/Tk not found or wrong path | Verify `--with-tcltk` path in Step 4 |
| `DLL load failed: %1 is not a valid Win32 application` | Architecture mismatch | Ensure all builds use `x64` (or all use `x86`) |
| `Can't find sitecustomize.py` | Patch not applied | Re-apply `sitecustomize-inject.patch` |
| Registry writes still occurring | `NO_REGISTRY` define missing | Check `ghost.props` is loaded |

---

## Automating with GitHub Actions

For CI builds, use the following workflow snippet (`.github/workflows/build.yml`):

```yaml
name: Build Ghost Python

on:
  push:
    branches: [main]
  workflow_dispatch:

jobs:
  build:
    runs-on: windows-2022
    steps:
      - uses: actions/checkout@v4

      - name: Setup Visual Studio
        uses: ilammy/msvc-dev-cmd@v1

      - name: Download Python source
        run: |
          curl -L -o python-src.tar.xz https://www.python.org/ftp/python/3.14.0/Python-3.14.0.tar.xz
          tar -xf python-src.tar.xz
          move Python-3.14.0 python-src

      - name: Apply patches
        run: |
          cd python-src
          git apply ../patches/*.patch

      - name: Build
        run: |
          cd python-src\PCbuild
          .\build.bat -c Release -p x64

      - name: Assemble distribution
        run: python tools\assemble_ghost.py

      - name: Upload artifact
        uses: actions/upload-artifact@v4
        with:
          name: GhostPython
          path: ghost-output/
```

---

## References

- [CPython Developer Guide](https://devguide.python.org/)
- [Python on Windows documentation](https://docs.python.org/3/using/windows.html)
- [Tcl/Tk for Windows build instructions](https://www.tcl.tk/doc/howto/compile.html)
- [Ghost Python GitHub Repository](https://github.com/DAPOWER99/GhostPython)

---

**Maintainer:** @DAPOWER99  
**Last updated:** 2026-06-11

---

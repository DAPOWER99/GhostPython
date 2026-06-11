# GhostPython
(Ghost Python – A zero‑install, fully portable Python 3.14 distribution for Windows that never touches the Registry. Ships with surgically aligned Tcl/Tk libraries (no more tk.tcl errors), a smart silent/ interactive banner, and built‑in shell commands (tree, clean, libs, clear). Run from USB, CI runners, or embedded systems. Leave no trace.)


---

# 👻 Ghost Python

**High-performance · Ultra-minimal · Fully portable · Python 3.14 for Windows**

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Python 3.14](https://img.shields.io/badge/python-3.14-blue.svg)](https://www.python.org/)
[![Platform](https://img.shields.io/badge/platform-Windows%20%7C%20WinPE%20%7C%20USB-blue)](https://github.com/DAPOWER99)

Ghost Python is a surgically clean, zero‑footprint Python 3.14 distribution that runs entirely from a single folder.  
It never touches the Windows Registry, never writes to `System32`, and eliminates the infamous *Tcl/Tk `tk.tcl` not found* errors once and for all.

> Perfect for USB drives, CI runners, embedded systems, and developers who want a **ghost** of a setup — present when you need it, gone when you don't.

---

## Why Ghost Python?

| Problem | Ghost Solution |
|---------|----------------|
| Bloated installers, registry pollution | One folder. Copy → run → delete. No traces. |
| `_tkinter.TclError: Can't find a usable tk.tcl` | Binary‑aligned, pre‑linked Tcl/Tk libraries. |
| Cluttered terminal after `import` | Silent during scripts, custom banner only in interactive mode. |
| No custom shell shortcuts | Built‑in commands (`tree`, `clean`, `libs`, `clear`) in `builtins`. |
| Locked Python version | Ships with **Python 3.14** – fresh, minimal, portable. |

---

## Quick Start (60 seconds)

### 1. Download
Get the latest release from:  
👉 [https://github.com/DAPOWER99/GhostPython/releases](https://github.com/DAPOWER99/GhostPython/releases)

### 2. Extract
Unzip the archive to any location – for example:
```
C:\tools\GhostPython\
D:\USB\GhostPython\
```

### 3. Run
```cmd
cd C:\tools\GhostPython
.\python.exe
```

You will see the Ghost banner (only in interactive sessions):
```
👻 Ghost Python 3.14 ready.
Type 'help', 'tree', 'clean', 'libs', 'clear'
>>>
```

### 4. Use as a portable environment
```cmd
.\python.exe -m pip install requests
.\python.exe my_script.py
```

To remove Ghost Python completely — **delete the folder**. No uninstaller, no registry keys.

---

## Custom Commands Reference

Ghost Python injects several helper commands directly into the `builtins` namespace via `sitecustomize.py`.  
These work **only in the interactive REPL** and do not interfere with normal scripts.

| Command | Description | Example |
|---------|-------------|---------|
| `tree` | Display directory tree of current working directory | `tree` or `tree("C:\\projects")` |
| `clean` | Remove `__pycache__` and `.pyc` files recursively | `clean()` or `clean("src/")` |
| `libs` | List all installed pip packages with versions | `libs` |
| `clear` | Clear terminal screen (cross‑platform) | `clear` |

All commands support optional paths/arguments and integrate seamlessly with Python’s `help()` system.

---

## Architecture & Configuration

### `settings.config` — Toggle core modules on/off
Ghost Python uses a lightweight `settings.config` file in the root folder to enable/disable major components at runtime.
# feature of these can be disabled by changing the value to anything other than "enabled"
```ini
[Features]
Lib=enabled "libs important"

DLLs=enabled "Dlls"

tcl=enabled "Tcl folder"

Title=enabled "credits non important""


```

Disabling `include_tcl` removes tkinter entirely – useful for headless or server‑side deployments.

### Smart Banner Logic
- **Interactive session** → Shows the Ghost banner.
- **Script execution** (`python script.py`) → No banner, no output.
- **Piped input** → Automatically silent.

This guarantees zero noise in automation while keeping the REPL friendly.

### Portability Guarantees
- No `PYTHONPATH`, no `PATH` modifications.
- No `%APPDATA%` or `%LOCALAPPDATA%` writes.
- All relative paths resolved from the executable’s folder.

---

## Use Cases

| Scenario | Why Ghost Python fits |
|----------|----------------------|
| USB‑based development | Run Python from a flash drive on any Windows PC. |
| CI/CD pipelines (Windows) | No admin rights, no install step. |
| Teaching / workshops | Students download one folder – no environment issues. |
| Embedded / kiosk systems | Minimal disk footprint, toggle‑off features. |
| Legacy Tcl/Tk conflict resolution | Pre‑aligned libraries override system globals. |

---

## Project Structure (simplified)

```
GhostPython/
├── python.exe
├── python3.dll
├── settings.config
├── sitecustomize.py          # Custom commands + banner logic
├── Lib/                      # Standard library (toggle via config)
├── DLLs/                     # Core DLLs (toggle via config)
├── tcl/                      # Tcl/Tk runtime (toggle via config)
├── Scripts/                  # pip, wheel, etc.
└── Tools/                    # Optional (toggle via config)
```

---

## Building from source (for maintainers)

Ghost Python is built from an official Python 3.14 source tarball with:
- `--disable-posix-shm` (Windows portability)
- `--enable-shared` (DLL separation)
- Custom `_tkinter` compilation with bundled Tcl/Tk 8.6.

See [`BUILD.md`](BUILD.md) for the exact patchset and Windows SDK requirements.

---

## License

MIT License – use freely, commercially or otherwise.  
Attribution appreciated but not required.

---

## Contributing

Issues and PRs are welcome at:  
🔗 [https://github.com/DAPOWER99/GhostPython](https://github.com/DAPOWER99/GhostPython)

Focus areas:  
- Reducing DLL dependencies further  
- Adding `settings.config` toggles for more sub‑libraries  
- Supporting Windows on ARM (WoA)


---

## Acknowledgments

Built with respect for the Python Software Foundation, the Tcl/Tk community, and every depressed developer who has ever debugged `tk.tcl` at 2 AM.

**Ghost Python – leave no trace, solve real problems.**

---

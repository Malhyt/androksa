## Overview

`reportMissingModuleSource` is a diagnostic in Pylance and Pyright that warns when an imported module is found, but its source code cannot be located. This typically occurs with:

- **Compiled extensions** (C/C++ modules like `.so`, `.pyd`, `.dylib`)
- **Wheel distributions** built without source (`.pyc`-only packages)
- **Platform-specific binaries** that don't include `.py` source files

**This warning is usually safe to ignore** for compiled modules—they don't contain readable Python source by design. However, it can indicate missing type information or configuration issues in some cases.

---

## Quick Start: What This Means & How to Fix It

| Scenario | Root Cause | Fix |
|---|---|---|
| **Compiled extension** (e.g., `cv2`, `numpy.core._multiarray`) | Native binary module; no `.py` source exists | Safe to ignore or suppress; install type stubs if needed |
| **Stubs installed but runtime package missing** | `pip install types-requests` but `requests` not installed | Install the main package: `pip install requests` |
| **Type stubs conflict with source** | Stubs point to source that isn't available | Check `pyrightconfig.json` for path issues; verify `useLibraryCodeForTypes` setting |
| **Dynamic import or platform-specific module** | Module only available on certain OS/Python version | Use `TYPE_CHECKING` guard or conditional import; see [Conditional Imports](#conditional-imports-for-platform-specific-modules) |

---

## When to Suppress This Diagnostic

You should **suppress** `reportMissingModuleSource` if:

- The imported module is a **compiled extension** (C/C++ library) with no `.py` source
- You've confirmed the module works at runtime (`python -c "import module_name"` succeeds)
- Type stubs are installed or not needed for your use case

**Per-line suppression:**
```python
import cv2  # type: ignore[reportMissingModuleSource]
```

**Per-file suppression (at the top):**
```python
# pyright: reportMissingModuleSource=false
import cv2
```

**In config (global):**
```json
{
    "python.analysis.diagnosticSeverityOverrides": {
        "reportMissingModuleSource": "none"
    }
}
```

---

## Diagnostic Steps

### 1. Confirm the Module Works at Runtime
```bash
python -c "import cv2; print(cv2.__file__)"
# Output: /path/to/site-packages/cv2/__init__.cpython-39-darwin.so
```

If this **fails**, the real issue is `reportMissingImports` (module not installed), not `reportMissingModuleSource`.

### 2. Check If the Module Is Compiled
```bash
python -c "import cv2; import inspect; print(inspect.getsourcefile(cv2))"
# Output: None  → compiled module
# Output: /path/to/cv2/__init__.py  → has source
```

### 3. Check Pylance's Module Resolution

Enable trace logging in your VS Code settings:

```json
{
    "python.analysis.logLevel": "Trace"
}
```

Then check **Output → Pylance** for the import resolution log. Look for:
- Which path Pylance found the module in
- Whether it found `.pyi` stubs
- Whether it's looking for source files that don't exist

See [How to Read Pylance Import Resolution Logs](../howto/reading-pylance-logs.md) for details.

---

## Common Causes & Solutions

### Cause 1: Native Compiled Module (Normal)

**Symptom**: Warning on imports like `cv2`, `numpy.core._multiarray`, `_ssl`, etc.

**Root cause**: The module is a compiled C/C++ extension with no Python source.

**Solutions** (in order):

1. **Suppress the diagnostic** (recommended for most cases):
   ```python
   import cv2  # type: ignore[reportMissingModuleSource]
   ```

2. **Install type stubs** (if available):
   ```bash
   pip install opencv-stubs
   ```

3. **Disable globally** if it's a frequent false positive:
   ```json
   {
       "python.analysis.diagnosticSeverityOverrides": {
           "reportMissingModuleSource": "none"
       }
   }
   ```

---

### Cause 2: Type Stubs Installed but Runtime Package Missing

**Symptom**: Warning on `reportMissingModuleSource` for a package like `requests`, `flask`, etc.

**Root cause**: A `types-*` stub package is installed, but the actual runtime package is not.

**Solution**: Install the runtime package in your Python environment:

```bash
# Example: types-requests is installed, but requests isn't
pip install requests
```

**Check**:
```bash
pip show requests
pip show types-requests
```

Both should be installed.

---

### Cause 3: Incorrect `pyrightconfig.json` or `extraPaths` Configuration

**Symptom**: Warning on packages that should have source code available.

**Root cause**: Path configuration points to the wrong directory or includes non-existent paths.

**Solutions**:

1. **Check your config file** (`pyrightconfig.json` or `pyproject.toml`):
   ```bash
   # Does this file exist in your project root?
   cat pyrightconfig.json
   ```

   If it does, verify the `include` and `exclude` patterns are correct.

2. **Verify `extraPaths` points to the correct location**:
   ```json
   {
       "python.analysis.extraPaths": ["./src"]
   }
   ```

   The path should point to the **parent directory** of your package's `__init__.py`, not the package itself.

   ❌ Wrong:
   ```json
   {
       "python.analysis.extraPaths": ["./src/mypackage"]
   }
   ```

   ✅ Correct:
   ```json
   {
       "python.analysis.extraPaths": ["./src"]
   }
   ```

3. **Verify the Python interpreter is correct**:
   - Click the Python version in the VS Code status bar
   - Ensure it points to the virtual environment with the packages installed
   - Run "Python: Restart Language Server" after changing it

---

### Cause 4: Dynamic Imports or Platform-Specific Modules

**Symptom**: Warning on modules that are only available on certain platforms or Python versions.

**Example**:
```python
import win32api  # Only available on Windows; warns on Linux/macOS
```

**Solution**: Use `TYPE_CHECKING` guards with conditional imports:

```python
from typing import TYPE_CHECKING

if TYPE_CHECKING:
    import win32api

def my_function() -> None:
    # At runtime, check if the module is available
    import sys
    if sys.platform == "win32":
        win32api.SetClipboardViewer(...)
```

See [Conditional Imports for Platform-Specific Modules](#conditional-imports-for-platform-specific-modules) below.

---

### Cause 5: `useLibraryCodeForTypes` Configuration

**Symptom**: Inconsistent behavior between Pyright CLI and Pylance; warnings appear only in certain contexts.

**Root cause**: The `useLibraryCodeForTypes` setting controls whether Pylance uses compiled library source or bundled stubs. Mismatches between CLI and IDE can cause this.

**Solution**:

```json
{
    "python.analysis.useLibraryCodeForTypes": true
}
```

Set consistently in both:
- VS Code `settings.json` (for Pylance)
- `pyrightconfig.json` (for Pyright CLI)

See [`python.analysis.useLibraryCodeForTypes`](../settings/python_analysis_useLibraryCodeForTypes.md) for details.

---

## Conditional Imports for Platform-Specific Modules

Use `TYPE_CHECKING` to import types without triggering the diagnostic at runtime:

```python
from __future__ import annotations
from typing import TYPE_CHECKING
import sys

if TYPE_CHECKING:
    import win32api  # Only imported for type checking

def clipboard_copy(text: str) -> None:
    """Copy text to clipboard on Windows."""
    if sys.platform == "win32":
        import win32api
        win32api.SetClipboardData(text)
    else:
        print("clipboard_copy is Windows-only")
```

**Key points**:
- `TYPE_CHECKING` is `False` at runtime, so the import is skipped
- Pylance still sees the type annotation and provides IntelliSense
- `from __future__ import annotations` (PEP 563) converts all annotations to strings, preventing runtime `NameError`

---

## Configuration Examples

### Suppress for Specific Packages

```json
{
    "python.analysis.diagnosticSeverityOverrides": {
        "reportMissingModuleSource": "information"
    }
}
```

Or suppress only for specific packages in a `pyrightconfig.json`:

```json
{
    "reportMissingModuleSource": "none",
    "include": ["src/"],
    "exclude": ["tests/"]
}
```

### Use Library Code When Type Stubs Are Missing

If you prefer to use source code from installed packages rather than bundled stubs:

```json
{
    "python.analysis.useLibraryCodeForTypes": true
}
```

### Adjust Diagnostic Severity

Make it a warning instead of an error:

```json
{
    "python.analysis.diagnosticSeverityOverrides": {
        "reportMissingModuleSource": "warning"
    }
}
```

---

## Representative Issues

The diagnostic is motivated by real-world issues and user configurations:

**Installation & Environment Issues:**
- [#2202](https://github.com/microsoft/pylance-release/issues/2202): Module installation problems
- [#2411](https://github.com/microsoft/pylance-release/issues/2411): Python environment selection
- [#1585](https://github.com/microsoft/pyright/issues/1585): CI/CD environment setup

**Type Stubs & Source Mismatch:**
- [#242](https://github.com/microsoft/pylance-release/issues/242): Missing `.pyi` files for compiled modules
- [#4163](https://github.com/microsoft/pylance-release/issues/4163): Consistency between Pyright CLI and Pylance
- [#4286](https://github.com/microsoft/pyright/issues/4286): Incorrect stub imports

**Configuration:**
- [#295](https://github.com/microsoft/pylance-release/issues/295): Diagnostic configuration
- [#509](https://github.com/microsoft/pylance-release/issues/509): Respecting user disabling preferences
- [#5200](https://github.com/microsoft/pylance-release/issues/5200): Severity override customization

**Dynamic Imports & Runtime Behavior:**
- [#5073](https://github.com/microsoft/pylance-release/issues/5073): Conditional imports with `TYPE_CHECKING`
- [#4976](https://github.com/microsoft/pylance-release/issues/4976): Dynamic module availability
- [#8902](https://github.com/microsoft/pyright/issues/8902): Runtime script metadata handling

**Type Hints & Compatibility:**
- [#7832](https://github.com/microsoft/pyright/issues/7832): Type hints in libraries
- [#8558](https://github.com/microsoft/pyright/issues/8558): Callable object compatibility

---

## See Also

- [Fixing unresolved imports](../howto/unresolved-imports.md) — comprehensive guide for all import issues
- [How to Read Pylance Import Resolution Logs](../howto/reading-pylance-logs.md) — debug import resolution
- [`python.analysis.extraPaths`](../settings/python_analysis_extraPaths.md) — add search paths for imports
- [`python.analysis.useLibraryCodeForTypes`](../settings/python_analysis_useLibraryCodeForTypes.md) — use library source vs. stubs
- [`python.analysis.diagnosticSeverityOverrides`](../settings/python_analysis_diagnosticSeverityOverrides.md) — suppress or adjust this diagnostic
- [Pyright Configuration](https://github.com/microsoft/pyright/blob/main/docs/configuration.md#reportMissingModuleSource) — full configuration reference

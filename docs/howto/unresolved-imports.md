# How to Fix Unresolved Import Errors in Pylance

Pylance uses static analysis to resolve Python imports — it does not execute your code. When it cannot find a module, it reports one of several diagnostics depending on what is missing. This guide covers all of them.

---

## Table of Contents

- [Import Could Not Be Resolved (`reportMissingImports`)](#import-could-not-be-resolved-reportmissingimports)
- [Import Could Not Be Resolved from Source (`reportMissingModuleSource`)](#import-could-not-be-resolved-from-source-reportmissingmodulesource)
- [Stub File Not Found (`reportMissingTypeStubs`)](#stub-file-not-found-reportmissingtypestubs)
- [Circular Import Detected (`reportImportCycles`)](#circular-import-detected-reportimportcycles)
- [Works at Runtime but Pylance Shows Errors](#works-at-runtime-but-pylance-shows-errors)
- [How Pylance Resolves Imports](#how-pylance-resolves-imports)
- [Diagnostic Checklist](#diagnostic-checklist)

---

## Import Could Not Be Resolved (`reportMissingImports`)

**Symptom**: `Import "mypackage" could not be resolved` [`Pylance(reportMissingImports)`](../diagnostics/reportMissingImports.md)

### Step 1: Check the Python Interpreter

Click the Python version in the VS Code status bar. Verify it points to the correct virtual environment.

```bash
# In the VS Code terminal, confirm:
which python    # Linux/macOS
where python    # Windows
python -c "import sys; print(sys.path)"
```

### Step 2: Check if the Package is Installed

```bash
pip show mypackage
# or
python -c "import mypackage; print(mypackage.__file__)"
```

### Step 3: Check `extraPaths`

Open Settings (JSON) and verify `python.analysis.extraPaths` includes the correct import root:

```json
{
    "python.analysis.extraPaths": ["./src"]
}
```

### Step 4: Check for Editable Install Issues

If using `pip install -e`:

```bash
# Look for .pth files:
find .venv/lib -name "*.pth" | xargs cat
# Any line starting with "import" will be ignored by Pylance
```

### Step 5: Enable Verbose Logging

Add to settings.json:

```json
{
    "python.analysis.logLevel": "Trace"
}
```

Then check the **Output** panel → **Pylance** for import resolution attempts.

### Step 6: Check `pyrightconfig.json`

If it exists, it may override VS Code settings. Verify `include` and `exclude` patterns:

```json
{
    "include": ["src"],
    "exclude": ["tests"]
}
```

### Common Causes and Fixes

| Cause | Fix |
|---|---|
| Wrong interpreter selected | Select the correct venv in VS Code status bar |
| Package not installed in the venv | `pip install mypackage` in the correct venv |
| `extraPaths` missing a directory | Add the package source dir to `extraPaths` |
| Configured path exists but points to wrong level | Change path to point at import root (parent of `__init__.py`) |
| Editable install uses import hooks | Reinstall with `--config-settings editable_mode=compat` |
| `pyrightconfig.json` overrides settings | Check and update the config file |
| `include` setting excludes the file | Verify `python.analysis.include` covers your files |
| File is in `exclude` patterns | Remove from `python.analysis.exclude` |
| Namespace package (no `__init__.py`) | Pylance supports namespace packages; verify structure |
| Missing `__init__.py` in regular package | Add `__init__.py` to each package directory |

---

## Import Could Not Be Resolved from Source (`reportMissingModuleSource`)

**Symptom**: `Import "mypackage" could not be resolved from source` `Pylance(reportMissingModuleSource)`

**What it means**: Pylance found a type stub (`.pyi`) for the package but could not find the corresponding Python source (`.py`). This is different from `reportMissingImports` — the import _does_ exist.

For detailed solutions, see [reportMissingModuleSource Documentation](../diagnostics/reportMissingModuleSource.md).

### Quick Reference: Common Causes and Fixes

| Cause | Fix |
|---|---|
| Package is a C extension or native library (`.so` / `.pyd`) | Safe to ignore — native extensions don't have `.py` source. Suppress with `# type: ignore[reportMissingModuleSource]` |
| Package installed as `.pyc`-only (compiled wheel without source) | Install a source distribution, or suppress the diagnostic |
| Stubs installed separately (e.g., `types-requests`) but main package not in venv | Install the main package: `pip install requests` |
| Stub-only package with no corresponding runtime package | Suppress for that specific package |

---

## Stub File Not Found (`reportMissingTypeStubs`)

**Symptom**: `Stub file not found for "mypackage"` `Pylance(reportMissingTypeStubs)`

**What it means**: The package is installed and importable, but it has no type stubs (no `.pyi` files) and is not marked as `py.typed`. This diagnostic is **off by default**.

### Common Causes and Fixes

| Cause | Fix |
|---|---|
| Third-party package without type support | Install stubs if available: `pip install types-mypackage` |
| No stubs available for the package | Suppress: `"reportMissingTypeStubs": "none"` in `diagnosticSeverityOverrides` |
| Package _is_ typed but missing `py.typed` marker | Report to the package maintainer, or create a local stub |

---

## Circular Import Detected (`reportImportCycles`)

**Symptom**: `Cycle detected in import chain` `Pylance(reportImportCycles)` — followed by a list of file paths forming the cycle.

**What it means**: Files form a circular dependency (A imports B, B imports C, C imports A). This diagnostic is **off by default**.

### Mitigation Strategies

- **Move shared types to a separate module** that both sides can import without creating a cycle
- **Use `TYPE_CHECKING` imports** for type-only references:

```python
from __future__ import annotations
from typing import TYPE_CHECKING

if TYPE_CHECKING:
    from mypackage.models import User  # only used for type hints, not at runtime
```

- **Restructure the dependency** so the cycle is broken at the module level
- **Suppress per-file**: `# pyright: reportImportCycles=false` at the top of files

---

## Works at Runtime but Pylance Shows Errors

**Symptom**: `import mypackage` works in the terminal, but Pylance shows errors.

This happens because Pylance does **static analysis** and cannot execute dynamic Python mechanisms.

### Things Pylance Cannot Follow

| Runtime Mechanism | Why Pylance Can't Follow It |
|---|---|
| **Import-hook `.pth` files** (lines starting with `import`) | Pylance reads `.pth` files but only processes plain path lines — it skips `import` lines |
| **`sys.path.append()` in code** | Pylance does not execute Python code |
| **`PYTHONPATH` environment variable** | Pylance does not read environment variables |
| **`pkgutil.extend_path()` in `__init__.py`** | Pylance resolves statically; picks first match |
| **`importlib` dynamic imports** | All programmatic imports are invisible to static analysis |
| **Namespace packages across separate roots** | Python merges same-named packages; Pylance picks first match |
| **pytest `conftest.py` path injection** | pytest adds to `sys.path` at runtime; Pylance has no awareness |
| **Django `INSTALLED_APPS`** | Django dynamically discovers apps; Pylance can't execute `django.setup()` |
| **Conda environment path layout** | Conda uses different path structures; configure with `extraPaths` |
| **`sys.modules` patching** | Some libraries inject modules at runtime; invisible to static analysis |
| **`site.addsitedir()`** | Adds to `sys.path` at runtime; Pylance doesn't call this |

### Framework-Specific Examples

**pytest**: If tests import helper modules from a `tests/` directory:

```text
project/
├── src/mypackage/
├── tests/
│   ├── conftest.py
│   └── helpers/
│       └── test_utils.py
```

Fix:

```json
{
    "python.analysis.extraPaths": ["./tests"]
}
```

**Django**: If using app discovery via `INSTALLED_APPS`:

```text
myproject/
├── myproject/
│   └── settings.py
├── users/
│   └── models.py
└── billing/
    └── models.py
```

Fix:

```json
{
    "python.analysis.extraPaths": ["."]
}
```

**Flask**: If using app factories with blueprints:

```text
myproject/
├── app/
│   ├── __init__.py
│   └── extensions.py
├── blueprints/
│   ├── auth/
│   │   └── routes.py
│   └── api/
│       └── routes.py
```

Fix:

```json
{
    "python.analysis.extraPaths": ["."]
}
```

---

## How Pylance Resolves Imports

Pylance searches for modules in this specific order, stopping at the first match:

| Priority | Search Location | Notes |
|---|---|---|
| 1 | `stubPath` (custom stubs) | Default: `./typings` |
| 2 | Source files in project root | Your workspace files |
| 3 | `extraPaths` entries | Searched in order listed |
| 4 | `src/` directory (if enabled) | Automatic convenience path |
| 5 | Typeshed stdlib stubs | Standard library type info |
| 6 | Python interpreter search paths (`site-packages`) | Installed packages from selected interpreter |
| 7 | Bundled third-party stubs | Pylance ships with stubs for popular packages |
| 8 | Typeshed third-party stubs | Community-maintained fallback stubs |

**Key implications**:

- `extraPaths` entries have **higher priority** than installed packages in `site-packages`
- `stubPath` has **higher priority** than bundled third-party stubs
- Installed packages with type information take priority over bundled stubs
- If `pyrightconfig.json` exists, it overrides VS Code settings

---

## Diagnostic Checklist

When imports aren't resolving, run through this checklist:

- [ ] **Interpreter**: Is the correct Python interpreter selected? (status bar)
- [ ] **Package installed**: Is the package installed? (`pip show <package>`)
- [ ] **Editable install**: If editable, is the `.pth` file path-based (not import-hook)?
- [ ] **`extraPaths`**: Do paths exist and point to the import root?
- [ ] **Path-based settings**: Do `stubPath` or `typeshedPaths` point to valid locations?
- [ ] **`pyrightconfig.json`**: Does a config file exist that might override settings?
- [ ] **`include`/`exclude`**: Are the correct files included and not excluded?
- [ ] **Verbose logging**: Enable `"python.analysis.logLevel": "Trace"` to debug
- [ ] **Restart**: Run "Python: Restart Language Server" after configuration changes

---

## See Also

- [reportMissingModuleSource Documentation](../diagnostics/reportMissingModuleSource.md) — module found but source missing
- [Pyright Configuration Reference](https://github.com/microsoft/pyright/blob/main/docs/configuration.md)

# python.analysis.extraPaths

## Setting

```json
{
    "python.analysis.extraPaths": ["./src", "./lib"]
}
```

## What It Does

Adds extra directories to Pylance's module search path. These directories are searched **before** installed packages in `site-packages`, giving them higher priority.

## Common Use Cases

### Monorepo or Multi-Root Project

```text
project/
├── src/
│   └── mypackage/
│       └── __init__.py
├── lib/
│   └── shared/
│       └── __init__.py
└── tests/
```

Configuration:

```json
{
    "python.analysis.extraPaths": ["./src", "./lib"]
}
```

### Framework-Specific Patterns

**Django** (project root is import root):

```json
{
    "python.analysis.extraPaths": ["."]  
}
```

**Flask** (app and blueprints at project root):

```json
{
    "python.analysis.extraPaths": ["."]  
}
```

**pytest** (tests directory needs to be on path):

```json
{
    "python.analysis.extraPaths": ["./tests"]
}
```

## ❌ Common Mistakes

### Mistake 1: Pointing to the Package Directory Instead of Parent

❌ Wrong:
```json
{
    "python.analysis.extraPaths": ["./src/mypackage"]
}
```

This makes Pylance look for `from __init__ import ...` instead of `from mypackage import ...`.

✅ Correct:
```json
{
    "python.analysis.extraPaths": ["./src"]
}
```

### Mistake 2: Using PYTHONPATH Instead of extraPaths

Pylance **does not read** `PYTHONPATH` environment variables. You must use `extraPaths`:

❌ Wrong (won't work for Pylance):
```bash
export PYTHONPATH="./src:./lib"
```

✅ Correct:
```json
{
    "python.analysis.extraPaths": ["./src", "./lib"]
}
```

### Mistake 3: Non-Existent Paths

❌ Wrong:
```json
{
    "python.analysis.extraPaths": ["./nonexistent/path"]
}
```

✅ Correct:
```json
{
    "python.analysis.extraPaths": ["./src"]  # path exists
}
```

## How to Debug

1. Enable trace logging:
   ```json
   {
       "python.analysis.logLevel": "Trace"
   }
   ```

2. Check **Output → Pylance** for import resolution logs

3. Look for lines showing which directories are being searched

## See Also

- [How to Fix Unresolved Import Errors](../howto/unresolved-imports.md)
- [reportMissingImports Diagnostic](../diagnostics/reportMissingImports.md)

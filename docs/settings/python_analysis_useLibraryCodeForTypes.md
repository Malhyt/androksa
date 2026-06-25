# python.analysis.useLibraryCodeForTypes

## Setting

```json
{
    "python.analysis.useLibraryCodeForTypes": true
}
```

## What It Does

Controls whether Pylance uses compiled library source code or bundled type stubs for type information.

- **`true`**: Use library source code when available (even if stubs exist)
- **`false`** (default): Prefer bundled type stubs over library source

## When to Use Each

### Use `true` When:

- You want the most up-to-date type information from the library
- The library includes inline type hints (PEP 561: `py.typed` marker)
- You're debugging type issues and need to see the actual source

```json
{
    "python.analysis.useLibraryCodeForTypes": true
}
```

### Use `false` (default) When:

- You prefer stability of bundled stubs
- The library source is incomplete or has outdated type information
- You want consistent behavior across different Pylance versions

```json
{
    "python.analysis.useLibraryCodeForTypes": false
}
```

## Common Issue: Inconsistent Behavior

**Symptom**: Pyright CLI shows different results than Pylance (IDE).

**Cause**: Settings are configured differently.

**Fix**: Set `useLibraryCodeForTypes` consistently in both:

1. **VS Code settings.json** (for Pylance):
   ```json
   {
       "python.analysis.useLibraryCodeForTypes": true
   }
   ```

2. **pyrightconfig.json** (for Pyright CLI):
   ```json
   {
       "useLibraryCodeForTypes": true
   }
   ```

## See Also

- [reportMissingModuleSource Diagnostic](../diagnostics/reportMissingModuleSource.md)
- [Pyright Configuration Reference](https://github.com/microsoft/pyright/blob/main/docs/configuration.md)

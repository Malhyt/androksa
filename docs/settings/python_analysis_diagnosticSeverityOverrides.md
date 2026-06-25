# python.analysis.diagnosticSeverityOverrides

## Setting

```json
{
    "python.analysis.diagnosticSeverityOverrides": {
        "reportMissingImports": "error",
        "reportMissingModuleSource": "warning",
        "reportMissingTypeStubs": "none"
    }
}
```

## What It Does

Overrides the default severity level for specific diagnostics. This allows you to:

- Suppress warnings you don't care about
- Promote warnings to errors for stricter type checking
- Convert errors to information messages

## Severity Levels

| Level | Effect | Use When |
|---|---|---|
| `"error"` | Appears as error (red squiggle) | You want to enforce strict type checking |
| `"warning"` | Appears as warning (yellow squiggle) | You want to be notified but don't want to block work |
| `"information"` | Appears as info (blue squiggle) | You want to be aware but keep it subtle |
| `"none"` | Disabled completely | You don't care about this diagnostic |

## Common Examples

### Suppress Compiled Module Warnings

```json
{
    "python.analysis.diagnosticSeverityOverrides": {
        "reportMissingModuleSource": "none"
    }
}
```

### Strict Type Checking

```json
{
    "python.analysis.diagnosticSeverityOverrides": {
        "reportMissingImports": "error",
        "reportMissingTypeStubs": "error",
        "reportUnusedVariable": "error"
    }
}
```

### Lenient Type Checking

```json
{
    "python.analysis.diagnosticSeverityOverrides": {
        "reportMissingTypeStubs": "none",
        "reportUnknownArgumentType": "none",
        "reportUnknownMemberType": "none"
    }
}
```

## Per-File Suppression

You can also suppress diagnostics at the top of individual files:

```python
# pyright: reportMissingModuleSource=false
import cv2  # This warning is suppressed
```

## Per-Line Suppression

Or suppress specific lines:

```python
import cv2  # type: ignore[reportMissingModuleSource]
```

## See Also

- [reportMissingModuleSource Diagnostic](../diagnostics/reportMissingModuleSource.md)
- [Pyright Configuration Reference](https://github.com/microsoft/pyright/blob/main/docs/configuration.md)

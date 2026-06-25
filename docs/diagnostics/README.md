# Diagnostics Documentation

This directory contains comprehensive guides for understanding and fixing diagnostic warnings in Pylance and Pyright.

## Available Diagnostics

### Import & Module Resolution

- **[reportMissingModuleSource](./reportMissingModuleSource.md)** — Module found but source code cannot be located
  - Typically for compiled C/C++ extensions
  - Safe to ignore in most cases
  - Configure suppression per-package

## Structure

Each diagnostic guide includes:

1. **Overview** — What the diagnostic means
2. **Quick Start** — Instant solutions for common scenarios
3. **When to Suppress** — Clear guidance on when to ignore
4. **Diagnostic Steps** — How to investigate
5. **Common Causes & Solutions** — Root cause analysis with fixes
6. **Configuration Examples** — Real-world setup patterns
7. **Representative Issues** — Related GitHub issues organized by theme

## General Principles

- **Static vs. Dynamic**: Pylance cannot execute code, so some runtime behaviors are invisible
- **Configuration Priority**: `pyrightconfig.json` > VS Code settings > defaults
- **Path Resolution**: `extraPaths` entries take priority over `site-packages`
- **Type Information**: Stubs (`.pyi`) are preferred over source when available

## See Also

- [How to Fix Unresolved Import Errors](../howto/unresolved-imports.md) — comprehensive import troubleshooting
- [Pyright Configuration Reference](https://github.com/microsoft/pyright/blob/main/docs/configuration.md)

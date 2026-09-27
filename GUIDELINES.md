# Bookshelf Development & Contribution Guidelines

This document outlines the strict architectural standards and conventions for code modifications in this repository fork.

---

## 1. Zero Loose Changes Directive (Absolute Rule)

**EVERY single code modification—without exception—must be encapsulated within C# preprocessor directives.**

There are **zero loose changes** permitted:
* Adding a `using` statement? It MUST be wrapped in `#if true // [TAG] ... #endif`.
* Modifying a `catch` block or exception filter? It MUST be wrapped in `#if true // [TAG] ... #else ... #endif`.
* Adding a method, property, parameter, or comment? It MUST be wrapped.
* Modifying existing lines? The original upstream code MUST be preserved in the `#else` block.

This ensures:
1. `git diff` immediately shows every fork modification with its technical justification.
2. Future upstream merges from Readarr/Bookshelf can instantly spot, preserve, or reconcile our custom modifications.
3. Features or fixes can be toggled on/off cleanly if needed.

---

## 2. Standard Tags

* `[FIX]`: Bug fixes, deadlock resolution, race-condition mitigation.
* `[FEATURE]`: Architectural extensions, new workflows, UX additions.
* `[HARDCOVER]`: Changes specifically bridging or fixing Hardcover metadata compatibility.
* `[WORKAROUND]`: Temporary mitigations for external API or library limitations.

---

## 3. Syntax Standards

### A. Modifications / Replacements (Mandatory `#else` preservation)
Always retain the original upstream code in the `#else` branch:
```csharp
#if true // [HARDCOVER] Catch all exceptions from Goodreads proxy so upstream blocks do not crash sync
    catch (Exception ex)
    {
        _logger.Debug(ex, $"Nothing found for edition [{report.EditionGoodreadsId}]");
        report.EditionGoodreadsId = null;
    }
#else
    catch (BookNotFoundException)
    {
        _logger.Debug($"Nothing found for edition [{report.EditionGoodreadsId}]");
        report.EditionGoodreadsId = null;
    }
#endif
```

### B. Pure Additions (Using Directives, Methods, Logic)
```csharp
#if true // [FIX] Needed for QualityModel in DownloadIgnoredEvent
using NzbDrone.Core.Qualities;
#endif
```

```csharp
#if true // [FEATURE] Ingest unmonitored series books for complete series visibility
    // Additions here
#endif
```

---

## 4. Engineering Standards

1. **SOLID Principles**: Keep methods single-responsibility; extract helper classes/methods instead of inlining complex branches.
2. **Defensive Validation**: Validate inputs at boundary entry points (APIs, parsers, queue ingestion).
3. **Trace Logging**: Use appropriate NLog levels (`Trace` for high-frequency steps, `Debug` for internal state transitions, `Info` for completed workflows, `Warn` for recoverable issues).
4. **Non-Destructive Defaults**: Never silently purge user data or library entities unless explicitly instructed by the user or required by unambiguous configuration.

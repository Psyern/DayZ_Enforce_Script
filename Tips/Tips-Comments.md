# Tips: Comments

Comment styles and documentation practices in EnScript.

## Single-line Comments

```c
// This is a single-line comment
int value = 10; // Inline comment
```

## Multi-line Comments

```c
/*
 * This is a multi-line comment
 * spanning multiple lines
 */
```

## Documentation Comments

```c
//! Brief description
//! @param value The value to process
//! @return True if successful
bool ProcessValue(int value);
```

## Doxygen-style Comments

```c
//! Toggle between hidden and previously set visibility state for each marker category
int visibility;
int previousVisibility;

visibility |= m_MarkerModule.GetVisibility(ExpansionMapMarkerType.SERVER);
previousVisibility |= m_MarkerModule.GetPreviousVisibility(ExpansionMapMarkerType.SERVER);
```

## Comment Style

* Use `//` for single-line comments
* Use `/* */` for multi-line comments
* Use `//!` for documentation comments

## Best Practices

1. **Document complex logic** - Explain why, not what
2. **Keep comments up to date** - Outdated comments are worse than no comments
3. **Use documentation comments** - For public APIs and important methods
4. **Avoid obvious comments** - Code should be self-documenting

---

**Related Guides:**
- [EnScript Style Guide](EnScript-Style-Guide.md) - Complete style reference
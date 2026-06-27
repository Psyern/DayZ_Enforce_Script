# Tips: Preprocessor Directives

Using preprocessor directives for conditional compilation and defines.

## Conditional Compilation

```c
#ifdef EXPANSION_DEBUG
	Print("Debug message");
#endif

#ifndef EXPANSION_RELEASE
	// Development code
#endif
```

## Defines

Use `#define` for global constants and feature flags:

```c
#define MAX_PLAYERS 60
#define EXPANSION_FEATURE_ENABLED
```

## Debug Defines

```c
#ifdef EXPANSION_DEBUG
	// Debug-only code
#endif
```

## Common Patterns

### Feature Flags

```c
#define MYMOD_FEATURE_ENABLED

#ifdef MYMOD_FEATURE_ENABLED
	// Feature code
#endif
```

### Version-Specific Code

```c
#ifdef DAYZ_1_29
	// DayZ 1.29+ specific code
	// Use g_Game directly
	if (g_Game.IsDedicatedServer())
#endif

#ifndef DAYZ_1_29
	// DayZ 1.28 and earlier
	// Use g_Game
	if (g_Game && g_Game.IsDedicatedServer())
#endif
```

## Best Practices

1. **Use descriptive define names** - Make it clear what the define controls
2. **Group related defines** - Keep feature flags together
3. **Document defines** - Comment what each define does
4. **Avoid empty preprocessor blocks** - Must contain at least one statement

---

**Related Guides:**
- [EnScript Style Guide](EnScript-Style-Guide.md) - Complete style reference
- [Tips: Debugging](Tips-Debugging.md) - Debugging techniques
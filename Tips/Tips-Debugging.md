# Tips: Debugging

Debugging techniques, error handling, and log file understanding.

## Debug Traces (Expansion-Specific)

```c
#ifdef EXPANSION_DEBUG
	ExpansionDebug.Trace("Debug message", "Category");
#endif
```

## Logging

```c
Print("Log message");
```

**See:** [How to Create a Logger](../How-To/How-To-Logger.md) and [How to Use CF Logging](../Frameworks/Community-Framework/CF/How-To-CF-Logger.md) for logging guides.

## Error Logs

* **Exception logs:** `crash_<date>_<time>.log` - Contains exceptions (NOT crashes)
* **Actual crashes:** Segfaults create different crash dumps
* Check both log types when debugging issues

## Error Handling

### Existence/Null Checks

Always check for null before using objects:

```c
PlayerBase player = PlayerBase.Cast(entity);
if (player)
{
	// Safe to use player
}
```

### Value Validation

```c
if (value < 0 || value > MAX_VALUE)
{
	return false;
}
```

### Client/Server Checks

```c
// ❌ WRONG - Returns true on client during load
if (g_Game && g_Game.IsClient())

// ✅ CORRECT - Reliable client check
if (g_Game && !g_Game.IsDedicatedServer())

// ❌ WRONG - Returns true on client during load
if (g_Game && g_Game.IsServer())

// ✅ CORRECT - Reliable server check (dedicated only)
if (g_Game && g_Game.IsDedicatedServer())

// ✅ CORRECT - Server check including offline/singleplayer
if (g_Game && g_Game.IsServer() && !g_Game.IsMultiplayer())
```

**Rule:** Use `IsDedicatedServer()` for reliable client/server detection, especially in initialization code.

**See:** [Tips: g_Game vs GetGame()](Tips-g_Game-GetGame.md) for detailed client/server detection.

## Compiler Error Messages

**⚠️ Misleading Error Locations:**

Compile errors involving:

* Undefined classes (missing addon not loaded)
* Variable name conflicts

Will **NOT** show the correct filename and line number. Instead, they show the last successfully parsed `.c` file at EOF, which is misleading and unhelpful.

**What to do:**

* If error location seems wrong, search for the class/variable name in your entire codebase
* Check that all required addons are loaded
* Look for name conflicts across files

## Understanding Log Files

* **Exception logs:** `crash_<date>_<time>.log` - Contains exceptions (NOT crashes)
* **Actual crashes:** Segfaults create different crash dumps
* Check both log types when debugging issues

## Common Segfault Causes

1. **Complex array assignments** - Use intermediate variables
2. **Empty preprocessor blocks** - Must contain at least one statement
3. **Misused `delete` keyword** - Let garbage collection handle it
4. **Over-specified `ref` usage** - Only use on members/typedefs

**See:** [Tips: Memory Management](Tips-Memory-Management.md) for detailed segfault prevention.

---

**Related Guides:**
- [EnScript Style Guide](EnScript-Style-Guide.md) - Complete style reference
- [Tips: Memory Management](Tips-Memory-Management.md) - Understanding segfaults
- [Tips: g_Game vs GetGame()](Tips-g_Game-GetGame.md) - Client/server detection
- [How to Create a Logger](../How-To/How-To-Logger.md) - Logging implementation
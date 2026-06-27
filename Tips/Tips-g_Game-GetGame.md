# Tips: g_Game vs GetGame()

When to use `g_Game` directly versus `GetGame()` function in DayZ modding.

## The Difference

### g_Game (Global Variable)
- Direct access to global game instance
- Used in DayZ 1.29+ experimental
- Faster access (no function call)
- Must check if null

### GetGame() (Function)
- Function that returns game instance
- Used in DayZ 1.28 and earlier
- Always returns valid instance (never null)
- Slightly slower (function call overhead)

## DayZ Version Differences

### DayZ 1.28.161464 (Current Stable)

**Uses:** `GetGame()` function

```c
// DayZ 1.28
DayZGame game = GetGame();
if (game.IsDedicatedServer())
{
	// Server code
}

// Or direct call
if (GetGame().IsDedicatedServer())
{
	// Server code
}
```

### DayZ 1.29.161219 (Experimental)

**Uses:** `g_Game` global variable

```c
// DayZ 1.29
if (g_Game.IsDedicatedServer())
{
	// Server code
}

// Always check for null first
if (!g_Game)
	return;

g_Game.RPC(null, RPC_TEST, data, false);
```

## When to Use Which

### Use g_Game (DayZ 1.29+)

**When:**
- Targeting DayZ 1.29+ experimental
- Need direct access
- Performance critical code
- Code is clearly for newer versions

```c
// ✅ Correct for 1.29+
if (!g_Game)
	return;

if (g_Game.IsDedicatedServer())
{
	// Server code
}
```

### ~~Use GetGame()~~ - DO NOT USE

**⚠️ DEPRECATED: Do not use `GetGame()`**

**Always use `g_Game` instead:**

```c
// ❌ WRONG - Do not use GetGame()
DayZGame game = GetGame();
if (game.IsDedicatedServer())
{
	// Server code
}

// ✅ CORRECT - Always use g_Game with null check
if (!g_Game)
	return;
	
if (g_Game.IsDedicatedServer())
{
	// Server code
}
```

## Version Compatibility Pattern

### Check Version and Use Appropriate Method

```c
void MyFunction()
{
#ifdef DAYZ_1_29
	// DayZ 1.29+ code
	if (!g_Game)
		return;
	
	if (g_Game.IsDedicatedServer())
	{
		// Server code
	}
#else
	// DayZ 1.28 code
	DayZGame game = GetGame();
	if (game.IsDedicatedServer())
	{
		// Server code
	}
#endif
}
```

### Universal Pattern (Works on Both)

```c
void MyFunction()
{
	DayZGame game;
	
#ifdef DAYZ_1_29
	game = g_Game;
#else
	game = GetGame();
#endif
	
	if (!game)
		return;
	
	if (game.IsDedicatedServer())
	{
		// Server code
	}
}
```

## Common Patterns

### Pattern 1: Null Check First (1.29+)

```c
// ✅ Good - Always check first
if (!g_Game)
	return;

if (g_Game.IsDedicatedServer())
{
	// Server code
}
```

### Pattern 2: ~~Using Function~~ - DO NOT USE

```c
// ❌ WRONG - Do not use GetGame()
DayZGame game = GetGame();
if (game.IsDedicatedServer())
{
	// Server code
}

// ✅ CORRECT - Always use g_Game
if (!g_Game)
	return;
	
if (g_Game.IsDedicatedServer())
{
	// Server code
}
```

### Pattern 3: Inline Use

```c
// DayZ 1.29+
if (g_Game && g_Game.IsDedicatedServer())
{
	// Server code
}

// ❌ WRONG - Do not use GetGame()
// Always use g_Game instead
```

### Pattern 4: Combined Null and Server Check

```c
// ✅ Good - Combined check for early return
if (!g_Game || !g_Game.IsServer())
	return;

// Then use g_Game safely
g_Game.GetCallQueue(CALL_CATEGORY_GAMEPLAY).CallLater(...);
```

### Pattern 5: Null Check Before Multiple Calls

```c
// ✅ Good - Check once, use multiple times
if (!g_Game)
	return;

g_Game.GetCallQueue(CALL_CATEGORY_SYSTEM).CallLater(...);
g_Game.GetCallQueue(CALL_CATEGORY_GAMEPLAY).CallLater(...);
g_Game.ObjectDelete(target);
```

### Pattern 6: Conditional Method Call

```c
// ✅ Good - Check before each call if needed
if (g_Game)
	g_Game.GetCallQueue(CALL_CATEGORY_GAMEPLAY).CallLater(...);

if (g_Game)
	g_Game.ObjectDelete(item);
```

## Real-World Examples

### Example 1: RPC Calls

```c
// DayZ 1.29+
void SendRPC(Param data)
{
	if (!g_Game)
		return;
	
	g_Game.RPC(null, MYMOD_RPC_TEST, data, false);
}

// ❌ WRONG - Do not use GetGame()
// Always use g_Game instead:
void SendRPC(Param data)
{
	if (!g_Game)
		return;
		
	g_Game.RPC(null, MYMOD_RPC_TEST, data, false);
}
```

### Example 2: Server Check

```c
// DayZ 1.29+
bool IsServer()
{
	return (g_Game && g_Game.IsDedicatedServer());
}

// ❌ WRONG - Do not use GetGame()
// Always use g_Game instead:
bool IsServer()
{
	return (g_Game && g_Game.IsDedicatedServer());
}
```

### Example 3: Profile Settings

```c
// DayZ 1.29+
void SaveSettings()
{
	if (!g_Game)
		return;
	
	g_Game.SetProfileString("MyMod.Setting", "value");
	g_Game.SaveProfile();
}

// ❌ WRONG - Do not use GetGame()
// Always use g_Game instead:
void SaveSettings()
{
	if (!g_Game)
		return;
		
	g_Game.SetProfileString("MyMod.Setting", "value");
	g_Game.SaveProfile();
}
```

### Example 4: CallQueue Usage

```c
// DayZ 1.29+ - Multiple patterns
void ScheduleTask()
{
	// Pattern 1: Early return
	if (!g_Game || !g_Game.IsServer())
		return;
	
	g_Game.GetCallQueue(CALL_CATEGORY_GAMEPLAY).CallLater(MyMethod, 1000, false);
}

void ScheduleTask2()
{
	// Pattern 2: Inline check
	if (g_Game)
		g_Game.GetCallQueue(CALL_CATEGORY_GAMEPLAY).CallLater(MyMethod, 1000, false);
}

void ScheduleTask3()
{
	// Pattern 3: Check once, use multiple times
	if (!g_Game)
		return;
	
	g_Game.GetCallQueue(CALL_CATEGORY_SYSTEM).CallLater(Task1, 100, false);
	g_Game.GetCallQueue(CALL_CATEGORY_GAMEPLAY).CallLater(Task2, 2000, false);
}
```

### Example 5: Object Creation

```c
// DayZ 1.29+
EntityAI CreateItem(string className)
{
	EntityAI item = null;
	if (g_Game)
		item = EntityAI.Cast(g_Game.CreateObject(className, "0 0 0", false));
	return item;
}

// ❌ WRONG - Do not use GetGame()
// Always use g_Game instead:
EntityAI CreateItem(string className)
{
	if (!g_Game)
		return null;
		
	return EntityAI.Cast(g_Game.CreateObject(className, "0 0 0", false));
}
```

## Best Practices

### 1. Always Check for Null (g_Game)

```c
// ✅ Good - Always check
if (!g_Game)
	return;

// Use g_Game safely

// ❌ Avoid - May crash
g_Game.RPC(null, RPC_TEST, data, false);  // Crashes if g_Game is null!
```

### 2. Use Appropriate Version

```c
// ✅ Good - Version-specific
#ifdef DAYZ_1_29
	if (!g_Game)
		return;
#else
	DayZGame game = GetGame();
#endif

// ❌ Avoid - Mixing versions incorrectly
if (GetGame().IsDedicatedServer())  // Wrong for 1.29!
```

### 3. Be Consistent

```c
// ✅ Good - Consistent throughout
void MyMod_Module::OnInit()
{
	if (!g_Game)
		return;
	
	g_Game.GetPlayers(players);
}

// ❌ Avoid - Inconsistent
void MyMod_Module::OnInit()
{
	if (!g_Game)
		return;
	
	GetGame().GetPlayers(players);  // Mixed usage!
}
```

### 4. Use Function Result Immediately

```c
// ✅ Good - Always use g_Game with null check
if (g_Game && g_Game.IsDedicatedServer())
{
	// ...
}

// ❌ WRONG - Do not use GetGame()
if (GetGame().IsDedicatedServer())  // DO NOT USE
```

## Migration Guide

### Migration: Replace GetGame() with g_Game

**❌ Old Code (DO NOT USE):**
```c
DayZGame game = GetGame();
if (game.IsDedicatedServer())
{
	game.RPC(null, RPC_TEST, data, false);
}
```

**✅ New Code (REQUIRED):**
```c
if (!g_Game)
	return;

if (g_Game.IsDedicatedServer())
{
	g_Game.RPC(null, RPC_TEST, data, false);
}
```

**Migration Steps:**
1. Replace all `GetGame()` calls with `g_Game`
2. Add null checks: `if (!g_Game) return;`
3. Remove any `DayZGame game = GetGame();` variable assignments
4. Use `g_Game` directly with null checks

## Summary

**⚠️ CRITICAL: Always use `g_Game`, never use `GetGame()`**

**Required Standard:**
- **MUST use `g_Game` global variable** - This is the required standard
- **MUST check `g_Game` for null** before every use
- **NEVER use `GetGame()`** - This is deprecated and should not be used
- Direct access (faster than function call)
- Matches real-world mod patterns and DayZ best practices

**DayZ 1.28 (Legacy):**
- ~~Use `GetGame()` function~~ - **DO NOT USE**
- ~~Never null~~ - **DO NOT RELY ON THIS**
- Function call overhead

**DayZ 1.29+ (Current Standard):**
- **Use `g_Game` global variable** - **REQUIRED**
- **Must check for null** - **REQUIRED**
- Direct access (faster)

**Wiki Standard:**
- **This wiki uses `g_Game` consistently** for all examples
- **Always check `g_Game` for null before use** - **REQUIRED**
- This matches real-world mod patterns (e.g., bounty system, PvP mods)
- **Never use `GetGame()`** - Always use `g_Game` with null checks

**Best Practices:**
- **ALWAYS use `g_Game`** - Never use `GetGame()`
- **ALWAYS check `g_Game` for null** before use
- Be consistent throughout your mod
- Use `g_Game` for all DayZ versions

**Quick Reference:**
- ~~`GetGame()`~~ - **DO NOT USE** - Deprecated
- `g_Game` - **REQUIRED** - Must check null (used in this wiki)

---

**Related Guides:**
- [DayZ 1.28.161464](../DayZGame/DayZ-1.28.161464.md)
- [DayZ 1.29.161219](../DayZGame/DayZ-1.29.161219.md)
- [How to Create Profile Settings](../How-To/How-To-Profile-Settings.md)
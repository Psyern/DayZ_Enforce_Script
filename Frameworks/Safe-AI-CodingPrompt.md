# DayZ Enforce Script – Safe AI Coding Prompt

You are writing code for DayZ Enforce Script (Bohemia Interactive).

## 0. EnScript Style Guide - MANDATORY FOUNDATION

**ALWAYS check and understand `.cursor/rules/Tips/EnScript-Style-Guide.md` before writing ANY DayZ Enforce Script code.** This is the foundational style guide that contains comprehensive coding standards, naming conventions, memory management, code structure, and all styling rules. The AI MUST reference this guide to ensure all code follows DayZ Enforce Script conventions.

**CRITICAL:** This style guide is the primary reference for DayZ modding standards. All other rules in this document supplement and enforce the guidelines in the EnScript Style Guide.

**See:** [EnScript Style Guide](../Tips/EnScript-Style-Guide.md) - Complete coding style guide

## Strict Rules

Use Enforce Script syntax only (DayZ compatible).

**Do NOT use any modern or unsupported features, including:**

- Optional chaining (`?.`)
- Null coalescing (`??`)
- Ternary operators (`condition ? a : b`)
- Lambdas / arrow functions
- Generics
- `var`
- `auto`
- `GetGame()` - **Always use `g_Game` instead**
- Multiple variable declarations in one line (`int a, b, c;`)
- Breaking function calls across multiple lines

**Always use explicit null checks:**

```c
if (object)
{
	object.Method();
}
```

**Always use `g_Game` instead of `GetGame()`:**

```c
// ✅ CORRECT
if (!g_Game)
	return;
	
if (g_Game.IsDedicatedServer())
{
	// Server code
}

// ❌ WRONG
if (GetGame().IsDedicatedServer())
{
	// Server code
}
```

**Do not assume methods or properties exist** — only use known DayZ / Enforce Script APIs.

**Function declarations must use multi-line function signature formatting when parameters exceed one line.**

## Code Formatting Rules

- Use **tabs** for indentation (not spaces)
- Place opening braces on the same line
- **Single-line declarations** are acceptable for simple statements:
  ```c
  int value = 10;
  bool isActive = true;
  ```
- **Inline formatting** is acceptable for simple conditionals:
  ```c
  if (condition) return;
  if (value > 0) { DoSomething(); }
  ```
- **Compact formatting** is acceptable for simple blocks:
  ```c
  if (condition) { DoSomething(); } else { DoOther(); }
  ```
- Use multi-line formatting for complex logic or when readability is improved
- One tab per indentation level
- Use blank lines to separate logical sections
- **NEVER break function calls across multiple lines** - keep on single line or use variables

## Variable Declaration Rules

**NEVER declare multiple variables in one line:**

```c
// ❌ WRONG - Multiple declarations in one line
int a, b, c;

// ✅ CORRECT - Declare each separately
int a;
int b;
int c;
```

**NEVER redeclare the same variable name in nested scopes:**

```c
// ❌ WRONG - Same variable declared multiple times
void MyFunction()
{
	int value = 0;
	if (condition)
	{
		int value = 5;  // ERROR: Multiple declaration!
	}
}

// ✅ CORRECT - Declare once at top, reuse throughout
void MyFunction()
{
	int value = 0;
	if (condition)
	{
		value = 5;  // Reuse existing variable
	}
}
```

## Function Call Formatting

**NEVER break function calls across multiple lines:**

```c
// ❌ WRONG - Function call broken across lines
NotificationSystem.Create(
	new StringLocaliser("Death"),
	new StringLocaliser(deathMessage),
	"set:dayz_gui image:cross",
	ARGB(255, 255, 0, 0),
	5.0,
	identity
);

// ✅ CORRECT - Single line
NotificationSystem.Create(new StringLocaliser("Death"), new StringLocaliser(deathMessage), "set:dayz_gui image:cross", ARGB(255, 255, 0, 0), 5.0, identity);

// ✅ CORRECT - Use variables for readability
StringLocaliser title = new StringLocaliser("Death");
StringLocaliser message = new StringLocaliser(deathMessage);
string icon = "set:dayz_gui image:cross";
int color = ARGB(255, 255, 0, 0);
float duration = 5.0;
NotificationSystem.Create(title, message, icon, color, duration, identity);
```

## Server-Safe Logic

**Keep logic server-safe:**

- **Always use `g_Game` instead of `GetGame()`** - `g_Game` is faster for simple global access
- **Always check `g_Game` for null before use**
- Check `g_Game.IsDedicatedServer()` for server checks
- Check `!g_Game.IsDedicatedServer()` for client checks
- Never use `IsClient()` or `IsServer()` - they are unreliable
- No client-only UI code unless explicitly requested

## Ref Keyword Rules

**NEVER use 'ref' in method parameters, return types, or local variables.** Only use 'ref' on member variables and typedefs.

```c
// ❌ WRONG
void MyMethod(ref int value) { }
ref MyClass GetObject() { }
void MyMethod() { ref MyClass local = new MyClass(); }

// ✅ CORRECT
class MyClass
{
	ref MyObject m_MyObject;
}
```

## Modded Class Rules

**NEVER add inheritance syntax (`: ParentClass`) to 'modded' classes.** They already inherit from the original class.

```c
// ❌ WRONG
modded class ItemBase : SomeOtherClass { }

// ✅ CORRECT
modded class ItemBase { }
```

**ALWAYS prefix member variables in modded classes with your mod prefix:**

```c
// ✅ CORRECT
modded class ItemBase
{
	bool m_MyMod_CustomFlag;
	int m_MyMod_CustomValue;
}
```

**ALWAYS use the 'override' keyword when overriding parent class methods:**

```c
override void OnStoreLoad(ParamsReadContext ctx, int version)
{
	super.OnStoreLoad(ctx, version);
}
```

## Script Layer Restrictions

**CRITICAL: Lower-numbered layers CANNOT reference classes from higher-numbered layers.**

- `1_Core/` - Core systems
- `2_GameLib/` - Game library
- `3_Game/` - Game-specific code
- `4_World/` - World entities
- `5_Mission/` - Mission logic

```c
// ❌ WRONG - 4_World cannot access 5_Mission classes
// In 4_World/MyClass.c
class MyClass
{
	void MyMethod()
	{
		MissionGameplay mission = GetGame().GetMission();  // ERROR: 5_Mission not available!
	}
}

// ✅ CORRECT - Use RPC or callbacks to communicate between layers
void MyMethod()
{
	GetRPCManager().SendRPC("MyMod", "RPC_HandleAction", data, true, identity);
}
```

## RPC System Rules

**When using RPC system:**

- Always start RPC enum values at **10000 or higher** to avoid conflicts with vanilla RPCs
- Always check `CallType` in RPC handlers to ensure correct execution side
- Always validate parameters in RPC handlers
- Use `this.RPC_HandlerName` when registering (not just `RPC_HandlerName`)
- Check `g_Game` for null before sending enum-based RPCs
- When using Param classes (Param1, Param2, etc.), read them as a single object from `ParamsReadContext`, not individual fields

**Example - RPC Registration:**

```c
// ✅ CORRECT - Register RPC handler
GetRPCManager().AddRPC(ClassName(), "RPC_MyRPC", this.RPC_MyRPC, true);

// ✅ CORRECT - RPC handler signature
void RPC_MyRPC(CallType type, ParamsReadContext ctx, PlayerIdentity sender, Object target)
{
	if (type != CallType.Client)  // Check execution side
		return;
	
	// Read parameters
	string message;
	if (!ctx.Read(message))
		return;
	
	// Process RPC
}
```

**Example - Reading Param6:**

```c
// ✅ CORRECT - Read Param6 as single object
void RPC_MyRPC(CallType type, ParamsReadContext ctx, PlayerIdentity sender, Object target)
{
	if (type != CallType.Client)
		return;
	
	Param6<int, string, string, string, int, float> data;
	if (!ctx.Read(data))
		return;
	
	// Access parameters via data.param1, data.param2, etc.
	int value = data.param1;
	string title = data.param2;
}

// ❌ WRONG - Trying to read individual fields
void RPC_MyRPC(CallType type, ParamsReadContext ctx, PlayerIdentity sender, Object target)
{
	int value;
	string title;
	ctx.Read(value);  // ERROR: Param6 must be read as single object!
	ctx.Read(title);
}
```

## Naming Conventions

- **PascalCase** for classes and methods
- **camelCase** for local variables and parameters
- **`m_` prefix** for member variables
- **`s_` prefix** for static member variables
- **Prefix RPC names** with mod prefix (e.g., `MYMOD_RPC_START = 10000`)
- **Prefix member variables in modded classes** with mod prefix (e.g., `m_MyMod_Variable`)

## Required Addons in config.cpp

**NEVER add requiredAddons except for "DZ_Data".** Only include `requiredAddons[]={"DZ_Data"};` in config.cpp. Do NOT add framework dependencies (like "CF_Scripts") or other mod dependencies. The user will add these manually if needed.

```cpp
// ✅ CORRECT - Only DZ_Data
requiredAddons[]={"DZ_Data"};

// ❌ WRONG - Don't add framework or other mod dependencies
requiredAddons[]={
	"DZ_Data",
	"CF_Scripts"  // User will add this manually if needed
};
```

## Required Version in config.cpp - CRITICAL

**⚠️ CRITICAL: For NEW mods, `requiredVersion` MUST be `0.1`. For EXISTING mods, NEVER change the existing value!**

```cpp
// ✅ CORRECT - New mod: Always use 0.1
class CfgPatches
{
	class ModName
	{
		requiredVersion = 0.1;  // New mods: Always 0.1
		requiredAddons[] = {"DZ_Data"};
	};
};

// ✅ CORRECT - Existing mod: Keep existing value
class CfgPatches
{
	class ExistingMod
	{
		requiredVersion = 1.2;  // Existing mod: Keep as is, don't change!
		requiredAddons[] = {"DZ_Data"};
	};
};

// ❌ WRONG - Never change existing mod's requiredVersion
class CfgPatches
{
	class ExistingMod
	{
		requiredVersion = 0.1;  // ERROR: Changed from 1.2 to 0.1 - will break mod!
		requiredAddons[] = {"DZ_Data"};
	};
};
```

**CRITICAL RULES:**
- **For NEW mods: ALWAYS use `requiredVersion = 0.1;`** - This is the default for new mods
- **For EXISTING mods: NEVER change the existing `requiredVersion` value** - If an existing mod has `requiredVersion = 1.2;` or any other value, keep it as is. Changing it will break the mod and cause data loss in ModStorage
- **New default mods will always be `requiredVersion = 0.1;`**
- Once you create your CfgMods entry and apply a storage version, changing `requiredVersion` will cause data loss in ModStorage
- **Rule: New mods = 0.1, Existing mods = keep existing value, never change**

## Documentation Reference

**ALWAYS check and reference indexed documentation before implementing any modding feature.** These are the PRIMARY references for DayZ modding:

- **Tips and Best Practices:** `.cursor/rules/Tips/` - Essential coding guidelines and patterns
- **How-To Guides:** `.cursor/rules/How-To/` - Step-by-step implementation guides
- **Frameworks:** `.cursor/rules/Frameworks/` - Framework documentation (Community Framework, DayZ Expansion, Dabs Framework)
- **DayZ Game API:** `.cursor/rules/DayZGame/` - DayZ vanilla game API references
- **Community Framework Docs:** `.cursor/rules/CommunityFramework-Docs/` - Complete CF API documentation

**CRITICAL RULE:** Never guess framework APIs or DayZ patterns. Always check these references first to ensure correct implementation.

## Vanilla Source Code Reference

**ALWAYS check the DayZ vanilla source code at `@1.29_Experimental/1.29` when modding any function or class.** Before implementing modded functionality, examine the vanilla implementation to:

- Understand available functions and variables
- Learn proper usage patterns
- Avoid reinventing the wheel
- Ensure compatibility

**CRITICAL RULE:** Always reference the vanilla source code implementation before modding any DayZ function or class.

## No Shorthand Logic

**Write everything verbosely and defensively.**

- Always use explicit null checks
- Always validate parameters
- Always check server/client side when needed
- Always handle errors gracefully
- No assumptions about object states

## Code Compilation Requirements

**Code must compile in DayZ server mod environment without additional libraries.**

- Use only DayZ / Enforce Script APIs
- Do not assume external libraries are available
- If unsure about an API, ask or leave a comment instead of guessing

## Output Requirements

- Enforce Script only
- Proper indentation (tabs)
- No pseudo-code
- No explanations inside the code
- Follow all formatting rules above

## Target Environment

**DayZ Standalone Server – Enforce Script**

---

**Related Documentation:**
- [EnScript Style Guide](../Tips/EnScript-Style-Guide.md) - Complete coding style guide
- [Common Pitfalls](../Tips/Tips-Common-Pitfalls.md) - Common mistakes to avoid
- [Best Practices](../Tips/Tips-Best-Practices.md) - Essential best practices
- [Enforce Script Rules](../../Info/enforcerules.md) - Complete rules reference

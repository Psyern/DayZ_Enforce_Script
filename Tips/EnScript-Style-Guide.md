# EnScript (Enforce Script) Style Guide

Complete coding style guide for DayZ modding based on conventions observed in the official Bohemia Interactive DayZ Script Diff and the DayZ Expansion mod codebase.

## Introduction

EnScript (Enforce Script) is the object-oriented scripting language used by the Enfusion engine in DayZ Standalone. This style guide provides a comprehensive overview of coding standards and best practices for DayZ modding.

**Note:** This guide provides an overview. For detailed information on specific topics, see the dedicated guides linked below.

---

## File Organization

### File Extensions

* Use `.c` extension for all EnScript files
* File names should match the primary class name when possible

### Directory Structure

Scripts are typically organized by layer inside a root folder called `Scripts/`:

* `1_Core/` - Core systems
* `2_GameLib/` - Game library
* `3_Game/` - Game-specific code
* `4_World/` - World entities
* `5_Mission/` - Mission logic

**See:** [Script Layers Guide](../How-To/Script-Layers-Guide.md) for detailed information on what code belongs in each layer.

---

## Naming Conventions

### Classes

**PascalCase** for all class names:

```c
class ExpansionGarageSettings
class PlayerBase
class ItemBase
class ExpansionAIPatrolBase
```

**Prefixes:**

* Module/mod-specific prefixes are common (e.g., `Expansion`, `CF_`, `MyMod_`)
* Base classes often end with `Base`
* Abstract classes may start with `Abstract`

**Modded Classes:**

* **ALWAYS prefix member variables** in modded classes or classes inheriting from vanilla/other mods:

```c
// ✅ CORRECT - Prefixed members in modded class
modded class ItemBase
{
	bool m_Expansion_CustomFlag;
	int m_Expansion_CustomValue;
}

// ✅ CORRECT - Prefixed members in inherited class
class ExpansionSomeItem : ItemBase
{
	bool m_Expansion_SpecialProperty;
}
```

This prevents name conflicts with vanilla code or other mods.

**See:** [Tips: Naming Conventions and Prefixes](Tips/Tips-Prefixes-Naming.md) for detailed prefix guidelines.

### Variables

**Member Variables:**

* Use `m_` prefix for private/protected member variables
* Use PascalCase after the prefix

```c
bool m_IsLoaded;
ExpansionMarkerModule m_MarkerModule;
ref MapMenu m_Expansion_MapMenu;
```

**Static Variables:**

* Use `s_` prefix for static member variables
* Use PascalCase after the prefix

```c
static ref array<EffectArea> s_Expansion_DangerousAreas;
```

**Local Variables:**

* Use camelCase for local variables
* No prefix required

```c
int bitmask;
string lootingBehavior;
vector hitPosition;
```

### Methods/Functions

**PascalCase** for method names:

```c
void SetDefaultLootingBehaviour()
int GetLootingBehaviour()
void OnSend(ParamsWriteContext ctx)
bool OnRecieve(ParamsReadContext ctx)
```

**Override Methods:**

* Always use the `override` keyword when overriding parent methods

```c
override void OnSend(ParamsWriteContext ctx)
override bool OnRecieve(ParamsReadContext ctx)
```

**Proto Methods vs Custom Methods:**

When available, prefer using proto (native engine) methods over custom script implementations **when the native method performs expensive logic**. Proto methods are faster for complex operations as they run in native code.

```c
// ✅ PREFERRED - Proto method for expensive operations
proto native vector GetPosition();

// ❌ AVOID - Custom reimplementation of expensive native logic
vector GetPosition() 
{
	// Custom expensive calculation
}
```

**Obsolete Attribute:**

If someone is writing a mod that will likely be modded/overridden, suggest using the Obsolete attribute on a method/field/variable instead of removing them altogether. The workbench will show a warning when used.

```c
[Obsolete()]
vector GetCustomPosition() 
{
	// Custom expensive calculation
}
```

**Specific use note:** If at all possible, avoid using `GetObjectsAtPosition` and `GetObjectsAtPosition3D` as much as possible. Static arrays on the class you're looking for, triggers, or the GetScene methods are all much better options available.

**Important Note:** There is overhead from calling native methods from script. For simple operations, custom script methods may actually be faster. This is why `proto native CGame GetGame()` was replaced with `DayZGame GetGame() { return g_Game; }` - the script version is faster for simple global access.

**Rule:** Use proto methods when they perform complex/expensive operations. For simple operations (like returning a global), script implementations are often faster.

### Parameters

Use camelCase for parameters:

```c
void Log(string msg)
void SetValue(int value, bool isEnabled)
static vector GetPlacementPosition(vector pos)
```

### Constants and Defines

**Constants:**

Use `const` for class-specific constants, or enums for grouped constants:

```c
class VehicleManager
{
	const int MAX_VEHICLES = 10;
	const float SEARCH_RADIUS = 100.0;
}
```

**Enums:**

Use enums to group related constants - **avoid creating many similarly named global constants**:

```c
// ✅ PREFERRED - Grouped with enum
enum EVehicleType
{
	Car,
	Truck,
	Helicopter
}

// ✅ ALSO ACCEPTABLE - PASCAL_CASE enums
enum EVehicleType
{
	CAR,
	TRUCK,
	HELICOPTER
}

// ❌ AVOID - Multiple global constants
const int VEHICLE_TYPE_CAR = 0;
const int VEHICLE_TYPE_TRUCK = 1;
const int VEHICLE_TYPE_HELICOPTER = 2;
```

**Enum Naming:**

* **CamelCase** is generally nicer to read: `enum EStatus { Idle, Running, Stopped }`
* **PASCAL_CASE** can be used for clarity in some cases: `enum EStatus { IDLE, RUNNING, STOPPED }`
* Choose one style and be consistent within your codebase

**See:** [How to Use Enums](How-To/How-To-Enums.md) for detailed enum guide.

**Preprocessor Defines:**

Use `#define` for global constants and feature flags:

```c
#define MAX_PLAYERS 60
#define EXPANSION_DEBUG
```

---

## Code Structure

**See:** [Tips: Code Structure](Tips/Tips-Code-Structure.md) for detailed code structure guidelines.

### Quick Reference

* Use **tabs** for indentation (not spaces)
* One tab per indentation level
* Place opening braces on the same line
* Use blank lines to separate logical sections

---

## Type System

**See:** [Tips: Type System](Tips/Tips-Type-System.md) for detailed type system information.

### Quick Reference

* **Primitive Types:** `int, float, bool, string, vector`
* **Type vs Instance:** Types are class definitions, instances are objects
* **Type Casting:** Use `.Cast()` for safe conversion
* **Collections:** `array<T>`, `map<K,V>`, `TStringArray`

**See:** [Tips: Memory Management](Tips/Tips-Memory-Management.md) for detailed `ref` usage guide.

---

## Memory Management

**See:** [Tips: Memory Management](Tips/Tips-Memory-Management.md) for comprehensive memory management guide.

### Quick Reference

* **Object Creation:** `MyClass obj = new MyClass();`
* **Garbage Collection:** Automatic, aggressive for function-scope instances
* **Deletion:** Never use `delete`, set to `null` instead
* **ref Rules:** Only on member variables and typedefs

---

## Control Flow

**See:** [Tips: Code Structure](Tips/Tips-Code-Structure.md) for detailed control flow examples.

### ⚠️ Important: Ternary Operator Not Supported

**DayZ EnScript does NOT support the ternary operator (`? :`).**

```c
// ❌ WRONG - Causes compilation error: "Broken expression (missing ';'?)"
string result = condition ? "true" : "false";

// ✅ CORRECT - Use if-else statements
string result;
if (condition)
{
	result = "true";
}
else
{
	result = "false";
}
```

**See:** [Tips: Common Pitfalls](Tips/Tips-Common-Pitfalls.md#ternary-operator-not-supported) for detailed information.

---

## Preprocessor Directives

**See:** [Tips: Preprocessor Directives](Tips/Tips-Preprocessor-Directives.md) for detailed preprocessor guide.

---

## Comments

**See:** [Tips: Comments](Tips/Tips-Comments.md) for detailed comment guidelines.

---

## Modding Patterns

### Modded Classes

```c
modded class ItemBase
{
	override void OnStoreLoad(ParamsReadContext ctx, int version)
	{
		super.OnStoreLoad(ctx, version);
		// Custom logic
	}
}
```

**Important:** `modded` classes already inherit from the original class. **NEVER** add inheritance syntax:

```c
// ❌ WRONG
modded class ItemBase : SomeOtherClass { }

// ✅ CORRECT
modded class ItemBase { }
```

**See:** [Tips: Modded Classes](Tips/Tips-Modded-Classes.md) for comprehensive modded class guide.

### Accessing Private Members

Use the class name prefix to access private members:

```c
modded class ItemBase
{
	void MyMethod()
	{
		ItemBase.m_SomePrivateMember = value;
	}
}
```

### Calling Parent Methods

Use `super` to call parent class methods:

```c
override void OnStoreLoad(ParamsReadContext ctx, int version)
{
	super.OnStoreLoad(ctx, version);
	// Additional logic
}
```

### Global Variables and Singletons

**Avoid Creating New Globals:**

Prefer using existing global variables or creating module-based singletons:

```c
// ✅ PREFERRED - Use existing global
if (!g_Game)
	return;

DayZGame game = g_Game;

// ❌ AVOID - Creating new global
int g_MyGlobalValue = 0;
```

### Accessing Globals

```c
// Always check g_Game for null first
if (!g_Game)
	return;

DayZGame game = g_Game;
Mission mission = g_Game.GetMission();
```

**See:** [Tips: g_Game vs GetGame()](Tips/Tips-g_Game-GetGame.md) for when to use which.

### Property Access

**Direct Property Access vs Getters:**

In EnScript, direct property access is common and acceptable:

```c
// Both are acceptable
player.m_Health = 100;
player.SetHealth(100);
```

### Serialization

**Common Pattern:**

```c
override void OnStoreSave(ParamsWriteContext ctx)
{
	super.OnStoreSave(ctx);
	ctx.Write(m_CustomValue);
}

override void OnStoreLoad(ParamsReadContext ctx, int version)
{
	super.OnStoreLoad(ctx, version);
	ctx.Read(m_CustomValue);
}
```

### Input Handling

**Standard Pattern:**

```c
override bool OnInput(InputEvent e)
{
	if (e.IsKeyPress(KeyCode.KC_E))
	{
		// Handle input
		return true;
	}
	return super.OnInput(e);
}
```

### Module System

**Module Registration:**

```c
class MyModule : ExpansionModule
{
	override void OnInit()
	{
		super.OnInit();
		// Module initialization
	}
}
```

**See:** [Module System](How-To/Module-System.md) for comprehensive module guide.

**Module Callbacks:**

```c
override void OnUpdate(float delta_time)
{
	super.OnUpdate(delta_time);
	// Update logic
}
```

---

## Debugging and Tracing

**See:** [Tips: Debugging](Tips/Tips-Debugging.md) for comprehensive debugging guide.

---

## Best Practices

**See:** [Tips: Best Practices](Tips/Tips-Best-Practices.md) for comprehensive best practices guide.

---

## Common Pitfalls and Issues

**See:** [Tips: Common Pitfalls](Tips/Tips-Common-Pitfalls.md) for comprehensive pitfalls guide.

---

## Conclusion

This style guide reflects the conventions used in professional DayZ modding. Consistency with these patterns will make your code more maintainable and easier to integrate with existing mods and the vanilla codebase.

### Key Takeaways

* Use PascalCase for classes and methods
* Use `m_` prefix for member variables, `s_` for static
* Use tabs for indentation
* Always use `override` keyword when overriding
* Check for null before using objects
* Document complex logic with comments
* Use preprocessor directives for conditional compilation
* Follow the modded class pattern for extending vanilla code
* Where possible, use proto methods over custom methods for expensive logic
* CfgMod classnames are defines in script as of 1.26
* **NEVER use `ref` in method parameters, return types, or local variables** - only on member variables and templates
* **NEVER add inheritance (`: ParentClass`) to `modded` classes** - they already inherit from the original class
* **NEVER declare multiple variables in one line** - declare each separately
* **NEVER use ternary operators (`? :`)** - use if-else statements instead
* **NEVER break function calls across multiple lines** - keep on single line or use variables
* **ALWAYS check indexed documentation** before implementing features
* **ALWAYS check vanilla source code** at `@1.29_Experimental/1.29` before modding
* **NEVER add requiredAddons except DZ_Data** - let users add dependencies manually

### References

* DayZ Script Diff (Bohemia Interactive)
* DayZ Expansion Scripts
* Bohemia Interactive Community Wiki - Enforce Script

---

**Related Wiki Guides:**

### Detailed Guides
- [Tips: Naming Conventions and Prefixes](Tips/Tips-Prefixes-Naming.md) - Detailed prefix guidelines
- [Tips: Code Structure](Tips/Tips-Code-Structure.md) - Code organization and control flow
- [Tips: Type System](Tips/Tips-Type-System.md) - Types, collections, and casting
- [Tips: Memory Management](Tips/Tips-Memory-Management.md) - Comprehensive `ref` usage
- [Tips: Preprocessor Directives](Tips/Tips-Preprocessor-Directives.md) - Conditional compilation
- [Tips: Comments](Tips/Tips-Comments.md) - Comment styles and documentation
- [Tips: Modded Classes](Tips/Tips-Modded-Classes.md) - How to extend vanilla classes
- [Tips: g_Game vs GetGame()](Tips/Tips-g_Game-GetGame.md) - Global access patterns
- [Tips: Debugging](Tips/Tips-Debugging.md) - Debugging techniques
- [Tips: Best Practices](Tips/Tips-Best-Practices.md) - Essential best practices
- [Tips: Common Pitfalls](Tips/Tips-Common-Pitfalls.md) - Common mistakes to avoid

### How-To Guides
- [How to Use Enums](How-To/How-To-Enums.md) - Enum creation and usage
- [How to Create a Logger](How-To/How-To-Logger.md) - Logging implementation
- [Module System](How-To/Module-System.md) - Module creation and management
- [Script Layers Guide](../How-To/Script-Layers-Guide.md) - Understanding where code belongs
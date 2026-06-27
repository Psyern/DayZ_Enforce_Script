# Tips: Common Pitfalls

Common mistakes and how to avoid them in DayZ modding.

## Using GetGame() Instead of g_Game

**❌ CRITICAL: Always use `g_Game`, never use `GetGame()`.**

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

**Why:** `GetGame()` is deprecated and should not be used. Always use `g_Game` global variable with proper null checks.

**Rule:** Always use `g_Game` and check for null before use. Never use `GetGame()`.

**See:** [Tips: g_Game vs GetGame()](Tips-g_Game-GetGame.md) for detailed information.

## Multiple Variable Declaration - COMPILATION ERROR

**❌ CRITICAL: Multiple variable declaration in one line is NOT allowed. Declare each variable separately at the top of the scope and reuse it throughout. NEVER declare the same variable name multiple times in the same function scope - this causes compilation errors.**

```c
// ❌ WRONG - Multiple declarations in one line
int a, b, c;

// ❌ WRONG - Same variable declared multiple times in same function
void MyFunction()
{
	int value = 0;
	if (condition)
	{
		int value = 5;  // ERROR: Multiple declaration!
	}
}

// ✅ CORRECT - Declare once at top, reuse
int a;
int b;
int c;

// ✅ CORRECT - Declare once at top of function, reuse throughout
void MyFunction()
{
	int value = 0;
	if (condition)
	{
		value = 5;  // Reuse existing variable
	}
}
```

**Error Message:**
```
Variable 'value' already declared
```

**Why:** EnScript doesn't support multiple variable declarations in one line, and redeclaring the same variable name in nested scopes causes compilation errors.

**Rule:** Always declare each variable separately. Declare variables once at the top of their scope and reuse them throughout. Never redeclare the same variable name in nested scopes.

## Ternary Operator Not Supported

**❌ CRITICAL: DayZ EnScript does NOT support the ternary operator (`? :`).**

```c
// ❌ WRONG - Ternary operator causes compilation error
string result = condition ? "true" : "false";
int value = isEnabled ? 1 : 0;
string side = GetGame().IsServer() ? "SERVER" : "CLIENT";

// ✅ CORRECT - Use if-else statements instead
string result;
if (condition)
{
	result = "true";
}
else
{
	result = "false";
}

int value;
if (isEnabled)
{
	value = 1;
}
else
{
	value = 0;
}

string side;
if (GetGame().IsServer())
{
	side = "SERVER";
}
else
{
	side = "CLIENT";
}
```

**Error Message:**
```
Broken expression (missing ';'?)
```

**Why:** The EnScript compiler doesn't recognize the `?` character as a valid operator, causing it to misinterpret the expression and report confusing error messages.

**Rule:** Always use `if-else` statements instead of ternary operators in DayZ scripts.

## Compiler Error Messages

**⚠️ Misleading Error Locations:**

Compile errors involving:

* Undefined classes (missing addon not loaded)
* Variable name conflicts
* Ternary operators (unsupported syntax)

Will **NOT** show the correct filename and line number. Instead, they show the last successfully parsed `.c` file at EOF, which is misleading and unhelpful.

**What to do:**

* If error location seems wrong, search for the class/variable name in your entire codebase
* Check that all required addons are loaded
* Look for name conflicts across files
* Check for ternary operators (`? :`) and replace with if-else statements

## Common Segfault Causes

1. **Complex array assignments** - Use intermediate variables
2. **Empty preprocessor blocks** - Must contain at least one statement
3. **Misused `delete` keyword** - Let garbage collection handle it
4. **Over-specified `ref` usage** - Only use on members/typedefs

### Complex Array Assignments

**⚠️ Segfault Risk:**

```c
class MyClass
{
	bool m_IsInside[3];
	
	// ❌ WRONG - Causes segfault
	void TestBad(int index, vector a, vector b, float distSq)
	{
		m_IsInside[index] = vector.DistanceSq(a, b) <= distSq;
	}
	
	// ✅ CORRECT - Use intermediate variable
	void TestGood(int index, vector a, vector b, float distSq)
	{
		bool isInside = vector.DistanceSq(a, b) <= distSq;
		m_IsInside[index] = isInside;
	}
}
```

**Rule:** For array assignments with complex expressions, use an intermediate variable.

### Empty Preprocessor Blocks

```c
// ❌ WRONG - Empty block causes segfault
#ifdef DEBUG
#endif

// ✅ CORRECT - Must have at least one statement
#ifdef DEBUG
	// Debug code
#endif
```

### Misused Delete Keyword

```c
// ❌ WRONG
delete myObject;

// ✅ CORRECT - Let garbage collection handle it
myObject = null;
```

## Integer Comparison Quirks

**⚠️ Integer Comparison Quirk:**

```c
// int.MIN = -2147483648

1 < int.MIN;      // ⚠️ Returns TRUE in EnScript!
1 < -2147483647;  // ⚠️ Also returns TRUE!

// Always be careful with MIN/MAX integer comparisons
```

## Incorrect ref Usage

**Common Mistakes:**

```c
// ❌ WRONG - ref in parameter
void MyMethod(ref MyClass obj) { }

// ❌ WRONG - ref in return type
ref MyClass GetObject() { }

// ❌ WRONG - ref in local variable
void MyMethod()
{
	ref MyClass localObj = new MyClass();  // ❌
}
```

**See:** [Tips: Memory Management](Tips-Memory-Management.md) for correct `ref` usage.

## Modded Class Inheritance

**⚠️ Common Mistake:**

```c
// ❌ WRONG - modded classes already inherit
modded class ItemBase : SomeOtherClass { }

// ✅ CORRECT - No inheritance syntax needed
modded class ItemBase { }
```

**See:** [Tips: Modded Classes](Tips-Modded-Classes.md) for correct usage.

## Client/Server Detection

**⚠️ Common Mistakes:**

```c
// ❌ WRONG - Returns true on client during load
if (g_Game && g_Game.IsClient())

// ✅ CORRECT - Reliable client check
if (g_Game && !g_Game.IsDedicatedServer())

// ❌ WRONG - Returns true on client during load
if (g_Game && g_Game.IsServer())

// ✅ CORRECT - Reliable server check (dedicated only)
if (g_Game && g_Game.IsDedicatedServer())
```

**See:** [Tips: g_Game vs GetGame()](Tips-g_Game-GetGame.md) for detailed client/server detection.

## Performance Pitfalls

* **Creating temporary objects in tight loops** - Causes excessive garbage collection
* **Not using ref on member variables** - Objects get garbage collected prematurely
* **Using GetObjectsAtPosition** - Very expensive, use static arrays or triggers instead

---

**Related Guides:**
- [EnScript Style Guide](EnScript-Style-Guide.md) - Complete style reference
- [Tips: Memory Management](Tips-Memory-Management.md) - Memory pitfalls
- [Tips: Debugging](Tips-Debugging.md) - Debugging techniques
- [Tips: Best Practices](Tips-Best-Practices.md) - Best practices
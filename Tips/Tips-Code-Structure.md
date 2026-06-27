# Tips: Code Structure

Best practices for organizing and structuring EnScript code.

## Class Declaration

```c
class MyClass : ParentClass
{
	// Member variables
	protected bool m_IsInitialized;
	
	// Constructor
	void MyClass()
	{
		m_IsInitialized = false;
	}
	
	// Methods
	void Initialize()
	{
		m_IsInitialized = true;
	}
}
```

## Indentation and Spacing

* Use **tabs** for indentation (not spaces)
* One tab per indentation level
* Place opening braces on the same line
* Use blank lines to separate logical sections

```c
void MyMethod()
{
	if (condition)
	{
		DoSomething();
	}
	else
	{
		DoSomethingElse();
	}
}
```

## Control Flow

### Conditionals

**⚠️ Important: DayZ EnScript does NOT support ternary operators (`? :`). Always use if-else statements.**

```c
if (condition)
{
	DoSomething();
}
else if (otherCondition)
{
	DoSomethingElse();
}
else
{
	DoDefault();
}
```

**Ternary Operator Alternative:**

```c
// ❌ WRONG - Ternary operator not supported
string result = condition ? "yes" : "no";

// ✅ CORRECT - Use if-else
string result;
if (condition)
{
	result = "yes";
}
else
{
	result = "no";
}
```

### Loops

**For Loop:**

```c
for (int i = 0; i < array.Count(); i++)
{
	ProcessItem(array[i]);
}
```

**While Loop:**

```c
while (condition)
{
	DoSomething();
}
```

### Foreach Loops

```c
foreach (ItemBase item : items)
{
	ProcessItem(item);
}
```

### Switch Statements

```c
switch (value)
{
	case 1:
		DoFirst();
		break;
	case 2:
		DoSecond();
		break;
	default:
		DoDefault();
		break;
}
```

### Array Bounds Checking

Always check bounds before accessing array elements:

```c
if (index >= 0 && index < array.Count())
{
	ProcessItem(array[index]);
}
```

## Function Call Formatting

**❌ CRITICAL: NEVER break function calls across multiple lines. Keep function calls on a single line. If parameters are too long, format them inline or use variables.**

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

**Why:** Breaking function calls across multiple lines can cause compilation errors or unexpected behavior in EnScript.

**Rule:** Always keep function calls on a single line. If parameters are too long, extract them to variables first, then call the function with those variables.

## Best Practices

### 1. Keep Methods Focused

Each method should do one thing well:

```c
void SetDefaultLootingBehaviour()
{
	if (Behaviour == "ROAMING")
		LootingBehaviour = "ALL";
	else
		LootingBehaviour = "DEFAULT";
}
```

### 2. Use Constants for Magic Numbers

```c
// Good
const int DEFAULT_MAX_VEHICLES = 10;
MaxStorableVehicles = DEFAULT_MAX_VEHICLES;

// Avoid
MaxStorableVehicles = 10;
```

### 3. Initialize Member Variables

Initialize in constructor or at declaration:

```c
void ExpansionGarageSettings()
{
	EntityWhitelist = new TStringArray;
	m_IsLoaded = false;
}
```

### 4. Document Complex Logic

```c
//! Toggle between hidden and previously set visibility state for each marker category
int visibility;
int previousVisibility;

visibility |= m_MarkerModule.GetVisibility(ExpansionMapMarkerType.SERVER);
previousVisibility |= m_MarkerModule.GetPreviousVisibility(ExpansionMapMarkerType.SERVER);
```

---

**Related Guides:**
- [EnScript Style Guide](EnScript-Style-Guide.md) - Complete style reference
- [Tips: Naming Conventions](Tips-Prefixes-Naming.md) - Naming guidelines
- [Tips: Memory Management](Tips-Memory-Management.md) - Memory best practices
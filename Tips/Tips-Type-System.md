# Tips: Type System

Understanding EnScript types, collections, and type casting.

## Primitive Types

```
int, float, bool, string, vector
```

## Understanding Types and Instances

**Important:** In EnScript, types and instances are different concepts:

* **Type:** The class definition itself (e.g., `ItemBase`)
* **Instance:** An actual object of that type (e.g., `myItem`)

```c
// Type reference
Class myClass = ItemBase;

// Instance reference
ItemBase myItem = ItemBase.Cast(entity);
```

## Collections

### Arrays

```c
array<int> numbers;
array<ref MyClass> objects;
TStringArray strings;
```

### Maps

```c
map<string, int> nameToId;
```

## Type Casting

Use `.Cast()` for safe type conversion:

```c
PlayerBase player = PlayerBase.Cast(entity);
if (player)
{
	// Safe to use player
}
```

## Leverage Type Safety

```c
// Use enums
eAILootingBehavior behavior;

// Use proper casting
PlayerBase player = PlayerBase.Cast(entity);
if (player)
{
	// Safe to use player
}
```

---

**Related Guides:**
- [EnScript Style Guide](EnScript-Style-Guide.md) - Complete style reference
- [How to Use Enums](../How-To/How-To-Enums.md) - Creating and using enums
- [Tips: Memory Management](Tips-Memory-Management.md) - Understanding ref
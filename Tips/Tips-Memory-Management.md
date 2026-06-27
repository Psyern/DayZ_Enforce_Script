# Tips: Memory Management

Understanding references, `ref`, and garbage collection in EnScript.

## Overview

EnScript uses automatic garbage collection. Objects are automatically destroyed when no longer referenced. However, you need to understand when to use `ref` to prevent premature garbage collection.

## The `ref` Keyword - CRITICAL

### ✅ CORRECT Usage of `ref`

**ONLY use `ref` on:**

1. **Member variables** (to prevent garbage collection)
2. **Static member variables**
3. **In typedef declarations**

```c
class MyMod_Module
{
	// ✅ CORRECT - ref on member variable
	ref MyMod_Settings m_Settings;
	
	// ✅ CORRECT - ref on static member
	static ref array<ItemBase> s_Items;
	
	// ✅ CORRECT - ref in typedef
	typedef ref array<MyMod_Item> MyMod_ItemArray;
}
```

### ❌ WRONG Usage of `ref`

**NEVER use `ref` on:**

1. **Method parameters**
2. **Return types**
3. **Local variables**

```c
// ❌ WRONG - ref in parameter
void MyMethod(ref MyClass obj) { }

// ❌ WRONG - ref in return type
ref MyClass GetObject() { }

// ❌ WRONG - ref in local variable
void MyMethod()
{
	ref MyClass localObj = new MyClass();  // ❌ NO!
	
	// ✅ CORRECT - No ref on local
	MyClass localObj = new MyClass();  // ✅ Correct
}
```

## Why `ref` Matters

### Without `ref` on Members

```c
class MyMod_Module
{
	// ❌ WRONG - No ref on member
	MyMod_Settings m_Settings;  // May be garbage collected!
	
	void OnInit()
	{
		m_Settings = new MyMod_Settings();  // Created
	}
	
	// Later, m_Settings may be garbage collected if no other references
}
```

### With `ref` on Members

```c
class MyMod_Module
{
	// ✅ CORRECT - ref on member
	ref MyMod_Settings m_Settings;  // Protected from GC
	
	void OnInit()
	{
		m_Settings = new MyMod_Settings();  // Created and protected
	}
	
	// m_Settings stays alive as long as the module exists
}
```

## Reference Rules Summary

### Quick Reference

| Location | Use `ref`? | Example |
|----------|------------|---------|
| **Member variable** | ✅ YES | `ref MyClass m_Instance;` |
| **Static member** | ✅ YES | `static ref array<int> s_Items;` |
| **Typedef** | ✅ YES | `typedef ref array<Item> ItemArray;` |
| **Method parameter** | ❌ NO | `void Method(MyClass obj)` |
| **Return type** | ❌ NO | `MyClass GetClass()` |
| **Local variable** | ❌ NO | `MyClass local = new MyClass();` |

## Common Patterns

### Pattern 1: Member Variable

```c
class MyMod_Module
{
	// ✅ CORRECT - ref on member
	ref MyMod_Settings m_Settings;
	
	void OnInit()
	{
		// Create instance - ref keeps it alive
		m_Settings = new MyMod_Settings();
	}
}
```

### Pattern 2: Static Member

```c
class MyMod_Manager
{
	// ✅ CORRECT - ref on static member
	static ref MyMod_Manager s_Instance;
	
	static MyMod_Manager GetInstance()
	{
		if (!s_Instance)
		{
			s_Instance = new MyMod_Manager();
		}
		return s_Instance;
	}
}
```

### Pattern 3: Array of References

```c
class MyMod_Module
{
	// ✅ CORRECT - ref array as member
	ref array<ref MyMod_Item> m_Items;
	
	void OnInit()
	{
		m_Items = new array<ref MyMod_Item>;
		
		// Items stay alive because of ref array
		MyMod_Item item = new MyMod_Item();
		m_Items.Insert(item);
	}
}
```

### Pattern 4: Local Variable (No ref)

```c
void MyMethod()
{
	// ✅ CORRECT - No ref on local
	MyMod_Settings settings = new MyMod_Settings();
	settings.Load();
	
	// settings will be garbage collected after method ends
	// This is fine for temporary objects
}
```

## Garbage Collection

### How Garbage Collection Works

EnScript uses **aggressive garbage collection**:

- Objects are destroyed immediately when no longer referenced
- Function-scope instances are collected when function ends
- Member variables with `ref` are protected from collection
- Set objects to `null` to allow collection

### Example: Garbage Collection

```c
void MyMethod()
{
	// Created in function scope
	MyMod_Settings settings = new MyMod_Settings();
	
	// Use settings
	settings.Load();
	
} // settings may be garbage collected here

class MyMod_Module
{
	ref MyMod_Settings m_Settings;  // Protected with ref
	
	void OnInit()
	{
		m_Settings = new MyMod_Settings();
	}
	
	// m_Settings stays alive as long as module exists
}
```

## Setting to null

### ✅ CORRECT - Set to null When Done

```c
class MyMod_Module
{
	ref MyMod_Manager m_Manager;
	
	void Cleanup()
	{
		if (m_Manager)
		{
			m_Manager.Cleanup();
			m_Manager = null;  // ✅ Allow garbage collection
		}
	}
}
```

### ❌ WRONG - Using delete

```c
// ❌ WRONG - Never use delete
delete myObject;  // NO! This doesn't exist in EnScript!

// ✅ CORRECT - Set to null
myObject = null;  // Let garbage collection handle it
```

## Common Mistakes

### Mistake 1: ref in Parameter

```c
// ❌ WRONG - ref in parameter
void ProcessItem(ref ItemBase item)
{
	// This causes compiler errors!
}

// ✅ CORRECT - No ref in parameter
void ProcessItem(ItemBase item)
{
	// Correct!
}
```

### Mistake 2: ref in Return Type

```c
// ❌ WRONG - ref in return type
ref MyClass GetClass()
{
	return new MyClass();
}

// ✅ CORRECT - No ref in return type
MyClass GetClass()
{
	return new MyClass();
}
```

### Mistake 3: ref on Local Variable

```c
// ❌ WRONG - ref on local
void MyMethod()
{
	ref MyClass obj = new MyClass();  // NO!
}

// ✅ CORRECT - No ref on local
void MyMethod()
{
	MyClass obj = new MyClass();  // Correct!
}
```

### Mistake 4: Missing ref on Member

```c
class MyMod_Module
{
	// ❌ WRONG - Missing ref on member
	MyMod_Settings m_Settings;  // May be garbage collected!
	
	void OnInit()
	{
		m_Settings = new MyMod_Settings();
		// Later, m_Settings may be collected!
	}
}

// ✅ CORRECT - ref on member
class MyMod_Module
{
	ref MyMod_Settings m_Settings;  // Protected!
	
	void OnInit()
	{
		m_Settings = new MyMod_Settings();
		// m_Settings stays alive
	}
}
```

## Best Practices

### 1. Use ref on All Member Variables

```c
// ✅ Good - All members have ref
class MyMod_Module
{
	ref MyMod_Settings m_Settings;
	ref MyMod_Manager m_Manager;
	ref array<ItemBase> m_Items;
}

// ❌ Avoid - Missing ref on some members
class MyMod_Module
{
	ref MyMod_Settings m_Settings;  // Has ref
	MyMod_Manager m_Manager;  // Missing ref!
}
```

### 2. Use ref on Static Members

```c
// ✅ Good - ref on static member
class MyMod_Manager
{
	static ref MyMod_Manager s_Instance;
	
	static MyMod_Manager GetInstance()
	{
		if (!s_Instance)
		{
			s_Instance = new MyMod_Manager();
		}
		return s_Instance;
	}
}
```

### 3. Set to null When Done

```c
// ✅ Good - Clean up references
void Cleanup()
{
	if (m_Manager)
	{
		m_Manager.Cleanup();
		m_Manager = null;  // Allow GC
	}
}
```

### 4. Never Use delete

```c
// ❌ Never use delete
delete myObject;  // This doesn't exist!

// ✅ Use null instead
myObject = null;
```

## Summary

**`ref` Usage:**
- ✅ **YES** on member variables
- ✅ **YES** on static members
- ✅ **YES** in typedefs
- ❌ **NO** in method parameters
- ❌ **NO** in return types
- ❌ **NO** on local variables

**Memory Management:**
- EnScript uses automatic garbage collection
- `ref` protects objects from collection
- Set to `null` when done with objects
- Never use `delete` keyword

**Quick Rules:**
- Member variables → `ref`
- Parameters → No `ref`
- Return types → No `ref`
- Local variables → No `ref`
- Set to `null` when done
- Never use `delete`

---

**Related Guides:**
- [Tips: Modded Classes](Tips-Modded-Classes.md)
- [EnScript Style Guide](EnScript-Style-Guide.md)
# How to Use CF ModStorage

Guide to using Community Framework's ModStorage system for persisting data across server restarts.

## Overview

CF ModStorage allows you to save and load custom data on items, players, and other entities. This data persists across server restarts and is stored separately from vanilla DayZ save data.

**⚠️ CRITICAL:** Once you create your `CfgMods` entry and apply a storage version, it will be impossible to rename the entry. Any data written to ModStorage will be lost if you change the mod name.

## Setup

### Config.cpp Configuration

Before using ModStorage, configure your `CfgMods` entry with a `storageVersion`:

```cpp
class CfgMods
{
	class NameOfYourMod
	{
		type = "mod";
		version = "1.2";        // The actual version of the mod
		storageVersion = 2;     // Storage version (must be > 0 to enable ModStorage)
	};
};
```

**Important:**
- `storageVersion` must be greater than 0 for ModStorage to be registered
- `storageVersion` is separate from `version` (mod version string)
- Increment `storageVersion` when you change the data structure

### Advanced Setup (Script-Based)

You can also set the storage version in script by creating a class that inherits from `ModStructure`:

```c
class NameOfYourMod : ModStructure
{
	override void LoadData()
	{
		super.LoadData();
		SetStorageVersion(1);
	}
}
```

## Accessing ModStorage Context

To access your ModStorage context, pass your mod name as a string:

```c
override bool CF_OnStoreLoad(CF_ModStorageMap storage)
{
	if (!super.CF_OnStoreLoad(storage))
		return false;
	
	auto ctx = storage["NameOfYourMod"];
	if (!ctx)
		return true;  // ModStorage doesn't exist yet (first time)
	
	// Read your data from ctx
	return true;
}
```

**Using Class Type (Advanced):**

If using a custom `ModStructure` class, you can use the class type instead of string:

```c
override bool CF_OnStoreLoad(CF_ModStorageMap storage)
{
	if (!super.CF_OnStoreLoad(storage))
		return false;
	
	auto ctx = storage[NameOfYourMod];  // Class type instead of string
	if (!ctx)
		return true;
	
	return true;
}
```

## Writing Data

### Writing Primitive Types

Only primitive data types can be written to ModStorage:

```c
modded class ItemBase
{
	int m_MyMod_SomeVariable;
	
	override void CF_OnStoreSave(CF_ModStorageMap storage)
	{
		super.CF_OnStoreSave(storage);
		
		auto ctx = storage["NameOfYourMod"];
		if (!ctx)
			return;
		
		ctx.Write(m_MyMod_SomeVariable);
	}
}
```

### Writing Arrays

```c
modded class ItemBase
{
	ref array<int> m_MyMod_SomeArray = new array<int>();
	
	override void CF_OnStoreSave(CF_ModStorageMap storage)
	{
		super.CF_OnStoreSave(storage);
		
		auto ctx = storage["NameOfYourMod"];
		if (!ctx)
			return;
		
		int count = m_MyMod_SomeArray.Count();
		ctx.Write(count);
		
		for (int i = 0; i < count; i++)
		{
			ctx.Write(m_MyMod_SomeArray[i]);
		}
	}
}
```

### Writing Custom Classes

For custom classes, create a method within the class to handle writing:

```c
class MyMod_DataClass
{
	string m_SomeVariable;
	int m_SomeValue;
	
	void OnStoreSave(CF_ModStorage ctx)
	{
		ctx.Write(m_SomeVariable);
		ctx.Write(m_SomeValue);
	}
}

modded class ItemBase
{
	ref MyMod_DataClass m_MyMod_DataInstance = new MyMod_DataClass();
	
	override void CF_OnStoreSave(CF_ModStorageMap storage)
	{
		super.CF_OnStoreSave(storage);
		
		auto ctx = storage["NameOfYourMod"];
		if (!ctx)
			return;
		
		m_MyMod_DataInstance.OnStoreSave(ctx);
	}
}
```

## Reading Data

### Reading Primitive Types

Always check for errors when reading:

```c
modded class ItemBase
{
	int m_MyMod_SomeVariable;
	
	override bool CF_OnStoreLoad(CF_ModStorageMap storage)
	{
		if (!super.CF_OnStoreLoad(storage))
			return false;
		
		auto ctx = storage["NameOfYourMod"];
		if (!ctx)
			return true;  // No ModStorage data exists yet
		
		if (ctx.GetVersion() >= 1)
		{
			if (!ctx.Read(m_MyMod_SomeVariable))
				return false;
		}
		
		return true;
	}
}
```

### Reading Arrays

```c
modded class ItemBase
{
	ref array<int> m_MyMod_SomeArray = new array<int>();
	
	override bool CF_OnStoreLoad(CF_ModStorageMap storage)
	{
		if (!super.CF_OnStoreLoad(storage))
			return false;
		
		auto ctx = storage["NameOfYourMod"];
		if (!ctx)
			return true;
		
		int count;
		if (!ctx.Read(count))
			return false;
		
		m_MyMod_SomeArray.Resize(count);
		
		for (int i = 0; i < count; i++)
		{
			int value;
			if (!ctx.Read(value))
				return false;
			
			m_MyMod_SomeArray[i] = value;
		}
		
		return true;
	}
}
```

### Reading Custom Classes

```c
class MyMod_DataClass
{
	string m_SomeVariable;
	int m_SomeValue;
	
	bool OnStoreLoad(CF_ModStorage ctx)
	{
		if (!ctx.Read(m_SomeVariable))
			return false;
		if (!ctx.Read(m_SomeValue))
			return false;
		return true;
	}
}

modded class ItemBase
{
	ref MyMod_DataClass m_MyMod_DataInstance = new MyMod_DataClass();
	
	override bool CF_OnStoreLoad(CF_ModStorageMap storage)
	{
		if (!super.CF_OnStoreLoad(storage))
			return false;
		
		auto ctx = storage["NameOfYourMod"];
		if (!ctx)
			return true;
		
		if (!m_MyMod_DataInstance.OnStoreLoad(ctx))
			return false;
		
		return true;
	}
}
```

## Version Compatibility

Always check `ctx.GetVersion()` to maintain backward compatibility:

```c
override bool CF_OnStoreLoad(CF_ModStorageMap storage)
{
	if (!super.CF_OnStoreLoad(storage))
		return false;
	
	auto ctx = storage["NameOfYourMod"];
	if (!ctx)
		return true;
	
	// Version 1: Only had int variable
	if (ctx.GetVersion() >= 1)
	{
		if (!ctx.Read(m_MyMod_IntVariable))
			return false;
	}
	
	// Version 2: Added string variable
	m_MyMod_StringVariable = "Default Value";  // Default for older versions
	if (ctx.GetVersion() >= 2)
	{
		if (!ctx.Read(m_MyMod_StringVariable))
			return false;
	}
	
	return true;
}
```

## Complete Example

```c
modded class ItemBase
{
	int m_MyMod_IntVariable;
	string m_MyMod_StringVariable;
	ref array<int> m_MyMod_IntArray = new array<int>();
	
	override void CF_OnStoreSave(CF_ModStorageMap storage)
	{
		super.CF_OnStoreSave(storage);
		
		auto ctx = storage["NameOfYourMod"];
		if (!ctx)
			return;
		
		// Write primitive
		ctx.Write(m_MyMod_IntVariable);
		
		// Write string
		ctx.Write(m_MyMod_StringVariable);
		
		// Write array
		int count = m_MyMod_IntArray.Count();
		ctx.Write(count);
		for (int i = 0; i < count; i++)
		{
			ctx.Write(m_MyMod_IntArray[i]);
		}
	}
	
	override bool CF_OnStoreLoad(CF_ModStorageMap storage)
	{
		if (!super.CF_OnStoreLoad(storage))
			return false;
		
		auto ctx = storage["NameOfYourMod"];
		if (!ctx)
		{
			// First time - use defaults
			m_MyMod_IntVariable = 0;
			m_MyMod_StringVariable = "Default";
			return true;
		}
		
		// Read primitive
		if (!ctx.Read(m_MyMod_IntVariable))
			return false;
		
		// Read string with version check
		m_MyMod_StringVariable = "Default";
		if (ctx.GetVersion() >= 2)
		{
			if (!ctx.Read(m_MyMod_StringVariable))
				return false;
		}
		
		// Read array
		int count;
		if (!ctx.Read(count))
			return false;
		
		m_MyMod_IntArray.Resize(count);
		for (int i = 0; i < count; i++)
		{
			int value;
			if (!ctx.Read(value))
				return false;
			m_MyMod_IntArray[i] = value;
		}
		
		return true;
	}
}
```

## Best Practices

### 1. Always Call super First

```c
override void CF_OnStoreSave(CF_ModStorageMap storage)
{
	super.CF_OnStoreSave(storage);  // Always first!
	// Your code
}

override bool CF_OnStoreLoad(CF_ModStorageMap storage)
{
	if (!super.CF_OnStoreLoad(storage))  // Always first!
		return false;
	// Your code
}
```

### 2. Check Context Existence

```c
auto ctx = storage["NameOfYourMod"];
if (!ctx)
	return true;  // No data exists yet - this is OK
```

### 3. Handle Read Errors

```c
if (!ctx.Read(m_MyVariable))
	return false;  // Failed to read - return false
```

### 4. Maintain Version Compatibility

```c
// Always provide defaults for new fields
m_NewField = "Default";

// Check version before reading new fields
if (ctx.GetVersion() >= 2)
{
	if (!ctx.Read(m_NewField))
		return false;
}
```

### 5. Write and Read in Same Order

```c
// Save order
ctx.Write(variable1);
ctx.Write(variable2);
ctx.Write(variable3);

// Load order (must match!)
ctx.Read(variable1);
ctx.Read(variable2);
ctx.Read(variable3);
```

## Summary

**CF ModStorage:**
- Requires `storageVersion > 0` in config.cpp
- Use `CF_OnStoreSave()` to write data
- Use `CF_OnStoreLoad()` to read data
- Always check for context existence
- Always handle read errors
- Maintain version compatibility
- Write and read in the same order

**Best Practices:**
- Always call `super` first
- Check context existence
- Handle all read errors
- Use version checks for backward compatibility
- Provide defaults for new fields

---

**Related Guides:**
- [How to Create a Mod](../../How-To/How-To-Create-Mod.md)
- [Community Framework](../Community-Framework.md) - CF framework overview
- [Module System](../../How-To/Module-System.md)

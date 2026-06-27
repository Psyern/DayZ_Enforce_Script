# Tips: Modded Classes

How to correctly extend vanilla DayZ classes using the `modded` keyword.

## What is a Modded Class?

A `modded` class extends an existing vanilla class without modifying the original code. This is the **only way** to extend vanilla DayZ classes in your mod.

## Basic Syntax

### ✅ CORRECT - Basic Modded Class

```c
modded class ItemBase
{
	override void OnStoreLoad(ParamsReadContext ctx, int version)
	{
		super.OnStoreLoad(ctx, version);
		// Your custom code here
	}
}
```

### ⚠️ Note on Inheritance Syntax

**Generally avoid adding inheritance to modded classes:**

```c
// ✅ CORRECT - No inheritance syntax (preferred)
modded class ItemBase { }

// ⚠️ NOTE: Some vanilla mission classes (like MissionServer, MissionGameplay) 
// may use "extends" when modded, but this is the exception, not the rule
modded class MissionServer extends MissionBase { }

// ❌ Avoid - Don't add inheritance to modded classes unless vanilla class uses it
modded class ItemBase : SomeOtherClass { }  // Generally avoid!
```

**Important:** `modded` classes already inherit from the original class. The style guide recommends **avoiding** inheritance syntax with modded classes. However, check the vanilla class structure first - some mission classes like `MissionServer` may legitimately use `extends`.

**Best Practice:** Check vanilla class structure. If vanilla class uses inheritance, your modded class may need it. Otherwise, avoid inheritance syntax.

**CRITICAL: Always Check Vanilla Source Code**

**❌ CRITICAL: ALWAYS check the DayZ vanilla source code at `@1.29_Experimental/1.29` when modding any function or class.** Before implementing modded functionality, examine the vanilla implementation to:

- **Understand available functions and variables** - See what methods and properties are available in the vanilla class
- **Learn proper usage patterns** - Understand how vanilla code uses these functions
- **Avoid reinventing the wheel** - Use existing vanilla functions instead of custom implementations
- **Ensure compatibility** - Understand the vanilla behavior to maintain compatibility

**Example:** When modding `PlayerBase.OnDeath()`, check `@1.29_Experimental/1.29` to see:
- What parameters `OnDeath()` receives
- What member variables are available (e.g., `m_Health`, `m_LastDamageSource`)
- What helper functions exist (e.g., `GetIdentity()`, `GetPosition()`)
- How vanilla code handles death logic

**CRITICAL RULE:** Always reference the vanilla source code implementation before modding any DayZ function or class. This ensures you use the correct API and understand available functionality.

## Correct Modded Class Pattern

### Example 1: Extending ItemBase

```c
modded class ItemBase
{
	// ✅ ALWAYS prefix member variables in modded classes
	bool m_MyMod_CustomFlag;
	int m_MyMod_CustomValue;
	
	override void OnStoreLoad(ParamsReadContext ctx, int version)
	{
		super.OnStoreLoad(ctx, version);  // Always call super first
		
		// Your custom load logic
		if (version >= 12345)  // Use version from your mod
		{
			ctx.Read(m_MyMod_CustomFlag);
			ctx.Read(m_MyMod_CustomValue);
		}
	}
	
	override void OnStoreSave(ParamsWriteContext ctx)
	{
		super.OnStoreSave(ctx);  // Always call super first
		
		// Your custom save logic
		ctx.Write(m_MyMod_CustomFlag);
		ctx.Write(m_MyMod_CustomValue);
	}
}
```

### Example 2: Extending PlayerBase

```c
modded class PlayerBase
{
	// ✅ ALWAYS prefix member variables
	bool m_MyMod_HasCustomFeature;
	int m_MyMod_CustomHealth;
	
	override void OnStoreLoad(ParamsReadContext ctx, int version)
	{
		super.OnStoreLoad(ctx, version);
		
		if (version >= 12345)
		{
			ctx.Read(m_MyMod_HasCustomFeature);
			ctx.Read(m_MyMod_CustomHealth);
		}
	}
	
	override void OnStoreSave(ParamsWriteContext ctx)
	{
		super.OnStoreSave(ctx);
		
		ctx.Write(m_MyMod_HasCustomFeature);
		ctx.Write(m_MyMod_CustomHealth);
	}
}
```

### Example 3: Extending Mission Classes

```c
// ✅ CORRECT - No inheritance syntax (preferred)
modded class MissionServer
{
	override void OnInit()
	{
		super.OnInit();
		// Your initialization
	}
}

// ⚠️ NOTE: Some vanilla mission classes may use "extends" in modded form:
// modded class MissionServer extends MissionBase { }
// This is valid for certain mission classes, but check vanilla structure first.
// The style guide recommends avoiding inheritance syntax with modded classes.
```

## Accessing Private Members

In modded classes, use the class name prefix to access private members:

```c
modded class ItemBase
{
	void MyCustomMethod()
	{
		// Access private member using class name prefix
		ItemBase.m_SomePrivateMember = value;
		
		// Call private method
		ItemBase.SomePrivateMethod();
	}
}
```

## Calling Parent Methods

**Always** call `super` first when overriding methods:

```c
modded class ItemBase
{
	override void OnStoreLoad(ParamsReadContext ctx, int version)
	{
		// ✅ ALWAYS call super first
		super.OnStoreLoad(ctx, version);
		
		// Then your custom code
		m_MyMod_CustomValue = 100;
	}
	
	override void OnStoreSave(ParamsWriteContext ctx)
	{
		// ✅ ALWAYS call super first
		super.OnStoreSave(ctx);
		
		// Then your custom code
		ctx.Write(m_MyMod_CustomValue);
	}
}
```

### ❌ WRONG - Missing super Call

```c
modded class ItemBase
{
	// ❌ WRONG - Missing super call!
	override void OnStoreLoad(ParamsReadContext ctx, int version)
	{
		// Your code only - vanilla load never happens!
		m_MyMod_CustomValue = 100;
	}
	
	// ❌ WRONG - Missing super call!
	override void OnStoreSave(ParamsWriteContext ctx)
	{
		// Your code only - vanilla save never happens!
		ctx.Write(m_MyMod_CustomValue);
	}
}
```

## Prefixing Member Variables

### ✅ CRITICAL - Always Prefix Members

```c
modded class ItemBase
{
	// ✅ ALWAYS prefix member variables in modded classes
	bool m_MyMod_IsCustom;
	int m_MyMod_CustomValue;
	ref MyMod_Manager m_MyMod_Manager;
}

// ❌ WRONG - No prefix (conflicts!)
modded class ItemBase
{
	bool m_IsCustom;        // Conflicts with other mods!
	int m_CustomValue;      // Conflicts with other mods!
	ref Manager m_Manager;  // Conflicts!
}
```

**Rule:** **ALWAYS** prefix member variables in modded classes to prevent conflicts.

## Complete Examples

### Example 1: Custom Item Behavior

```c
modded class ItemBase
{
	// ✅ Prefixed members
	bool m_MyMod_IsSpecial;
	int m_MyMod_SpecialDurability;
	
	override void OnStoreLoad(ParamsReadContext ctx, int version)
	{
		super.OnStoreLoad(ctx, version);  // Always super first
		
		if (version >= 12345)
		{
			if (!ctx.Read(m_MyMod_IsSpecial))
				m_MyMod_IsSpecial = false;
			
			if (!ctx.Read(m_MyMod_SpecialDurability))
				m_MyMod_SpecialDurability = 100;
		}
	}
	
	override void OnStoreSave(ParamsWriteContext ctx)
	{
		super.OnStoreSave(ctx);  // Always super first
		
		ctx.Write(m_MyMod_IsSpecial);
		ctx.Write(m_MyMod_SpecialDurability);
	}
	
	// New method (not overriding)
	void MyMod_ApplySpecialEffect()
	{
		if (m_MyMod_IsSpecial)
		{
			// Apply special effect
		}
	}
}
```

### Example 2: Player Modifications

```c
modded class PlayerBase
{
	// ✅ Prefixed members
	bool m_MyMod_HasBoost;
	float m_MyMod_BoostMultiplier = 1.0;
	
	override void OnStoreLoad(ParamsReadContext ctx, int version)
	{
		super.OnStoreLoad(ctx, version);
		
		if (version >= 12345)
		{
			ctx.Read(m_MyMod_HasBoost);
			ctx.Read(m_MyMod_BoostMultiplier);
		}
	}
	
	override void OnStoreSave(ParamsWriteContext ctx)
	{
		super.OnStoreSave(ctx);
		
		ctx.Write(m_MyMod_HasBoost);
		ctx.Write(m_MyMod_BoostMultiplier);
	}
}
```

## Common Mistakes

### Mistake 1: Adding Inheritance Syntax

```c
// ❌ WRONG - Never add inheritance
modded class ItemBase : SomeClass { }

// ✅ CORRECT - No inheritance syntax
modded class ItemBase { }
```

### Mistake 2: Forgetting super Call

```c
// ❌ WRONG - Missing super
override void OnStoreLoad(ParamsReadContext ctx, int version)
{
	ctx.Read(m_MyMod_Value);  // Vanilla load never happens!
}

// ✅ CORRECT - Always call super first
override void OnStoreLoad(ParamsReadContext ctx, int version)
{
	super.OnStoreLoad(ctx, version);  // Call super first!
	ctx.Read(m_MyMod_Value);
}
```

### Mistake 3: Not Prefixing Members

```c
// ❌ WRONG - No prefix
modded class ItemBase
{
	bool m_IsCustom;  // Conflicts!
}

// ✅ CORRECT - Prefixed
modded class ItemBase
{
	bool m_MyMod_IsCustom;  // Safe!
}
```

### Mistake 4: Missing override Keyword

```c
// ❌ WRONG - Missing override
void OnStoreLoad(ParamsReadContext ctx, int version)
{
	super.OnStoreLoad(ctx, version);
}

// ✅ CORRECT - Use override keyword
override void OnStoreLoad(ParamsReadContext ctx, int version)
{
	super.OnStoreLoad(ctx, version);
}
```

## Best Practices

### 1. Always Use override Keyword

```c
// ✅ Good - Always use override
override void OnStoreLoad(ParamsReadContext ctx, int version)
{
	super.OnStoreLoad(ctx, version);
}

// ❌ Avoid - Missing override
void OnStoreLoad(ParamsReadContext ctx, int version)  // Wrong!
{
	super.OnStoreLoad(ctx, version);
}
```

### 2. Always Call super First

```c
// ✅ Good - super first
override void OnStoreLoad(ParamsReadContext ctx, int version)
{
	super.OnStoreLoad(ctx, version);  // First!
	// Your code after
}

// ❌ Avoid - Your code first
override void OnStoreLoad(ParamsReadContext ctx, int version)
{
	// Your code first - wrong!
	super.OnStoreLoad(ctx, version);  // Should be first!
}
```

### 3. Always Prefix Member Variables

```c
// ✅ Good - Prefixed
modded class ItemBase
{
	bool m_MyMod_Flag;
	int m_MyMod_Value;
}

// ❌ Avoid - No prefix
modded class ItemBase
{
	bool m_Flag;  // Conflicts!
	int m_Value;  // Conflicts!
}
```

### 4. Use Version Checking for Serialization

```c
// ✅ Good - Version check
override void OnStoreLoad(ParamsReadContext ctx, int version)
{
	super.OnStoreLoad(ctx, version);
	
	if (version >= 12345)  // Your mod's version
	{
		ctx.Read(m_MyMod_Value);
	}
}

// This ensures backward compatibility with older saves
```

### 5. Handle Read/Write Errors

```c
// ✅ Good - Error handling
override void OnStoreLoad(ParamsReadContext ctx, int version)
{
	super.OnStoreLoad(ctx, version);
	
	if (version >= 12345)
	{
		if (!ctx.Read(m_MyMod_Value))
		{
			// Handle error - use default value
			m_MyMod_Value = 100;  // Default
		}
	}
}

// ✅ Good - Error handling
override void OnStoreSave(ParamsWriteContext ctx)
{
	super.OnStoreSave(ctx);
	
	if (!ctx.Write(m_MyMod_Value))
	{
		// Handle error
		Print("[MyMod] Failed to save value");
	}
}
```

## Summary

**Modded Class Rules:**
- ✅ Use `modded class ClassName { }` syntax
- ✅ **NEVER** add inheritance syntax (`: ParentClass` or `extends`)
- ✅ **ALWAYS** prefix member variables in modded classes
- ✅ **ALWAYS** use `override` keyword when overriding
- ✅ **ALWAYS** call `super` first in overridden methods
- ✅ Use version checking for serialization

**Critical Rules:**
- `modded` classes already inherit from the original class
- **Never** add `: ParentClass` or `extends` to modded classes
- **Always** prefix member variables with your mod prefix
- **Always** call `super` before your custom code

**Quick Reference:**
- Basic: `modded class ItemBase { override void Method() { super.Method(); } }`
- Members: `bool m_MyMod_Flag;` (always prefixed)
- Super: Always call `super.Method()` first

---

**Related Guides:**
- [Tips: Naming Conventions and Prefixes](Tips-Prefixes-Naming.md)
- [How to Create a Mod](../How-To/How-To-Create-Mod.md)
- [EnScript Style Guide](../EnScript-Style-Guide.md)
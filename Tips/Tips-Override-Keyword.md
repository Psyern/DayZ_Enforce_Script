# Tips: Override Keyword

Complete guide to using the `override` keyword correctly in DayZ modding, including inheritance chains and common patterns.

## What is Override?

The `override` keyword explicitly marks a method that overrides a parent class method. It's **required** when overriding methods and helps catch errors at compile time.

## Basic Usage

### ✅ CORRECT - Always Use Override

```c
modded class ItemBase
{
	override void OnStoreLoad(ParamsReadContext ctx, int version)
	{
		super.OnStoreLoad(ctx, version);
		// Your code
	}
}
```

### ❌ WRONG - Missing Override

```c
modded class ItemBase
{
	// ❌ Missing override keyword - compiler warning!
	void OnStoreLoad(ParamsReadContext ctx, int version)
	{
		super.OnStoreLoad(ctx, version);
	}
}
```

## Override in Inheritance Chains

### Example: Multi-Level Inheritance

When you have base classes and variant classes, **always** use `override` at each level:

```c
// Base class defines methods
class LockedLoot_Crate_Base: Container_Base
{
	string GetCrateOpenSoundset()
	{
		return "LLC_CrateOpen_SoundSet";
	}
	
	string GetCrateCloseSoundset()
	{
		return "LLC_CrateClose_SoundSet";
	}
	
	protected void UpdateVisualState()
	{
		if (IsOpen())
			SetAnimationPhase("lid", 1);
		else
			SetAnimationPhase("lid", 0);
	}
}

// Variant base class overrides with override keyword
class LLC_ToolBox_Base: LockedLoot_Crate_Base
{
	// ✅ CORRECT - Override keyword used
	override string GetCrateOpenSoundset()
	{
		return "LLC_LatchOpen_SoundSet";
	}
	
	override string GetCrateCloseSoundset()
	{
		return "LLC_LatchClose_SoundSet";
	}
	
	// ✅ CORRECT - Protected override
	protected override void UpdateVisualState()
	{
		if (IsOpen())
			SetAnimationPhase("lid", 1);
		else
			SetAnimationPhase("lid", 0);
	}
}
```

### ❌ WRONG - Missing Override in Variant Classes

```c
class LLC_ToolBox_Base: LockedLoot_Crate_Base
{
	// ❌ WRONG - Missing override keyword
	string GetCrateOpenSoundset()
	{
		return "LLC_LatchOpen_SoundSet";
	}
	
	// ❌ WRONG - Missing override keyword
	protected void UpdateVisualState()
	{
		// Compiler warning: "FIX-ME: Overriding function but not marked as 'override'"
	}
}
```

## Protected Override Methods

When overriding `protected` methods, include both keywords:

```c
class BaseClass: ItemBase
{
	protected void UpdateVisualState()
	{
		// Base implementation
	}
}

class DerivedClass: BaseClass
{
	// ✅ CORRECT - Both protected and override
	protected override void UpdateVisualState()
	{
		// Derived implementation
	}
}
```

## Real-World Example: Crate Variant System

Complete example showing proper override usage in a variant system:

```c
// Base crate class
class LockedLoot_Crate_Base: Container_Base
{
	string GetCrateOpenSoundset()
	{
		return "LLC_CrateOpen_SoundSet";
	}
	
	string GetCrateCloseSoundset()
	{
		return "LLC_CrateClose_SoundSet";
	}
	
	protected void UpdateVisualState()
	{
		if (IsOpen())
			SetAnimationPhase("lid", 1);
		else
			SetAnimationPhase("lid", 0);
	}
}

// Toolbox variant - overrides sound methods
class LLC_ToolBox_Base: LockedLoot_Crate_Base
{
	override string GetCrateOpenSoundset()
	{
		return "LLC_LatchOpen_SoundSet";
	}
	
	override string GetCrateCloseSoundset()
	{
		return "LLC_LatchClose_SoundSet";
	}
	
	protected override void UpdateVisualState()
	{
		if (IsOpen())
			SetAnimationPhase("lid", 1);
		else
			SetAnimationPhase("lid", 0);
	}
}

// Fridge variant - overrides with different animation
class LLC_Fridge_Base: LockedLoot_Crate_Base
{
	override string GetCrateOpenSoundset()
	{
		return "LLC_FridgeOpen_SoundSet";
	}
	
	override string GetCrateCloseSoundset()
	{
		return "LLC_FridgeClose_SoundSet";
	}
	
	protected override void UpdateVisualState()
	{
		if (IsOpen())
		{
			SetAnimationPhase("lid", 1);
			SetAnimationPhase("lid2", 1);  // Fridge has two lids
		}
		else
		{
			SetAnimationPhase("lid", 0);
			SetAnimationPhase("lid2", 0);
		}
	}
}
```

## Common Patterns

### Pattern 1: Override Virtual Methods

```c
modded class ItemBase
{
	override bool IsOpen()
	{
		if (m_MyMod_Openable)
			return m_MyMod_Openable.IsOpened();
		return false;
	}
}
```

### Pattern 2: Override Protected Methods

```c
class MyMod_BaseItem: ItemBase
{
	protected void UpdateVisualState()
	{
		// Base implementation
	}
}

class MyMod_DerivedItem: MyMod_BaseItem
{
	protected override void UpdateVisualState()
	{
		super.UpdateVisualState();  // Call parent first
		// Additional derived logic
	}
}
```

### Pattern 3: Override with Different Return Types

```c
class BaseClass
{
	string GetSoundset()
	{
		return "BaseSound";
	}
}

class DerivedClass: BaseClass
{
	override string GetSoundset()
	{
		return "DerivedSound";
	}
}
```

## When Override is Required

### ✅ Always Required For:

1. **Overriding parent class methods**
2. **Overriding modded class methods**
3. **Overriding protected methods**
4. **Overriding virtual methods**
5. **Overriding methods in inheritance chains**

### ❌ Not Required For:

1. **New methods** (not overriding anything)
2. **Methods in base classes** (defining, not overriding)
3. **Static methods** (unless overriding static)

## Compiler Warnings

If you forget `override`, DayZ compiler will warn:

```
SCRIPT (W): @"path/to/file.c,10": FIX-ME: Overriding function 'MethodName' but not marked as 'override'
```

**Always fix these warnings** by adding the `override` keyword.

## Best Practices

### 1. Always Use Override

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

### 2. Use in Inheritance Chains

```c
// ✅ Good - Override at each level
class Base { string GetSound() { return "base"; } }
class Derived: Base { override string GetSound() { return "derived"; } }
class Variant: Derived { override string GetSound() { return "variant"; } }
```

### 3. Include Access Modifiers

```c
// ✅ Good - Include protected/private with override
protected override void UpdateVisualState() { }
private override void InternalMethod() { }
```

### 4. Call Super First

```c
// ✅ Good - Super first, then override logic
override void OnStoreLoad(ParamsReadContext ctx, int version)
{
	super.OnStoreLoad(ctx, version);  // Always first
	// Your override logic here
}
```

## Common Mistakes

### Mistake 1: Forgetting Override in Variant Classes

```c
// ❌ WRONG
class VariantBase: BaseClass
{
	string GetSound()  // Missing override!
	{
		return "variant";
	}
}

// ✅ CORRECT
class VariantBase: BaseClass
{
	override string GetSound()
	{
		return "variant";
	}
}
```

### Mistake 2: Missing Override on Protected Methods

```c
// ❌ WRONG
class Derived: Base
{
	protected void UpdateVisual()  // Missing override!
	{
		// ...
	}
}

// ✅ CORRECT
class Derived: Base
{
	protected override void UpdateVisual()
	{
		// ...
	}
}
```

### Mistake 3: Override on Non-Overriding Methods

```c
// ❌ WRONG - Method doesn't exist in parent
class Derived: Base
{
	override void NewMethod()  // Parent doesn't have this!
	{
		// Compiler error!
	}
}

// ✅ CORRECT - No override for new methods
class Derived: Base
{
	void NewMethod()  // New method, no override
	{
		// ...
	}
}
```

## Summary

**Override Keyword Rules:**
- ✅ **ALWAYS** use `override` when overriding parent methods
- ✅ **ALWAYS** use `override` in inheritance chains
- ✅ **ALWAYS** use `override` for protected/private overrides
- ✅ **ALWAYS** fix compiler warnings about missing override

**Quick Reference:**
- Basic: `override void Method() { super.Method(); }`
- Protected: `protected override void Method() { }`
- Inheritance: Use `override` at each level
- Warning: Fix "FIX-ME: Overriding function but not marked as 'override'"

**Related Guides:**
- [Tips: Modded Classes](Tips-Modded-Classes.md)
- [Tips: Naming Conventions](Tips-Prefixes-Naming.md)
- [EnScript Style Guide](EnScript-Style-Guide.md)

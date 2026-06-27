# Tips: Naming Conventions and Prefixes

Guide to proper naming conventions and when to use your own prefix to prevent mod conflicts.

## Why Prefixes Matter

Prefixes prevent conflicts between:
- Your mod and vanilla code
- Your mod and other mods
- Multiple mods on the same server

**Without proper prefixes, your mod may crash or conflict with other mods.**

## When to Use Your Own Prefix

### ✅ ALWAYS Use Your Prefix For:

1. **Member variables in modded classes**
2. **New classes you create**
3. **Enums you create**
4. **Static variables**
5. **Settings files**
6. **Module names**
7. **RPC function names** (if using string-based RPC)

### ❌ Don't Use Prefix For:

1. **Local variables** - Use camelCase without prefix
2. **Parameters** - Use camelCase without prefix
3. **Vanilla classes you're extending** - The class name stays the same

## Prefix Guidelines

### Choose a Unique Prefix

Use a **short, unique prefix** for your mod:

```c
// ✅ Good - Short and unique
MyMod_ClassName      // "MyMod" prefix
CF_ClassName         // "CF" prefix
ASS_ClassName        // "ASS" prefix (from your memory)

// ❌ Avoid - Too generic
Mod_ClassName        // Too generic, likely conflicts
Test_ClassName       // Too generic
Class_ClassName      // Redundant
```

### Prefix Format

**Format:** `YourPrefix_ClassName` or `YourPrefix_MemberName`

- Use **PascalCase** for classes
- Use **PascalCase** after the prefix underscore
- Keep prefix **short** (2-5 characters recommended)

## Member Variables in Modded Classes

### ✅ CORRECT - Always Prefix Members

```c
modded class ItemBase
{
	// ✅ ALWAYS prefix member variables in modded classes
	bool m_MyMod_IsCustom;
	int m_MyMod_CustomValue;
	ref MyMod_Manager m_MyMod_Manager;
}

// ✅ Also prefix in new classes
class MyMod_MyClass : ItemBase
{
	bool m_MyMod_SpecialFlag;
	int m_MyMod_SpecialValue;
}
```

### ❌ WRONG - No Prefix

```c
modded class ItemBase
{
	// ❌ NO! This conflicts with vanilla code and other mods
	bool m_IsCustom;
	int m_CustomValue;
	ref Manager m_Manager;
}
```

## Class Naming

### ✅ CORRECT - Prefix Your Classes

```c
// Your mod classes
class MyMod_Settings {}
class MyMod_Module {}
class MyMod_Manager {}
class MyMod_Logger {}

// ✅ Even if extending vanilla classes
class MyMod_CustomItem : ItemBase {}
class MyMod_CustomWeapon : WeaponBase {}
```

### ❌ WRONG - Generic Names

```c
// ❌ NO! Too generic, conflicts with other mods
class Settings {}
class Module {}
class Manager {}
class CustomItem : ItemBase {}  // Missing prefix!
```

## Enum Naming

### ✅ CORRECT - Prefix Your Enums

```c
enum EMyModRPCs
{
	MYMOD_RPC_START = 10000,
	MYMOD_RPC_TEST,
	MYMOD_RPC_SEND_DATA
}

enum MyModLogLevel
{
	Debug = 0,
	Info,
	Warning,
	Error
}

enum EMyModSettings
{
	Low,
	Medium,
	High
}
```

### ❌ WRONG - Generic Enum Names

```c
// ❌ NO! Generic names conflict
enum ERPCs {}  // Conflicts with vanilla
enum LogLevel {}  // Conflicts with other mods
enum Settings {}  // Too generic
```

## Settings Files

### ✅ CORRECT - Namespace Settings

```c
// Constants.c
const string MyMod_ROOT_FOLDER = "$profile:MyMod\\";
const string MyMod_CONFIG_FILE = MyMod_ROOT_FOLDER + "Config\\Settings.json";

// Settings class
class MyMod_Settings
{
	int m_MyMod_LogLevel = 1;
	bool m_MyMod_Enabled = true;
}
```

### ❌ WRONG - Generic Paths

```c
// ❌ NO! Generic paths conflict
const string CONFIG_FILE = "$profile:Config.json";  // Conflicts!
const string SETTINGS_FILE = "$profile:Settings.json";  // Conflicts!

class Settings  // Missing prefix!
{
	int m_LogLevel;  // Missing prefix!
}
```

## Preventing Mod Conflicts

### 1. Use Unique Class Names

```c
// ✅ Good - Unique name
class MyMod_SpawnManager {}

// ❌ Avoid - Generic name
class SpawnManager {}  // Other mods might use this!
```

### 2. Prefix Member Variables in Modded Classes

```c
modded class PlayerBase
{
	// ✅ ALWAYS prefix members in modded classes
	bool m_MyMod_CustomFlag;
	int m_MyMod_CustomValue;
	
	// ❌ NEVER use unprefixed members
	// bool m_CustomFlag;  // Conflicts!
}
```

### 3. Namespace File Paths

```c
// ✅ Good - Mod-specific folder
const string MyMod_CONFIG_FILE = "$profile:MyMod\\Config\\Settings.json";

// ❌ Avoid - Generic folder
const string CONFIG_FILE = "$profile:Config\\Settings.json";  // Conflicts!
```

### 4. Prefix RPC Names

```c
// ✅ Good - String-based RPC with prefix
CF_GetRPCManager().RegisterServerRPC("MyMod", "RPC_Test", RPC_Test, ...);

// ✅ Good - Enum-based RPC with prefix
enum EMyModRPCs
{
	MYMOD_RPC_START = 10000,  // Start at 10000 to avoid vanilla conflicts
	MYMOD_RPC_TEST,
	MYMOD_RPC_SEND_DATA
}

// ❌ Avoid - Generic RPC names
enum ERPCs  // Conflicts with vanilla ERPCs!
{
	RPC_TEST,  // Conflicts!
}
```

### 5. Use Unique Module Names

```c
// ✅ Good - Unique module name
class MyMod_Module : MissionModule {}
class MyMod_LoggingModule : CF_ModuleGame {}

// ❌ Avoid - Generic module name
class Module : MissionModule {}  // Conflicts!
```

### 6. JSON Config Classes - Exception to Prefix Rule

**Important Exception:** Config classes used for JSON serialization should **NOT** prefix member variable names to maintain JSON file compatibility.

```c
// ✅ CORRECT - Config class for JSON (NO prefix on members)
class LLCSpawnCrateConfig
{
	string name;        // No prefix - matches JSON field name
	int colorR;         // No prefix - matches JSON field name
	int colorG;
	int colorB;
	int colorA;
	int maxItems;
	float lootrandomize;
}

// ✅ CORRECT - Entity class (HAS prefix)
class LockedLoot_Crate_Base: Container_Base
{
	bool m_LLC_IsLocked;      // Prefixed - entity class
	ref OpenableBehaviour m_LLC_Openable;  // Prefixed
}

// ❌ WRONG - Prefixing config class members breaks JSON
class LLCSpawnCrateConfig
{
	string m_LLC_name;     // ❌ JSON files won't match!
	int m_LLC_colorR;      // ❌ Breaks existing JSON configs!
}
```

**Why No Prefix for Config Classes?**
- JSON files use member variable names as keys
- Changing member names breaks all existing JSON config files
- Users have already configured their servers with these names
- This is an acceptable exception for JSON serialization classes

**Rule:** Prefix entity classes, but **don't prefix** config class members used for JSON serialization.

## Real-World Examples

### Example 1: Modded ItemBase

```c
// ✅ CORRECT - Properly prefixed
modded class ItemBase
{
	// Prefixed member variables
	bool m_MyMod_HasCustomProperty;
	int m_MyMod_CustomDurability;
	ref MyMod_Component m_MyMod_Component;
	
	override void OnStoreLoad(ParamsReadContext ctx, int version)
	{
		super.OnStoreLoad(ctx, version);
		
		// Your custom load logic
		if (version >= 12345)  // Use version from your mod
		{
			ctx.Read(m_MyMod_HasCustomProperty);
			ctx.Read(m_MyMod_CustomDurability);
		}
	}
	
	override void OnStoreSave(ParamsWriteContext ctx)
	{
		super.OnStoreSave(ctx);
		
		// Your custom save logic
		ctx.Write(m_MyMod_HasCustomProperty);
		ctx.Write(m_MyMod_CustomDurability);
	}
}
```

### Example 2: Custom Class

```c
// ✅ CORRECT - Prefixed class and members
class MyMod_ItemManager
{
	static ref MyMod_ItemManager s_Instance;
	ref array<ItemBase> m_MyMod_ManagedItems;
	int m_MyMod_ItemCount = 0;
	
	static MyMod_ItemManager GetInstance()
	{
		if (!s_Instance)
		{
			s_Instance = new MyMod_ItemManager();
		}
		return s_Instance;
	}
}
```

### Example 3: Settings with Prefix

```c
// ✅ CORRECT - All namespaced
const string MyMod_ROOT_FOLDER = "$profile:MyMod\\";
const string MyMod_CONFIG_FILE = MyMod_ROOT_FOLDER + "Config\\Settings.json";

class MyMod_Settings
{
	int m_MyMod_LogLevel = 1;
	bool m_MyMod_Enabled = true;
	string m_MyMod_ServerName = "My Server";
	
	void Save()
	{
		MakeDirectoryIfNotExists();
		JsonFileLoader<MyMod_Settings>.JsonSaveFile(MyMod_CONFIG_FILE, this);
	}
}
```

### Example 4: JSON Config Class (No Prefix Exception)

```c
// ✅ CORRECT - Config class for JSON serialization (NO prefix on members)
class LLCSpawnCrateConfig
{
	string name;              // No prefix - matches JSON
	string title;
	string messageOnOpen;
	string iconPath;
	int colorR;               // No prefix - matches JSON
	int colorG;
	int colorB;
	int colorA;
	int maxItems;
	float lootrandomize;
	ref array<string> loot;
	
	// Class name IS prefixed, but members are NOT (JSON compatibility)
}

// JSON file uses these exact names:
// {
//   "name": "My Crate",
//   "colorR": 255,
//   "colorG": 90,
//   "colorB": 0,
//   "colorA": 255
// }

// ✅ CORRECT - Entity class (HAS prefix)
class LockedLoot_Crate_Base: Container_Base
{
	bool m_LLC_IsLocked;           // Prefixed - entity class
	bool m_LLC_HasToolsCache;      // Prefixed
	ref OpenableBehaviour m_LLC_Openable;  // Prefixed
}
```

## Common Mistakes

### Mistake 1: Forgetting Prefix in Modded Class

```c
// ❌ WRONG
modded class ItemBase
{
	bool m_IsCustom;  // Missing prefix!
}

// ✅ CORRECT
modded class ItemBase
{
	bool m_MyMod_IsCustom;  // Properly prefixed
}
```

### Mistake 2: Generic Class Names

```c
// ❌ WRONG
class Manager {}  // Too generic!
class Settings {}  // Too generic!

// ✅ CORRECT
class MyMod_Manager {}  // Unique name
class MyMod_Settings {}  // Unique name
```

### Mistake 3: Unprefixed Enums

```c
// ❌ WRONG
enum ERPCs  // Conflicts with vanilla!
{
	RPC_TEST
}

// ✅ CORRECT
enum EMyModRPCs  // Unique name
{
	MYMOD_RPC_START = 10000,
	MYMOD_RPC_TEST
}
```

### Mistake 4: Generic File Paths

```c
// ❌ WRONG
const string CONFIG_FILE = "$profile:Config.json";  // Conflicts!

// ✅ CORRECT
const string MyMod_CONFIG_FILE = "$profile:MyMod\\Config\\Settings.json";  // Unique path
```

## Prefix Best Practices

### 1. Be Consistent

Use the **same prefix** throughout your entire mod:

```c
// ✅ Good - Consistent prefix
class MyMod_Settings {}
class MyMod_Module {}
class MyMod_Manager {}
enum EMyModRPCs {}

// ❌ Avoid - Inconsistent prefixes
class MyMod_Settings {}
class MyMod2_Module {}  // Different prefix!
class Settings_Manager {}  // Different format!
```

### 2. Keep It Short

Prefix should be **2-5 characters** for readability:

```c
// ✅ Good - Short and clear
CF_ClassName      // Community Framework
ASS_ClassName     // Advance Spawn System
MyMod_ClassName   // Still readable

// ❌ Avoid - Too long
MyVeryLongModName_ClassName  // Hard to read!
```

### 3. Use Project Convention

Based on your project memory, you prefer **ASS** prefix:

```c
// ✅ Following project convention
class ASS_Settings {}
class ASS_Manager {}
bool m_ASS_IsEnabled;
```

### 4. Document Your Prefix

```c
// Constants.c
// MOD_PREFIX: Use "MyMod" prefix for all classes, enums, and member variables
// to prevent conflicts with vanilla code and other mods.

const string MyMod_ROOT_FOLDER = "$profile:MyMod\\";
```

## Summary

**When to Use Prefix:**
- ✅ ALWAYS for member variables in modded classes
- ✅ ALWAYS for member variables in entity classes
- ✅ ALWAYS for new classes
- ✅ ALWAYS for enums
- ✅ ALWAYS for settings paths
- ✅ ALWAYS for module names
- ✅ ALWAYS for RPC names

**When NOT to Use Prefix:**
- ❌ Local variables
- ❌ Parameters
- ❌ Vanilla class names you're extending
- ❌ **Member variables in JSON config classes** (maintains JSON file compatibility)

**Exception: JSON Config Classes**
- Config classes used for JSON serialization should **NOT** prefix member names
- This maintains compatibility with existing JSON configuration files
- Class name should still be prefixed: `MyMod_ConfigClass`
- But members should match JSON field names: `string name;` not `string m_MyMod_name;`

**Best Practices:**
- Use short, unique prefix (2-5 characters)
- Be consistent throughout mod
- Prefix all member variables in modded classes
- Namespace file paths
- Use unique class/enum names

**Quick Reference:**
- **Member in modded class:** `m_YourPrefix_MemberName`
- **Member in entity class:** `m_YourPrefix_MemberName`
- **Member in JSON config class:** `memberName` (NO prefix - JSON compatibility)
- **New class:** `YourPrefix_ClassName`
- **Enum:** `EYourPrefixEnumName` or `YourPrefixEnumName`
- **Settings path:** `$profile:YourPrefix\\...`
- **RPC:** `YourPrefix_RPC_Name` or `YOURPREFIX_RPC_START = 10000`

---

**Related Guides:**
- [How to Create a Mod](../How-To/How-To-Create-Mod.md)
- [Tips: Modded Classes](Tips-Modded-Classes.md)
- [EnScript Style Guide](EnScript-Style-Guide.md)
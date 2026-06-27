# Basic Mod Structure

Understanding how DayZ mods are organized is crucial for effective modding. This guide covers the standard mod structure used in DayZ Standalone.

## Directory Structure

A typical DayZ mod follows this structure:

```
MyMod/
├── scripts/
│   └── [YourNamespace]/
│       └── [YourModName]/
│           ├── 1_Core/          # Core systems (optional)
│           ├── 2_GameLib/       # Game library (optional)
│           ├── 3_Game/          # Game logic and modules
│           ├── 4_World/         # World entities
│           ├── 5_Mission/       # Mission logic and GUI
│           ├── Data/            # JSON data files
│           └── config.cpp       # Mod configuration
├── gui/                         # GUI layouts and resources (optional)
└── other/                       # Additional resources (optional)
```

## Config.cpp Structure

Every mod requires a `config.cpp` file that defines the mod to the game engine.

### Basic Example

```cpp
class CfgPatches
{
	class MyMod_Scripts
	{
		units[] = {};
		weapons[] = {};
		requiredVersion = 0.1;  // ⚠️ CRITICAL: NEVER change this value! Always use 0.1
		requiredAddons[] = {"DZ_Data"};
	};
};

class CfgMods
{
	class MyMod
	{
		dir = "YourNamespace/YourModName";
		picture = "";
		action = "";
		hideName = 0;
		hidePicture = 1;
		name = "My Mod";
		credits = "Your Name";
		creditsJson = "YourNamespace/YourModName/Scripts/Data/Credits.json";
		version = "1.0.0";  // Version string (e.g., "1.0", "1.1", "1.0.0")
		// Note: versionPath is optional - most mods just use version directly
		author = "Your Name";
		authorID = "76561198000000000";
		extra = 0;
		type = "mod";
		defines[] = {"MYMOD_FEATURE_ENABLED"};
		dependencies[] = {"Core", "Game", "World", "Mission"};
		
		class defs
		{
			class engineScriptModule
			{
				value = "";
				files[] = {"YourNamespace/YourModName/Scripts/1_Core"};
			};
			class gameLibScriptModule
			{
				value = "";
				files[] = {"YourNamespace/YourModName/Scripts/2_GameLib"};
			};
			class gameScriptModule
			{
				value = "";  // Optional: entry point function
				files[] = {"YourNamespace/YourModName/Scripts/3_Game"};
			};
			class worldScriptModule
			{
				value = "";
				files[] = {"YourNamespace/YourModName/Scripts/4_World"};
			};
			class missionScriptModule
			{
				value = "";
				files[] = {"YourNamespace/YourModName/Scripts/5_Mission"};
			};
		};
	};
};
```

### Key Configuration Elements

#### CfgPatches
- **units[]** - Array of units/objects this patch provides
- **weapons[]** - Array of weapons this patch provides
- **requiredVersion** - ⚠️ **CRITICAL: For NEW mods, ALWAYS use `0.1`. For EXISTING mods, NEVER change the existing value (e.g., if it's `1.2`, keep it as `1.2`).** Once you create your CfgMods entry and apply a storage version, changing this will cause data loss in ModStorage. New default mods will always be `requiredVersion = 0.1;`, but existing mods may have different values that must be preserved.
- **requiredAddons[]** - Dependencies on other mods/addons

#### CfgMods
- **dir** - Path to mod directory (relative to scripts/)
- **name** - Display name of the mod
- **version** - Mod version string
- **defines[]** - Preprocessor defines available in your mod
- **dependencies[]** - Required script layers
- **class defs** - Script module definitions for each layer

## Script Layers Explained

### 1_Core Layer
**Purpose:** Core engine systems and utilities

- Used for fundamental systems that don't depend on game logic
- Common uses: Logging, utilities, base classes
- **Example:** CF mod uses this for core framework utilities

```cpp
class engineScriptModule
{
	value = "";
	files[] = {"YourNamespace/YourModName/Scripts/1_Core"};
};
```

### 2_GameLib Layer
**Purpose:** Game library code

- Shared utilities and game-level abstractions
- Common uses: Helpers, shared game logic
- **Example:** Game library utilities and managers

```cpp
class gameLibScriptModule
{
	value = "";
	files[] = {"YourNamespace/YourModName/Scripts/2_GameLib"};
};
```

### 3_Game Layer
**Purpose:** Game logic and modules

- Main game logic, modules, managers
- Common uses: Game modules, settings, managers
- **Example:** Most mod logic goes here

```cpp
class gameScriptModule
{
	value = "";  // Usually blank - entry point is rarely needed
	files[] = {"YourNamespace/YourModName/Scripts/3_Game"};
};
```

### 4_World Layer
**Purpose:** World entities and objects

- Entity classes, world objects, items
- Common uses: Custom items, entities, world objects
- **Example:** ItemBase extensions, custom entities

```cpp
class worldScriptModule
{
	value = "";
	files[] = {"YourNamespace/YourModName/Scripts/4_World"};
};
```

### 5_Mission Layer
**Purpose:** Mission logic and GUI

- Mission-specific code, GUI, player interactions
- Common uses: GUI layouts, mission logic, player scripts
- **Example:** Menu systems, player interactions

```cpp
class missionScriptModule
{
	value = "";
	files[] = {"YourNamespace/YourModName/Scripts/5_Mission"};
};
```

## File Organization Within Layers

Within each layer, organize files by functionality:

```
3_Game/
└── YourModName/
    ├── Modules/
    │   └── MyModule.c
    ├── Settings/
    │   └── MySettings.c
    ├── Managers/
    │   └── MyManager.c
    └── Utils/
        └── MyUtils.c
```

## Naming Conventions

### Directory Names
- Use PascalCase: `MyModName`
- Match namespace in config: `YourNamespace/YourModName`

### File Names
- Use PascalCase matching class name: `MyClass.c`
- Match the primary class name in the file

### Class Names
- Use PascalCase with mod prefix: `MyMod_ClassName`
- Example: `CF_Module`, `ExpansionGarageSettings`

## Real-World Examples

### Community Framework Structure

```
CF/
└── scripts/
    └── JM/
        └── CF/
            └── Scripts/
                ├── 1_Core/
                │   └── CommunityFramework/
                │       ├── Module/
                │       ├── Logging/
                │       └── Utils/
                ├── 3_Game/
                │   └── CommunityFramework/
                │       ├── Config/
                │       ├── Module/
                │       └── RPC/
                ├── 4_World/
                └── config.cpp
```

### DayZ Expansion Structure

```
DayZExpansion/
└── Core/
    └── Scripts/
        └── 3_Game/
            └── DayZExpansion_Core/
                ├── Modules/
                ├── Settings/
                └── Managers/
```

## Best Practices

### 1. Use Appropriate Layers
- Don't put game logic in 1_Core
- Don't put core utilities in 5_Mission
- Follow the layer purposes

### 2. Organize by Functionality
- Group related files in subdirectories
- Use clear, descriptive names
- Keep related functionality together

### 3. Namespace Your Code
- Always use mod prefixes on classes
- Prefix member variables in modded classes
- Avoid name conflicts

### 4. Keep Config Clean
- Only include necessary script modules
- Use defines for feature flags
- Document dependencies clearly

### 5. File Structure Consistency
- Follow established patterns
- Keep similar mods organized similarly
- Make it easy for others to navigate

## Common Mistakes

### ❌ Wrong Layer Usage
```cpp
// ❌ Game logic in 1_Core
1_Core/MyGameLogic.c

// ✅ Should be in 3_Game
3_Game/MyGameLogic.c
```

### ❌ Incorrect Directory Paths
```cpp
// ❌ Mismatch between dir and actual structure
dir = "MyMod/Scripts";  // But files are in "MyMod/Code"
```

### ❌ Missing Dependencies
```cpp
// ❌ Missing required addon
requiredAddons[] = {};  // But mod uses DZ_Data classes

// ✅ Include dependencies
requiredAddons[] = {"DZ_Data"};
```

## Entry Points (Rarely Needed)

**Note:** Entry points are rarely needed. Most mods use `value = ""` and let `MissionModule` handle initialization automatically.

**If you need a custom entry point** (very rare cases):

```cpp
class gameScriptModule
{
	value = "MyMod_CreateGame";  // Only if you need custom initialization
	files[] = {"YourNamespace/YourModName/Scripts/3_Game"};
};
```

The entry point function:

```c
void MyMod_CreateGame()
{
	// Custom initialization code (rarely needed)
	// Modules using MissionModule are auto-registered
}
```

**For 99% of mods:** Just use `value = ""` and skip entry points entirely.

## Summary

Understanding mod structure is fundamental to DayZ modding:

- **config.cpp** defines your mod to the engine
- **Script layers** organize code by purpose
- **File organization** within layers keeps code maintainable
- **Naming conventions** prevent conflicts
- **Appropriate layer usage** ensures proper initialization order

---

**Related Topics:**
- [How to Create a Mod](How-To-Create-Mod.md) - Step-by-step tutorial
- [Script Layers Guide](Script-Layers-Guide.md) - Detailed layer information
- [Module System](Module-System.md) - Module creation guide
- [EnScript Style Guide](../Tips/EnScript-Style-Guide.md) - Coding standards
- [Community Framework](../Frameworks/Community-Framework.md) - CF documentation
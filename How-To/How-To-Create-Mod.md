# How to Create a Mod

Step-by-step guide to creating a DayZ mod from scratch.

## Step 1: Create Mod Directory Structure

Create the following directory structure:

```
MyMod/
├── scripts/
│   ├── 3_Game/
│   │   └── YourModName_Module.c
│   ├── 4_World/          (optional - for world entities)
│   ├── 5_Mission/        (optional - for mission scripts)
│   └── Common/           (optional - shared code across layers)
│       └── Debug.c
├── gui/                      (optional - for UI layouts and icons)
│   ├── layouts/
│   └── icons/
├── config.cpp
└── datasets/                 (optional - for 3D models, textures, etc.)
```

**Important:** 
- Use PascalCase for mod names
- `dir` in config.cpp is the mod identifier (e.g., `"YourModName"`)
- `files[]` paths must use the pattern `"YourModName/scripts/3_Game"`
- The physical folder structure is `MyMod/scripts/3_Game` (no mod name folder between scripts and layer folders)

**See:** [Tips: Code Organization](../Tips/Tips-Code-Organization.md) for detailed folder organization patterns.

## Step 2: Create config.cpp

Create `config.cpp` in your mod root:

**⚠️ CRITICAL WARNING - requiredVersion:**
- **For NEW mods: ALWAYS use `requiredVersion = 0.1;`** - This is the default for new mods
- **For EXISTING mods: NEVER change the existing `requiredVersion` value** - If an existing mod has `requiredVersion = 1.2;` or any other value, keep it as is. Changing it will break the mod and cause data loss in ModStorage
- **New default mods will always be `requiredVersion = 0.1;`**
- Once you create your CfgMods entry and apply a storage version, changing `requiredVersion` will cause data loss in ModStorage
- **Rule: New mods = 0.1, Existing mods = keep existing value, never change**

```cpp
class CfgPatches
{
	class YourModName_Scripts
	{
		units[] = {};
		weapons[] = {};
		requiredVersion = 0.1;  // ⚠️ CRITICAL: For NEW mods, always use 0.1. For EXISTING mods, keep existing value!
		requiredAddons[] = {"DZ_Data"};
	};
};

class CfgMods
{
	class YourModName
	{
		dir = "YourModName";  // Mod identifier (used in files[] paths, not a physical folder)
		picture = "";  // Optional: mod icon path
		action = "";  // Optional: URL or action
		hideName = 0;  // Show mod name in launcher
		hidePicture = 1;  // Hide mod picture in launcher
		name = "Your Mod Name";  // Display name
		credits = "Your Name";  // Credits string
		version = "1.0.0";  // Version string (e.g., "1.0", "1.1", "1.0.0")
		// Version string for mod launcher
		author = "Your Name";
		authorID = "76561198000000000";  // Your Steam ID (64-bit)
		extra = 0;
		type = "mod";
		defines[] = {};  // Optional: feature flags (e.g., {"DEBUG_MODE"})
		dependencies[] = {"Game", "World", "Mission"};  // Required script layers
		
		class defs
		{
			// Game layer scripts (initialization, modules, managers)
			class gameScriptModule
			{
				value = "";  // Optional: entry point function name
				files[] = {
					"YourModName/scripts/Common",  // Optional: shared code
					"YourModName/scripts/3_Game"   // Game layer scripts
				};
			};
			
			// World layer scripts (entities, items, vehicles) - Optional
			class worldScriptModule
			{
				value = "";
				files[] = {
					"YourModName/scripts/Common",
					"YourModName/scripts/4_World"
				};
			};
			
			// Mission layer scripts (mission logic, player scripts) - Optional
			class missionScriptModule
			{
				value = "";
				files[] = {
					"YourModName/scripts/Common",
					"YourModName/scripts/5_Mission"
				};
			};
		};
	};
};
```

## Step 3: Create Basic Module

**Without CF (Native DayZ):**

Create `scripts/3_Game/YourModName_Module.c`:

```c
class YourModName_Module : MissionModule
{
	override void OnInit()
	{
		super.OnInit();
		
		// Initialize your mod here
		Print("YourModName: Module initialized");
	}
	
	override void OnUpdate(float delta_time)
	{
		super.OnUpdate(delta_time);
		// Update logic here (optional)
	}
}
```

## Step 4: Version Constants (Optional)

If you want to track and display your mod version in code, create a Constants file:

**Create `scripts/3_Game/Constants/YourModName_Constants.c`:**

```c
// Version constant
const string YourModName_VERSION = "1.0.1";
```

**Then log it in your MissionServer** (`scripts/5_Mission/MissionServer.c`):

```c
modded class MissionServer extends MissionBase
{
	override void OnInit()
	{
		super.OnInit();
		
		if (YourModName_Logger)  // If you have a logger
		{
			YourModName_Logger.Notice("YourModName mod initialized - Version " + YourModName_VERSION);
		}
		else
		{
			Print("[YourModName] Mod initialized - Version " + YourModName_VERSION);
		}
	}
}
```

**Note:** The `version = "1.0.0"` in `config.cpp` is separate and used by the mod launcher. The constant in code is for runtime version display/logging.

## Step 5: Additional Script Layers (Optional)

If your mod needs world entities or mission logic, you can add additional layers:

**World Layer (`4_World`)** - For entities, items, vehicles:
```c
// scripts/4_World/YourItem/YourItem.c
modded class ItemBase
{
	override void OnStoreLoad(ParamsReadContext ctx, int version)
	{
		super.OnStoreLoad(ctx, version);
		// Your code
	}
}
```

**Mission Layer (`5_Mission`)** - For mission scripts, player modifications:
```c
// scripts/5_Mission/MissionServer.c
modded class MissionServer extends MissionBase
{
	void MissionServer()
	{
		Print("[YourMod] MissionServer initialized");
	}
}
```

**Common Folder** - Shared code across all layers:
```c
// scripts/Common/Debug.c
void YourMod_DebugPrint(string message)
{
	Print("[YourMod] " + message);
}
```

Don't forget to add these to `config.cpp` `defs` section if you use them (see Step 2).

## Step 6: Register Your Module (Optional - Rarely Needed)

**Note:** The `value` field is almost always blank (`value = ""`). Entry point functions are rarely needed since modules using `MissionModule` are automatically registered.

**If you ever need a custom entry point** (which is very rare):
1. Create `scripts/3_Game/YourModName_CreateGame.c`:

```c
void YourModName_CreateGame()
{
	// Custom initialization code (rarely needed)
	// Module registration happens automatically if using MissionModule
}
```

2. Then update `config.cpp`:

```cpp
class gameScriptModule
{
	value = "YourModName_CreateGame";  // Only if you need custom entry point
	files[] = {"YourModName/scripts/3_Game"};
};
```

**For 99% of mods:** Just use `value = ""` and let `MissionModule` handle initialization automatically.

## Basic Mod with Logger

Here's a complete example with logging:

### config.cpp

```cpp
class CfgPatches
{
	class MyFirstMod_Scripts
	{
		units[] = {};
		weapons[] = {};
		requiredVersion = 0.1;  // ⚠️ CRITICAL: For NEW mods, always use 0.1. For EXISTING mods, keep existing value!
		requiredAddons[] = {"DZ_Data"};
	};
};

class CfgMods
{
	class MyFirstMod
	{
		dir = "MyFirstMod";  // Mod identifier (used in files[] paths)
		name = "My First Mod";
		version = "1.0.0";
		author = "Your Name";
		authorID = "76561198000000000";
		type = "mod";
		dependencies[] = {"Game", "World", "Mission"};
		
		class defs
		{
			class gameScriptModule
			{
				value = "";  // Optional entry point
				files[] = {
					"MyFirstMod/scripts/Common",  // Optional: shared code
					"MyFirstMod/scripts/3_Game"
				};
			};
		};
	};
};
```

### Module File

```c
class MyFirstMod_Module : MissionModule
{
	override void OnInit()
	{
		super.OnInit();
		Print("[MyFirstMod] Module initialized successfully!");
	}
}
```

## Adding Dependencies

**❌ CRITICAL: NEVER add requiredAddons except for "DZ_Data".** Only include `requiredAddons[]={"DZ_Data"};` in config.cpp. Do NOT add framework dependencies (like "CF_Scripts") or other mod dependencies. The user will add these manually if needed.

```cpp
// ✅ CORRECT - Only DZ_Data
class CfgPatches
{
	class MyMod_Scripts
	{
		units[] = {};
		weapons[] = {};
		requiredVersion = 0.1;  // ⚠️ CRITICAL: For NEW mods, always use 0.1. For EXISTING mods, keep existing value!
		requiredAddons[] = {"DZ_Data"};
	};
};

// ❌ WRONG - Don't add framework or other mod dependencies
class CfgPatches
{
	class MyMod_Scripts
	{
		requiredAddons[] = {
			"DZ_Data",
			"CF_Scripts"  // User will add this manually if needed
		};
	};
};
```

**Why:** Users may not have all dependencies installed, or may want to use different versions. Let users add dependencies manually based on their setup.

**Rule:** Always use `requiredAddons[]={"DZ_Data"};` only. Never add framework or other mod dependencies.

## Complete Example: Simple Mod with Settings

### Directory Structure

```
MyMod/
├── scripts/
│   ├── 3_Game/
│   │   ├── MyFirstMod_Module.c
│   │   └── MyFirstMod_Settings.c
│   └── Common/
│       └── Debug.c
├── config.cpp
└── datasets/
```

### config.cpp

```cpp
class CfgPatches
{
	class MyFirstMod_Scripts
	{
		units[] = {};
		weapons[] = {};
		requiredVersion = 0.1;  // ⚠️ CRITICAL: For NEW mods, always use 0.1. For EXISTING mods, keep existing value!
		requiredAddons[] = {"DZ_Data"};
	};
};

class CfgMods
{
	class MyFirstMod
	{
		dir = "MyFirstMod";  // Mod identifier (used in files[] paths)
		name = "My First Mod";
		version = "1.0.0";
		author = "Your Name";
		authorID = "76561198000000000";
		type = "mod";
		dependencies[] = {"Game", "World", "Mission"};
		
		class defs
		{
			class gameScriptModule
			{
				value = "";  // Optional entry point
				files[] = {
					"MyFirstMod/scripts/Common",  // Optional: shared code
					"MyFirstMod/scripts/3_Game"
				};
			};
		};
	};
};
```

### MyFirstMod_Module.c

```c
class MyFirstMod_Module : MissionModule
{
	ref MyFirstMod_Settings m_Settings;
	
	override void OnInit()
	{
		super.OnInit();
		
		// Initialize settings
		m_Settings = new MyFirstMod_Settings();
		
		Print("[MyFirstMod] Module initialized");
		Print("[MyFirstMod] Settings value: " + m_Settings.m_Value);
	}
}
```

### MyFirstMod_Settings.c

```c
class MyFirstMod_Settings
{
	int m_Value = 100;
	string m_Message = "Hello from MyFirstMod";
	
	void MyFirstMod_Settings()
	{
		// Default values set above
	}
}
```

## Testing Your Mod

1. **Compile** your mod using DayZ Tools
2. **Place** in `@MyMod` folder in your DayZ directory
3. **Start** DayZ with `-mod=@MyMod` parameter
4. **Check** logs for your print statements

## Next Steps

- [How to Create a Logger](How-To-Logger.md) - Add logging to your mod
- [How to Create Profile Settings](How-To-Profile-Settings.md) - Create persistent settings
- [Naming Conventions and Prefixes](Tips-Prefixes-Naming.md) - Learn proper naming
- [How to Use RPC](How-To-RPC.md) - Add network communication

## Common Mistakes

### ❌ Wrong Directory Structure

```
// ❌ Wrong - Missing proper structure
MyMod/scripts/MyModule.c

// ❌ Wrong - Extra nested folder
MyMod/scripts/YourModName/3_Game/YourModName/YourModName_Module.c

// ✅ Correct - Standard structure
MyMod/scripts/3_Game/YourModName_Module.c
```

### ❌ Wrong Config Paths

```cpp
// ❌ Wrong - Path doesn't match structure
dir = "MyMod/Scripts/MyMod";  // Includes "scripts" or wrong path
files[] = {"MyMod/Code"};  // Doesn't match actual folder
files[] = {"MyMod/scripts/3_Game/MyMod"};  // Extra nested path

// ✅ Correct - Path matches structure
// Physical folder structure: MyMod/scripts/3_Game/
// Config uses mod identifier:
dir = "YourModName";  // Mod identifier (can be different from folder name)
files[] = {"YourModName/scripts/3_Game"};  // Path uses mod identifier

// ✅ Example:
// Physical structure: MyMod/scripts/3_Game/...
// config.cpp:
dir = "MyMod";  // Mod identifier
files[] = {"MyMod/scripts/3_Game"};  // Path uses mod identifier
```

### ❌ Missing super Call

```c
// ❌ Wrong - Missing super.OnInit()
override void OnInit()
{
	Print("Initialized");
}

// ✅ Correct - Always call super
override void OnInit()
{
	super.OnInit();
	Print("Initialized");
}
```

## Summary

Creating a mod requires:

1. **Directory structure** - Follow naming conventions
2. **config.cpp** - Define your mod to the engine
3. **Module file** - Main mod logic
4. **Proper naming** - Use prefixes to avoid conflicts
5. **Testing** - Always test your mod

---

**Related Guides:**
- [Basic Mod Structure](Basic-Mod-Structure.md)
- [Tips: Code Organization](../Tips/Tips-Code-Organization.md) - Advanced folder organization
- [Naming Conventions and Prefixes](../Tips/Tips-Prefixes-Naming.md)
- [Tips: Modded Classes](../Tips/Tips-Modded-Classes.md)
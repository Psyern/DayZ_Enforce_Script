# Tips: Code Organization

Best practices for organizing your mod's code structure based on real-world mod patterns.

## Overview

Well-organized code is easier to maintain, debug, and extend. This guide shows proven organizational patterns used in professional DayZ mods.

## Common Folder Pattern

**Purpose:** Shared code accessible across all script layers (3_Game, 4_World, 5_Mission).

**Location:** `scripts/Common/`

**Example:**
```
scripts/
├── Common/
│   └── Debug.c          # Shared debug utilities
├── 3_Game/
├── 4_World/
└── 5_Mission/
```

**In config.cpp:**
```cpp
class gameScriptModule
{
	files[] = {
		"YourModName/scripts/Common",  // Include Common in all layers
		"YourModName/scripts/3_Game"
	};
};

class worldScriptModule
{
	files[] = {
		"YourModName/scripts/Common",
		"YourModName/scripts/4_World"
	};
};

class missionScriptModule
{
	files[] = {
		"YourModName/scripts/Common",
		"YourModName/scripts/5_Mission"
	};
};
```

**Common Use Cases:**
- Debug utilities
- Shared helper functions
- Common constants
- Utility classes used across layers

## 3_Game Layer Organization

The 3_Game layer typically contains modules, managers, settings, and configuration. Here's a proven organizational structure:

### Recommended Structure

```
scripts/3_Game/
├── Constants/
│   └── YourModName_Constants.c      # All constants (paths, IDs, values)
├── enums/
│   └── YourModName_Enums.c         # Custom enums
├── General Configs/                # JSON-based configuration classes
│   ├── YourModName_MainConfig.c
│   ├── YourModName_AdminConfig.c
│   └── YourModName_ZoneConfig.c
├── LoggerModule/                    # Logging system
│   ├── enum.c
│   ├── LoggingModule.c
│   └── LoggingSettings.c
├── Logging/                         # Alternative: Simple logging
│   └── Logging.c
├── Notifications/                   # Notification system
│   └── YourModName_Notifications.c
├── Webhook/                         # External integrations (Discord, etc.)
│   ├── YourModName_WebhookModule.c
│   ├── YourModName_WebhookMessage.c
│   └── YourModName_WebhookCallback.c
├── ZoneConfigs/                     # Zone-related configurations
│   ├── YourModName_ZoneSettings.c
│   └── YourModName_ZoneTimeSettings.c
└── YourModName_Module.c            # Main module
```

### Constants Folder

**Purpose:** Centralize all constants (paths, IDs, notification types, etc.)

**Example:**
```c
// scripts/3_Game/Constants/YourModName_Constants.c

const string YourModName_ROOT_FOLDER = "$profile:YourModName\\";
const string YourModName_LOG_FOLDER = YourModName_ROOT_FOLDER + "Logging\\";
const string YourModName_CONFIG_DIR = YourModName_ROOT_FOLDER + "Config\\";
const string YourModName_CONFIG_FILE = YourModName_CONFIG_DIR + "YourModNameConfig.json";

// Notification IDs
const int NOTIFICATION_PLACED = 0;
const int NOTIFICATION_EXPIRED = 1;
const int NOTIFICATION_REWARD = 2;

// Feature constants
const float DEFAULT_TELEPORT_DISTANCE = 50.0;
const int SAFEZONE_CHECK_DELAY_MS = 2000;
```

### General Configs Folder

**Purpose:** JSON-based configuration classes that are saved/loaded from profile

**Example:**
```c
// scripts/3_Game/General Configs/YourModName_MainConfig.c

ref YourModName_MainConfig g_YourModNameConfig;

class YourModName_MainConfig
{
	bool EnableFeature;
	int MaxItems;
	float SearchRadius;
	ref array<string> AllowedItems;
	
	void YourModName_MainConfig()
	{
		EnableFeature = true;
		MaxItems = 10;
		SearchRadius = 100.0;
		AllowedItems = new array<string>;
	}
}
```

### LoggerModule Folder

**Purpose:** Complete logging system with settings

**Structure:**
```
LoggerModule/
├── enum.c                    # Log level enums
├── LoggingModule.c          # Main logging class
└── LoggingSettings.c        # Logging configuration
```

### Notifications Folder

**Purpose:** Notification system for player messages

**Example:**
```c
// scripts/3_Game/Notifications/YourModName_Notifications.c

class YourModName_NotificationEntry
{
	bool Enabled;
	string Title;
	string Message;
	string IconPath;
	
	void YourModName_NotificationEntry()
	{
		Enabled = true;
		Title = "";
		Message = "";
		IconPath = "";
	}
}
```

## 4_World Layer Organization

The 4_World layer contains entity modifications, actions, and world-related code.

### Recommended Structure

```
scripts/4_World/
├── Actions/                          # Player actions
│   ├── ActionOpenMenu.c
│   └── ActionCustom.c
├── Anims/                            # Custom animations
│   └── Anim.c
├── entities/                         # Entity modifications
│   ├── game/
│   │   └── super/
│   │       └── YourCustomEntity.c
│   ├── manbase/
│   │   └── PlayerBase.c
│   ├── buildings/
│   │   └── YourBuilding.c
│   ├── creatures/
│   │   └── ZombieBase.c
│   └── itembase/
│       └── ItemBase.c
├── DamageSystem/                     # Damage-related systems
│   └── YourModName_DamageSystem.c
├── Plugins/                          # Plugin compatibility
│   ├── Expansion/
│   │   └── YourModName_Expansion.c
│   └── OtherMod/
│       └── YourModName_OtherMod.c
├── Zone Related/                     # Zone-related systems
│   ├── ZoneManager.c
│   ├── PlayerZoneHandler.c
│   └── ZoneController.c
└── YourModName_Module.c              # World module
```

### Actions Folder

**Purpose:** All player actions

**Example:**
```
Actions/
├── ActionOpenMenu.c
├── ActionCustom.c
└── ActionInteract.c
```

### Entities Folder Structure

**Purpose:** Organized by entity type

**Structure:**
```
entities/
├── game/              # Game entities
│   └── super/        # Static world objects
├── manbase/          # Player modifications
├── buildings/        # Building entities
├── creatures/       # Animal/zombie modifications
└── itembase/        # Item modifications
```

**Example:**
```c
// scripts/4_World/entities/manbase/PlayerBase.c
modded class PlayerBase
{
	override void OnStoreLoad(ParamsReadContext ctx, int version)
	{
		super.OnStoreLoad(ctx, version);
		// Your code
	}
}
```

### Plugins Folder

**Purpose:** Compatibility code for other mods

**Example:**
```
Plugins/
├── Expansion/
│   └── YourModName_Expansion.c
├── CommunityFramework/
│   └── YourModName_CF.c
└── OtherMod/
    └── YourModName_OtherMod.c
```

## 5_Mission Layer Organization

The 5_Mission layer contains mission logic, GUI, and player-facing systems.

### Recommended Structure

```
scripts/5_Mission/
├── GUI/                              # Menu classes
│   ├── YourModName_AdminMenu.c
│   └── YourModName_MainMenu.c
├── MapDrawer/                        # Map-related systems
│   └── YourModName_MapSystem.c
├── Plugins/                         # Plugin-specific mission code
│   ├── Maps/
│   │   ├── YourModName_MapMenu.c
│   │   ├── Expansion/
│   │   │   └── YourModName_ExpansionMapMenu.c
│   │   └── OtherMod/
│   │       └── YourModName_OtherModMap.c
│   └── OtherPlugin/
│       └── YourModName_OtherPlugin.c
├── missionGameplay.c                 # Mission gameplay modifications
└── MissionServer.c                  # Mission server modifications
```

### GUI Folder

**Purpose:** All UI menu classes

**Example:**
```
GUI/
├── YourModName_AdminMenu.c
├── YourModName_MainMenu.c
└── YourModName_SettingsMenu.c
```

### MapDrawer Folder

**Purpose:** Map drawing and visualization systems

**Example:**
```c
// scripts/5_Mission/MapDrawer/YourModName_MapSystem.c
class YourModName_MapSystem
{
	void DrawZonesOnMap()
	{
		// Map drawing logic
	}
}
```

## GUI Assets Organization

GUI assets (layouts, icons, textures) should be organized in the `gui/` folder.

### Recommended Structure

```
gui/
├── layouts/                          # UI layout files
│   ├── AdminMenu.layout
│   ├── MainMenu.layout
│   └── SettingsMenu.layout
├── icons/                            # Icon assets (.edds files)
│   ├── checkmark.edds
│   ├── error.edds
│   └── info.edds
└── textures/                         # Texture files (optional)
    └── background.edds
```

**Layout Path in Code:**
```c
// Load layout
if (!g_Game)
	return null;

Widget layoutRoot = g_Game.GetWorkspace().CreateWidgets("YourModName/GUI/layouts/MainMenu.layout");
```

## Complete Example Structure

Here's a complete example of a well-organized mod:

```
YourModName/
├── scripts/
│   ├── Common/
│   │   └── Debug.c
│   ├── 3_Game/
│   │   ├── Constants/
│   │   │   └── YourModName_Constants.c
│   │   ├── General Configs/
│   │   │   └── YourModName_MainConfig.c
│   │   ├── LoggerModule/
│   │   │   ├── enum.c
│   │   │   ├── LoggingModule.c
│   │   │   └── LoggingSettings.c
│   │   └── YourModName_Module.c
│   ├── 4_World/
│   │   ├── Actions/
│   │   │   └── ActionOpenMenu.c
│   │   ├── entities/
│   │   │   └── manbase/
│   │   │       └── PlayerBase.c
│   │   └── YourModName_Module.c
│   └── 5_Mission/
│       ├── GUI/
│       │   └── YourModName_MainMenu.c
│       └── missionGameplay.c
├── gui/
│   ├── layouts/
│   │   └── MainMenu.layout
│   └── icons/
│       └── icon.edds
└── config.cpp
```

## Best Practices

### 1. Group Related Functionality

```c
// ✅ Good - Related code together
scripts/3_Game/
├── LoggerModule/        # All logging code
├── Notifications/       # All notification code
└── General Configs/     # All config classes

// ❌ Avoid - Scattered code
scripts/3_Game/
├── Logger.c
├── Notification.c
├── Config.c
└── AnotherLogger.c
```

### 2. Use Descriptive Folder Names

```c
// ✅ Good - Clear purpose
General Configs/
LoggerModule/
Zone Related/

// ❌ Avoid - Generic names
Config/
Module/
Stuff/
```

### 3. Separate Plugin Code

```c
// ✅ Good - Plugin code isolated
Plugins/
├── Expansion/
└── OtherMod/

// ❌ Avoid - Mixed with main code
YourModName_Expansion.c  // In root folder
```

### 4. Organize Entities by Type

```c
// ✅ Good - Organized by entity type
entities/
├── manbase/
├── buildings/
└── itembase/

// ❌ Avoid - All entities in one folder
entities/
├── PlayerBase.c
├── Building.c
└── ItemBase.c
```

### 5. Keep Common Code Accessible

```c
// ✅ Good - Common folder included in all layers
files[] = {
	"YourModName/scripts/Common",
	"YourModName/scripts/3_Game"
};

// ❌ Avoid - Duplicating shared code
// Don't copy debug utilities to each layer
```

## Summary

**Key Organizational Principles:**

1. **Common Folder** - Shared code across all layers
2. **3_Game** - Constants/, Configs/, LoggerModule/, Notifications/, enums/
3. **4_World** - Actions/, entities/ (by type), Plugins/, feature folders
4. **5_Mission** - GUI/, MapDrawer/, Plugins/, mission files
5. **GUI Assets** - layouts/, icons/, textures/ in gui/ folder

**Benefits:**
- Easier to find code
- Better maintainability
- Clearer structure for contributors
- Reduced merge conflicts
- Professional appearance

---

**Related Guides:**
- [How to Create a Mod](../How-To/How-To-Create-Mod.md) - Basic mod structure
- [Basic Mod Structure](../How-To/Basic-Mod-Structure.md) - Mod organization overview
- [Script Layers Guide](../How-To/Script-Layers-Guide.md) - Understanding script layers
- [Tips: Code Structure](Tips-Code-Structure.md) - Code formatting and style

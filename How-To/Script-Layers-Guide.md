# Script Layers Guide - What Code Goes Where

Complete guide to DayZ script layers: what code belongs in each layer, what restrictions exist, and why the layer system exists.

## Overview

DayZ uses a **5-layer script system** to organize code by initialization order and dependency. Each layer has specific purposes and restrictions to prevent circular dependencies and ensure proper initialization.

## Layer Initialization Order

Scripts are loaded and initialized in this order:

1. **1_Core** - Core engine systems (loaded first)
2. **2_GameLib** - Game library (depends on Core)
3. **3_Game** - Game logic and modules (depends on GameLib)
4. **4_World** - World entities (depends on Game)
5. **5_Mission** - Mission logic and GUI (loaded last, depends on World)

**Critical Rule:** Lower-numbered layers **cannot** reference classes from higher-numbered layers.

## 1_Core Layer

**Purpose:** Core engine systems and fundamental utilities that don't depend on game logic.

### What Goes Here

✅ **CORRECT - Core Layer Code:**
- Base utility classes
- Logging systems
- String/array utilities
- Math helpers
- Constants and defines
- Proto method wrappers
- Base class definitions (not game-specific)
- Type definitions and typedefs

```c
// ✅ CORRECT - Core layer logging utility
class MyMod_Log
{
	static void Info(string message)
	{
		Print("[MyMod] " + message);
	}
}

// ✅ CORRECT - Core layer constants
const string MyMod_VERSION = "1.0.0";
const int MyMod_MAX_VALUE = 100;

// ✅ CORRECT - Core layer utility
class MyMod_StringUtils
{
	static string Format(string template, string param)
	{
		return template + param;
	}
}
```

### What CANNOT Go Here

❌ **INCORRECT - Core Layer Code:**
- **ANY** classes that inherit from `EntityAI`, `ItemBase`, `PlayerBase`, etc.
- Game-specific logic
- Modules (those go in 3_Game)
- Settings classes (those go in 3_Game)
- World entities (those go in 4_World)
- Mission classes (those go in 5_Mission)

```c
// ❌ WRONG - Cannot use EntityAI in Core layer
class MyMod_EntityHelper  // ❌ EntityAI is from 4_World!
{
	void ProcessEntity(EntityAI entity)  // ❌ 4_World classes not available yet!
	{
	}
}

// ❌ WRONG - Cannot use game modules in Core layer
class MyMod_Module : MissionModule  // ❌ MissionModule is from 3_Game!
{
}
```

### Restrictions

- **Cannot reference:** Any classes from 2_GameLib, 3_Game, 4_World, or 5_Mission
- **Can only use:** Primitive types (int, float, bool, string, vector), base types from engine, and classes defined in 1_Core
- **Purpose:** Provides foundation that all other layers can depend on

---

## 2_GameLib Layer

**Purpose:** Game library abstractions and shared utilities that depend on Core but not on game-specific logic.

### What Goes Here

✅ **CORRECT - GameLib Layer Code:**
- Game library base classes
- Shared utilities for game logic
- Input abstractions
- Menu system base classes
- Game-level helpers (not entity-specific)

```c
// ✅ CORRECT - GameLib layer input helper
class MyMod_InputHelper
{
	static bool IsKeyPressed(int keyCode)
	{
		// Input handling logic
		return false;
	}
}

// ✅ CORRECT - GameLib layer menu base
class MyMod_MenuBase
{
	protected Widget m_RootWidget;
	
	void Initialize()
	{
		// Menu initialization
	}
}
```

### What CANNOT Go Here

❌ **INCORRECT - GameLib Layer Code:**
- **ANY** classes that inherit from `EntityAI`, `ItemBase`, `PlayerBase`, etc.
- Game modules (those go in 3_Game)
- World entities (those go in 4_World)
- Mission classes (those go in 5_Mission)

```c
// ❌ WRONG - Cannot use EntityAI in GameLib layer
class MyMod_ItemHelper  // ❌ ItemBase is from 4_World!
{
	void ProcessItem(ItemBase item)  // ❌ 4_World classes not available yet!
	{
	}
}
```

### Restrictions

- **Cannot reference:** Any classes from 4_World or 5_Mission
- **Can reference:** 1_Core classes
- **Purpose:** Provides game-level abstractions without entity dependencies

**Note:** Most mods don't use 2_GameLib - it's optional and mainly for large frameworks.

---

## 3_Game Layer

**Purpose:** Game logic, modules, managers, and settings. This is where **most mod logic goes**.

### What Goes Here

✅ **CORRECT - Game Layer Code:**
- **Modules** (`MissionModule`, `CF_ModuleGame`, etc.)
- **Managers** (game logic managers, spawn managers, etc.)
- **Settings classes** (configuration, profile settings)
- **RPC handlers** (game-level RPCs)
- **Utilities** that work with game systems
- **Constants** specific to your mod's game logic

```c
// ✅ CORRECT - Game layer module
class MyMod_Module : MissionModule
{
	override void OnInit()
	{
		super.OnInit();
		Print("[MyMod] Module initialized");
	}
}

// ✅ CORRECT - Game layer settings
class MyMod_Settings
{
	int m_LogLevel = 1;
	bool m_Enabled = true;
	
	void Save()
	{
		JsonFileLoader<MyMod_Settings>.JsonSaveFile("$profile:MyMod/settings.json", this);
	}
}

// ✅ CORRECT - Game layer manager
class MyMod_Manager
{
	static ref MyMod_Manager s_Instance;
	
	static MyMod_Manager GetInstance()
	{
		if (!s_Instance)
			s_Instance = new MyMod_Manager();
		return s_Instance;
	}
}
```

### What CANNOT Go Here

❌ **INCORRECT - Game Layer Code:**
- **ANY** classes that inherit from `EntityAI`, `ItemBase`, `PlayerBase`, `VehicleBase`, etc.
- **ANY** `modded` classes that extend world entities
- World entity classes (those go in 4_World)
- Mission-specific classes like `MissionGameplay`, `MissionServer` (those go in 5_Mission)

```c
// ❌ WRONG - Cannot mod ItemBase in 3_Game layer
modded class ItemBase  // ❌ ItemBase is from 4_World!
{
	override void OnStoreLoad(ParamsReadContext ctx, int version)
	{
		super.OnStoreLoad(ctx, version);
		// ❌ 4_World classes not fully initialized yet!
	}
}

// ❌ WRONG - Cannot use PlayerBase directly in 3_Game
class MyMod_PlayerHelper
{
	void ProcessPlayer(PlayerBase player)  // ❌ PlayerBase is from 4_World!
	{
		// ❌ Technically this might compile, but breaks layer rules!
	}
}
```

### Restrictions

**Critical Restrictions for 3_Game:**
- **Cannot create/mod:** `modded class` for any `EntityAI`, `ItemBase`, `PlayerBase`, `VehicleBase`, or other 4_World classes
- **Cannot create:** New classes that inherit from 4_World classes
- **Can reference:** 4_World classes as types/parameters (but cannot modify them here)
- **Purpose:** Contains game logic and managers that work **with** entities, but doesn't **define** entities

**Why This Restriction Exists:**
- 3_Game loads **before** 4_World
- If 3_Game could modify 4_World classes, it would create circular dependencies
- Entity modifications must be in 4_World so they're available when entities are created

**What You CAN Do in 3_Game:**
```c
// ✅ CORRECT - Can reference EntityAI types (but not create/mod them)
class MyMod_Manager
{
	// ✅ OK - Using type as parameter/reference
	void ProcessEntityList(array<EntityAI> entities)
	{
		// Process list of entities
	}
	
	// ✅ OK - Getting entities from game
	EntityAI GetEntityByID(int id)
	{
		if (!g_Game)
			return null;
		
		return g_Game.GetObjectByID(id);
	}
}
```

---

## 4_World Layer

**Purpose:** World entities, items, vehicles, and all objects that exist in the game world.

### What Goes Here

✅ **CORRECT - World Layer Code:**
- **All `modded` classes** that extend world entities:
  - `modded class ItemBase`
  - `modded class PlayerBase`
  - `modded class EntityAI`
  - `modded class VehicleBase`
  - `modded class WeaponBase`
  - Any other entity classes
- **New entity classes** that inherit from world entities:
  - `class MyMod_CustomItem : ItemBase`
  - `class MyMod_CustomVehicle : VehicleBase`
- **Actions** - All player actions (eating, crafting, interacting):
  - `classes/useractionscomponent/actions/continuous/`
  - `classes/useractionscomponent/actions/singleuse/`
  - `classes/useractionscomponent/actions/interact/`
- **Recipes** - All crafting recipes:
  - `classes/recipes/recipes/`
- **Entity-related utilities** that work directly with entity classes

```c
// ✅ CORRECT - World layer modded class
modded class ItemBase
{
	bool m_MyMod_CustomFlag;
	
	override void OnStoreLoad(ParamsReadContext ctx, int version)
	{
		super.OnStoreLoad(ctx, version);
		
		if (version >= 12345)
		{
			ctx.Read(m_MyMod_CustomFlag);
		}
	}
	
	override void OnStoreSave(ParamsWriteContext ctx)
	{
		super.OnStoreSave(ctx);
		ctx.Write(m_MyMod_CustomFlag);
	}
}

// ✅ CORRECT - World layer custom item
class MyMod_CustomItem : ItemBase
{
	bool m_MyMod_SpecialProperty;
	
	override void OnStoreLoad(ParamsReadContext ctx, int version)
	{
		super.OnStoreLoad(ctx, version);
		// Custom item logic
	}
}

// ✅ CORRECT - World layer player mod
modded class PlayerBase
{
	int m_MyMod_CustomValue;
	
	override void OnStoreLoad(ParamsReadContext ctx, int version)
	{
		super.OnStoreLoad(ctx, version);
		// Player modification logic
	}
}
```

### What CANNOT Go Here

❌ **INCORRECT - World Layer Code:**
- **Modules** (those go in 3_Game)
- **Mission classes** like `MissionGameplay`, `MissionServer` (those go in 5_Mission)
- **GUI classes** (those go in 5_Mission)
- **Settings classes** (those go in 3_Game)

```c
// ❌ WRONG - Cannot put modules in 4_World
class MyMod_Module : MissionModule  // ❌ MissionModule belongs in 3_Game!
{
}

// ❌ WRONG - Cannot put mission classes in 4_World
modded class MissionGameplay  // ❌ MissionGameplay belongs in 5_Mission!
{
}
```

### Restrictions

- **Must be here:** All code that modifies or extends world entities
- **Cannot reference:** Mission-specific classes (5_Mission)
- **Can reference:** 3_Game modules/managers (but cannot be defined there)
- **Purpose:** Defines all objects that exist in the game world

---

## 5_Mission Layer

**Purpose:** Mission logic, GUI, player interactions, and mission-specific code that runs per-mission.

### What Goes Here

✅ **CORRECT - Mission Layer Code:**
- **Mission classes:**
  - `modded class MissionGameplay`
  - `modded class MissionServer`
  - `modded class MissionBase`
- **GUI classes:**
  - Menu classes (`UIScriptedMenu`, etc.)
  - Widget classes
  - UI controllers
- **Player interaction code:**
  - Input handlers
  - Action handlers
  - Client-side scripts
- **Mission-specific managers** that depend on mission being loaded

```c
// ✅ CORRECT - Mission layer mission mod
modded class MissionGameplay extends MissionBase
{
	override void OnInit()
	{
		super.OnInit();
		Print("[MyMod] Mission initialized");
	}
}

// ✅ CORRECT - Mission layer mission server
modded class MissionServer extends MissionBase
{
	void MissionServer()
	{
		Print("[MyMod] MissionServer started");
	}
}

// ✅ CORRECT - Mission layer GUI class
class MyMod_CustomMenu : UIScriptedMenu
{
	override Widget Init()
	{
		// Create menu widget
		return super.Init();
	}
}
```

### What CANNOT Go Here

❌ **INCORRECT - Mission Layer Code:**
- **Entity classes** (`modded class ItemBase`, etc. - those go in 4_World)
- **Game modules** (those go in 3_Game)
- **Settings classes** (those go in 3_Game)

```c
// ❌ WRONG - Cannot mod entities in 5_Mission
modded class ItemBase  // ❌ ItemBase belongs in 4_World!
{
	// ❌ Entities must be defined before mission loads!
}

// ❌ WRONG - Cannot put game modules in 5_Mission
class MyMod_Module : MissionModule  // ❌ MissionModule belongs in 3_Game!
{
}
```

### Restrictions

- **Must be here:** All mission and GUI code
- **Can reference:** All previous layers (Core, GameLib, Game, World)
- **Purpose:** Mission-specific logic that runs when a mission/level loads

---

## Common Mistakes

### Mistake 1: Putting Entity Mods in 3_Game

```c
// ❌ WRONG - In 3_Game layer
modded class ItemBase
{
	// ❌ This belongs in 4_World!
}

// ✅ CORRECT - Move to 4_World layer
// File: scripts/4_World/ItemBase.c
modded class ItemBase
{
	// ✅ Correct location
}
```

### Mistake 2: Putting Modules in 4_World

```c
// ❌ WRONG - In 4_World layer
class MyMod_Module : MissionModule
{
	// ❌ This belongs in 3_Game!
}

// ✅ CORRECT - Move to 3_Game layer
// File: scripts/3_Game/MyMod_Module.c
class MyMod_Module : MissionModule
{
	// ✅ Correct location
}
```

### Mistake 3: Putting Mission Classes in 4_World

```c
// ❌ WRONG - In 4_World layer
modded class MissionGameplay
{
	// ❌ This belongs in 5_Mission!
}

// ✅ CORRECT - Move to 5_Mission layer
// File: scripts/5_Mission/MissionGameplay.c
modded class MissionGameplay
{
	// ✅ Correct location
}
```

### Mistake 4: Referencing 4_World Classes in 3_Game (Wrong Way)

```c
// ❌ WRONG - Trying to modify entities from 3_Game
class MyMod_Manager
{
	void ModifyItem(ItemBase item)
	{
		// ❌ Cannot directly modify entity structure from 3_Game
		// Use 4_World modded class instead
	}
}

// ✅ CORRECT - Modify entity in 4_World, use from 3_Game
// In 4_World:
modded class ItemBase
{
	bool m_MyMod_CustomFlag;
}

// In 3_Game:
class MyMod_Manager
{
	void ProcessItem(ItemBase item)
	{
		// ✅ OK - Can read/use entity, but modifications are in 4_World
		if (item)
		{
			// Use entity (modifications were done in 4_World)
		}
	}
}
```

---

## Quick Reference: Where Does My Code Go?

| Code Type | Layer | Example |
|-----------|-------|---------|
| **Utilities, Logging, Constants** | 1_Core | `MyMod_Log`, `MyMod_Constants` |
| **Game Modules** | 3_Game | `MyMod_Module : MissionModule` |
| **Settings Classes** | 3_Game | `MyMod_Settings` |
| **Managers** | 3_Game | `MyMod_Manager` |
| **RPC Handlers** | 3_Game | `MyMod_RPCManager` |
| **`modded class ItemBase`** | 4_World | `modded class ItemBase { ... }` |
| **`modded class PlayerBase`** | 4_World | `modded class PlayerBase { ... }` |
| **Custom Items/Entities** | 4_World | `class MyItem : ItemBase` |
| **Actions** | 4_World | `ActionEatMeat`, `ActionOpen` |
| **Recipes** | 4_World | `RecipeCraftKnife`, `RecipeCookMeat` |
| **`modded class MissionGameplay`** | 5_Mission | `modded class MissionGameplay` |
| **GUI Classes** | 5_Mission | `MyMod_Menu : UIScriptedMenu` |
| **Player Input Handlers** | 5_Mission | Input handling code |

---

## Real-World Examples

### Example 1: Mod with Settings and Item Modifications

**3_Game Layer:**
```c
// scripts/3_Game/MyMod_Module.c
class MyMod_Module : MissionModule
{
	ref MyMod_Settings m_Settings;
	
	override void OnInit()
	{
		super.OnInit();
		m_Settings = MyMod_Settings.Load();
	}
}

// scripts/3_Game/MyMod_Settings.c
class MyMod_Settings
{
	int m_LogLevel = 1;
	
	static ref MyMod_Settings Load()
	{
		MyMod_Settings settings = new MyMod_Settings();
		// Load from JSON
		return settings;
	}
}
```

**4_World Layer:**
```c
// scripts/4_World/ItemBase.c
modded class ItemBase
{
	bool m_MyMod_HasCustomFeature;
	
	override void OnStoreLoad(ParamsReadContext ctx, int version)
	{
		super.OnStoreLoad(ctx, version);
		if (version >= 12345)
		{
			ctx.Read(m_MyMod_HasCustomFeature);
		}
	}
}
```

**5_Mission Layer:**
```c
// scripts/5_Mission/MissionGameplay.c
modded class MissionGameplay
{
	override void OnInit()
	{
		super.OnInit();
		Print("[MyMod] Mission initialized");
	}
}
```

### Example 2: What Happens if You Break the Rules

```c
// ❌ WRONG - ItemBase mod in 3_Game layer
// File: scripts/3_Game/ItemBase.c
modded class ItemBase  // ❌ ERROR!
{
	// This will cause:
	// 1. Compile errors (ItemBase not available in 3_Game)
	// 2. Runtime errors (entity modifications not loaded)
	// 3. Serialization issues (data won't save/load correctly)
}
```

**Solution:** Move to 4_World layer where `ItemBase` is available.

---

## Summary

**Key Rules:**

1. **1_Core** - Only base utilities, no game/entity code
2. **2_GameLib** - Game library abstractions (optional, rarely used)
3. **3_Game** - Modules, managers, settings (most mod logic)
   - **CRITICAL:** Cannot mod entities here (no `modded class ItemBase`!)
4. **4_World** - All entity modifications and custom entities
   - **MUST** put all `modded class ItemBase/PlayerBase/etc.` here
5. **5_Mission** - Mission logic and GUI only

**The Golden Rule:**
- **Lower layers (1-3) cannot create/mod classes from higher layers (4-5)**
- **Entity modifications MUST be in 4_World**
- **Module code MUST be in 3_Game**
- **Mission code MUST be in 5_Mission**

**When in Doubt:**
- Settings/Modules/Managers → **3_Game**
- Entity mods (`modded class ItemBase`) → **4_World**
- Mission/GUI code → **5_Mission**

---

**Related Guides:**
- [Basic Mod Structure](Basic-Mod-Structure.md) - Mod organization overview
- [How to Create a Mod](How-To-Create-Mod.md) - Step-by-step mod creation
- [How to Create Actions](How-To-Actions.md) - Creating player actions
- [How to Create Recipes](How-To-Recipes.md) - Creating crafting recipes
- [Tips: Modded Classes](../Tips/Tips-Modded-Classes.md) - How to extend vanilla classes
- [Module System](Module-System.md) - Understanding the module system
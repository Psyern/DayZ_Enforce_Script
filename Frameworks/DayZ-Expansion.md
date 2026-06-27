# DayZ Expansion

DayZ Expansion is a comprehensive modding framework and collection of mods for DayZ Standalone.

## Overview

**Location:** `Mods/DayZ-Expansion-Scripts-release/`

DayZ Expansion provides:
- Core framework systems
- AI systems
- Base building enhancements
- Vehicle systems
- Market system
- Quest system
- Teleporter
- And many more features

## Structure

DayZ Expansion uses a modular architecture with preload modules:

```
DayZ-Expansion-Scripts-release/
├── 0_DayZExpansion_Preload/       # Main preload
├── 0_DayZExpansion_Core_Preload/  # Core preload
├── 0_DayZExpansion_AI_Preload/    # AI preload
├── DayZExpansion/
│   ├── Core/                      # Core framework
│   ├── AI/                        # AI systems
│   ├── BaseBuilding/              # Base building
│   ├── Vehicles/                  # Vehicle systems
│   ├── Market/                    # Market system
│   ├── Quests/                    # Quest system
│   └── ...                        # Many more modules
```

## Core Framework

### Expansion Module System

Expansion uses a module-based architecture:

```c
class MyExpansionModule : ExpansionModule
{
	override void OnInit()
	{
		super.OnInit();
		// Initialize module
	}
	
	override void OnUpdate(float delta_time)
	{
		super.OnUpdate(delta_time);
		// Update logic
	}
	
	override void EnableRPC()
	{
		super.EnableRPC();
		Expansion_EnableRPCManager();
		Expansion_RegisterServerRPC("RPC_MyServerRPC");
		Expansion_RegisterClientRPC("RPC_MyClientRPC");
	}
	
	void RPC_MyServerRPC(PlayerIdentity sender, Object target, ParamsReadContext ctx)
	{
		// Handle server RPC
	}
	
	void RPC_MyClientRPC(PlayerIdentity sender, Object target, ParamsReadContext ctx)
	{
		// Handle client RPC
	}
}
```

### Expansion RPC System

Expansion provides a string-based RPC system:

```c
// Register RPCs
override void EnableRPC()
{
	super.EnableRPC();
	Expansion_EnableRPCManager();
	Expansion_RegisterServerRPC("RPC_Test");
	Expansion_RegisterClientRPC("RPC_Update");
}

// Send RPCs
auto rpc = Expansion_CreateRPC("RPC_Update");
rpc.Write(value);
rpc.Expansion_Send(true, identity);  // true = guaranteed, identity = target player
```

### Expansion Settings

Expansion uses a settings system for configuration:

```c
class ExpansionGarageSettings : ExpansionSettingsBase
{
	TStringArray EntityWhitelist = new TStringArray;
	int MaxStorableVehicles = 10;
	float SearchRadius = 100.0;
	
	void ExpansionGarageSettings()
	{
		EntityWhitelist.Insert("CarSedan");
		EntityWhitelist.Insert("Truck_01");
	}
}
```

## Key Systems

### AI System

Expansion provides comprehensive AI systems:

- Patrol system
- AI vehicles
- AI behaviors
- AI spawning

### Base Building

Expansion enhances base building:

- Territory system
- Raiding system
- Building restrictions
- Material systems

### Vehicles

Expansion adds vehicle features:

- Vehicle storage (Garage)
- Vehicle customization
- Vehicle spawning
- Vehicle persistence

### Market System

Expansion includes a market system:

- Item trading
- Currency system
- NPC traders
- Market UI

### Quest System

Expansion provides a quest framework:

- Quest creation
- Quest progression
- Quest rewards
- Quest UI

### Teleporter

Expansion includes teleportation:

- Teleport points
- Teleport restrictions
- Teleport UI
- Teleport permissions

## Preload System

Expansion uses preload modules for initialization order:

### Preload Structure

Each expansion module has a corresponding preload:

```
0_DayZExpansion_AI_Preload/
├── 4_world/           # Early world initialization
├── 5_mission/         # Early mission initialization
├── Common/            # Shared preload code
└── config.cpp         # Preload configuration
```

Preload modules ensure initialization order:

1. Main Preload (`0_DayZExpansion_Preload`)
2. Core Preload (`0_DayZExpansion_Core_Preload`)
3. Feature Preloads (AI, BaseBuilding, etc.)
4. Main Modules (`DayZExpansion/`)

## Module Patterns

### Creating Expansion Modules

```c
class MyExpansionModule : ExpansionModule
{
	override void OnInit()
	{
		super.OnInit();
		
		// Initialize settings
		LoadSettings();
		
		// Register RPCs
		EnableRPC();
		
		// Initialize systems
		InitSystems();
	}
	
	void LoadSettings()
	{
		// Load from JSON or default
	}
	
	void InitSystems()
	{
		// Initialize module systems
	}
}
```

### Expansion Settings Pattern

```c
class MyExpansionSettings : ExpansionSettingsBase
{
	bool Enabled = true;
	int MaxItems = 100;
	float SearchRadius = 50.0;
	
	void MyExpansionSettings()
	{
		// Initialize defaults
	}
	
	override void LoadDefaults()
	{
		Enabled = true;
		MaxItems = 100;
		SearchRadius = 50.0;
	}
}
```

### Expansion RPC Pattern

```c
class MyExpansionModule : ExpansionModule
{
	override void EnableRPC()
	{
		super.EnableRPC();
		Expansion_EnableRPCManager();
		
		// Register server RPCs
		Expansion_RegisterServerRPC("RPC_RequestData");
		Expansion_RegisterServerRPC("RPC_Action");
		
		// Register client RPCs
		Expansion_RegisterClientRPC("RPC_ReceiveData");
		Expansion_RegisterClientRPC("RPC_UpdateUI");
	}
	
	// Server RPC handlers
	void RPC_RequestData(PlayerIdentity sender, Object target, ParamsReadContext ctx)
	{
		// Validate sender
		if (!GetPermissionsManager().HasPermission("MyMod.View", sender))
			return;
		
		// Get data
		MyData data = GetData(sender);
		
		// Send response
		auto rpc = Expansion_CreateRPC("RPC_ReceiveData");
		rpc.Write(data);
		rpc.Expansion_Send(true, sender);
	}
	
	// Client RPC handlers
	void RPC_ReceiveData(PlayerIdentity sender, Object target, ParamsReadContext ctx)
	{
		MyData data;
		if (!ctx.Read(data))
			return;
		
		// Process data on client
		ProcessData(data);
	}
}
```

## Using Expansion Framework

### Dependencies

If creating an addon mod for Expansion:

```cpp
// config.cpp
class CfgPatches
{
	class MyMod_Scripts
	{
		requiredAddons[] = {
			"DZ_Data",
			"DayZExpansion_Core_Scripts"
		};
	};
};
```

### Creating Addon Modules

```c
// Create module that integrates with Expansion
class MyAddonModule : ExpansionModule
{
	override void OnInit()
	{
		super.OnInit();
		
		// Integrate with Expansion systems
		if (GetExpansionSettings().IsFeatureEnabled("MyFeature"))
		{
			InitializeFeature();
		}
	}
	
	void InitializeFeature()
	{
		// Initialize feature
	}
}
```

### Using Expansion Systems

```c
// Use Expansion permissions
if (GetPermissionsManager().HasPermission("MyMod.Action", identity))
{
	// Perform action
}

// Use Expansion logging
EXLogPrint("MyMod", "Information message");

// Use Expansion settings
ExpansionGarageSettings settings = ExpansionSettings.GetInstance().GetGarageSettings();
int maxVehicles = settings.MaxStorableVehicles;
```

## Common Expansion Features

### Territory System

```c
// Check if player is in territory
ExpansionTerritoryModule territoryModule = ExpansionTerritoryModule.Cast(GetExpansionModuleManager().GetModule(ExpansionTerritoryModule));
if (territoryModule)
{
	if (territoryModule.IsPlayerInTerritory(player))
	{
		// Handle territory logic
	}
}
```

### Permission System

```c
// Check permissions
ExpansionPermissionsModule permModule = ExpansionPermissionsModule.Cast(GetExpansionModuleManager().GetModule(ExpansionPermissionsModule));
if (permModule)
{
	if (permModule.HasPermission("MyMod.Action", identity))
	{
		// Allowed action
	}
}
```

### Market Integration

```c
// Use market system
ExpansionMarketModule marketModule = ExpansionMarketModule.Cast(GetExpansionModuleManager().GetModule(ExpansionMarketModule));
if (marketModule)
{
	// Interact with market
}
```

## Best Practices

### 1. Use Expansion Module System

Create modules that integrate with Expansion:

```c
// ✅ Good - Expansion module
class MyModule : ExpansionModule
{
	// Module logic
}

// ❌ Avoid - Standalone system
class MyManager
{
	// Not integrated
}
```

### 2. Use Expansion RPC System

Use Expansion's RPC system for consistency:

```c
// ✅ Good - Expansion RPC
Expansion_RegisterServerRPC("RPC_Test");

// ❌ Avoid - Native RPC
	if (!g_Game)
		return;
	
	g_Game.RPC(null, MYMOD_RPC_TEST, data, false);
```

### 3. Respect Expansion Permissions

Check permissions before actions:

```c
// ✅ Good - Permission check
if (GetPermissionsManager().HasPermission("MyMod.Action", identity))
{
	// Action
}

// ❌ Avoid - No check
// Direct action
```

### 4. Use Expansion Settings

Use Expansion settings system for configuration:

```c
// ✅ Good - Expansion settings
class MySettings : ExpansionSettingsBase
{
	// Settings
}

// ❌ Avoid - Global variables
int g_MySetting = 100;
```

## Related Documentation

- [Basic Mod Structure](Basic-Mod-Structure.md)
- [How to Use RPC](../How-To/How-To-RPC.md)
- [Module System](Module-System.md)

## Notes

- Expansion uses preload modules for initialization order
- Expansion provides many integrated systems
- Addon mods can integrate with Expansion features
- Expansion has its own RPC and settings systems

---

**Location:** `Mods/DayZ-Expansion-Scripts-release/`
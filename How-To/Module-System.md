# Module System

Understanding and using the module system in DayZ mods is essential for organized, maintainable code.

## What is a Module?

A module is a self-contained unit of functionality that:
- Initializes when the game starts
- Updates during the game loop (optional)
- Handles cleanup when unloaded
- Manages its own state and resources

## Module Types

Different frameworks provide different module base classes:

### Community Framework (CF)

CF provides layer-specific module classes. CF modules are the standard approach when using Community Framework.

**See:** [Community Framework](../Frameworks/Community-Framework/Community-Framework.md) for complete CF module documentation.

**Quick Overview:**
- **Module Base Classes:**
  - `CF_ModuleCore` - For 1_Core and 2_GameLib layers
  - `CF_ModuleGame` - For 3_Game layer
  - `CF_ModuleWorld` - For 4_World and 5_Mission layers

- **⚠️ Important:** All CF module events are disabled by default. You must call the corresponding `Enable*()` method to activate an event.

- **Key Events:**
  - `CF_ModuleCore`: `OnMissionStart`, `OnUpdate`, `OnMPSessionStart`, `OnRespawn`, `OnChat`, etc.
  - `CF_ModuleGame`: `OnRPC`
  - `CF_ModuleWorld`: `OnInvokeConnect`, `OnClientNew`, `OnClientReady`, etc.

**Basic Example:**
```c
class MyMod_Module : CF_ModuleWorld
{
	override void OnInit()
	{
		super.OnInit();
		EnableInvokeConnect();  // Enable player connect event
		EnableUpdate();          // Enable update loop
	}
	
	override void OnInvokeConnect(Class sender, CF_EventArgs args)
	{
		super.OnInvokeConnect(sender, args);
		// Handle player connect
	}
}
```

**Registration:**
```c
void MyMod_Init()
{
	CF_ModuleWorldManager.GetInstance().RegisterModule(MyMod_WorldModule);
}
```

### DayZ Expansion

Expansion provides a unified module class:

```c
class MyExpansionModule : ExpansionModule
{
	override void OnInit()
	{
		super.OnInit();
		// Initialize
	}
	
	override void EnableRPC()
	{
		super.EnableRPC();
		// Register RPCs
	}
}
```

### DayZ Native

DayZ provides base module classes:

```c
// Mission module
class MyMod_Module : MissionModule
{
	override void OnInit()
	{
		super.OnInit();
		// Initialize
	}
	
	override void OnUpdate(float delta_time)
	{
		super.OnUpdate(delta_time);
		// Update logic
	}
}
```

## Basic Module Structure

A basic module follows this pattern:

```c
class MyMod_Module : CF_ModuleWorld
{
	// Member variables
	ref MyMod_Settings m_Settings;
	ref MyMod_Manager m_Manager;
	
	// Constructor
	void MyMod_Module()
	{
		m_Settings = new MyMod_Settings();
		m_Manager = new MyMod_Manager();
	}
	
	// Initialization
	override void OnInit()
	{
		super.OnInit();
		
		// Load settings
		m_Settings.Load();
		
		// Initialize manager
		m_Manager.Initialize();
		
		// Register RPCs
		RegisterRPCs();
		
		// Enable update loop (if needed)
		EnableUpdate();
	}
	
	// Update loop (optional)
	override void OnUpdate(float delta_time)
	{
		super.OnUpdate(delta_time);
		
		// Update logic
		m_Manager.Update(delta_time);
	}
	
	// RPC registration
	void RegisterRPCs()
	{
		CF_GetRPCManager().RegisterServerRPC("MyMod", "RPC_Test", RPC_Test, SingleplayerExecutionType.Server);
	}
	
	// RPC handler
	void RPC_Test(Param4<CallType, ParamsReadContext, PlayerIdentity, Object> data)
	{
		// Handle RPC
	}
	
	// Cleanup
	override void OnUnloadModule()
	{
		super.OnUnloadModule();
		
		// Save settings
		m_Settings.Save();
		
		// Cleanup manager
		m_Manager.Cleanup();
	}
}
```

## Module Lifecycle

Modules follow a specific lifecycle:

### 1. Construction

```c
void MyMod_Module()
{
	// Initialize member variables
	m_Settings = new MyMod_Settings();
}
```

### 2. Initialization

```c
override void OnInit()
{
	super.OnInit();
	// Initialize module systems
}
```

### 3. Update (Optional)

```c
override void OnUpdate(float delta_time)
{
	super.OnUpdate(delta_time);
	// Update logic
}
```

### 4. Cleanup

```c
override void OnUnloadModule()
{
	super.OnUnloadModule();
	// Cleanup resources
}
```

## Module Registration

### CF Module Registration

```c
// In initialization function
void MyMod_Init()
{
	// Register world module
	CF_ModuleWorldManager.GetInstance().RegisterModule(MyMod_WorldModule);
	
	// Register game module
	CF_ModuleGameManager.GetInstance().RegisterModule(MyMod_GameModule);
}
```

### Expansion Module Registration

```c
// Expansion modules are auto-registered via config
// Or manually:
ExpansionModuleManager.GetInstance().RegisterModule(MyExpansionModule);
```

### DayZ Native Module Registration

```c
// Native modules are registered via config.cpp
// Or in mission init:
MissionModule module = new MyMod_Module();
module.Init();
```

## Module Managers

Module managers handle module lifecycle:

### CF Module Managers

```c
// Get module managers
CF_ModuleWorldManager worldMgr = CF_ModuleWorldManager.GetInstance();
CF_ModuleGameManager gameMgr = CF_ModuleGameManager.GetInstance();

// Register modules
worldMgr.RegisterModule(MyMod_WorldModule);
gameMgr.RegisterModule(MyMod_GameModule);

// Get modules
MyMod_WorldModule module = worldMgr.GetModule(MyMod_WorldModule);
```

### Expansion Module Manager

```c
// Get module manager
ExpansionModuleManager moduleMgr = ExpansionModuleManager.GetInstance();

// Get module
MyExpansionModule module = ExpansionModuleManager.GetModule(MyExpansionModule);
```

## Module Patterns

### Singleton Module Pattern

```c
class MyMod_Module : CF_ModuleWorld
{
	static MyMod_Module s_Instance;
	
	static MyMod_Module GetInstance()
	{
		if (!s_Instance)
		{
			s_Instance = new MyMod_Module();
			s_Instance.Init();
		}
		return s_Instance;
	}
	
	override void OnInit()
	{
		super.OnInit();
		s_Instance = this;
	}
}
```

### Settings Module Pattern

```c
class MyMod_SettingsModule : CF_ModuleGame
{
	ref MyMod_Settings m_Settings;
	
	override void OnInit()
	{
		super.OnInit();
		m_Settings = new MyMod_Settings();
		m_Settings.Load();
	}
	
	MyMod_Settings GetSettings()
	{
		return m_Settings;
	}
	
	override void OnUnloadModule()
	{
		super.OnUnloadModule();
		m_Settings.Save();
	}
}
```

### RPC Module Pattern

```c
class MyMod_RPCModule : CF_ModuleWorld
{
	override void OnInit()
	{
		super.OnInit();
		RegisterRPCs();
	}
	
	void RegisterRPCs()
	{
		CF_GetRPCManager().RegisterServerRPC("MyMod", "RPC_Test", RPC_Test, SingleplayerExecutionType.Server);
		CF_GetRPCManager().RegisterClientRPC("MyMod", "RPC_Update", RPC_Update, SingleplayerExecutionType.Client);
	}
	
	void RPC_Test(Param4<CallType, ParamsReadContext, PlayerIdentity, Object> data)
	{
		// Handle server RPC
	}
	
	void RPC_Update(Param4<CallType, ParamsReadContext, PlayerIdentity, Object> data)
	{
		// Handle client RPC
	}
}
```

### Update Module Pattern

```c
class MyMod_UpdateModule : CF_ModuleWorld
{
	array<MyMod_Updateable> m_Updateables = {};
	
	override void OnInit()
	{
		super.OnInit();
		EnableUpdate();
	}
	
	override void OnUpdate(float delta_time)
	{
		super.OnUpdate(delta_time);
		
		foreach (MyMod_Updateable updateable : m_Updateables)
		{
			updateable.Update(delta_time);
		}
	}
	
	void RegisterUpdateable(MyMod_Updateable updateable)
	{
		m_Updateables.Insert(updateable);
	}
	
	void UnregisterUpdateable(MyMod_Updateable updateable)
	{
		int index = m_Updateables.Find(updateable);
		if (index >= 0)
		{
			m_Updateables.Remove(index);
		}
	}
}
```

## Module Communication

### Accessing Other Modules

```c
// CF modules
CF_ModuleWorldManager worldMgr = CF_ModuleWorldManager.GetInstance();
MyMod_OtherModule otherModule = worldMgr.GetModule(MyMod_OtherModule);
if (otherModule)
{
	otherModule.DoSomething();
}

// Expansion modules
ExpansionModuleManager moduleMgr = ExpansionModuleManager.GetInstance();
MyExpansionModule module = moduleMgr.GetModule(MyExpansionModule);
if (module)
{
	module.DoSomething();
}
```

### Module Events

```c
// Publish event
CF_EventHandler.Invoke("MyEvent", new Param1<string>("data"));

// Subscribe to event
class MyMod_Module : CF_ModuleWorld
{
	override void OnInit()
	{
		super.OnInit();
		CF_EventHandler.Subscribe("MyEvent", OnMyEvent);
	}
	
	void OnMyEvent(Param params)
	{
		Param1<string> data = Param1<string>.Cast(params);
		// Handle event
	}
}
```

## Best Practices

### 1. Organize by Functionality

Each module should have a clear, focused purpose:

```c
// ✅ Good - Focused module
class MyMod_InventoryModule : CF_ModuleWorld
{
	// Only handles inventory
}

// ❌ Avoid - Mixed concerns
class MyMod_EverythingModule : CF_ModuleWorld
{
	// Handles everything
}
```

### 2. Initialize in Correct Order

Modules may depend on each other, initialize in order:

```c
// ✅ Good - Initialize dependencies first
void MyMod_Init()
{
	// Settings module first
	CF_ModuleGameManager.GetInstance().RegisterModule(MyMod_SettingsModule);
	
	// Then modules that use settings
	CF_ModuleWorldManager.GetInstance().RegisterModule(MyMod_WorldModule);
}

// ❌ Avoid - Random order
void MyMod_Init()
{
	CF_ModuleWorldManager.GetInstance().RegisterModule(MyMod_WorldModule);  // Needs settings!
	CF_ModuleGameManager.GetInstance().RegisterModule(MyMod_SettingsModule);
}
```

### 3. Clean Up Resources

Always clean up resources in OnUnloadModule:

```c
// ✅ Good - Cleanup
override void OnUnloadModule()
{
	super.OnUnloadModule();
	m_Settings.Save();
	m_Manager.Cleanup();
}

// ❌ Avoid - No cleanup
override void OnUnloadModule()
{
	super.OnUnloadModule();
	// Resources leak
}
```

### 4. Use Appropriate Layer

Put modules in the correct layer:

```c
// ✅ Good - Game logic in game layer
class MyMod_GameModule : CF_ModuleGame
{
	// Game logic
}

// ❌ Avoid - Game logic in core layer
class MyMod_CoreModule : CF_ModuleCore
{
	// Game logic (wrong layer!)
}
```

### 5. Enable Update Only When Needed

Don't enable update loop unnecessarily:

```c
// ✅ Good - Update only when needed
override void OnInit()
{
	super.OnInit();
	if (m_NeedsUpdate)
	{
		EnableUpdate();
	}
}

// ❌ Avoid - Always enabling update
override void OnInit()
{
	super.OnInit();
	EnableUpdate();  // Even when not needed
}
```

## Common Mistakes

### Missing Super Call

Always call super methods:

```c
// ❌ Wrong
override void OnInit()
{
	// Missing super.OnInit()
}

// ✅ Correct
override void OnInit()
{
	super.OnInit();
}
```

### Not Checking Module Existence

Always check if module exists before using:

```c
// ❌ Wrong
MyMod_Module module = moduleMgr.GetModule(MyMod_Module);
module.DoSomething();  // Crashes if module doesn't exist

// ✅ Correct
MyMod_Module module = moduleMgr.GetModule(MyMod_Module);
if (module)
{
	module.DoSomething();
}
```

### Circular Dependencies

Avoid circular dependencies between modules:

```c
// ❌ Wrong - Circular dependency
class ModuleA
{
	void Init()
	{
		ModuleB.GetInstance().DoSomething();
	}
}

class ModuleB
{
	void Init()
	{
		ModuleA.GetInstance().DoSomething();
	}
}

// ✅ Correct - One-way dependency
class ModuleA
{
	void Init()
	{
		// Initialize first
	}
}

class ModuleB
{
	void Init()
	{
		ModuleA.GetInstance().DoSomething();  // ModuleB depends on ModuleA
	}
}
```

## Summary

Modules are essential for organized DayZ modding:

- **Use appropriate module types** for your layer
- **Follow lifecycle patterns** (Init, Update, Cleanup)
- **Register modules** with managers
- **Communicate between modules** properly
- **Clean up resources** in OnUnloadModule
- **Avoid circular dependencies**
- **Organize by functionality**

---

**Related Topics:**
- [Basic Mod Structure](Basic-Mod-Structure.md)
- [Community Framework](Community-Framework.md)
- [DayZ Expansion](DayZ-Expansion.md)
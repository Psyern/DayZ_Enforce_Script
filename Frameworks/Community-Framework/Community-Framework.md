# Community Framework (CF)

Community Framework is a comprehensive modding framework for DayZ Standalone that provides utilities, systems, and best practices for mod development.

**⚠️ Framework Standard:** We always load **CF, Dabs, and your mod together**. CF is the primary framework for mod development. All examples and guides in this wiki prioritize CF usage when applicable.

## Overview

**Location:** `Mods/CF/`

Community Framework provides:
- Module system for organized mod functionality
- RPC management system (string-based registration)
- Logging and debugging utilities (with context tracing)
- Notification system for in-game messages
- ModStorage for data persistence
- NetworkedVariables for variable synchronization
- Localiser for multi-part localized strings
- LifecycleEvents for game/mission lifecycle hooks
- XML parsing and file utilities
- Type conversion and serialization (basic and advanced)
- Event system (simple and typed)
- Expression VM
- InputBindings for custom input handling
- Cryptography (SHA-256)
- And much more

## Key Features

### Module System

CF provides a robust module system for organizing mod functionality:

```c
class MyMod_Module : CF_ModuleWorld
{
	override void OnInit()
	{
		super.OnInit();
		
		// Enable events you need
		EnableInvokeConnect();  // Player connect event
		EnableUpdate();          // Update loop
	}
	
	override void OnInvokeConnect(Class sender, CF_EventArgs args)
	{
		super.OnInvokeConnect(sender, args);
		auto playerArgs = CF_EventPlayerArgs.Cast(args);
		PlayerBase player = playerArgs.Player;
		// Handle player connect
	}
	
	override void OnUpdate(Class sender, CF_EventArgs args)
	{
		super.OnUpdate(sender, args);
		auto updateArgs = CF_EventUpdateArgs.Cast(args);
		float deltaTime = updateArgs.DeltaTime;
		// Update logic
	}
}
```

**Module types:**
- `CF_ModuleCore` - For 1_Core and 2_GameLib layers
- `CF_ModuleGame` - For 3_Game layer
- `CF_ModuleWorld` - For 4_World and 5_Mission layers

**⚠️ Important:** All CF module events are disabled by default. You must call the corresponding `Enable*()` method to activate an event. Once enabled, events cannot be disabled.

**See:** [Module System](../How-To/Module-System.md) for complete module events list and examples.

### RPC Management

CF's RPC system uses string-based registration instead of enum values:

```c
class MyMod_Module : CF_ModuleWorld
{
	override void OnInit()
	{
		super.OnInit();
		
		// Register Server RPC
		CF_GetRPCManager().RegisterServerRPC("MyMod", "RPC_Test", RPC_Test, SingleplayerExecutionType.Server);
		
		// Register Client RPC
		CF_GetRPCManager().RegisterClientRPC("MyMod", "RPC_UpdateClient", RPC_UpdateClient, SingleplayerExecutionType.Client);
	}
	
	void RPC_Test(Param4<CallType, ParamsReadContext, PlayerIdentity, Object> data)
	{
		// Handle RPC
	}
}
```

Sending RPCs:

```c
// Client to Server
CF_GetRPCManager().SendRPC("MyMod", "RPC_Test", new Param1<string>("data"), false);

// Server to Client
CF_GetRPCManager().SendRPC("MyMod", "RPC_UpdateClient", new Param1<int>(42), true, identity);

// Broadcast to All
CF_GetRPCManager().SendRPC("MyMod", "RPC_Broadcast", new Param1<string>("message"), true);
```

### Logging System

CF provides a comprehensive logging system:

```c
class MyMod_Logger
{
	static CF_Log s_Log = new CF_Log("MyMod");

	void LogExample()
	{
		s_Log.Info("Information message");
		s_Log.Warn("Warning message");
		s_Log.Error("Error message");
		s_Log.Critical("Critical error");
		
		// With formatting
		s_Log.Info("Player %1 joined", playerName);
	}
}
```

**Log levels:**
- `Trace` - Most detailed (requires `CF_TRACE_ENABLED`)
- `Debug` - Debug info (requires `CF_DEBUG_ENABLED`)
- `Info` - General information
- `Warn` - Warnings
- `Error` - Errors (produces stack trace)
- `Critical` - Critical errors (produces stack trace)

**See:** [How to Use CF Logging](CF/How-To-CF-Logger.md) for complete guide including context tracing.

### XML Parsing

CF includes XML parsing utilities:

```c
CF_XML xml = new CF_XML();
if (xml.LoadFile("MyMod/Config/Settings.xml"))
{
	CF_XMLNode root = xml.GetRoot();
	CF_XMLNode node = root.Find("MySetting");
	if (node)
	{
		string value = node.GetValue();
	}
}
```

### File Utilities

CF provides comprehensive file and directory utilities:

```c
// Check if file exists
if (CF_File.Exists("MyMod/Config/settings.json"))
{
	// Read file
	string content = CF_File.Read("MyMod/Config/settings.json");
	
	// Write file
	CF_File.Write("MyMod/Config/output.json", content);
}

// Directory operations
if (CF_Directory.Exists("MyMod/Data"))
{
	array<ref CF_File> files();
	CF_Directory.GetFiles("MyMod/Data/*.json", files);
	
	foreach (auto file : files)
	{
		Print(file.GetFileName());
	}
}

// Path utilities
string fileName = CF_Path.GetFileName("MyMod/Config/settings.json");
string extension = CF_Path.GetExtension("MyMod/Config/settings.json");
string directory = CF_Path.GetDirectoryName("MyMod/Config/settings.json");
```

**CF_File methods:**
- `GetFullPath()`, `GetFileName()`, `GetExtension()`
- `CreateStream()`, `Delete()`, `Rename()`, `Move()`, `Copy()`

**CF_Directory methods:**
- `GetFiles()` - Get files matching pattern
- `CreateDirectory()` - Create directories recursively

**CF_Path methods:**
- `GetDirectoryName()`, `GetFileName()`, `GetExtension()`

### Type Conversion

CF includes type conversion utilities:

**Basic Conversion (CF_Convert):**
```c
// String to number
int value = CF_Convert.ToInt("42");
float fvalue = CF_Convert.ToFloat("3.14");
bool bvalue = CF_Convert.ToBool("1");

// Number to string
string svalue = CF_Convert.ToString(42);
```

**Advanced Type Converters (CF_TypeConverter):**
For custom type conversion and serialization:

```c
// Get type converter for a class
auto converter = CF_TypeConverter.Get(PlayerStatBase);

// Read from instance variable
converter.Read(GetGame().GetPlayer(), "m_StatDiet");

// Convert to different types
float fvalue = converter.GetFloat();
string svalue = converter.GetString();

// Set and write back
converter.SetFloat(0.5);
converter.Write(GetGame().GetPlayer(), "m_StatDiet");
```

**Custom Type Converters:**
Create custom converters for your classes:

```c
[CF_RegisterTypeConverter(CF_TypeConverterMyClass)]
class CF_TypeConverterMyClass : CF_TypeConverterClass
{
	override void SetFloat(float value)
	{
		// Custom conversion logic
	}
	
	override float GetFloat()
	{
		// Custom conversion logic
		return 0.0;
	}
}
```

### Notification System

CF provides a notification system for displaying in-game messages to players:

```c
// Send notification to specific player
void SendNotification(PlayerBase player, string message)
{
	PlayerIdentity identity = player.GetIdentity();
	if (!identity)
		return;
	
	NotificationSystem.Create(
		new StringLocaliser("MyMod Title"),
		new StringLocaliser(message),
		"set:dayz_gui image:checkmark",
		ARGB(255, 0, 255, 0),
		5.0,
		identity
	);
}

// Broadcast to all players
void BroadcastNotification(string message)
{
	NotificationSystem.Create(
		new StringLocaliser("Server Announcement"),
		new StringLocaliser(message),
		"set:dayz_gui image:icon_gear",
		ARGB(255, 255, 255, 0),
		10.0,
		NULL  // NULL = broadcast to all
	);
}
```

**⚠️ CRITICAL:** `NotificationSystem` is a static system class. **NEVER** create an instance of it. Always use the static methods directly.

**See:** [How to Use CF Notifications](CF/How-To-CF-Notifications.md) for complete guide.

### Event System

CF provides a comprehensive event system for reactive programming:

**Simple Event Handler:**
```c
// Register event handler
CF_EventHandler.Subscribe("MyEvent", OnMyEvent);

// Trigger event
CF_EventHandler.Invoke("MyEvent", new Param1<string>("data"));

// Event handler
void OnMyEvent(Param params)
{
	Param1<string> data = Param1<string>.Cast(params);
	Print("Event received: " + data.param1);
}
```

**Typed Event Handler (CF_EventHandlerT):**
```c
class MyCustomEventArgs : CF_EventArgs
{
	int someValue;
	float moreData;
}

// Create typed event handler
ref CF_EventHandlerT<MyCustomEventArgs> m_MyEvent = new CF_EventHandlerT<MyCustomEventArgs>();

// Subscribe
void RegisterEvent()
{
	m_MyEvent.AddSubscriber(ScriptCaller.Create(this.OnMyEvent));
}

// Invoke
void TriggerEvent()
{
	MyCustomEventArgs args = new MyCustomEventArgs();
	args.someValue = 42;
	args.moreData = 3.14;
	m_MyEvent.Invoke(this, args);
}

// Handler
void OnMyEvent(Class sender, MyCustomEventArgs args)
{
	Print("Value: " + args.someValue);
}
```

**Lifecycle Events:**
Subscribe to game and mission lifecycle events:

```c
void MyMod_Module()
{
	// Subscribe to lifecycle events
	CF_LifecycleEvents.OnGameCreate.AddSubscriber(ScriptCaller.Create(this.OnGameCreate));
	CF_LifecycleEvents.OnMissionCreate.AddSubscriber(ScriptCaller.Create(this.OnMissionCreate));
	CF_LifecycleEvents.OnMissionDestroy.AddSubscriber(ScriptCaller.Create(this.OnMissionDestroy));
}

void OnGameCreate()
{
	Print("Game created - initialize global cache");
}

void OnMissionCreate()
{
	Print("Mission created - reset per-mission state");
}

void OnMissionDestroy()
{
	Print("Mission destroyed - cleanup");
}
```

**Available Lifecycle Events:**
- `OnGameCreate` - Once per game launch
- `OnGameDestroy` - When closing the game
- `OnMissionCreate` - Every time a local mission starts or during server join
- `OnMissionDestroy` - Every time a local mission ends or during disconnect

## Structure

```
CF/
└── scripts/
    └── JM/
        └── CF/
            └── Scripts/
                ├── 1_Core/
                │   └── CommunityFramework/
                │       ├── Module/          # Module system
                │       ├── Logging/         # Logging utilities
                │       ├── Files/           # File utilities
                │       └── Utils/
                ├── 2_GameLib/
                │   └── CommunityFramework/
                │       └── EventHandler/    # Event system
                ├── 3_Game/
                │   └── CommunityFramework/
                │       ├── Config/          # Configuration
                │       ├── Module/          # Game modules
                │       ├── RPC/             # RPC management
                │       └── XML/             # XML parsing
                ├── 4_World/
                │   └── CommunityFramework/
                │       └── Module/          # World modules
                └── config.cpp
```

## Entry Point

CF replaces the game creation function:

```c
// config.cpp
class gameScriptModule
{
	value = "CF_CreateGame";
	files[] = {"JM/CF/Scripts/3_Game"};
}

// CommunityFramework.c
CGame CF_CreateGame()
{
	CreateGame();
	CF._GameInit();
	return g_Game;
}
```

This allows CF to initialize before other mods.

## Using CF in Your Mod

### Basic Setup

1. **Depend on CF:**

```cpp
// config.cpp
class CfgPatches
{
	class MyMod_Scripts
	{
		requiredAddons[] = {"DZ_Data", "CF_Scripts"};
	};
};
```

2. **Create a Module:**

```c
class MyMod_Module : CF_ModuleWorld
{
	override void OnInit()
	{
		super.OnInit();
		
		// Initialize RPCs
		CF_GetRPCManager().RegisterServerRPC("MyMod", "RPC_Test", RPC_Test, SingleplayerExecutionType.Server);
		
		// Enable update loop
		EnableUpdate();
	}
	
	override void OnUpdate(float delta_time)
	{
		super.OnUpdate(delta_time);
		// Update logic
	}
	
	void RPC_Test(Param4<CallType, ParamsReadContext, PlayerIdentity, Object> data)
	{
		// Handle RPC
	}
}
```

3. **Register Module:**

```c
void MyMod_Init()
{
	CF_ModuleWorldManager.GetInstance().RegisterModule(MyMod_Module);
}
```

### Common Patterns

#### Settings Management

```c
class MyMod_Settings : CF_ConfigBase
{
	int m_Value = 100;
	string m_Message = "Hello";
	
	override void LoadDefaults()
	{
		m_Value = 100;
		m_Message = "Hello";
	}
	
	override void LoadFile()
	{
		if (CF_File.Exists("MyMod/settings.json"))
		{
			// Load from JSON
		}
	}
}
```

#### Object Manager

CF provides object management utilities:

```c
// Get object manager
CF_ObjectManager objMgr = CF_ObjectManager.GetInstance();

// Track objects
objMgr.RegisterObject(myObject);
objMgr.UnregisterObject(myObject);
```

## When to Use CF vs DayZ Native

**Use CF when:**
- ✅ You're using Community Framework (we always load CF, Dabs, and your mod together)
- ✅ You need string-based RPC registration (easier to manage than enums)
- ✅ You want integrated module system with events
- ✅ You need notification system
- ✅ You need ModStorage for data persistence
- ✅ You want better logging with levels and formatting

**Use DayZ Native when:**
- ✅ Not using CF (rare, as CF is standard)
- ✅ Working with vanilla DayZ systems that require native APIs
- ✅ Need enum-based RPC (though CF string-based is preferred)

**Note:** We always load CF, Dabs, and your mod together, so CF features are the standard approach.

## Best Practices

### 1. Use CF Module System

Organize your mod using CF modules rather than global singletons:

```c
// ✅ Good - Using CF module
class MyMod_Module : CF_ModuleWorld
{
	override void OnInit()
	{
		super.OnInit();
		EnableUpdate();
	}
}

// ❌ Avoid - Global singleton
MyMod_Manager g_MyModManager = new MyMod_Manager();
```

### 2. Use CF Logging

Use CF logging instead of Print statements:

```c
// ✅ Good
static CF_Log s_Log = new CF_Log("MyMod");
s_Log.Info("Message %1", param);

// ❌ Avoid
Print("Message");
```

### 3. Use CF RPC System

Use CF's RPC system for better organization:

```c
// ✅ Good - String-based, easier to manage
CF_GetRPCManager().RegisterServerRPC("MyMod", "RPC_Test", RPC_Test, SingleplayerExecutionType.Server);

// ❌ Avoid - Enum-based, harder to extend
if (!g_Game)
	return;
g_Game.RPC(null, MYMOD_RPC_TEST, data, false);
```

### 4. Use CF Notifications

Use CF notification system for in-game messages:

```c
// ✅ Good - CF notifications
NotificationSystem.Create(title, text, icon, color, duration, identity);

// ❌ Avoid - Custom notification systems
// (unless you have specific requirements)
```

### 5. Leverage CF Utilities

Use CF utilities instead of reimplementing:

- File operations → `CF_File`, `CF_Directory`
- Type conversion → `CF_Convert`
- XML parsing → `CF_XML`
- Events → `CF_EventHandler`
- Data persistence → `CF_ModStorage`

## Advanced Features

### Expression VM

CF includes an Expression VM for evaluating expressions:

```c
CF_ExpressionVM vm = new CF_ExpressionVM();
vm.SetVariable("x", 10);
vm.SetVariable("y", 20);
int result = vm.Evaluate("x + y");
```

### NetworkedVariables

CF provides NetworkedVariables for synchronizing variables over the network:

```c
class MyMod_Item : ItemBase
{
	ref CF_NetworkedVariables m_NetworkVariables = new CF_NetworkedVariables(this);
	int m_SomeIntVariable;
	float m_SomeFloatVariable;
	
	void MyMod_Item()
	{
		// Register variables to synchronize
		m_NetworkVariables.Register("m_SomeIntVariable");
		m_NetworkVariables.Register("m_SomeFloatVariable");
	}
	
	void OnVariablesSynchronized()
	{
		// Called after variables are synchronized
		Print("Variables synced: " + m_SomeIntVariable + ", " + m_SomeFloatVariable);
	}
	
	void SetSynchDirty()
	{
		// Write variables to RPC
		ScriptRPC ctx = new ScriptRPC();
		m_NetworkVariables.Write(ctx);
		ctx.Send(null, RPC_ID, true, null);
	}
	
	void OnRPC(PlayerIdentity sender, Object target, int rpc_type, ParamsReadContext ctx)
	{
		if (rpc_type != RPC_ID)
			return;
		
		// Read synchronized variables
		m_NetworkVariables.Read(ctx);
		OnVariablesSynchronized();
	}
}
```

**Note:** Variables within variables can be registered using dot notation (e.g., `"m_DataHolder.m_SomeVariable"`), with a maximum depth of 3.

### Localiser

CF provides `CF_Localiser` for constructing multi-part localized strings:

```c
// Create localiser with string table key
CF_Localiser localiser = new CF_Localiser("console_log_in"); // "Press %1 to log in."

// Set parameters (index 0-9, or -1 for main text)
localiser[0] = "STR_NOVEMBER"; // "Nov"
// Or use Add() method
localiser.Add("STR_NOVEMBER");

// Format and use
Print(localiser.Format()); // "Press Nov to log in."

// Can also use plain strings
CF_Localiser localiser2 = new CF_Localiser("Press %1 to log in.");
localiser2[0] = "Nov";
// Or non-string values
localiser2[0] = 5; // "Press 5 to log in."
```

**Note:** `CF_Localiser` is different from `StringLocaliser` (used in notifications). `CF_Localiser` supports multi-part strings with indexed parameters.

### InputBindings

CF provides input binding system for custom input handling (client-side only):

```c
class MyMod_InputHandler
{
	autoptr CF_InputBindings m_CF_Bindings = new CF_InputBindings(this);
	
	void MyMod_InputHandler()
	{
		// Bind input by name
		m_CF_Bindings.Bind("PrintMessage", "UAUIBack", false);
		
		// Or by input ID
		m_CF_Bindings.Bind("PrintMessage", UAUIBack, false);
	}
	
	void PrintMessage(UAInput input)
	{
		Print("Hello, World!");
	}
}
```

### Cryptography

CF provides cryptography functions:

```c
// SHA-256 hashing
CF_FileStream input = new CF_FileStream("$profile:test.txt", FileMode.READ);
CF_TextReader reader = new CF_TextReader(input);
CF_Base16Stream output = new CF_Base16Stream();

CF_SHA256.Process(reader, output);
string hash = output.Encode(); // "2263D8DD95CCFE1AD45D732C6EAAF59B3345E6647331605CB15AAE52002DFF75"

reader.Close();
```

### Doubly Linked Nodes

CF provides doubly linked node structures for efficient data management.

### Mod Storage

CF includes ModStorage system for persisting custom data on items and entities across server restarts.

**See:** [How to Use CF ModStorage](CF/How-To-CF-ModStorage.md) for complete guide.

**Quick Example:**
```c
modded class ItemBase
{
	int m_MyMod_Value;
	
	override void CF_OnStoreSave(CF_ModStorageMap storage)
	{
		super.CF_OnStoreSave(storage);
		auto ctx = storage["MyMod"];
		if (ctx)
			ctx.Write(m_MyMod_Value);
	}
	
	override bool CF_OnStoreLoad(CF_ModStorageMap storage)
	{
		if (!super.CF_OnStoreLoad(storage))
			return false;
		auto ctx = storage["MyMod"];
		if (ctx)
			ctx.Read(m_MyMod_Value);
		return true;
	}
}
```

**Setup:** Requires `storageVersion > 0` in config.cpp `CfgMods` entry.

## Configuration

CF can be configured through defines:

```cpp
// config.cpp
defines[] = {
	"CF_MODULE_CONFIG",
	"CF_EXPRESSION",
	"CF_TRACE_STACK_NAME_ASSUMPTION_FIX",
	"CF_GHOSTICONS",
	"CF_MODSTORAGE",
	"CF_SURFACES",
	"CF_MODULES",
	"CF_REF_FIX",
	"CF_BUGFIX_REF",
	"CF_BUGFIX_XML",
	"CF_VIRTUALSTORAGE_FIX",
	"CF_DOUBLYLINKEDNODES",
	"CF_ONUPDATE_RATE_LIMIT",
	"CF_LOG_TIMESTAMP"
};
```

## CF Guides

Complete guides for using CF features:

- **[How to Use CF Logging](CF/How-To-CF-Logger.md)** - CF logging system and context tracing
- **[How to Use CF RPC](CF/How-To-CF-RPC.md)** - CF RPC system with string-based registration
- **[How to Use CF Notifications](CF/How-To-CF-Notifications.md)** - CF notification system
- **[How to Use CF ModStorage](CF/How-To-CF-ModStorage.md)** - CF ModStorage for data persistence

## Related Documentation

- [Basic Mod Structure](../How-To/Basic-Mod-Structure.md)
- [How to Use RPC](../How-To/How-To-RPC.md) - General RPC guide (includes DayZ native)
- [Module System](../How-To/Module-System.md) - Module system guide (includes CF modules)

## Quick Reference

### Module Registration

```c
void MyMod_Init()
{
	CF_ModuleWorldManager.GetInstance().RegisterModule(MyMod_WorldModule);
	CF_ModuleGameManager.GetInstance().RegisterModule(MyMod_GameModule);
}
```

### RPC Registration

```c
CF_GetRPCManager().RegisterServerRPC("MyMod", "RPC_Name", RPC_Handler, SingleplayerExecutionType.Server);
CF_GetRPCManager().RegisterClientRPC("MyMod", "RPC_Name", RPC_Handler, SingleplayerExecutionType.Client);
```

### Logging

```c
static CF_Log s_Log = new CF_Log("MyMod");
s_Log.Info("Message %1", param);
```

### Notifications

```c
NotificationSystem.Create(title, text, icon, ARGB(a, r, g, b), duration, identity);
```

### ModStorage

```c
override void CF_OnStoreSave(CF_ModStorageMap storage)
override bool CF_OnStoreLoad(CF_ModStorageMap storage)
```

### NetworkedVariables

```c
ref CF_NetworkedVariables m_NetworkVariables = new CF_NetworkedVariables(this);
m_NetworkVariables.Register("m_VariableName");
m_NetworkVariables.Write(ctx);  // In RPC
m_NetworkVariables.Read(ctx);  // In RPC handler
```

### Localiser

```c
CF_Localiser localiser = new CF_Localiser("string_key");
localiser[0] = "param1";
string formatted = localiser.Format();
```

### LifecycleEvents

```c
CF_LifecycleEvents.OnGameCreate.AddSubscriber(ScriptCaller.Create(this.OnGameCreate));
CF_LifecycleEvents.OnMissionCreate.AddSubscriber(ScriptCaller.Create(this.OnMissionCreate));
```

### Type Converters

```c
auto converter = CF_TypeConverter.Get(MyClass);
converter.Read(instance, "m_VariableName");
float value = converter.GetFloat();
```

### File Operations

```c
CF_File.Exists(path);
CF_File.Read(path);
CF_File.Write(path, content);
CF_Directory.GetFiles(pattern, files);
CF_Path.GetFileName(path);
```

## Notes

- **CF replaces the game creation function**, so it initializes early
- **We always load CF, Dabs, and your mod together** - CF is the standard framework
- CF provides many utilities that can replace custom implementations
- The RPC system uses string-based registration for flexibility
- CF's module system helps organize complex mods
- All module events are disabled by default - enable them with `Enable*()` methods

---

**Location:** `Mods/CF/`  
**Author:** Jacob_Mango, Arkensor
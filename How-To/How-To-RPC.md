# How to Use RPC

Practical guide to using Remote Procedure Calls (RPC) in your DayZ mod.

## Quick Start

RPCs let you call functions across the network (client ↔ server).

## Method 1: Using CF RPC System (Recommended)

If using Community Framework, use CF's string-based RPC system. CF RPC is the standard approach when using CF.

**See:** [How to Use CF RPC](../Frameworks/Community-Framework/CF/How-To-CF-RPC.md) for the complete CF RPC guide.

**Quick Overview:**
- String-based registration (easier than enums)
- Use `CF_GetRPCManager().RegisterServerRPC()` and `RegisterClientRPC()`
- Handler signature: `Param4<CallType, ParamsReadContext, PlayerIdentity, Object>`
- Send with `CF_GetRPCManager().SendRPC()`

## Method 3: DayZ Expansion RPC System

If using DayZ Expansion, use Expansion's RPC system:

```c
class MyExpansionModule : ExpansionModule
{
	override void EnableRPC()
	{
		super.EnableRPC();
		
		Expansion_EnableRPCManager();
		Expansion_RegisterServerRPC("RPC_MyServerRPC");
		Expansion_RegisterClientRPC("RPC_MyClientRPC");
	}
	
	// Server RPC handler (Expansion naming convention)
	void RPC_MyServerRPC(PlayerIdentity sender, Object target, ParamsReadContext ctx)
	{
		// Read and process
		string data;
		ctx.Read(data);
		// ... process ...
	}
}

// Sending Expansion RPCs
void SendExpansionRPC(PlayerIdentity identity, int value)
{
	auto rpc = Expansion_CreateRPC("RPC_MyClientRPC");
	rpc.Write(value);
	rpc.Expansion_Send(true, identity);  // true = guaranteed, identity = target player
}
```

## Method 2: Native DayZ RPC System

Without CF, use DayZ's native enum-based RPC system:

**⚠️ REQUIRED FILES:**
- `5_Mission/missionGameplay.c` - **MUST** exist to register client-side RPC handlers
- `5_Mission/MissionServer.c` - **MUST** exist for server-side RPC sending logic

**Key Points:**
- Use **enum values** (start at 10000+) for RPC IDs
- Register handlers in `missionGameplay.c` constructor
- Send RPCs using `g_Game.RPC()` with enum values
- **DO NOT** use `GetRPCManager().SendRPC()` - that's for CF RPC only

### Step 1: Create RPC Enum

```c
// MyMod_RPCs.c
enum EMyModRPCs
{
	MYMOD_RPC_START = 10000,  // Start at 10000 to avoid vanilla conflicts
	MYMOD_RPC_TEST,
	MYMOD_RPC_SEND_DATA,
	MYMOD_RPC_END
}
```

**Important:** Always start your RPC enum values at **10000 or higher** to avoid conflicts with vanilla RPCs.

### Step 2: Register RPC Handler

**⚠️ IMPORTANT: Required Files for Native RPC**

For native DayZ RPC to work, you **MUST** have these files in your `5_Mission/` directory:

- **`missionGameplay.c`** - Registers client-side RPC handlers (receives RPCs from server)
- **`MissionServer.c`** - Handles server-side logic and sends RPCs to clients

These files are **required** because:
- RPC registration happens in the mission layer (`5_Mission`)
- `missionGameplay.c` extends `MissionBase` and handles client-side RPC reception
- `MissionServer.c` extends `MissionServer` and handles server-side RPC sending
- Without these files, RPCs will not be registered or processed

**Example Structure:**
```
5_Mission/
├── missionGameplay.c    # REQUIRED: Client-side RPC handler registration
└── MissionServer.c      # REQUIRED: Server-side RPC sending logic
```

**Option A: Register in Module (Alternative)**

```c
class MyMod_Module : MissionModule
{
	override void OnInit()
	{
		super.OnInit();
		
		// Register RPC handler
		GetRPCManager().AddRPC(ClassName(), "RPC_MyModTest", RPC_MyModTest, true);
	}
	
	// Handler function
	void RPC_MyModTest(PlayerIdentity sender, Object target, ParamsReadContext ctx)
	{
		// Read parameters
		string message;
		if (!ctx.Read(message))
			return;
		
		// Process RPC
		Print("[MyMod] Received: " + message);
	}
}
```

**Option B: Register in missionGameplay.c (Recommended for Native RPC)**

```c
// 5_Mission/missionGameplay.c
modded class MissionGameplay extends MissionBase
{
	void MissionGameplay()
	{
		// Register client-side RPC handler
		GetRPCManager().AddRPC("MyMod", "RPC_MyModTest", this, SingleplayerExecutionType.Client);
	}
	
	// Handler function - receives RPCs from server
	void RPC_MyModTest(CallType type, ParamsReadContext ctx, PlayerIdentity sender, Object target)
	{
		// Only process on client
		if (type != CallType.Client)
			return;
		
		// Read parameters
		string message;
		if (!ctx.Read(message))
			return;
		
		// Process RPC
		Print("[MyMod] Received: " + message);
	}
}
```

### Step 3: Send RPCs

**⚠️ IMPORTANT: Use `g_Game.RPC()` for Native RPC**

Native DayZ RPC **MUST** use `g_Game.RPC()` with enum values. Do **NOT** use `GetRPCManager().SendRPC()` - that is for CF RPC only.

**Server → Client (in MissionServer.c or server-side code):**

```c
// 5_Mission/MissionServer.c (or any server-side code)
modded class MissionServer
{
	override void InvokeOnConnect(PlayerBase player, PlayerIdentity identity)
	{
		super.InvokeOnConnect(player, identity);
		
		// Send RPC to client when they connect
		if (g_Game && g_Game.IsServer() && identity)
		{
			string data = "Welcome!";
			auto param = new Param1<string>(data);
			g_Game.RPC(null, MYMOD_RPC_SEND_DATA, param, true, identity);
		}
	}
}
```

**Client → Server:**

```c
// Any client-side code
void SendToServer(string message)
{
	if (!g_Game)
		return;
	
	auto data = new Param1<string>(message);
	g_Game.RPC(null, MYMOD_RPC_TEST, data, false);
}
```

**Server → Client:**

```c
// Any server-side code (MissionServer.c, modules, etc.)
void SendToClient(PlayerIdentity identity, string data)
{
	if (!g_Game || !g_Game.IsServer())
		return;
	
	auto param = new Param1<string>(data);
	g_Game.RPC(null, MYMOD_RPC_SEND_DATA, param, true, identity);
}
```

**g_Game.RPC() Parameters:**
- `null` - Target object (usually null for player RPCs)
- `MYMOD_RPC_TEST` - RPC enum value
- `data` - Param class with data
- `true/false` - Guaranteed delivery (true = guaranteed, false = best effort)
- `identity` - Target player identity (for server→client, omit for broadcast)

## Complete Example: Client → Server → Client

```c
// Client requests data from server
void RequestPlayerData()
{
	CF_GetRPCManager().SendRPC("MyMod", "RPC_RequestData", null, false);
}

// Server receives request and responds
void RPC_RequestData(Param4<CallType, ParamsReadContext, PlayerIdentity, Object> data)
{
	PlayerIdentity sender = data.param3;
	
	// Only handle on server
	if (data.param1 != CallType.Server)
		return;
	
	// Get data
	PlayerData playerData = GetPlayerData(sender);
	
	// Send response back to client
	CF_GetRPCManager().SendRPC("MyMod", "RPC_ReceiveData", 
		new Param1<PlayerData>(playerData), true, sender);
}

// Client receives response
void RPC_ReceiveData(Param4<CallType, ParamsReadContext, PlayerIdentity, Object> data)
{
	ParamsReadContext ctx = data.param2;
	
	// Only handle on client
	if (data.param1 != CallType.Client)
		return;
	
	// Read data
	PlayerData playerData;
	if (!ctx.Read(playerData))
		return;
	
	// Process data
	UpdateUI(playerData);
}
```

## Sending Multiple Parameters

### Using Param Classes

```c
// Two parameters
void SendTwoParams(string name, int value)
{
	auto data = new Param2<string, int>(name, value);
	CF_GetRPCManager().SendRPC("MyMod", "RPC_Test", data, false);
}

// Handler
void RPC_Test(Param4<CallType, ParamsReadContext, PlayerIdentity, Object> data)
{
	ParamsReadContext ctx = data.param2;
	
	string name;
	int value;
	
	// Read in order
	if (!ctx.Read(name))
		return;
	if (!ctx.Read(value))
		return;
	
	// Use parameters
	Print("[MyMod] Name: " + name + ", Value: " + value);
}
```

### Using More Parameters

```c
// Three parameters
Param3<bool, float, string> data = new Param3<bool, float, string>(true, 3.14, "Test");
CF_GetRPCManager().SendRPC("MyMod", "RPC_Test", data, false);

// Four parameters
Param4<int, string, bool, float> data = new Param4<int, string, bool, float>(1, "two", false, 2.5);
CF_GetRPCManager().SendRPC("MyMod", "RPC_Test", data, false);
```

### Custom Param Classes

For complex data structures, create custom Param classes:

```c
class MyMod_DataParam : Param
{
	int m_Value;
	string m_Message;
	bool m_Flag;
	
	override bool OnSerialize(ParamsWriteContext ctx)
	{
		if (!ctx.Write(m_Value))
			return false;
		if (!ctx.Write(m_Message))
			return false;
		if (!ctx.Write(m_Flag))
			return false;
		return true;
	}
	
	override bool OnDeserialize(ParamsReadContext ctx)
	{
		if (!ctx.Read(m_Value))
			return false;
		if (!ctx.Read(m_Message))
			return false;
		if (!ctx.Read(m_Flag))
			return false;
		return true;
	}
}

// Usage
void SendComplexData(MyMod_DataParam data)
{
	CF_GetRPCManager().SendRPC("MyMod", "RPC_ComplexData", data, false);
}
```

## Best Practices

### 1. Always Validate Parameters

```c
void RPC_Test(Param4<CallType, ParamsReadContext, PlayerIdentity, Object> data)
{
	ParamsReadContext ctx = data.param2;
	
	string message;
	if (!ctx.Read(message))
	{
		Print("[MyMod] Invalid RPC: missing parameter");
		return;  // Always return on error
	}
	
	// Validate value
	if (message == "")
	{
		Print("[MyMod] Invalid RPC: empty message");
		return;
	}
	
	// Process valid RPC
	ProcessMessage(message);
}
```

### 2. Check Server/Client Side

```c
void RPC_Test(Param4<CallType, ParamsReadContext, PlayerIdentity, Object> data)
{
	// Only process on server
	if (data.param1 != CallType.Server)
		return;
	
	// Or check directly
	if (!g_Game || !g_Game.IsDedicatedServer())
		return;
	
	// Process server-side logic
}
```

### 3. Use Guaranteed RPCs Appropriately

```c
// ✅ Use guaranteed (true) for important updates
void SendImportantUpdate(PlayerIdentity identity, ImportantData data)
{
	CF_GetRPCManager().SendRPC("MyMod", "RPC_ImportantUpdate", 
		new Param1<ImportantData>(data), true, identity);  // true = guaranteed
}

// ✅ Use non-guaranteed (false) for frequent updates
void SendPositionUpdate(PlayerIdentity identity, vector position)
{
	CF_GetRPCManager().SendRPC("MyMod", "RPC_PositionUpdate", 
		new Param1<vector>(position), false, identity);  // false = not guaranteed
}
```

### 4. Handle Singleplayer Mode

RPCs behave differently in singleplayer/offline mode. Use `SingleplayerExecutionType`:

```c
// Server only - executes on server side in singleplayer
CF_GetRPCManager().RegisterServerRPC("MyMod", "RPC_ServerOnly", RPC_ServerOnly, SingleplayerExecutionType.Server);

// Client only - executes on client side in singleplayer
CF_GetRPCManager().RegisterClientRPC("MyMod", "RPC_ClientOnly", RPC_ClientOnly, SingleplayerExecutionType.Client);

// Both - executes on both sides in singleplayer (most common for gameplay)
CF_GetRPCManager().RegisterServerRPC("MyMod", "RPC_Both", RPC_Both, SingleplayerExecutionType.Both);
```

**SingleplayerExecutionType options:**
- **Server**: Only executes on server side
- **Client**: Only executes on client side
- **Both**: Executes on both sides (most common for gameplay RPCs)

### 5. Prefix Your RPC Names

```c
// ✅ Good - Prefixed RPC names
CF_GetRPCManager().RegisterServerRPC("MyMod", "RPC_TestServer", ...);
CF_GetRPCManager().RegisterServerRPC("MyMod", "RPC_SendData", ...);

// ❌ Avoid - Generic RPC names
CF_GetRPCManager().RegisterServerRPC("MyMod", "RPC_Test", ...);  // Too generic
```

## Common Patterns

### Request-Response Pattern

```c
// Client requests data
void RequestServerInfo()
{
	CF_GetRPCManager().SendRPC("MyMod", "RPC_RequestInfo", null, false);
}

// Server responds
void RPC_RequestInfo(Param4<CallType, ParamsReadContext, PlayerIdentity, Object> data)
{
	if (data.param1 != CallType.Server)
		return;
	
	PlayerIdentity sender = data.param3;
	ServerInfo info = GetServerInfo();
	
	// Send response
	CF_GetRPCManager().SendRPC("MyMod", "RPC_ReceiveInfo", 
		new Param1<ServerInfo>(info), true, sender);
}
```

### Broadcast Pattern

```c
// Server broadcasts to all clients
void BroadcastMessage(string message)
{
	CF_GetRPCManager().SendRPC("MyMod", "RPC_Broadcast", 
		new Param1<string>(message), true);  // NULL identity = all clients
}
```

## Debugging RPCs

### Logging RPC Calls

```c
#ifdef MYMOD_DEBUG
	void RPC_Test(Param4<CallType, ParamsReadContext, PlayerIdentity, Object> data)
	{
		Print("[MyMod] [DEBUG] RPC_Test called from: " + data.param3.GetPlainId());
		
		// ... handle RPC ...
	}
#endif
```

### Tracing RPC Flow

```c
// On send
void SendTestRPC(int value)
{
	Print("[MyMod] Sending RPC_Test with value: " + value);
	CF_GetRPCManager().SendRPC("MyMod", "RPC_Test", new Param1<int>(value), false);
}

// On receive
void RPC_Test(Param4<CallType, ParamsReadContext, PlayerIdentity, Object> data)
{
	int value;
	data.param2.Read(value);
	Print("[MyMod] Received RPC_Test with value: " + value);
}
```

## Summary

**Quick Steps:**
1. **Register RPC** in module's `OnInit()`
2. **Create handler** function with proper signature
3. **Send RPC** when needed
4. **Validate** parameters in handler
5. **Check** server/client side

**Best Practices:**
- Use CF RPC system if available (easier)
- Always validate parameters
- Check server/client side
- Use guaranteed RPCs for important updates
- Prefix RPC names
- Debug with logging

---

**Related Guides:**
- [How to Create a Mod](How-To-Create-Mod.md)
- [Module System](Module-System.md)
- [Tips: g_Game vs GetGame()](../Tips/Tips-g_Game-GetGame.md)
- [Community Framework](../Frameworks/Community-Framework/Community-Framework.md) - CF framework overview
- [How to Use CF RPC](../Frameworks/Community-Framework/CF/How-To-CF-RPC.md) - Complete CF RPC guide
- [DayZ Expansion](../Frameworks/DayZ-Expansion.md) - Expansion documentation
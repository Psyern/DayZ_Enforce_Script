# How to Use CF RPC System

Guide to using Community Framework's RPC system in your DayZ mod.

## Overview

CF's RPC system uses string-based registration instead of enum values, making it easier to manage and extend. It's the recommended approach when using Community Framework.

**⚠️ Framework Standard:** We always load CF, Dabs, and your mod together. CF RPC is the standard for mods using CF.

## Quick Start

RPCs let you call functions across the network (client ↔ server).

### Step 1: Register RPCs

```c
class MyMod_Module : CF_ModuleWorld
{
	override void OnInit()
	{
		super.OnInit();
		
		// Register Server RPC (client → server)
		CF_GetRPCManager().RegisterServerRPC("MyMod", "RPC_TestServer", RPC_TestServer, SingleplayerExecutionType.Server);
		
		// Register Client RPC (server → client)
		CF_GetRPCManager().RegisterClientRPC("MyMod", "RPC_TestClient", RPC_TestClient, SingleplayerExecutionType.Client);
	}
	
	// Server RPC handler (called from client)
	void RPC_TestServer(Param4<CallType, ParamsReadContext, PlayerIdentity, Object> data)
	{
		ParamsReadContext ctx = data.param2;
		PlayerIdentity sender = data.param3;
		
		// Read parameters
		string message;
		if (!ctx.Read(message))
			return;
		
		// Process on server
		Print("[MyMod] Server received: " + message + " from " + sender.GetPlainId());
	}
	
	// Client RPC handler (called from server)
	void RPC_TestClient(Param4<CallType, ParamsReadContext, PlayerIdentity, Object> data)
	{
		ParamsReadContext ctx = data.param2;
		
		// Read parameters
		int value;
		if (!ctx.Read(value))
			return;
		
		// Process on client
		Print("[MyMod] Client received value: " + value);
	}
}
```

### Step 2: Send RPCs

```c
// Client → Server
void SendToServer(string message)
{
	CF_GetRPCManager().SendRPC("MyMod", "RPC_TestServer", new Param1<string>(message), false);
}

// Server → Client
void SendToClient(PlayerIdentity identity, int value)
{
	CF_GetRPCManager().SendRPC("MyMod", "RPC_TestClient", new Param1<int>(value), true, identity);
}

// Server → All Clients (broadcast)
void BroadcastToAll(int value)
{
	CF_GetRPCManager().SendRPC("MyMod", "RPC_TestClient", new Param1<int>(value), true);
}
```

## RPC Registration

### Server RPCs (Client → Server)

```c
CF_GetRPCManager().RegisterServerRPC("MyMod", "RPC_Name", RPC_Handler, SingleplayerExecutionType.Server);
```

**Parameters:**
- `"MyMod"` - Module name (usually your mod name)
- `"RPC_Name"` - RPC identifier (string, not enum)
- `RPC_Handler` - Handler method name
- `SingleplayerExecutionType.Server` - Execute on server

### Client RPCs (Server → Client)

```c
CF_GetRPCManager().RegisterClientRPC("MyMod", "RPC_Name", RPC_Handler, SingleplayerExecutionType.Client);
```

**Parameters:**
- `"MyMod"` - Module name
- `"RPC_Name"` - RPC identifier
- `RPC_Handler` - Handler method name
- `SingleplayerExecutionType.Client` - Execute on client

### Both (Server and Client)

```c
CF_GetRPCManager().RegisterServerRPC("MyMod", "RPC_Both", RPC_Both, SingleplayerExecutionType.Both);
```

## RPC Handler Signature

CF RPC handlers use this signature:

```c
void RPC_HandlerName(Param4<CallType, ParamsReadContext, PlayerIdentity, Object> data)
{
	CallType type = data.param1;
	ParamsReadContext ctx = data.param2;
	PlayerIdentity sender = data.param3;
	Object target = data.param4;
	
	// Read parameters from ctx
	string message;
	if (!ctx.Read(message))
		return;
	
	// Process RPC
}
```

**Important:** Always read parameters from `ParamsReadContext` using `ctx.Read()`.

## Sending RPCs

### Client to Server

```c
// Send from client to server
CF_GetRPCManager().SendRPC("MyMod", "RPC_TestServer", new Param1<string>("Hello"), false);
```

**Parameters:**
- `"MyMod"` - Module name
- `"RPC_TestServer"` - RPC identifier
- `new Param1<string>("Hello")` - Parameters (Param1, Param2, etc.)
- `false` - Not reliable (use `true` for important messages)

### Server to Client

```c
// Send to specific client
CF_GetRPCManager().SendRPC("MyMod", "RPC_TestClient", new Param1<int>(42), true, identity);

// Broadcast to all clients
CF_GetRPCManager().SendRPC("MyMod", "RPC_TestClient", new Param1<int>(42), true);
```

**Parameters:**
- `"MyMod"` - Module name
- `"RPC_TestClient"` - RPC identifier
- `new Param1<int>(42)` - Parameters
- `true` - Reliable (guaranteed delivery)
- `identity` - Target player identity (omit for broadcast)

## Parameter Classes

CF RPCs use `Param` classes for data:

```c
// Single parameter
new Param1<string>("Hello")
new Param1<int>(42)

// Multiple parameters
new Param2<string, int>("Name", 100)
new Param3<string, int, float>("Name", 100, 3.14)
new Param4<string, int, float, bool>("Name", 100, 3.14, true)
// Up to Param6
```

### Reading Parameters

```c
void RPC_Example(Param4<CallType, ParamsReadContext, PlayerIdentity, Object> data)
{
	ParamsReadContext ctx = data.param2;
	
	// Read single parameter
	string message;
	if (!ctx.Read(message))
		return;
	
	// Read multiple parameters
	string name;
	int value;
	float fvalue;
	if (!ctx.Read(name))
		return;
	if (!ctx.Read(value))
		return;
	if (!ctx.Read(fvalue))
		return;
}
```

**⚠️ CRITICAL:** When reading `Param` classes (like `Param6`), read them as a **single object**, not individual fields:

```c
// ✅ Correct - Read Param6 as single object
void RPC_ComplexData(Param4<CallType, ParamsReadContext, PlayerIdentity, Object> data)
{
	ParamsReadContext ctx = data.param2;
	
	Param6<string, string, int, int, int, int> notificationData;
	if (!ctx.Read(notificationData))
		return;
	
	// Now access fields
	string title = notificationData.param1;
	string message = notificationData.param2;
	int r = notificationData.param3;
	int g = notificationData.param4;
	int b = notificationData.param5;
	int a = notificationData.param6;
}

// ❌ Wrong - Don't read Param6 fields individually
void RPC_ComplexData(Param4<CallType, ParamsReadContext, PlayerIdentity, Object> data)
{
	ParamsReadContext ctx = data.param2;
	
	// This will fail!
	string title;
	if (!ctx.Read(title))  // Wrong!
		return;
}
```

## Complete Examples

### Example 1: Simple Message

```c
class MyMod_Module : CF_ModuleWorld
{
	override void OnInit()
	{
		super.OnInit();
		CF_GetRPCManager().RegisterServerRPC("MyMod", "RPC_SendMessage", RPC_SendMessage, SingleplayerExecutionType.Server);
	}
	
	void RPC_SendMessage(Param4<CallType, ParamsReadContext, PlayerIdentity, Object> data)
	{
		ParamsReadContext ctx = data.param2;
		PlayerIdentity sender = data.param3;
		
		string message;
		if (!ctx.Read(message))
			return;
		
		Print("[MyMod] Message from " + sender.GetPlainId() + ": " + message);
	}
}

// Client sends:
CF_GetRPCManager().SendRPC("MyMod", "RPC_SendMessage", new Param1<string>("Hello Server!"), false);
```

### Example 2: Complex Data

```c
class MyMod_Module : CF_ModuleWorld
{
	override void OnInit()
	{
		super.OnInit();
		CF_GetRPCManager().RegisterClientRPC("MyMod", "RPC_UpdateUI", RPC_UpdateUI, SingleplayerExecutionType.Client);
	}
	
	void RPC_UpdateUI(Param4<CallType, ParamsReadContext, PlayerIdentity, Object> data)
	{
		ParamsReadContext ctx = data.param2;
		
		Param3<string, int, float> uiData;
		if (!ctx.Read(uiData))
			return;
		
		string title = uiData.param1;
		int value = uiData.param2;
		float progress = uiData.param3;
		
		// Update UI
	}
}

// Server sends:
CF_GetRPCManager().SendRPC("MyMod", "RPC_UpdateUI", 
	new Param3<string, int, float>("Status", 100, 0.75), true, identity);
```

### Example 3: Request/Response Pattern

```c
class MyMod_Module : CF_ModuleWorld
{
	override void OnInit()
	{
		super.OnInit();
		CF_GetRPCManager().RegisterServerRPC("MyMod", "RPC_RequestData", RPC_RequestData, SingleplayerExecutionType.Server);
		CF_GetRPCManager().RegisterClientRPC("MyMod", "RPC_ReceiveData", RPC_ReceiveData, SingleplayerExecutionType.Client);
	}
	
	// Client requests data
	void RPC_RequestData(Param4<CallType, ParamsReadContext, PlayerIdentity, Object> data)
	{
		PlayerIdentity sender = data.param3;
		
		// Process request and send response
		CF_GetRPCManager().SendRPC("MyMod", "RPC_ReceiveData", 
			new Param2<string, int>("Response", 42), true, sender);
	}
	
	// Client receives data
	void RPC_ReceiveData(Param4<CallType, ParamsReadContext, PlayerIdentity, Object> data)
	{
		ParamsReadContext ctx = data.param2;
		
		Param2<string, int> response;
		if (!ctx.Read(response))
			return;
		
		Print("[MyMod] Received: " + response.param1 + " = " + response.param2);
	}
}
```

## Best Practices

### 1. Use Descriptive RPC Names

```c
// ✅ Good - Descriptive
CF_GetRPCManager().RegisterServerRPC("MyMod", "RPC_RequestPlayerInfo", ...);
CF_GetRPCManager().RegisterServerRPC("MyMod", "RPC_SendPlayerData", ...);

// ❌ Avoid - Generic
CF_GetRPCManager().RegisterServerRPC("MyMod", "RPC_Test", ...);
```

### 2. Always Check Read Results

```c
// ✅ Good - Check read result
string message;
if (!ctx.Read(message))
	return;  // Failed to read

// ❌ Avoid - No check
string message;
ctx.Read(message);  // May fail silently
```

### 3. Use Reliable for Important Messages

```c
// ✅ Good - Reliable for important data
CF_GetRPCManager().SendRPC("MyMod", "RPC_ImportantUpdate", data, true, identity);

// ✅ Good - Unreliable for frequent updates
CF_GetRPCManager().SendRPC("MyMod", "RPC_PositionUpdate", data, false, identity);
```

### 4. Register in Module OnInit

```c
class MyMod_Module : CF_ModuleWorld
{
	override void OnInit()
	{
		super.OnInit();
		
		// Register all RPCs here
		CF_GetRPCManager().RegisterServerRPC("MyMod", "RPC_TestServer", ...);
		CF_GetRPCManager().RegisterClientRPC("MyMod", "RPC_TestClient", ...);
	}
}
```

## When to Use CF RPC vs DayZ Native RPC

**Use CF RPC when:**
- ✅ Using Community Framework
- ✅ Want string-based registration (easier to manage)
- ✅ Need better organization and module integration
- ✅ Working with CF modules

**Use DayZ Native RPC when:**
- ✅ Not using CF
- ✅ Need enum-based registration
- ✅ Working with vanilla DayZ systems

**Note:** We always load CF, Dabs, and your mod together, so CF RPC is the standard approach.

## Summary

**CF RPC System:**
- String-based registration (easier than enums)
- Integrated with CF modules
- Use `CF_GetRPCManager()` for registration and sending
- Handler signature: `Param4<CallType, ParamsReadContext, PlayerIdentity, Object>`
- Read `Param` classes as single objects

**Best Practices:**
- Use descriptive RPC names
- Always check read results
- Use reliable for important messages
- Register in module `OnInit()`

---

**Related Guides:**
- [How to Use RPC](../../How-To/How-To-RPC.md) - General RPC guide (includes DayZ native)
- [Module System](../../How-To/Module-System.md) - CF module system
- [Community Framework](../Community-Framework.md) - CF framework overview

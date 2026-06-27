# How to Use Enums

Practical guide to creating and using enums in your DayZ mod.

## What is an Enum?

An enum (enumeration) groups related constants together. Instead of creating many separate constants, use an enum to group them.

## Basic Enum Syntax

### Simple Enum

```c
enum MyModLogLevel
{
	Debug = 0,
	Info,
	Warning,
	Error
}
```

**Notes:**
- First value can have explicit value: `Debug = 0`
- Following values auto-increment: `Info = 1`, `Warning = 2`, `Error = 3`
- Use **CamelCase** or **PASCAL_CASE** consistently

### Enum with Explicit Values

```c
enum MyModRPCs
{
	MYMOD_RPC_START = 10000,  // Explicit value
	MYMOD_RPC_TEST,           // 10001
	MYMOD_RPC_SEND_DATA,      // 10002
	MYMOD_RPC_RECEIVE_DATA    // 10003
}
```

## Enum Naming Conventions

### CamelCase (Recommended)

```c
// ✅ Good - CamelCase is easier to read
enum EMyModStatus
{
	Idle,
	Running,
	Stopped,
	Error
}

enum MyModLogLevel
{
	Debug,
	Info,
	Warning,
	Error
}
```

### PASCAL_CASE (Also Valid)

```c
// ✅ Also acceptable - PASCAL_CASE for clarity
enum EMyModStatus
{
	IDLE,
	RUNNING,
	STOPPED,
	ERROR
}

enum MyModRPCs
{
	MYMOD_RPC_START = 10000,
	MYMOD_RPC_TEST,
	MYMOD_RPC_SEND_DATA
}
```

**Choose one style and be consistent within your mod.**

## Using Enums

### Variable Declaration

```c
// Declare variable with enum type
MyModLogLevel logLevel = MyModLogLevel.Info;
EMyModRPCs rpcType = EMyModRPCs.MYMOD_RPC_TEST;

// Use in functions
void SetLogLevel(MyModLogLevel level)
{
	logLevel = level;
}
```

### Comparing Enum Values

```c
void CheckLogLevel(MyModLogLevel level)
{
	if (level == MyModLogLevel.Debug)
	{
		// Debug level
	}
	else if (level == MyModLogLevel.Error)
	{
		// Error level
	}
}
```

### Switch Statement with Enums

```c
string GetLogLevelString(MyModLogLevel level)
{
	switch (level)
	{
		case MyModLogLevel.Debug:
			return "DEBUG";
		
		case MyModLogLevel.Info:
			return "INFO";
		
		case MyModLogLevel.Warning:
			return "WARNING";
		
		case MyModLogLevel.Error:
			return "ERROR";
		
		default:
			return "UNKNOWN";
	}
	
	return "";
}
```

## Real-World Examples

### Example 1: Log Level Enum

```c
// enum.c
enum MyModLogLevel
{
	Debug = 0,
	Info,
	Warning,
	Error
}

// Usage
class MyMod_Logger
{
	MyModLogLevel m_LogLevel = MyModLogLevel.Info;
	
	void Log(string message, MyModLogLevel level)
	{
		if (level < m_LogLevel)
			return;  // Skip if below current log level
		
		string levelStr = GetLevelString(level);
		Print("[MyMod] [" + levelStr + "] " + message);
	}
	
	string GetLevelString(MyModLogLevel level)
	{
		switch (level)
		{
			case MyModLogLevel.Debug:
				return "DEBUG";
			case MyModLogLevel.Info:
				return "INFO";
			case MyModLogLevel.Warning:
				return "WARNING";
			case MyModLogLevel.Error:
				return "ERROR";
		}
		return "";
	}
}
```

### Example 2: RPC Enum

```c
// MyMod_RPCs.c
enum EMyModRPCs
{
	MYMOD_RPC_START = 10000,  // Always start at 10000+
	MYMOD_RPC_TEST,
	MYMOD_RPC_SEND_DATA,
	MYMOD_RPC_RECEIVE_DATA,
	MYMOD_RPC_UPDATE_UI,
	MYMOD_RPC_END
}

// Usage
void SendTestRPC()
{
	if (!g_Game)
		return;
	
	auto data = new Param1<string>("test");
	g_Game.RPC(null, MYMOD_RPC_TEST, data, false);
}
```

### Example 3: Status Enum

```c
enum EMyModConnectionStatus
{
	Disconnected,
	Connecting,
	Connected,
	Error
}

// Usage
class MyMod_Connection
{
	EMyModConnectionStatus m_Status = EMyModConnectionStatus.Disconnected;
	
	void SetStatus(EMyModConnectionStatus status)
	{
		m_Status = status;
		
		if (status == EMyModConnectionStatus.Connected)
		{
			OnConnected();
		}
		else if (status == EMyModConnectionStatus.Error)
		{
			OnError();
		}
	}
}
```

## Prefixing Enums

### ✅ CORRECT - Prefix Your Enums

```c
// ✅ Good - Prefixed enum
enum EMyModRPCs
{
	MYMOD_RPC_START = 10000,
	MYMOD_RPC_TEST
}

enum MyModLogLevel
{
	Debug,
	Info
}

// ❌ Avoid - Generic names
enum ERPCs {}  // Conflicts with vanilla!
enum LogLevel {}  // Conflicts with other mods!
```

## Best Practices

### 1. Always Prefix Enums

```c
// ✅ Good - Prefixed
enum EMyModRPCs {}
enum MyModLogLevel {}
enum EMyModStatus {}

// ❌ Avoid - Generic
enum ERPCs {}  // Conflicts!
enum LogLevel {}  // Conflicts!
```

### 2. Use Appropriate Starting Values

```c
// ✅ Good - RPC enums start at 10000+
enum EMyModRPCs
{
	MYMOD_RPC_START = 10000,  // Avoids vanilla conflicts
	MYMOD_RPC_TEST
}

// ❌ Avoid - Starting at 0 for RPCs
enum EMyModRPCs
{
	MYMOD_RPC_TEST = 0  // May conflict with vanilla!
}
```

### 3. Be Consistent with Naming

```c
// ✅ Good - Consistent style
enum MyModLogLevel
{
	Debug,    // CamelCase
	Info,
	Warning,
	Error
}

// ✅ Also good - Consistent PASCAL_CASE
enum MyModLogLevel
{
	DEBUG,    // PASCAL_CASE
	INFO,
	WARNING,
	ERROR
}

// ❌ Avoid - Mixed styles
enum MyModLogLevel
{
	Debug,    // CamelCase
	INFO,     // PASCAL_CASE - inconsistent!
	Warning   // CamelCase again
}
```

### 4. Document Enum Purpose

```c
// Enum for logging levels
// Debug = Most detailed, never enable in production
// Info = General information
// Warning = Abnormal but non-fatal
// Error = Execution stopped
enum MyModLogLevel
{
	Debug = 0,
	Info,
	Warning,
	Error
}

// Enum for RPC identifiers
// Start at 10000 to avoid conflicts with vanilla RPCs
enum EMyModRPCs
{
	MYMOD_RPC_START = 10000,
	MYMOD_RPC_TEST,
	MYMOD_RPC_SEND_DATA
}
```

## Common Patterns

### Pattern 1: Status Enum

```c
enum EMyModItemStatus
{
	New,
	Used,
	Damaged,
	Ruined
}

void CheckItemStatus(ItemBase item)
{
	EMyModItemStatus status = GetItemStatus(item);
	
	switch (status)
	{
		case EMyModItemStatus.New:
			Print("Item is new");
			break;
		
		case EMyModItemStatus.Used:
			Print("Item is used");
			break;
		
		case EMyModItemStatus.Damaged:
			Print("Item is damaged");
			break;
		
		case EMyModItemStatus.Ruined:
			Print("Item is ruined");
			break;
	}
}
```

### Pattern 2: Option Enum (Bit Flags)

```c
enum EMyModOptions
{
	None = 0,
	Option1 = 1,
	Option2 = 2,
	Option3 = 4,
	All = 7  // 1 + 2 + 4
}

void SetOptions(int options)
{
	if (options & EMyModOptions.Option1)
	{
		// Option 1 enabled
	}
	
	if (options & EMyModOptions.Option2)
	{
		// Option 2 enabled
	}
	
	// Enable multiple options
	int myOptions = EMyModOptions.Option1 | EMyModOptions.Option2;
}
```

## Summary

**Creating Enums:**
- Use `enum EnumName {}` syntax
- First value can be explicit: `First = 0`
- Following values auto-increment
- Use CamelCase or PASCAL_CASE consistently

**Using Enums:**
- Declare: `MyEnum value = MyEnum.ValueName;`
- Compare: `if (value == MyEnum.ValueName)`
- Use in switch statements

**Best Practices:**
- **Always prefix** your enum names
- Start RPC enums at 10000+
- Be consistent with naming style
- Document enum purpose
- Use enums instead of many constants

**Quick Reference:**
- Enum declaration: `enum EMyModRPCs { MYMOD_RPC_START = 10000, MYMOD_RPC_TEST }`
- Variable: `EMyModRPCs rpc = EMyModRPCs.MYMOD_RPC_TEST;`
- Comparison: `if (rpc == EMyModRPCs.MYMOD_RPC_TEST)`

---

**Related Guides:**
- [How to Use RPC](How-To-RPC.md)
- [How to Create a Logger](How-To-Logger.md)
- [Tips: Naming Conventions and Prefixes](Tips-Prefixes-Naming.md)
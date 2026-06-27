# How to Use CF Logging

Guide to using Community Framework's logging system in your mod.

## Overview

CF provides a built-in logging system that's better than using `Print()` statements. It includes timestamps, log levels, and formatted output.

## Basic Usage

### Creating a Logger Instance

```c
class MyMod_Module : CF_ModuleWorld
{
	static CF_Log s_Log = new CF_Log("MyMod");
	
	override void OnInit()
	{
		super.OnInit();
		
		// Use logger
		s_Log.Info("Module initialized");
	}
}
```

**Note:** Use `static` instance so the logger persists and can be used throughout your module.

### Log Levels

CF_Log provides these log levels (in order of severity):

```c
// Most detailed (never enable in production)
s_Log.Trace("Detailed trace information");

// Debugging information
s_Log.Debug("Debug information");

// General information
s_Log.Info("General information message");

// Warning messages
s_Log.Warn("Warning message");

// Error (produces stack trace)
s_Log.Error("Error occurred");

// Critical error (produces stack trace)
s_Log.Critical("Critical error!");
```

## Formatting Messages

### Using Parameters

CF_Log supports formatted messages with up to 9 parameters:

```c
// Single parameter
s_Log.Info("Player %1 joined", playerName);

// Multiple parameters
s_Log.Info("Player %1 (ID: %2) joined server %3", playerName, playerId, serverName);

// With numbers
s_Log.Info("Health: %1, Stamina: %2", health.ToString(), stamina.ToString());

// Up to 9 parameters
s_Log.Info("Value1: %1, Value2: %2, Value3: %3, Value4: %4", 
	value1.ToString(), value2.ToString(), value3.ToString(), value4.ToString());
```

### Example Output

```c
s_Log.Info("Player %1 joined with ID %2", "John", "76561198000000000");
// Output: [12:34:56] [INFO]	Player John joined with ID 76561198000000000
```

## Controlling Log Level

### Using Defines in config.cpp

Control which log messages are displayed using defines:

```cpp
class CfgMods
{
	class MyMod
	{
		defines[] = {
			"CF_DEBUG_ENABLED",  // Enables DEBUG and above (Info, Warn, Error, Critical)
			// "CF_INFO_ENABLED",  // Enables INFO and above (default)
			// "CF_TRACE_ENABLED"  // Enables all levels including TRACE
		};
	};
};
```

### Log Level Hierarchy

**Default log level is WARNING** if no defines are set.

From most verbose to least:
1. **TRACE** - Most detailed (requires `CF_TRACE_ENABLED`)
2. **DEBUG** - Debug info (requires `CF_DEBUG_ENABLED`)
3. **INFO** - General info (requires `CF_INFO_ENABLED` or `CF_DEBUG_ENABLED`)
4. **WARNING** - Warnings (default, always enabled)
5. **ERROR** - Errors (always enabled, produces stack trace)
6. **CRITICAL** - Critical errors (always enabled, produces stack trace)

### Checking Log Level

```c
// Check if a log level is enabled
if (CF_Log.IsLogging(CF_LogLevel.DEBUG))
{
	s_Log.Debug("This debug message will be logged");
}
```

## Complete Example

### Module with Logger

```c
class MyMod_Module : CF_ModuleWorld
{
	static CF_Log s_Log = new CF_Log("MyMod");
	
	override void OnInit()
	{
		super.OnInit();
		
		s_Log.Info("Module initializing...");
		
		// Initialize systems
		InitializeSystems();
		
		s_Log.Info("Module initialized successfully");
	}
	
	void InitializeSystems()
	{
		s_Log.Debug("Initializing systems...");
		
		// Your initialization code
		if (SomeCondition())
		{
			s_Log.Warn("Some condition not met, using defaults");
		}
	}
	
	void OnError()
	{
		s_Log.Error("An error occurred in MyMod_Module");
		// Error() automatically produces stack trace
	}
	
	void OnCriticalError()
	{
		s_Log.Critical("Critical error - mod cannot continue!");
		// Critical() automatically produces stack trace
	}
}
```

## Best Practices

### 1. Use Appropriate Log Levels

```c
// ✅ Good - Appropriate levels
s_Log.Info("Player joined");           // Normal operation
s_Log.Warn("Low health detected");     // Abnormal but expected
s_Log.Error("Failed to save file");    // Error condition
s_Log.Critical("Cannot continue");     // Fatal error

// ❌ Avoid - Wrong levels
s_Log.Error("Player joined");          // This is normal, use Info
s_Log.Critical("Low health");          // Not critical, use Warning
```

### 2. Include Context in Messages

```c
// ✅ Good - Includes context
s_Log.Error("Failed to load config for player %1: %2", playerName, errorMessage);

// ❌ Avoid - No context
s_Log.Error("Failed");
```

### 3. Use Debug for Development

```c
// ✅ Good - Debug for development
#ifdef CF_DEBUG_ENABLED
	s_Log.Debug("Processing item: %1", itemName);
	s_Log.Debug("Current state: %1", currentState);
#endif

// Or check conditionally
if (CF_Log.IsLogging(CF_LogLevel.DEBUG))
{
	s_Log.Debug("Debug information");
}
```

### 4. Don't Log in Tight Loops

```c
// ❌ Bad - Logs every frame
override void OnUpdate(float delta_time)
{
	s_Log.Debug("Update: " + delta_time);  // Too much logging!
}

// ✅ Good - Log occasionally
float m_LastLogTime = 0;
override void OnUpdate(float delta_time)
{
	m_LastLogTime += delta_time;
	if (m_LastLogTime >= 60.0)  // Log every 60 seconds
	{
		s_Log.Info("Still running after %1 seconds", m_LastLogTime.ToString());
		m_LastLogTime = 0;
	}
}
```

### 5. Format Messages Properly

```c
// ✅ Good - Formatted message
s_Log.Info("Player %1 (%2) joined server", playerName, playerId);

// ❌ Avoid - String concatenation
s_Log.Info("Player " + playerName + " (" + playerId + ") joined server");
```

## Static Logger Pattern

Create a static logger that can be used anywhere:

```c
class MyMod_Logger
{
	static CF_Log s_Log = new CF_Log("MyMod");
	
	static void Trace(string message, string param1 = "", string param2 = "", string param3 = "")
	{
		s_Log.Trace(message, param1, param2, param3);
	}
	
	static void Debug(string message, string param1 = "", string param2 = "", string param3 = "")
	{
		s_Log.Debug(message, param1, param2, param3);
	}
	
	static void Info(string message, string param1 = "", string param2 = "", string param3 = "")
	{
		s_Log.Info(message, param1, param2, param3);
	}
	
	static void Warn(string message, string param1 = "", string param2 = "", string param3 = "")
	{
		s_Log.Warn(message, param1, param2, param3);
	}
	
	static void Error(string message, string param1 = "", string param2 = "", string param3 = "")
	{
		s_Log.Error(message, param1, param2, param3);
	}
	
	static void Critical(string message, string param1 = "", string param2 = "", string param3 = "")
	{
		s_Log.Critical(message, param1, param2, param3);
	}
}
```

### Usage

```c
// Use anywhere without needing module instance
MyMod_Logger.Info("Module initialized");
MyMod_Logger.Error("Something went wrong: %1", errorMessage);
```

## Conditional Logging

### Using Preprocessor Directives

```c
#ifdef CF_DEBUG_ENABLED
	s_Log.Debug("Debug information");
#endif

#ifdef CF_TRACE_ENABLED
	s_Log.Trace("Detailed trace");
#endif
```

### Runtime Level Checking

```c
// Check if level is enabled at runtime
if (CF_Log.IsLogging(CF_LogLevel.DEBUG))
{
	s_Log.Debug("This will only log if DEBUG is enabled");
}
```

## Error and Critical Logs

### Error Logs

`Error()` and `Critical()` automatically produce stack traces:

```c
void HandleError()
{
	s_Log.Error("Failed to process data");
	// Output includes:
	// [12:34:56] [ERROR]	Failed to process data
	// 	Stack trace:
	// 		at MyMod_Module::HandleError
	// 		at MyMod_Module::OnUpdate
	// 		...
}
```

### Critical Logs

Use `Critical()` for fatal errors:

```c
void FatalError()
{
	s_Log.Critical("Mod cannot continue - shutting down");
	// Output includes stack trace
	// Use this for unrecoverable errors
}
```

## Context Tracing (CF_Trace)

CF provides context tracing for debugging crashes and performance issues. **⚠️ WARNING:** Context tracing heavily impacts performance. Always place trace calls behind preprocessor defines and disable them in production.

### Basic Usage

The trace method outputs when a scope is entered and exited:

```c
void MyFunction()
{
	auto trace = CF_Trace_0();  // 0 = no parameters
	// Function body
}  // Trace outputs when function exits
```

### With Parameters

The number in `CF_Trace_X` matches the number of parameters:

```c
void MyFunction(int value)
{
	auto trace = CF_Trace_1().Add(value);  // 1 parameter
}

void MyFunction(string name, int value)
{
	auto trace = CF_Trace_2().Add(name).Add(value);  // 2 parameters
}
```

### With Class Methods

For class methods, pass `this` as the first parameter:

```c
class MyClass
{
	void MyMethod(string name, int value)
	{
		auto trace = CF_Trace_2(this).Add(name).Add(value);
	}
	
	static void MyStaticMethod(string name)
	{
		auto trace = CF_Trace_1("MyClass").Add(name);  // Class name as string
	}
}
```

### Conditional Tracing

Disable tracing programmatically by passing a boolean:

```c
static bool TRACE_MYFUNCTION = false;  // Disable tracing

void MyFunction(float deltaTime)
{
	auto trace = CF_Trace_1(TRACE_MYFUNCTION, this).Add(deltaTime);
}
```

### Complete Example

```c
static bool MYMOD_LOG_SOMECLASS_METHODONE = true;

void SomeGlobalFunction()
{
	auto trace = CF_Trace_0();
	
	SomeClass someClass = new SomeClass();
	someClass.MethodOne(0.5);
}

class SomeClass
{
	ref CF_Trace m_SomeClassTrace = CF_Trace_Instance(this);
	
	void SomeClass()
	{
		auto trace = CF_Trace_0(this);
	}
	
	void MethodOne(float deltaTime)
	{
		auto trace = CF_Trace_1(MYMOD_LOG_SOMECLASS_METHODONE, this).Add(deltaTime);
		
		MethodTwo("Test", GetGame().GetPlayer().GetParent());
	}
	
	void MethodTwo(string message, IEntity parent)
	{
		auto trace = CF_Trace_2(this).Add(message).Add(parent);
	}
}
```

### Output Example

```
SCRIPT : [TRACE] +::SomeGlobalFunction ()
SCRIPT : [TRACE]  +SomeClass<6ec92cd0> ()
SCRIPT : [TRACE]   +SomeClass<6ec92cd0>::SomeClass ()
SCRIPT : [TRACE]   -SomeClass<6ec92cd0>::SomeClass Time: 0.0098ms
SCRIPT : [TRACE]   +SomeClass<6ec92cd0>::MethodOne (0.5)
SCRIPT : [TRACE]    +SomeClass<6ec92cd0>::MethodTwo ("Test", NULL)
SCRIPT : [TRACE]    -SomeClass<6ec92cd0>::MethodTwo Time: 0.0019ms
SCRIPT : [TRACE]   -SomeClass<6ec92cd0>::MethodOne Time: 0.3468ms
SCRIPT : [TRACE]  -::SomeGlobalFunction Time: 1.6316ms
```

### Best Practices

**⚠️ CRITICAL:** Context tracing heavily impacts performance. Always:

1. **Use preprocessor defines:**
```c
#ifdef MYMOD_DEBUG_TRACE
	auto trace = CF_Trace_0(this);
#endif
```

2. **Disable in production:**
```c
static bool TRACE_ENABLED = false;  // Disable in production
auto trace = CF_Trace_0(TRACE_ENABLED, this);
```

3. **Only use for debugging crashes:**
- Don't leave trace calls in production code
- Use for finding crash locations
- Disable after debugging

## Summary

**CF Logging:**
- Use `CF_Log` with static instance
- Control level with defines in config.cpp
- Use appropriate log levels
- Format messages with parameters
- Error/Critical automatically produce stack traces

**CF Context Tracing:**
- Use `CF_Trace_X()` for debugging
- Heavily impacts performance - disable in production
- Use preprocessor defines to control tracing
- Outputs entry/exit times for scopes

**Quick Reference:**
- Create: `static CF_Log s_Log = new CF_Log("MyMod");`
- Log: `s_Log.Info("Message %1", param);`
- Levels: `Trace()`, `Debug()`, `Info()`, `Warn()`, `Error()`, `Critical()`
- Context Trace: `auto trace = CF_Trace_0();` (disable in production!)

**Best Practices:**
- Use appropriate log levels
- Include context in messages
- Don't log in tight loops
- Use debug for development
- Format messages properly
- **Never leave CF_Trace calls in production code**

---

**Related Guides:**
- [How to Create a Logger](../../How-To/How-To-Logger.md) - Creating custom loggers
- [How to Create a Mod](../../How-To/How-To-Create-Mod.md)
- [Community Framework](../Community-Framework.md) - CF framework overview
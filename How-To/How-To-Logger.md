# How to Create a Logger

Guide to creating logging systems in your DayZ mod.

## Why Use a Logger?

Instead of using `Print()` statements everywhere:
- **Consistent formatting** - Timestamps and log levels
- **Control verbosity** - Enable/disable debug logs
- **Better debugging** - Stack traces on errors
- **Professional output** - Formatted log messages

## Method 1: Using CF Logging (Recommended)

If you're using Community Framework, use CF's built-in logging. CF logging is the standard approach when using CF.

**See:** [How to Use CF Logging](../Frameworks/Community-Framework/CF/How-To-CF-Logger.md) for the complete CF logging guide including context tracing.

**Quick Overview:**
- Use `static CF_Log s_Log = new CF_Log("MyMod");`
- Log levels: `Trace()`, `Debug()`, `Info()`, `Warn()`, `Error()`, `Critical()`
- Format messages: `s_Log.Info("Message %1", param);`
- Control level with defines: `CF_DEBUG_ENABLED`, `CF_INFO_ENABLED`, `CF_TRACE_ENABLED`
- Includes context tracing (`CF_Trace`) for debugging (disable in production!)

## Method 2: Simple Custom Logger

If not using CF, create a simple custom logger:

### Basic Custom Logger

```c
class MyMod_Logger
{
	static const int LOG_LEVEL_NONE = 0;
	static const int LOG_LEVEL_ERROR = 1;
	static const int LOG_LEVEL_WARNING = 2;
	static const int LOG_LEVEL_INFO = 3;
	static const int LOG_LEVEL_DEBUG = 4;
	
	static int s_LogLevel = LOG_LEVEL_INFO;
	string m_ModName;
	
	void MyMod_Logger(string modName)
	{
		m_ModName = modName;
	}
	
	void Log(int level, string message)
	{
		if (level > s_LogLevel)
			return;
		
		string levelName = "";
		if (level == LOG_LEVEL_ERROR)
			levelName = "[ERROR]";
		else if (level == LOG_LEVEL_WARNING)
			levelName = "[WARNING]";
		else if (level == LOG_LEVEL_INFO)
			levelName = "[INFO]";
		else if (level == LOG_LEVEL_DEBUG)
			levelName = "[DEBUG]";
		
		string timeStr = GetTimeString();
		PrintFormat("%1 [%2] %3 %4", timeStr, m_ModName, levelName, message);
	}
	
	void Error(string message) { Log(LOG_LEVEL_ERROR, message); }
	void Warning(string message) { Log(LOG_LEVEL_WARNING, message); }
	void Info(string message) { Log(LOG_LEVEL_INFO, message); }
	void Debug(string message) { Log(LOG_LEVEL_DEBUG, message); }
	
	string GetTimeString()
	{
		int hours, minutes, seconds;
		GetHourMinuteSecond(hours, minutes, seconds);
		return hours.ToStringLen(2) + ":" + minutes.ToStringLen(2) + ":" + seconds.ToStringLen(2);
	}
}
```

### Usage

```c
class MyMod_Module : MissionModule
{
	static MyMod_Logger s_Logger = new MyMod_Logger("MyMod");
	
	override void OnInit()
	{
		super.OnInit();
		s_Logger.Info("Module initialized");
		s_Logger.Debug("Debug information");
		s_Logger.Warning("Warning message");
		s_Logger.Error("Error occurred");
	}
}
```

## Method 3: Using Global Static Logger

Create a global logger instance:

```c
// MyMod_Logger.c
class MyMod_Logger : Managed
{
	static MyMod_Logger s_Instance;
	string m_ModName;
	
	static MyMod_Logger GetInstance()
	{
		if (!s_Instance)
		{
			s_Instance = new MyMod_Logger("MyMod");
		}
		return s_Instance;
	}
	
	void MyMod_Logger(string modName)
	{
		m_ModName = modName;
	}
	
	void Log(string level, string message)
	{
		string timeStr = GetTimeString();
		PrintFormat("%1 [%2] [%3] %4", timeStr, m_ModName, level, message);
	}
	
	void Error(string message) { Log("ERROR", message); }
	void Warning(string message) { Log("WARNING", message); }
	void Info(string message) { Log("INFO", message); }
	void Debug(string message) { Log("DEBUG", message); }
	
	string GetTimeString()
	{
		int hours, minutes, seconds;
		GetHourMinuteSecond(hours, minutes, seconds);
		return hours.ToStringLen(2) + ":" + minutes.ToStringLen(2) + ":" + seconds.ToStringLen(2);
	}
}
```

### Usage

```c
// Use anywhere in your mod
MyMod_Logger.GetInstance().Info("Module initialized");
MyMod_Logger.GetInstance().Error("Something went wrong");
```

## Conditional Logging

Use preprocessor defines to control logging:

```c
class MyMod_Logger
{
#ifdef MYMOD_DEBUG
	static bool s_EnableDebug = true;
#else
	static bool s_EnableDebug = false;
#endif
	
	void Debug(string message)
	{
#ifdef MYMOD_DEBUG
		if (s_EnableDebug)
		{
			Print("[MyMod] [DEBUG] " + message);
		}
#endif
	}
}
```

Then in config.cpp:

```cpp
class CfgMods
{
	class MyMod
	{
		defines[] = {
			"MYMOD_DEBUG"  // Enables debug logging
		};
	};
};
```

## Best Practices

### 1. Use Appropriate Log Levels

```c
// ✅ Good - Appropriate levels
s_Log.Info("Player joined");           // Normal operation
s_Log.Warning("Low health");           // Abnormal but expected
s_Log.Error("Failed to save");         // Error condition
s_Log.Critical("Cannot continue");     // Fatal error

// ❌ Avoid - Wrong levels
s_Log.Error("Player joined");          // This is normal, use Info
s_Log.Critical("Low health");          // Not critical, use Warning
```

### 2. Include Context

```c
// ✅ Good - Includes context
s_Log.Error("Failed to load config for player %1: %2", playerName, errorMessage);

// ❌ Avoid - No context
s_Log.Error("Failed");
```

### 3. Don't Log in Loops

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
	if (!g_Game)
		return;
	
	if (g_Game.GetTime() - m_LastLogTime > 60)
	{
		s_Log.Debug("Still running");
		m_LastLogTime = g_Game.GetTime();
	}
}
```

### 4. Format Messages Properly

```c
// ✅ Good - Formatted message
s_Log.Info("Player %1 (%2) joined server", playerName, playerId);

// ❌ Avoid - String concatenation
s_Log.Info("Player " + playerName + " (" + playerId + ") joined server");
```

## Summary

**For CF Users:**
- Use `CF_Log` with static instance
- Control level with defines
- Use appropriate log levels

**Without CF:**
- Create simple custom logger
- Use preprocessor defines for debug
- Format messages with timestamps

**Best Practices:**
- Use appropriate log levels
- Include context in messages
- Don't log in tight loops
- Format messages properly

---

**Related Guides:**
- [Using CF Logging](../Frameworks/Community-Framework/CF/How-To-CF-Logger.md) - CF-specific logging guide
- [How to Create a Mod](How-To-Create-Mod.md) - Create your first mod
- [Tips: Memory Management](Tips-Memory-Management.md) - Understanding EnScript
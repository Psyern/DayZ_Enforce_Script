# Dabs Framework

Dabs Framework is a modding framework for DayZ Standalone that provides utilities and systems for mod development.

## Overview

**Location:** `Mods/Dabs Framework/`

Dabs Framework provides:
- MVC (Model-View-Controller) architecture
- Event management system
- Logging system
- Settings management
- GUI utilities
- File system utilities
- And more

## Key Features

### MVC Architecture

Dabs Framework implements MVC (Model-View-Controller) for organized GUI development:

```c
// Model
class MyMod_Model : DabsFramework_MVC_Model
{
	int m_Value;
	
	void SetValue(int value)
	{
		m_Value = value;
		NotifyPropertyChanged("Value");
	}
}

// View
class MyMod_View : DabsFramework_MVC_View
{
	override void UpdateView(MyMod_Model model)
	{
		// Update UI based on model
	}
}

// Controller
class MyMod_Controller : DabsFramework_MVC_Controller
{
	MyMod_Model m_Model;
	MyMod_View m_View;
	
	override void Init()
	{
		m_Model = new MyMod_Model();
		m_View = new MyMod_View();
		SetModel(m_Model);
		SetView(m_View);
	}
}
```

### Event Manager

Dabs Framework includes an event management system:

```c
// Subscribe to event
DabsFramework_EventManager.Subscribe("MyEvent", OnMyEvent);

// Publish event
DabsFramework_EventManager.Publish("MyEvent", new Param1<string>("data"));

// Event handler
void OnMyEvent(Param params)
{
	Param1<string> data = Param1<string>.Cast(params);
	Print("Event: " + data.param1);
}
```

### Logging System

Dabs Framework provides a logging system:

```c
// Get logger
DabsFramework_LoggerBase logger = DabsFramework_LoggerManager.GetLogger("MyMod");

// Log messages
logger.LogInfo("Information message");
logger.LogWarning("Warning message");
logger.LogError("Error message");

// Log levels
enum DabsFramework_LogLevel
{
	DEBUG,
	INFO,
	WARNING,
	ERROR,
	CRITICAL
}
```

### Settings System

Dabs Framework includes a settings management system:

```c
class MyMod_Settings : DabsFramework_SettingsBase
{
	int m_Value = 100;
	string m_Message = "Hello";
	
	override void LoadDefaults()
	{
		m_Value = 100;
		m_Message = "Hello";
	}
	
	override void Load()
	{
		// Load from file
	}
	
	override void Save()
	{
		// Save to file
	}
}
```

### File System Utilities

Dabs Framework provides file and directory utilities:

```c
// File operations
if (DabsFramework_File.Exists("MyMod/data.json"))
{
	string content = DabsFramework_File.Read("MyMod/data.json");
	DabsFramework_File.Write("MyMod/output.json", content);
}

// Directory operations
if (DabsFramework_Directory.Exists("MyMod/Data"))
{
	array<string> files = DabsFramework_Directory.GetFiles("MyMod/Data");
}
```

### Widget Utilities

Dabs Framework includes GUI widget utilities:

```c
// Find widget
Widget widget = DabsFramework_FindWidget.Find("MyWidget");

// Widget operations
DabsFramework_WidgetAnimator.Animate(widget, "fadeIn", 1000);
```

## Structure

```
Dabs Framework/
└── Scripts/
    └── DabsFramework/
        └── Scripts/
            ├── 1_Core/
            │   └── DabsFramework/
            │       ├── File/              # File utilities
            │       ├── Managed/           # Managed types
            │       └── Symbols/           # Symbol utilities
            ├── 3_Game/
            │   └── DabsFramework/
            │       ├── !Core/             # Core utilities
            │       ├── EventManager/      # Event system
            │       ├── Logging/           # Logging system
            │       ├── MVC/               # MVC architecture
            │       └── Settings/          # Settings management
            ├── 4_World/
            │   └── DabsFramework/
            │       └── Classes/           # World classes
            └── config.cpp
```

## Key Classes

### Core Classes

- `DabsFramework_DayZGame` - Game utilities
- `DabsFramework_Debug` - Debug utilities
- `DabsFramework_PlayerIdentity` - Player identity utilities
- `DabsFramework_Color` - Color utilities
- `DabsFramework_Math` - Math utilities

### MVC Classes

- `DabsFramework_MVC_Model` - Base model class
- `DabsFramework_MVC_View` - Base view class
- `DabsFramework_MVC_Controller` - Base controller class
- `DabsFramework_MVC_ViewModel` - ViewModel pattern

### Utility Classes

- `DabsFramework_File` - File operations
- `DabsFramework_Directory` - Directory operations
- `DabsFramework_Path` - Path utilities
- `DabsFramework_FindWidget` - Widget finding utilities

## Using Dabs Framework

### Basic Setup

1. **Depend on Dabs Framework:**

```cpp
// config.cpp
class CfgPatches
{
	class MyMod_Scripts
	{
		requiredAddons[] = {"DZ_Data", "DabsFramework_Scripts"};
	};
};
```

2. **Create a Controller:**

```c
class MyMod_Controller : DabsFramework_MVC_Controller
{
	MyMod_Model m_Model;
	MyMod_View m_View;
	
	override void Init()
	{
		m_Model = new MyMod_Model();
		m_View = new MyMod_View();
		SetModel(m_Model);
		SetView(m_View);
	}
}
```

3. **Use Logging:**

```c
class MyMod_Module
{
	DabsFramework_LoggerBase m_Logger;
	
	void MyMod_Module()
	{
		m_Logger = DabsFramework_LoggerManager.GetLogger("MyMod");
	}
	
	void DoSomething()
	{
		m_Logger.LogInfo("Doing something");
	}
}
```

### Common Patterns

#### Settings Pattern

```c
class MyMod_Settings : DabsFramework_SettingsBase
{
	int m_IntValue = 100;
	string m_StringValue = "Default";
	bool m_BoolValue = true;
	
	override void LoadDefaults()
	{
		m_IntValue = 100;
		m_StringValue = "Default";
		m_BoolValue = true;
	}
	
	override void Load()
	{
		// Load from JSON or other format
	}
	
	override void Save()
	{
		// Save to JSON or other format
	}
}
```

#### Event Pattern

```c
// Publisher
class MyMod_Publisher
{
	void PublishEvent()
	{
		DabsFramework_EventManager.Publish("MyEvent", new Param1<string>("data"));
	}
}

// Subscriber
class MyMod_Subscriber
{
	void MyMod_Subscriber()
	{
		DabsFramework_EventManager.Subscribe("MyEvent", OnMyEvent);
	}
	
	void OnMyEvent(Param params)
	{
		// Handle event
	}
}
```

## MVC Architecture

### Model

Models hold data and notify views of changes:

```c
class MyMod_Model : DabsFramework_MVC_Model
{
	int m_Health = 100;
	
	void SetHealth(int health)
	{
		m_Health = health;
		NotifyPropertyChanged("Health");
	}
	
	int GetHealth()
	{
		return m_Health;
	}
}
```

### View

Views display data from models:

```c
class MyMod_View : DabsFramework_MVC_View
{
	override void UpdateView(MyMod_Model model)
	{
		// Update UI widgets based on model
		Widget healthWidget = GetWidget().FindAnyWidget("HealthLabel");
		healthWidget.SetText("" + model.GetHealth());
	}
}
```

### Controller

Controllers coordinate between models and views:

```c
class MyMod_Controller : DabsFramework_MVC_Controller
{
	override void Init()
	{
		MyMod_Model model = new MyMod_Model();
		MyMod_View view = new MyMod_View();
		SetModel(model);
		SetView(view);
	}
	
	void OnHealthChanged()
	{
		m_Model.SetHealth(50);
		// View automatically updates
	}
}
```

## Best Practices

### 1. Use MVC for GUI

Organize GUI code using MVC pattern:

```c
// ✅ Good - MVC structure
class MyMod_Controller : DabsFramework_MVC_Controller
{
	// Controller logic
}

// ❌ Avoid - Mixed concerns
class MyMod_GUI
{
	// Everything mixed together
}
```

### 2. Use Event System

Use events for decoupled communication:

```c
// ✅ Good - Event-driven
DabsFramework_EventManager.Publish("PlayerDied", data);

// ❌ Avoid - Direct calls
player.DeathHandler.OnDeath();
```

### 3. Use Logging System

Use Dabs Framework logging instead of Print:

```c
// ✅ Good
logger.LogInfo("Message");

// ❌ Avoid
Print("Message");
```

### 4. Leverage Utilities

Use framework utilities instead of reimplementing:

- File operations → `DabsFramework_File`
- Path operations → `DabsFramework_Path`
- Widget finding → `DabsFramework_FindWidget`

## Related Documentation

- [Basic Mod Structure](Basic-Mod-Structure.md)
- [Module System](Module-System.md)
- [Event System](Event-System.md)

## Notes

- Dabs Framework emphasizes MVC architecture for GUI
- The event system enables decoupled communication
- Settings system provides structured configuration management
- File utilities simplify file operations

---

**Location:** `Mods/Dabs Framework/`
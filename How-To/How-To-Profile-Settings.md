# How to Create Profile Settings

Guide to creating settings that are saved to the Profile folder and persist across sessions.

## Overview

Profile settings are saved as **JSON files** in the `$profile:` directory. The `$profile:` path prefix expands differently depending on whether code runs on the server or client:

- **Server-side:** `$profile:` expands to the folder specified by the `-profiles` parameter in your server start .bat file (e.g., `ServerProfile/`, `MyDayZServer\profiles\`, `MyDayZServer\instance\`)
- **Client-side:** `$profile:` expands to `%localappdata%\dayz` (typically `C:\Users\[Username]\AppData\Local\dayz` on Windows)

These settings are **NOT** saved in the `.profile` file itself, but as separate JSON configuration files managed by your mod. This is useful for client-side settings like UI preferences, log levels, and personal configurations that persist across sessions.

### Key Points

- **Server Location:** `$profile:ModName/` → `[ServerProfilesFolder]/ModName/` (where `[ServerProfilesFolder]` is set via `-profiles=` parameter in server start .bat)
  - Example: If `-profiles=ServerProfile`, then `$profile:MyMod\` → `ServerProfile/MyMod\`
  - Example: If `-profiles=MyDayZServer\profiles\`, then `$profile:MyMod\` → `MyDayZServer\profiles\MyMod\`
- **Client Location:** `$profile:ModName/` → `%localappdata%\dayz\ModName/`
  - Windows example: `C:\Users\[Username]\AppData\Local\dayz\MyMod\Config\Settings.json`
- **Format:** JSON files (not binary `.profile` file)
- **Method:** Use `JsonFileLoader` to save/load

**Reference:** [DayZ Server Start .bat Guide](https://dzconfig.com/wiki/server-start-bat) - See `-profiles` parameter for server configuration

## Method 1: Using JSON File Loader (Standard DayZ)

The standard way to create profile settings in DayZ is using `JsonFileLoader` to save and load JSON files:

### Define Path Constants

First, define paths for your settings files:

```c
// Constants.c
const string MyMod_ROOT_FOLDER = "$profile:MyMod\\";
const string MyMod_CONFIG_DIR = MyMod_ROOT_FOLDER + "Config\\";
const string MyMod_CONFIG_FILE = MyMod_CONFIG_DIR + "Settings.json";
```

The `$profile:` prefix expands to:
- **Server:** The folder specified by `-profiles` parameter in your server start .bat file
  - Example: If your .bat has `set PROFILES_FOLDER=ServerProfile` and `-profiles=%PROFILES_FOLDER%`, then `$profile:` → `ServerProfile/`
  - Example: If your .bat has `-profiles=MyDayZServer\profiles\`, then `$profile:` → `MyDayZServer\profiles\`
  - Example: If your .bat has `-profiles=MyDayZServer\instance\`, then `$profile:` → `MyDayZServer\instance\`
- **Client:** `%localappdata%\dayz` (typically `C:\Users\[Username]\AppData\Local\dayz` on Windows)

**Server Configuration:** See [DayZ Server Start .bat Guide](https://dzconfig.com/wiki/server-start-bat) for how to set the `-profiles` parameter

### Create Settings Class

```c
// MyMod_Settings.c
class MyMod_Settings
{
	// Settings with default values
	int m_LogLevel = 1;
	int m_RefreshRateInSeconds = 60;
	bool m_EnableFeature = true;
	float m_Brightness = 1.0;
	string m_PlayerName = "Player";
	
	// Make directory if it doesn't exist
	void MakeDirectoryIfNotExists()
	{
		if (!FileExist(MyMod_ROOT_FOLDER))
			MakeDirectory(MyMod_ROOT_FOLDER);
		
		if (!FileExist(MyMod_CONFIG_DIR))
			MakeDirectory(MyMod_CONFIG_DIR);
	}
	
	// Save settings to JSON file
	void Save()
	{
		MakeDirectoryIfNotExists();
		JsonFileLoader<MyMod_Settings>.JsonSaveFile(MyMod_CONFIG_FILE, this);
	}
	
	// Load settings from JSON file
	static ref MyMod_Settings Load()
	{
		MyMod_Settings settings = new MyMod_Settings();
		settings.MakeDirectoryIfNotExists();
		
		if (FileExist(MyMod_CONFIG_FILE))
		{
			// Load existing settings
			JsonFileLoader<MyMod_Settings>.JsonLoadFile(MyMod_CONFIG_FILE, settings);
			return settings;
		}
		
		// Create new settings file with defaults
		settings.Save();
		return settings;
	}
}
```

### Using JSON Settings

```c
class MyMod_Module : MissionModule
{
	ref MyMod_Settings m_Settings;
	
	override void OnInit()
	{
		super.OnInit();
		
		// Load settings (creates file if doesn't exist)
		m_Settings = MyMod_Settings.Load();
		
		Print("[MyMod] Log level: " + m_Settings.m_LogLevel);
		Print("[MyMod] Feature enabled: " + m_Settings.m_EnableFeature);
	}
	
	void SaveSettings()
	{
		// Save settings to JSON file
		m_Settings.Save();
	}
	
	void UpdateSetting()
	{
		// Update setting
		m_Settings.m_LogLevel = 2;
		
		// Save immediately
		m_Settings.Save();
	}
}
```

## Method 2: Using Dabs Framework ProfileSettings

If using Dabs Framework, you can use the ProfileSettings system (different from JSON files):

Dabs Framework provides a different system that stores settings in the actual `.profile` file, not as separate JSON files.

## Profile Settings with Arrays

### Using Arrays with JSON

```c
class MyMod_Settings
{
	array<string> m_FavoriteItems = {};
	array<int> m_Numbers = {};
	
	// Arrays are automatically saved/loaded with JSON
	// Default values are set in constructor
	
	void MyMod_Settings()
	{
		// Default values
		m_FavoriteItems.Insert("AKM");
		m_FavoriteItems.Insert("M4A1");
	}
	
	// Save and Load work the same way
	void Save()
	{
		MakeDirectoryIfNotExists();
		JsonFileLoader<MyMod_Settings>.JsonSaveFile(MyMod_CONFIG_FILE, this);
	}
	
	static ref MyMod_Settings Load()
	{
		MyMod_Settings settings = new MyMod_Settings();
		settings.MakeDirectoryIfNotExists();
		
		if (FileExist(MyMod_CONFIG_FILE))
		{
			JsonFileLoader<MyMod_Settings>.JsonLoadFile(MyMod_CONFIG_FILE, settings);
			return settings;
		}
		
		settings.Save();
		return settings;
	}
}
```

## Complete Example

### Constants.c

```c
// Define paths for your mod's profile directory
const string MyMod_ROOT_FOLDER = "$profile:MyMod\\";
const string MyMod_CONFIG_DIR = MyMod_ROOT_FOLDER + "Config\\";
const string MyMod_CONFIG_FILE = MyMod_CONFIG_DIR + "Settings.json";
```

### MyMod_Settings.c

```c
class MyMod_Settings
{
	// Client-side settings with default values
	bool m_ShowNotifications = true;
	int m_NotificationDuration = 5;
	float m_UIScale = 1.0;
	string m_Theme = "Dark";
	
	// Hotkeys
	string m_HotkeyOpenMenu = "M";
	string m_HotkeyQuickAction = "F";
	
	// Preferences
	bool m_AutoSaveEnabled = true;
	int m_AutoSaveInterval = 300;  // Seconds
	array<string> m_DisabledFeatures = {};
	
	// Create directory structure
	void MakeDirectoryIfNotExists()
	{
		if (!FileExist(MyMod_ROOT_FOLDER))
			MakeDirectory(MyMod_ROOT_FOLDER);
		
		if (!FileExist(MyMod_CONFIG_DIR))
			MakeDirectory(MyMod_CONFIG_DIR);
	}
	
	// Save settings to JSON file
	void Save()
	{
		MakeDirectoryIfNotExists();
		JsonFileLoader<MyMod_Settings>.JsonSaveFile(MyMod_CONFIG_FILE, this);
	}
	
	// Load settings from JSON file
	static ref MyMod_Settings Load()
	{
		MyMod_Settings settings = new MyMod_Settings();
		settings.MakeDirectoryIfNotExists();
		
		if (FileExist(MyMod_CONFIG_FILE))
		{
			JsonFileLoader<MyMod_Settings>.JsonLoadFile(MyMod_CONFIG_FILE, settings);
			
			// Apply settings when loaded
			settings.ApplyOptions();
			return settings;
		}
		
		// Create new settings file with defaults
		settings.Save();
		settings.ApplyOptions();
		return settings;
	}
	
	// Apply settings to game
	void ApplyOptions()
	{
		ApplyUIScale();
		ApplyTheme();
	}
	
	void ApplyUIScale()
	{
		// Apply UI scale to widgets
		// Your UI scaling logic here
	}
	
	void ApplyTheme()
	{
		// Apply theme to UI
		// Your theme logic here
	}
}
```

### MyMod_Module.c

```c
class MyMod_Module : MissionModule
{
	ref MyMod_Settings m_Settings;
	
	override void OnInit()
	{
		super.OnInit();
		
		// Load settings (creates file if doesn't exist)
		m_Settings = MyMod_Settings.Load();
		
		Print("[MyMod] Settings loaded from: " + MyMod_CONFIG_FILE);
		Print("[MyMod] Notifications: " + m_Settings.m_ShowNotifications);
		Print("[MyMod] UI Scale: " + m_Settings.m_UIScale);
	}
	
	// Called when player wants to change settings
	void UpdateSettings(bool showNotifications, int duration, float uiScale)
	{
		m_Settings.m_ShowNotifications = showNotifications;
		m_Settings.m_NotificationDuration = duration;
		m_Settings.m_UIScale = uiScale;
		
		// Save immediately to JSON file
		m_Settings.Save();
		
		// Apply changes
		m_Settings.ApplyOptions();
	}
	
	override void OnUnloadModule()
	{
		super.OnUnloadModule();
		
		// Save settings on unload
		if (m_Settings)
		{
			m_Settings.Save();
		}
	}
}
```

## Where Settings Are Saved

Settings are saved to JSON files, not in the `.profile` file itself:

### Server-Side Location

On the server, `$profile:` expands to the folder specified by the `-profiles` parameter:

**Example Server Start .bat:**
```batch
set PROFILES_FOLDER=ServerProfile
DayZServer_x64.exe -profiles=%PROFILES_FOLDER% ...
```

**Settings saved to:**
- **Path:** `ServerProfile/MyMod/Config/Settings.json` (relative to server directory)
- **Expanded from:** `$profile:MyMod\\Config\\Settings.json`

**Other examples:**
- If `-profiles=MyDayZServer\profiles\` → Settings saved to `MyDayZServer\profiles\MyMod\Config\Settings.json`
- If `-profiles=MyDayZServer\instance\` → Settings saved to `MyDayZServer\instance\MyMod\Config\Settings.json`

### Client-Side Location

On the client, `$profile:` expands to `%localappdata%\dayz`:

**Settings saved to:**
- **Path:** `%localappdata%\dayz\MyMod\Config\Settings.json`
- **Windows Example:** `C:\Users\[Username]\AppData\Local\dayz\MyMod\Config\Settings.json`
- **Expanded from:** `$profile:MyMod\\Config\\Settings.json`

### Summary

| Side | `$profile:` Expands To | Example |
|------|------------------------|---------|
| **Server** | Folder from `-profiles=` parameter | `ServerProfile/MyMod/Config/Settings.json` |
| **Client** | `%localappdata%\dayz` | `C:\Users\[User]\AppData\Local\dayz\MyMod\Config\Settings.json` |

**Important:** Always use `$profile:` prefix in your paths - DayZ will automatically expand it to the correct location based on whether code runs on server or client.

**Reference:** [Server Start .bat Configuration](https://dzconfig.com/wiki/server-start-bat)

## Best Practices

### 1. Use Namespaced File Paths

```c
// ✅ Good - Namespaced paths
const string MyMod_ROOT_FOLDER = "$profile:MyMod\\";
const string MyMod_CONFIG_FILE = MyMod_CONFIG_DIR + "Settings.json";

// ❌ Avoid - Generic paths
const string CONFIG_FILE = "$profile:Config.json";  // May conflict with other mods
```

### 2. Provide Default Values

```c
// ✅ Good - Default values
bool m_EnableFeature = true;
int m_Volume = 50;

// ❌ Avoid - No defaults
bool m_EnableFeature;  // May be undefined
```

### 3. Save After Changes

```c
// ✅ Good - Save after change
m_Settings.m_Volume = 75;
m_Settings.Save();

// ❌ Avoid - Don't save
m_Settings.m_Volume = 75;  // Lost on restart!
```

### 4. Always Create Directories First

```c
// ✅ Good - Create directories
void MakeDirectoryIfNotExists()
{
	if (!FileExist(MyMod_ROOT_FOLDER))
		MakeDirectory(MyMod_ROOT_FOLDER);
	
	if (!FileExist(MyMod_CONFIG_DIR))
		MakeDirectory(MyMod_CONFIG_DIR);
}

void Save()
{
	MakeDirectoryIfNotExists();  // Always create first
	JsonFileLoader<MyMod_Settings>.JsonSaveFile(MyMod_CONFIG_FILE, this);
}

// ❌ Avoid - No directory creation
void Save()
{
	JsonFileLoader<MyMod_Settings>.JsonSaveFile(MyMod_CONFIG_FILE, this);  // May fail!
}
```

### 5. Use Appropriate Types (JSON Compatible)

```c
// ✅ Good - JSON-compatible types
bool m_Enabled = true;      // For on/off
int m_Volume = 50;          // For integers
float m_Brightness = 1.0;   // For decimals
string m_Theme = "Dark";    // For text
array<string> m_Items = {}; // For arrays

// ❌ Avoid - Non-JSON types
vector m_Position;          // Vectors not directly supported
ref MyClass m_Object;       // Object references not serialized
```

### 6. Reload Settings When Needed

```c
// ✅ Good - Reload settings periodically or on file change
void OnUpdate(float delta_time)
{
	m_ReloadTimer += delta_time;
	if (m_ReloadTimer >= m_Settings.m_RefreshRateInSeconds)
	{
		m_ReloadTimer = 0;
		
		// Reload settings from file
		m_Settings = MyMod_Settings.Load();
		Print("[MyMod] Settings reloaded");
	}
}

// This allows players to edit JSON file while game is running
```

## Accessing Settings

### From Anywhere in Your Mod

```c
// Store settings in module or singleton
class MyMod_Module : MissionModule
{
	static ref MyMod_Settings s_Settings;
	
	override void OnInit()
	{
		super.OnInit();
		s_Settings = MyMod_Settings.Load();
	}
}

// Get settings instance
static MyMod_Settings GetSettings()
{
	if (!MyMod_Module.s_Settings)
	{
		MyMod_Module.s_Settings = MyMod_Settings.Load();
	}
	return MyMod_Module.s_Settings;
}

// Use anywhere
void SomeFunction()
{
	MyMod_Settings settings = GetSettings();
	if (settings)
	{
		if (settings.m_ShowNotifications)
		{
			// Show notification
		}
	}
}
```

## Summary

**Profile Settings in DayZ:**
- Saved as **JSON files** in `$profile:ModName/` directory
- **Server:** Expands to folder specified by `-profiles=` parameter (e.g., `ServerProfile/ModName/Config/Settings.json`)
- **Client:** Expands to `%localappdata%\dayz\ModName/Config/Settings.json`
- **NOT** saved in the `.profile` file itself
- Persist across sessions
- Can be edited manually (JSON format)

**Creating Settings:**
- Use `JsonFileLoader` to save/load JSON files
- Define paths using `$profile:` prefix
- Create directory structure before saving
- Provide default values in class
- Use `Load()` and `Save()` methods

**Best Practices:**
- Namespace your file paths (use mod name)
- Always create directories before saving
- Provide default values
- Save after changes
- Use JSON-compatible types only
- Consider reloading settings periodically

---

**Related Guides:**
- [How to Create a Mod](How-To-Create-Mod.md)
- [Tips: g_Game vs GetGame()](Tips-g_Game-GetGame.md)
- [Module System](Module-System.md)
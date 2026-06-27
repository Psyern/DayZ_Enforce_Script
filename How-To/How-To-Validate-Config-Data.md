# How to Validate Config Data

Guide to validating and sanitizing configuration data loaded from JSON files, including color values, numeric ranges, and input validation.

## Why Validate Config Data?

Configuration files are user-editable and can contain invalid values:
- Colors outside valid range (0-255)
- Negative numbers where only positive allowed
- Empty strings where values required
- Invalid file paths
- Out-of-range numeric values

**Always validate config data** to prevent crashes and unexpected behavior.

## Color Validation

### ARGB Color Components

ARGB colors have 4 components, each 0-255:
- **A** (Alpha) - Transparency: 0 (transparent) to 255 (opaque)
- **R** (Red) - Red channel: 0 to 255
- **G** (Green) - Green channel: 0 to 255
- **B** (Blue) - Blue channel: 0 to 255

### ✅ CORRECT - Clamp Color Values

```c
class MyMod_Config
{
	int colorR;
	int colorG;
	int colorB;
	int colorA;
	
	void Validate()
	{
		// Clamp all color components to valid range
		colorR = Math.Clamp(colorR, 0, 255);
		colorG = Math.Clamp(colorG, 0, 255);
		colorB = Math.Clamp(colorB, 0, 255);
		colorA = Math.Clamp(colorA, 0, 255);
	}
	
	int GetARGBColor()
	{
		// Build ARGB integer from components
		return (colorA << 24) | (colorR << 16) | (colorG << 8) | colorB;
	}
}
```

### Real-World Example: Notification Colors

```c
void SendNotification(LLCSpawnCrateConfig crateConfig, PlayerIdentity identity)
{
	if (!crateConfig)
		return;
	
	// Validate and clamp color values
	int a = Math.Clamp(crateConfig.colorA, 0, 255);
	int r = Math.Clamp(crateConfig.colorR, 0, 255);
	int g = Math.Clamp(crateConfig.colorG, 0, 255);
	int b = Math.Clamp(crateConfig.colorB, 0, 255);
	
	// Build ARGB color integer
	int color = (a << 24) | (r << 16) | (g << 8) | b;
	
	// Use validated color
	NotificationSystem.Create(
		new StringLocaliser(crateConfig.title),
		new StringLocaliser(crateConfig.messageOnOpen),
		crateConfig.iconPath,
		color,
		5.0,
		identity
	);
}
```

### ❌ WRONG - No Validation

```c
// ❌ WRONG - No validation, can crash with invalid values
void SendNotification(MyMod_Config cfg)
{
	int color = (cfg.colorA << 24) | (cfg.colorR << 16) | (cfg.colorG << 8) | cfg.colorB;
	// If colorA is 999, this creates invalid color!
}
```

## Numeric Range Validation

### Clamp to Valid Range

```c
class MyMod_Config
{
	int maxItems = 10;
	float spawnChance = 0.5;
	int respawnTimer = 60;
	
	void Validate()
	{
		// Clamp maxItems to positive range
		maxItems = Math.Clamp(maxItems, 0, 1000);
		
		// Clamp spawn chance to 0.0-1.0
		spawnChance = Math.Clamp(spawnChance, 0.0, 1.0);
		
		// Clamp timer to reasonable range
		respawnTimer = Math.Clamp(respawnTimer, 0, 3600);
	}
}
```

### Real-World Example: Loot Randomization

```c
class LLCSpawnCrateConfig
{
	float lootrandomize = 1.0;
	int maxItems = 0;
	
	void Validate()
	{
		// Clamp randomization to 0.0-1.0 range
		lootrandomize = Math.Clamp(lootrandomize, 0.0, 1.0);
		
		// Clamp max items to non-negative
		if (maxItems < 0)
			maxItems = 0;
	}
	
	int CalculateTakeCount(int totalLootCount)
	{
		if (lootrandomize > 0 && lootrandomize <= 1.0)
		{
			// Use clamped value safely
			return Math.Round(Math.Clamp(lootrandomize, 0.0, 1.0) * totalLootCount);
		}
		return totalLootCount;
	}
}
```

## Boolean Validation

### Clamp Boolean Values

```c
class MyMod_Config
{
	int EnableLogging = 1;
	int EnableDebugLogging = 0;
	
	void Validate()
	{
		// Clamp to 0 or 1 (boolean in JSON as int)
		EnableLogging = Math.Clamp(EnableLogging, 0, 1);
		EnableDebugLogging = Math.Clamp(EnableDebugLogging, 0, 1);
	}
}
```

## String Validation

### Check for Empty/Invalid Strings

```c
class MyMod_Config
{
	string iconPath;
	string messageOnOpen;
	
	void Validate()
	{
		// Set defaults for empty strings
		if (iconPath == "" || iconPath == "0")
			iconPath = "set:dayz_gui image:icon_gear";
		
		// Optional: Allow empty message
		if (messageOnOpen == "0")
			messageOnOpen = "";
	}
	
	bool HasValidIcon()
	{
		return (iconPath != "" && iconPath != "0");
	}
}
```

## Complete Validation Example

### Config Class with Full Validation

```c
class LLCSpawnCrateConfig
{
	string name;
	string title;
	string messageOnOpen;
	string iconPath;
	int colorR;
	int colorG;
	int colorB;
	int colorA;
	int maxItems = 0;
	float lootrandomize = 1.0;
	int EnableLogging = 1;
	int EnableDebugLogging = 0;
	
	void Validate()
	{
		// Validate colors
		colorA = Math.Clamp(colorA, 0, 255);
		colorR = Math.Clamp(colorR, 0, 255);
		colorG = Math.Clamp(colorG, 0, 255);
		colorB = Math.Clamp(colorB, 0, 255);
		
		// Validate numeric ranges
		if (maxItems < 0)
			maxItems = 0;
		
		lootrandomize = Math.Clamp(lootrandomize, 0.0, 1.0);
		
		// Validate booleans (stored as int in JSON)
		EnableLogging = Math.Clamp(EnableLogging, 0, 1);
		EnableDebugLogging = Math.Clamp(EnableDebugLogging, 0, 1);
		
		// Validate strings
		if (iconPath == "0")
			iconPath = "";
		
		if (title == "0")
			title = "";
		
		if (messageOnOpen == "0")
			messageOnOpen = "";
	}
	
	int GetARGBColor()
	{
		// Values already validated, safe to use
		return (colorA << 24) | (colorR << 16) | (colorG << 8) | colorB;
	}
}
```

### Config Loader with Validation

```c
class MyMod_ConfigLoader
{
	void LoadConfigs()
	{
		LLCSpawnJsonConfig config = new LLCSpawnJsonConfig();
		
		if (FileExist(configPath))
		{
			JsonFileLoader<LLCSpawnJsonConfig>.JsonLoadFile(configPath, config);
			
			// ✅ ALWAYS validate after loading
			config.Validate();
			
			// Process validated configs
			foreach (LLCSpawnCrateConfig crateConfig : config.listOfCrates)
			{
				// Validate each crate config
				crateConfig.Validate();
			}
		}
	}
}
```

## Validation Patterns

### Pattern 1: Validate on Load

```c
void Load()
{
	JsonFileLoader<MyMod_Config>.JsonLoadFile(path, this);
	Validate();  // Always validate after load
}
```

### Pattern 2: Validate Before Use

```c
int GetColor()
{
	// Validate before building color
	colorR = Math.Clamp(colorR, 0, 255);
	colorG = Math.Clamp(colorG, 0, 255);
	colorB = Math.Clamp(colorB, 0, 255);
	colorA = Math.Clamp(colorA, 0, 255);
	
	return (colorA << 24) | (colorR << 16) | (colorG << 8) | colorB;
}
```

### Pattern 3: Validate in Setter

```c
void SetColorR(int value)
{
	colorR = Math.Clamp(value, 0, 255);
}

void SetMaxItems(int value)
{
	maxItems = Math.Max(0, value);  // Ensure non-negative
}
```

## Common Validation Functions

### Helper Functions

```c
class MyMod_ValidationHelper
{
	// Clamp color component
	static int ClampColor(int value)
	{
		return Math.Clamp(value, 0, 255);
	}
	
	// Clamp percentage (0.0-1.0)
	static float ClampPercentage(float value)
	{
		return Math.Clamp(value, 0.0, 1.0);
	}
	
	// Clamp positive integer
	static int ClampPositive(int value, int max = 1000)
	{
		return Math.Clamp(value, 0, max);
	}
	
	// Validate and fix empty string
	static string ValidateString(string value, string defaultValue = "")
	{
		if (value == "" || value == "0")
			return defaultValue;
		return value;
	}
}
```

### Usage

```c
void Validate()
{
	colorR = MyMod_ValidationHelper.ClampColor(colorR);
	spawnChance = MyMod_ValidationHelper.ClampPercentage(spawnChance);
	maxItems = MyMod_ValidationHelper.ClampPositive(maxItems, 100);
	iconPath = MyMod_ValidationHelper.ValidateString(iconPath, "default_icon");
}
```

## Best Practices

### 1. Always Validate After Load

```c
// ✅ Good - Validate immediately after load
void Load()
{
	JsonFileLoader<MyMod_Config>.JsonLoadFile(path, this);
	Validate();
}

// ❌ Avoid - Using unvalidated data
void Load()
{
	JsonFileLoader<MyMod_Config>.JsonLoadFile(path, this);
	// Using config without validation - dangerous!
}
```

### 2. Validate All User Input

```c
// ✅ Good - Validate all config values
void Validate()
{
	colorR = Math.Clamp(colorR, 0, 255);
	colorG = Math.Clamp(colorG, 0, 255);
	colorB = Math.Clamp(colorB, 0, 255);
	colorA = Math.Clamp(colorA, 0, 255);
	maxItems = Math.Max(0, maxItems);
	spawnChance = Math.Clamp(spawnChance, 0.0, 1.0);
}
```

### 3. Use Clamp for Ranges

```c
// ✅ Good - Clamp to valid range
value = Math.Clamp(value, min, max);

// ❌ Avoid - Manual checks
if (value < min)
	value = min;
else if (value > max)
	value = max;
```

### 4. Validate Before Critical Operations

```c
// ✅ Good - Validate before building color
int GetARGBColor()
{
	// Validate first
	colorR = Math.Clamp(colorR, 0, 255);
	colorG = Math.Clamp(colorG, 0, 255);
	colorB = Math.Clamp(colorB, 0, 255);
	colorA = Math.Clamp(colorA, 0, 255);
	
	// Then build
	return (colorA << 24) | (colorR << 16) | (colorG << 8) | colorB;
}
```

## Summary

**Validation Rules:**
- ✅ **ALWAYS** validate config data after loading from JSON
- ✅ **ALWAYS** clamp color values to 0-255 range
- ✅ **ALWAYS** validate numeric ranges (use Math.Clamp)
- ✅ **ALWAYS** validate boolean values (clamp to 0-1)
- ✅ **ALWAYS** check strings for empty/invalid values

**Common Validations:**
- Colors: `Math.Clamp(value, 0, 255)`
- Percentages: `Math.Clamp(value, 0.0, 1.0)`
- Positive integers: `Math.Max(0, value)`
- Booleans: `Math.Clamp(value, 0, 1)`
- Strings: Check for `""` or `"0"`

**Quick Reference:**
- Color component: `colorR = Math.Clamp(colorR, 0, 255);`
- Percentage: `chance = Math.Clamp(chance, 0.0, 1.0);`
- Positive: `maxItems = Math.Max(0, maxItems);`
- ARGB: `(a << 24) | (r << 16) | (g << 8) | b`

---

**Related Guides:**
- [How to Create Profile Settings](How-To-Profile-Settings.md)
- [Tips: Best Practices](Tips-Best-Practices.md)
- [Tips: Common Pitfalls](Tips-Common-Pitfalls.md)

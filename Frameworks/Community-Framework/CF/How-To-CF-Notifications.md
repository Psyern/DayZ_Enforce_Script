# How to Use CF Notification System

Guide to using Community Framework's notification system in your mod.

## Overview

CF's notification system allows you to send notifications to players. It handles both server-to-client communication and singleplayer/offline modes.

**⚠️ CRITICAL:** `NotificationSystem` is a static system class. **NEVER** create an instance of it. Always use the static methods directly.

## Basic Setup

### Dependencies

First, ensure CF is a dependency in your config.cpp:

```cpp
class CfgPatches
{
	class MyMod_Scripts
	{
		requiredAddons[] = {
			"DZ_Data",
			"CF_Scripts"  // CF dependency
		};
	};
};
```

## Basic Usage

### Simple Notification

```c
class MyMod_Module : MissionModule
{
	override void OnInit()
	{
		super.OnInit();
		
		// Send notification to all clients using DayZ built-in icon
		NotificationSystem.Create(new StringLocaliser("MyMod_Notification_Title"), new StringLocaliser("MyMod_Notification_Text"), "set:dayz_gui image:cross", ARGB(255, 255, 255, 255), 5.0);
		
		// Or using custom .edds file
		// NotificationSystem.Create(new StringLocaliser("MyMod_Notification_Title"), new StringLocaliser("MyMod_Notification_Text"), "MyMod\\gui\\alarm.edds", ARGB(255, 255, 255, 255), 5.0);
	}
}
```

### Notification to Specific Player

```c
void SendNotificationToPlayer(PlayerBase player)
{
	if (!player)
		return;
	
	PlayerIdentity identity = player.GetIdentity();
	if (!identity)
		return;
	
	NotificationSystem.Create(new StringLocaliser("MyMod_Notification_Title"), new StringLocaliser("MyMod_Notification_Text"), "set:dayz_gui image:cross", ARGB(255, 255, 255, 255), 3.0, identity);
}
```

## Notification Parameters

### Full Signature

```c
static void NotificationSystem.Create(
	StringLocaliser title,      // Notification title
	StringLocaliser text,       // Notification message
	string icon,                // Icon path
	int color,                  // Color using ARGB() function
	float time = 3,             // Duration in seconds
	PlayerIdentity sendTo = NULL // Target player (NULL = all)
)
```

### Parameters Explained

#### Title and Text (StringLocaliser)

Use `StringLocaliser` for translatable text:

```c
// Using string table entry
NotificationSystem.Create(
	new StringLocaliser("#STR_MYMOD_NOTIFICATION_TITLE"),
	new StringLocaliser("#STR_MYMOD_NOTIFICATION_TEXT"),
	// ...
);

// Using plain string (no localization)
NotificationSystem.Create(
	new StringLocaliser("My Title"),
	new StringLocaliser("My message"),
	// ...
);
```

#### Icon Path

Icons can use DayZ's built-in imagesets or custom .edds files from your mod's gui folder:

**DayZ Built-in Icons (imageset format):**
```c
"set:dayz_gui image:cross"           // Cross/X icon
"set:dayz_gui image:checkmark"       // Checkmark
"set:dayz_gui image:icon_gear"       // Gear icon
"set:dayz_gui image:icon_tick"      // Tick icon
"set:dayz_gui image:icon_warning"   // Warning icon
```

**Custom .edds Files (from your mod's gui folder):**
```c
"MyMod\\gui\\alarm.edds"             // Custom .edds file
"MyMod\\gui\\icons\\custom.edds"     // Custom .edds in subfolder
```

**Note:** Use double backslashes (`\\`) for Windows paths in string literals, or single forward slashes (`/`) which also work.

#### Color (ARGB Function)

Colors use the `ARGB()` function with Alpha, Red, Green, Blue values (0-255):

```c
ARGB(255, 255, 255, 255)  // White (Alpha=255, R=255, G=255, B=255)
ARGB(255, 0, 0, 0)        // Black
ARGB(255, 255, 0, 0)      // Red
ARGB(255, 0, 255, 0)      // Green
ARGB(255, 0, 0, 255)      // Blue
ARGB(255, 255, 255, 0)    // Yellow
ARGB(255, 255, 0, 255)    // Magenta
ARGB(255, 0, 255, 255)    // Cyan
```

With transparency:
```c
ARGB(128, 255, 255, 255)  // Semi-transparent white (50% alpha)
ARGB(64, 255, 255, 255)   // More transparent (25% alpha)
```

#### Duration

Duration in seconds (float):
```c
3.0   // 3 seconds (default)
5.0   // 5 seconds
10.0  // 10 seconds
1.5   // 1.5 seconds
```

#### Target Player

```c
NULL                // Send to all clients
playerIdentity      // Send to specific player
```

## Complete Examples

### Example 1: Welcome Message

```c
class MyMod_Module : MissionModule
{
	override void OnInit()
	{
		super.OnInit();
	}
	
	// Called when player joins
	void OnPlayerConnected(PlayerBase player)
	{
		PlayerIdentity identity = player.GetIdentity();
		if (!identity)
			return;
		
		NotificationSystem.Create(new StringLocaliser("Welcome!"), new StringLocaliser("Thanks for playing with MyMod!"), "set:dayz_gui image:checkmark", ARGB(255, 0, 255, 0), 5.0, identity);
	}
}
```

### Example 2: Warning Notification

```c
void WarnPlayer(PlayerBase player, string warningMessage)
{
	PlayerIdentity identity = player.GetIdentity();
	if (!identity)
		return;
	
	NotificationSystem.Create(new StringLocaliser("Warning"), new StringLocaliser(warningMessage), "set:dayz_gui image:icon_warning", ARGB(255, 255, 0, 0), 5.0, identity);
}
```

### Example 3: Server-Wide Announcement

```c
void BroadcastAnnouncement(string message)
{
	// NULL as sendTo sends to all clients
	NotificationSystem.Create(new StringLocaliser("Server Announcement"), new StringLocaliser(message), "set:dayz_gui image:icon_gear", ARGB(255, 255, 255, 0), 10.0, NULL);
}
```

### Example 4: Achievement Notification

```c
void ShowAchievement(PlayerBase player, string achievementName)
{
	PlayerIdentity identity = player.GetIdentity();
	if (!identity)
		return;
	
	NotificationSystem.Create(new StringLocaliser("Achievement Unlocked!"), new StringLocaliser(achievementName), "set:dayz_gui image:checkmark", ARGB(255, 255, 0, 255), 5.0, identity);
}
```

## Helper Function Pattern

Create helper functions for common notification types:

```c
class MyMod_NotificationHelper
{
	// Success notification
	static void Success(PlayerIdentity identity, string message)
	{
		NotificationSystem.Create(new StringLocaliser("Success"), new StringLocaliser(message), "set:dayz_gui image:checkmark", ARGB(255, 0, 255, 0), 3.0, identity);
	}
	
	// Error notification
	static void Error(PlayerIdentity identity, string message)
	{
		NotificationSystem.Create(new StringLocaliser("Error"), new StringLocaliser(message), "set:dayz_gui image:cross", ARGB(255, 255, 0, 0), 5.0, identity);
	}
	
	// Info notification
	static void Info(PlayerIdentity identity, string message)
	{
		NotificationSystem.Create(new StringLocaliser("Information"), new StringLocaliser(message), "set:dayz_gui image:icon_gear", ARGB(255, 255, 255, 255), 3.0, identity);
	}
	
	// Warning notification
	static void Warning(PlayerIdentity identity, string message)
	{
		NotificationSystem.Create(new StringLocaliser("Warning"), new StringLocaliser(message), "set:dayz_gui image:icon_warning", ARGB(255, 255, 255, 0), 5.0, identity);
	}
	
	// Broadcast to all
	static void Broadcast(string title, string message, int color = ARGB(255, 255, 255, 255), float duration = 5.0)
	{
		NotificationSystem.Create(new StringLocaliser(title), new StringLocaliser(message), "set:dayz_gui image:icon_gear", color, duration, NULL);
	}
}
```

### Usage

```c
// Use helper functions
MyMod_NotificationHelper.Success(player.GetIdentity(), "Action completed!");
MyMod_NotificationHelper.Error(player.GetIdentity(), "Action failed!");
MyMod_NotificationHelper.Broadcast("Server Restart", "Server restarting in 10 minutes");
```

## Best Practices

### 1. Check for Valid Identity

```c
// ✅ Good - Always check
void SendNotification(PlayerBase player)
{
	if (!player)
		return;
	
	PlayerIdentity identity = player.GetIdentity();
	if (!identity)
		return;
	
	NotificationSystem.Create(/* ... */, identity);
}

// ❌ Avoid - No checks
void SendNotification(PlayerBase player)
{
	NotificationSystem.Create(/* ... */, player.GetIdentity());  // May be null!
}
```

### 2. Use Appropriate Duration

```c
// ✅ Good - Appropriate durations
NotificationSystem.Create(/* ... */, 3.0);   // Short message
NotificationSystem.Create(/* ... */, 10.0);  // Important announcement

// ❌ Avoid - Too short/long
NotificationSystem.Create(/* ... */, 0.5);   // Too fast to read
NotificationSystem.Create(/* ... */, 60.0);  // Too long, annoying
```

### 3. Choose Appropriate Colors

```c
// ✅ Good - Meaningful colors
ARGB(255, 0, 255, 0)    // Green for success
ARGB(255, 255, 0, 0)    // Red for errors
ARGB(255, 255, 255, 0)  // Yellow for warnings
ARGB(255, 255, 255, 255) // White for info

// ❌ Avoid - Random colors
ARGB(255, 18, 52, 86)   // No meaning
```

### 4. Don't Spam Notifications

```c
// ❌ Bad - Too many notifications
override void OnUpdate(float delta_time)
{
	NotificationSystem.Create(/* ... */);  // Every frame!
}

// ✅ Good - Only when needed
void OnImportantEvent()
{
	NotificationSystem.Create(/* ... */);  // Only on event
}
```

## Localization Support

For translatable notifications, create string table entries:

### stringtable.csv

```csv
STR_MYMOD_NOTIFICATION_TITLE,"My Mod Title"
STR_MYMOD_NOTIFICATION_TEXT,"My notification message"
STR_MYMOD_SUCCESS,"Success!"
STR_MYMOD_ERROR,"Error occurred"
```

### Usage

```c
NotificationSystem.Create(
	new StringLocaliser("#STR_MYMOD_NOTIFICATION_TITLE"),
	new StringLocaliser("#STR_MYMOD_NOTIFICATION_TEXT"),
	// ...
);
```

## Summary

**CF Notification System:**
- Use `NotificationSystem.Create()` (static method - never instantiate!)
- Supports server-to-client and offline modes
- Requires CF as dependency
- Use `StringLocaliser` for text
- Use `ARGB()` function for colors (not hex format)
- NULL `sendTo` broadcasts to all

**Best Practices:**
- Always check for valid identity
- Use appropriate durations
- Choose meaningful colors
- Don't spam notifications
- Use helper functions for common types

---

**Related Guides:**
- [How to Create a Mod](../../How-To/How-To-Create-Mod.md)
- [Module System](../../How-To/Module-System.md)
- [How to Use RPC](../../How-To/How-To-RPC.md)
- [Community Framework](../Community-Framework.md) - CF framework overview
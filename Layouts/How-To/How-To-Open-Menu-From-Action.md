# How to Open Menu from Action and Keybind

Complete guide to opening menus from actions and setting up keybinds in DayZ.

## Overview

This guide shows how to:
1. Create an action that opens a menu
2. Register the action globally
3. Set up keybinds in Inputs.xml
4. Handle keybind input in MissionGameplay

## Step 1: Create the Action

**File:** `scripts/4_World/classes/useractionscomponent/actions/singleuse/MyMod_ActionOpenMyWidget.c`

```c
class MyMod_ActionOpenMyWidget: ActionSingleUseBase
{
	void MyMod_ActionOpenMyWidget()
	{
		m_CommandUID = DayZPlayerConstants.CMD_ACTIONMOD_INTERACTONCE;
		m_Text = "Open My Widget";
		m_MessageStart = "";
		m_MessageSuccess = "";
		m_MessageFail = "";
	}
	
	override void CreateConditionComponents()
	{
		m_ConditionItem = new CCINone;
		m_ConditionTarget = new CCTNone;
	}
	
	override bool ActionCondition(PlayerBase player, ActionTarget target, ItemBase item)
	{
		if (!player)
			return false;
		
		// Don't allow opening if another menu is open
		if (IsMissionClient() && g_Game && g_Game.GetUIManager() && g_Game.GetUIManager().GetMenu())
			return false;
		
		return true;
	}
	
	override void OnExecuteClient(ActionData action_data)
	{
		PlayerBase player = PlayerBase.Cast(action_data.m_Player);
		if (!player || !player.GetIdentity())
			return;
		
		// Open menu via CallLaterByName (recommended pattern)
		if (g_Game)
		{
			g_Game.GetCallQueue(CALL_CATEGORY_SYSTEM).CallLaterByName(
				g_Game.GetMission(),
				"OpenMyModMenuDirect",
				0,
				false
			);
		}
	}
}
```

## Step 2: Register the Action Globally

**File:** `scripts/4_World/classes/useractionscomponent/ActionConstructor.c`

```c
modded class ActionConstructor
{
	override void RegisterActions(TTypenameArray actions)
	{
		super.RegisterActions(actions);
		
		// Register your custom action
		actions.Insert(MyMod_ActionOpenMyWidget);
	}
}
```

## Step 3: Create Menu ID Constant

**File:** `scripts/3_Game/YourModName/Constants.c`

```c
const int MENU_MYMOD_MYWIDGET = 10000;  // Start at 10000+ for custom menus
```

## Step 4: Register Menu in MissionBase

**File:** `scripts/5_Mission/YourModName/MissionBase.c`

```c
modded class MissionBase
{
	override UIScriptedMenu CreateScriptedMenu(int id)
	{
		UIScriptedMenu menu = super.CreateScriptedMenu(id);
		
		if (menu)
			return menu;
		
		switch (id)
		{
			case MENU_MYMOD_MYWIDGET:
				menu = new MyMod_MyWidgetMenu();
				break;
		}
		
		return menu;
	}
}
```

## Step 5: Create Menu Opening Method in MissionGameplay

**File:** `scripts/5_Mission/YourModName/MissionGameplay.c`

```c
modded class MissionGameplay
{
	ref MyMod_MyWidgetMenu m_MyWidgetMenu;
	
	// Called from action via CallLaterByName
	void OpenMyModMenuDirect()
	{
		if (!g_Game || g_Game.IsDedicatedServer())
			return;
		
		UIManager uiManager = g_Game.GetUIManager();
		if (!uiManager)
			return;
		
		UIScriptedMenu currentMenu = uiManager.GetMenu();
		
		// Don't open if another menu is open (unless it's our menu)
		if (currentMenu && !currentMenu.IsInherited(MyMod_MyWidgetMenu))
			return;
		
		// Toggle: if already open, close it
		if (currentMenu && currentMenu.IsInherited(MyMod_MyWidgetMenu))
		{
			uiManager.HideScriptedMenu(currentMenu);
			m_MyWidgetMenu = null;
		}
		else
		{
			// Open menu
			m_MyWidgetMenu = new MyMod_MyWidgetMenu();
			uiManager.ShowScriptedMenu(m_MyWidgetMenu, null);
		}
	}
}
```

## Step 6: Set Up Keybind in Inputs.xml

**File:** `scripts/data/Inputs.xml`

```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes" ?>
<modded_inputs>
	<inputs>
		<actions>
			<input name="UAMyModOpenMyWidget" loc="#STR_MYMOD_OPEN_WIDGET" visible="true" />
		</actions>
	</inputs>
	<preset>
		<input name="UAMyModOpenMyWidget">
			<btn name="kF5" />
		</input>
	</preset>
</modded_inputs>
```

**Important:** 
- Use `UAMyMod` prefix to avoid conflicts
- `loc` is the localization key (optional)
- `visible="true"` makes it appear in keybind settings
- `kF5` is the default key (F5)

### Common Key Names

```xml
<!-- Function Keys -->
<btn name="kF1" />
<btn name="kF2" />
<btn name="kF5" />
<btn name="kF10" />

<!-- Number Keys -->
<btn name="k0" />
<btn name="k1" />
<btn name="k9" />

<!-- Letter Keys -->
<btn name="kM" />
<btn name="kN" />
<btn name="kT" />

<!-- Special Keys -->
<btn name="kReturn" />      <!-- Enter -->
<btn name="kSpace" />       <!-- Space -->
<btn name="kTab" />         <!-- Tab -->
<btn name="kEscape" />      <!-- Escape -->
<btn name="kBackspace" />   <!-- Backspace -->

<!-- Modifier Combinations -->
<input name="UAMyModOpenMyWidget">
	<btn name="kLCtrl" />
	<btn name="kM" />
</input>
```

## Step 7: Register Input in config.cpp

**File:** `scripts/config.cpp`

```cpp
class CfgMods
{
	class MyMod
	{
		dir = "MyMod";
		name = "My Mod";
		inputs = "MyMod/Scripts/Data/Inputs.xml";  // Path to Inputs.xml
		// ... other config
	};
};
```

## Step 8: Handle Keybind in MissionGameplay

**File:** `scripts/5_Mission/YourModName/MissionGameplay.c`

```c
modded class MissionGameplay
{
	ref MyMod_MyWidgetMenu m_MyWidgetMenu;
	
	override void OnUpdate(float timeslice)
	{
		super.OnUpdate(timeslice);
		
		// Handle keybind input
		if (!g_Game || g_Game.IsDedicatedServer())
			return;
		
		PlayerBase player = PlayerBase.Cast(GetGame().GetPlayer());
		if (!player)
			return;
		
		// Check if keybind is pressed
		if (GetUApi().GetInputByID(UAMyModOpenMyWidget).LocalPress())
		{
			// Don't open if another menu is open
			UIManager uiManager = g_Game.GetUIManager();
			if (uiManager && uiManager.GetMenu() && !uiManager.GetMenu().IsInherited(MyMod_MyWidgetMenu))
				return;
			
			// Toggle menu
			OpenMyModMenuDirect();
		}
	}
	
	void OpenMyModMenuDirect()
	{
		if (!g_Game || g_Game.IsDedicatedServer())
			return;
		
		UIManager uiManager = g_Game.GetUIManager();
		if (!uiManager)
			return;
		
		UIScriptedMenu currentMenu = uiManager.GetMenu();
		
		// Toggle: if already open, close it
		if (currentMenu && currentMenu.IsInherited(MyMod_MyWidgetMenu))
		{
			uiManager.HideScriptedMenu(currentMenu);
			m_MyWidgetMenu = null;
		}
		else
		{
			// Open menu
			m_MyWidgetMenu = new MyMod_MyWidgetMenu();
			uiManager.ShowScriptedMenu(m_MyWidgetMenu, null);
		}
	}
}
```

## Alternative: Direct Menu Opening (Simpler)

If you don't need an action and just want a keybind:

**File:** `scripts/5_Mission/YourModName/MissionGameplay.c`

```c
modded class MissionGameplay
{
	override void OnUpdate(float timeslice)
	{
		super.OnUpdate(timeslice);
		
		if (!g_Game || g_Game.IsDedicatedServer())
			return;
		
		PlayerBase player = PlayerBase.Cast(GetGame().GetPlayer());
		if (!player)
			return;
		
		// Check keybind
		if (GetUApi().GetInputByID(UAMyModOpenMyWidget).LocalPress())
		{
			UIManager uiManager = g_Game.GetUIManager();
			if (!uiManager)
				return;
			
			// Check if menu is already open
			if (uiManager.IsMenuOpen(MENU_MYMOD_MYWIDGET))
			{
				// Close menu
				UIScriptedMenu menu = uiManager.FindMenu(MENU_MYMOD_MYWIDGET);
				if (menu)
					menu.Close();
			}
			else
			{
				// Open menu
				uiManager.EnterScriptedMenu(MENU_MYMOD_MYWIDGET, null);
			}
		}
	}
}
```

## Complete Example

### Action File

**File:** `scripts/4_World/classes/useractionscomponent/actions/singleuse/MyMod_ActionOpenMyWidget.c`

```c
class MyMod_ActionOpenMyWidget: ActionSingleUseBase
{
	void MyMod_ActionOpenMyWidget()
	{
		m_CommandUID = DayZPlayerConstants.CMD_ACTIONMOD_INTERACTONCE;
		m_Text = "Open My Widget";
	}
	
	override void CreateConditionComponents()
	{
		m_ConditionItem = new CCINone;
		m_ConditionTarget = new CCTNone;
	}
	
	override bool ActionCondition(PlayerBase player, ActionTarget target, ItemBase item)
	{
		if (!player)
			return false;
		
		if (IsMissionClient() && g_Game && g_Game.GetUIManager() && g_Game.GetUIManager().GetMenu())
			return false;
		
		return true;
	}
	
	override void OnExecuteClient(ActionData action_data)
	{
		if (!g_Game)
			return;
		
		g_Game.GetCallQueue(CALL_CATEGORY_SYSTEM).CallLaterByName(
			g_Game.GetMission(),
			"OpenMyModMenuDirect",
			0,
			false
		);
	}
}
```

### ActionConstructor

**File:** `scripts/4_World/classes/useractionscomponent/ActionConstructor.c`

```c
modded class ActionConstructor
{
	override void RegisterActions(TTypenameArray actions)
	{
		super.RegisterActions(actions);
		actions.Insert(MyMod_ActionOpenMyWidget);
	}
}
```

### Inputs.xml

**File:** `scripts/data/Inputs.xml`

```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes" ?>
<modded_inputs>
	<inputs>
		<actions>
			<input name="UAMyModOpenMyWidget" loc="#STR_MYMOD_OPEN_WIDGET" visible="true" />
		</actions>
	</inputs>
	<preset>
		<input name="UAMyModOpenMyWidget">
			<btn name="kF5" />
		</input>
	</preset>
</modded_inputs>
```

### MissionGameplay

**File:** `scripts/5_Mission/YourModName/MissionGameplay.c`

```c
modded class MissionGameplay
{
	ref MyMod_MyWidgetMenu m_MyWidgetMenu;
	
	override void OnUpdate(float timeslice)
	{
		super.OnUpdate(timeslice);
		
		if (!g_Game || g_Game.IsDedicatedServer())
			return;
		
		PlayerBase player = PlayerBase.Cast(GetGame().GetPlayer());
		if (!player)
			return;
		
		// Handle keybind
		if (GetUApi().GetInputByID(UAMyModOpenMyWidget).LocalPress())
		{
			OpenMyModMenuDirect();
		}
	}
	
	void OpenMyModMenuDirect()
	{
		if (!g_Game || g_Game.IsDedicatedServer())
			return;
		
		UIManager uiManager = g_Game.GetUIManager();
		if (!uiManager)
			return;
		
		UIScriptedMenu currentMenu = uiManager.GetMenu();
		
		// Don't open if another menu is open
		if (currentMenu && !currentMenu.IsInherited(MyMod_MyWidgetMenu))
			return;
		
		// Toggle menu
		if (currentMenu && currentMenu.IsInherited(MyMod_MyWidgetMenu))
		{
			uiManager.HideScriptedMenu(currentMenu);
			m_MyWidgetMenu = null;
		}
		else
		{
			m_MyWidgetMenu = new MyMod_MyWidgetMenu();
			uiManager.ShowScriptedMenu(m_MyWidgetMenu, null);
		}
	}
}
```

## Adding Action to Item (Optional)

If you want the action to appear on an item:

**File:** `scripts/4_World/modded/MyMod_MyItem.c`

```c
modded class MyMod_MyItem: ItemBase
{
	override void SetActions()
	{
		super.SetActions();
		AddAction(MyMod_ActionOpenMyWidget);
	}
}
```

## Best Practices

### 1. Check for Existing Menus

```c
// ✅ Good - Check if menu is already open
if (uiManager.GetMenu() && !uiManager.GetMenu().IsInherited(MyMod_MyWidgetMenu))
	return;

// ❌ Bad - May open multiple menus
uiManager.EnterScriptedMenu(MENU_MYMOD_MYWIDGET, null);
```

### 2. Use CallLaterByName for Actions

```c
// ✅ Good - Delayed execution (recommended pattern)
g_Game.GetCallQueue(CALL_CATEGORY_SYSTEM).CallLaterByName(
	g_Game.GetMission(),
	"OpenMyModMenuDirect",
	0,
	false
);

// ⚠️ Alternative - Direct call (works but less safe)
g_Game.GetMission().OpenMyModMenuDirect();
```

### 3. Toggle Menu Instead of Always Opening

```c
// ✅ Good - Toggle behavior
if (currentMenu && currentMenu.IsInherited(MyMod_MyWidgetMenu))
{
	uiManager.HideScriptedMenu(currentMenu);
}
else
{
	uiManager.ShowScriptedMenu(new MyMod_MyWidgetMenu(), null);
}
```

### 4. Use Mod Prefix for Input Names

```c
// ✅ Good - Unique prefix
UAMyModOpenMyWidget

// ❌ Bad - May conflict
UAOpenMyWidget
```

### 5. Register Input in config.cpp

```cpp
// ✅ Good - Register inputs
inputs = "MyMod/Scripts/Data/Inputs.xml";
```

## Summary

**Action Pattern:**
1. Create action class extending `ActionSingleUseBase`
2. Register in `ActionConstructor`
3. Use `CallLaterByName` to open menu
4. Menu opening method in `MissionGameplay`

**Keybind Pattern:**
1. Define input in `Inputs.xml`
2. Register in `config.cpp`
3. Check input in `MissionGameplay.OnUpdate()`
4. Open/close menu based on input

**Key Files:**
- Action: `scripts/4_World/classes/useractionscomponent/actions/singleuse/`
- ActionConstructor: `scripts/4_World/classes/useractionscomponent/ActionConstructor.c`
- Inputs.xml: `scripts/data/Inputs.xml`
- MissionGameplay: `scripts/5_Mission/YourModName/MissionGameplay.c`

---

**Related Guides:**
- [How to Create Actions](../../../GIT/DAYZ_Enforce%20Script_StyleGuide/Wiki/How-To/How-To-Actions.md) - Complete action guide
- [How to Create UI Menus](../../../GIT/DAYZ_Enforce%20Script_StyleGuide/Wiki/How-To/How-To-UI-Menus.md) - Menu creation guide
- [How MVC Works](../How-MVC-Works.md) - Understanding menu architecture

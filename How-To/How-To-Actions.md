# How to Create Actions

Complete guide to creating and using actions in DayZ mods.

## What are Actions?

Actions are player interactions with items and world objects. They define what players can do (eat, open, craft, repair, etc.).

## Action Folder Structure

Actions belong in the **4_World** layer:

```
scripts/4_world/classes/useractionscomponent/actions/
├── actionbase.c                    # Base action class
├── actioncontinuousbase.c          # Base for continuous actions
├── actionsingleusebase.c           # Base for single-use actions
├── actioninteractbase.c           # Base for interact actions
├── actioninstantbase.c             # Base for instant actions
├── actionsequentialbase.c          # Base for sequential actions
├── continuous/                     # Continuous actions (210+ files)
│   ├── medical/                    # Medical actions subfolder
│   │   ├── actionbandagetarget.c
│   │   ├── actiontestbloodtarget.c
│   │   └── ...
│   ├── deployactions/              # Deploy actions subfolder
│   │   └── actiondeploybase.c
│   └── actioneatmeat.c
│   └── ...
├── singleuse/                      # Single-use actions (105+ files)
│   ├── actionopen.c
│   ├── actionturnonhelmetflashlight.c
│   └── ...
├── interact/                       # Interact actions (71+ files)
│   ├── vehicles/                   # Vehicle actions subfolder
│   │   └── actionswitchlights.c
│   ├── actionbuildshelter.c
│   └── ...
└── weapons/                        # Weapon-specific actions
    ├── firearmactionbase.c
    ├── firearmactionattachmagazine.c
    └── ...
```

**Important:** All actions go in **4_World** layer. See [Script Layers Guide](Script-Layers-Guide.md) for why.

## Action Class Naming Conventions

**Pattern:** `Action[Verb][Object]` (PascalCase, no spaces)

### Base Action Classes

```c
// Base classes (abstract, don't use directly)
class ActionBase                    // Root base class
class ActionContinuousBase          // For actions that take time
class ActionSingleUseBase           // For one-time actions
class ActionInteractBase            // For interaction actions
class ActionInstantBase             // For instant actions
class ActionSequentialBase          // For sequential actions
```

### Action Implementation Naming

**Format:** `Action[Verb][Object]` or `Action[ActionDescription]`

```c
// ✅ CORRECT - Action naming examples
class ActionEatMeat                 // Action + Verb + Object
class ActionOpen                     // Action + Verb
class ActionBandageTarget           // Action + Verb + Target
class ActionBuildPart               // Action + Verb + Object
class ActionCraftStoneKnifeEnv      // Action + Verb + Object + Context
class ActionTurnOnHelmetFlashlight  // Action + Verb + Object + Detail
class ActionSwitchLights            // Action + Verb + Object
class ActionUnpackGift              // Action + Verb + Object

// ❌ WRONG - Incorrect naming
class EatMeatAction                 // Wrong order
class action_eat_meat               // Wrong case and format
class ActionEat_Meat                // Wrong separator
```

### Action Callback Classes

**Pattern:** `Action[ActionName]CB` (Callback class)

```c
// ✅ CORRECT - Callback naming
class ActionEatMeatCB : ActionContinuousBaseCB
{
	override void CreateActionComponent()
	{
		m_ActionData.m_ActionComponent = new CAContinuousQuantityEdible(...);
	}
}

// ❌ WRONG
class EatMeatActionCB               // Wrong order
class ActionEatMeatCallback         // Should be "CB" not "Callback"
```

## Action Types

### 1. Continuous Actions (`ActionContinuousBase`)

**Use for:** Actions that take time and have progress (eating, crafting, repairing)

**Location:** `actions/continuous/`

```c
class ActionEatMeat : ActionEatBig  // Inherits from continuous action
{
	void ActionEatMeat()
	{
		m_CallbackClass = ActionEatMeatCB;  // Set callback
	}
	
	override void ApplyModifiers(ActionData action_data)
	{
		// Action logic
	}
}
```

### 2. Single-Use Actions (`ActionSingleUseBase`)

**Use for:** One-time actions (open, close, toggle)

**Location:** `actions/singleuse/`

```c
class ActionOpen : ActionSingleUseBase
{
	void ActionOpen()
	{
		m_CommandUID = DayZPlayerConstants.CMD_ACTIONMOD_OPENITEM_ONCE;
		m_Text = "#open";
	}
	
	override void OnExecuteServer(ActionData action_data)
	{
		action_data.m_MainItem.Open();
	}
}
```

### 3. Interact Actions (`ActionInteractBase`)

**Use for:** Actions that interact with world objects (build, toggle, set)

**Location:** `actions/interact/`

```c
class ActionBuildShelter : ActionInteractBase
{
	// Interact with world object
}
```

## Action Folder Organization

**Subfolders for organization:**

- `medical/` - Medical-related actions
- `vehicles/` - Vehicle-specific actions
- `deployactions/` - Deployment actions
- `weapons/` - Weapon-specific actions

**Example:**
```
actions/continuous/medical/actionbandagetarget.c
actions/interact/vehicles/actionswitchlights.c
```

## Registering Actions Globally (ActionConstructor)

**CRITICAL:** Before actions can be used in `SetActions()`, they must be registered globally via `ActionConstructor`.

### Creating ActionConstructor

Create a modded `ActionConstructor` class to register your actions globally:

**File:** `scripts/4_World/classes/useractionscomponent/ActionConstructor.c`

```c
modded class ActionConstructor
{
	override void RegisterActions(TTypenameArray actions)
	{
		super.RegisterActions(actions);
		
		// Register your custom actions here
		actions.Insert(MyMod_ActionCustom);
		actions.Insert(MyMod_ActionTurnOnHeater);
		actions.Insert(MyMod_ActionTurnOffHeater);
	}
}
```

### Why ActionConstructor is Required

1. **Global Registration**: Actions must be registered globally before they can be used
2. **System-Wide Availability**: Makes actions available to all items via `SetActions()`
3. **Initialization Order**: Actions are registered when the game starts, before items are created

### ActionConstructor Location

```
scripts/4_World/classes/useractionscomponent/ActionConstructor.c
```

**Important:** The path must match exactly: `classes/useractionscomponent/ActionConstructor.c`

### Complete Example

**File:** `scripts/4_World/classes/useractionscomponent/ActionConstructor.c`

```c
modded class ActionConstructor
{
	override void RegisterActions(TTypenameArray actions)
	{
		super.RegisterActions(actions);
		
		// Register custom actions
		actions.Insert(MyMod_ActionCustom);
		actions.Insert(MyMod_ActionTurnOnHeater);
		actions.Insert(MyMod_ActionTurnOffHeater);
	}
}
```

**File:** `scripts/4_World/actions/interact/MyMod_ActionTurnOnHeater.c`

```c
class MyMod_ActionTurnOnHeater : ActionInteractBase
{
	void MyMod_ActionTurnOnHeater()
	{
		m_CommandUID = DayZPlayerConstants.CMD_ACTIONMOD_INTERACTONCE;
		m_Text = "#switch_on";
	}
	
	override bool ActionCondition(PlayerBase player, ActionTarget target, ItemBase item)
	{
		// Action condition logic
		return true;
	}
	
	override void OnExecuteServer(ActionData action_data)
	{
		// Action execution logic
	}
}
```

**File:** `scripts/4_World/modded/MyMod_Heater.c`

```c
class MyMod_Heater : Inventory_Base
{
	override void SetActions()
	{
		super.SetActions();
		
		// Add globally registered actions
		AddAction(MyMod_ActionTurnOnHeater);
		AddAction(MyMod_ActionTurnOffHeater);
	}
}
```

### Registration Flow

```
1. Game Startup
   └── ActionConstructor.RegisterActions()
       └── Registers all actions globally
           └── Actions available system-wide

2. Item Creation
   └── Item.EEInit()
       └── Item.SetActions()
           └── AddAction(MyMod_ActionCustom)
               └── Uses globally registered action
```

### Common Mistakes

#### ❌ Missing ActionConstructor

```c
// ❌ WRONG - Action not registered globally
// File: scripts/4_World/modded/MyMod_Heater.c
override void SetActions()
{
	super.SetActions();
	AddAction(MyMod_ActionCustom);  // ❌ Action doesn't exist globally!
}
```

**Result:** Action won't appear, no error message

#### ✅ Correct - ActionConstructor + SetActions

```c
// ✅ CORRECT - Register globally first
// File: scripts/4_World/classes/useractionscomponent/ActionConstructor.c
modded class ActionConstructor
{
	override void RegisterActions(TTypenameArray actions)
	{
		super.RegisterActions(actions);
		actions.Insert(MyMod_ActionCustom);  // ✅ Register globally
	}
}

// File: scripts/4_World/modded/MyMod_Heater.c
override void SetActions()
{
	super.SetActions();
	AddAction(MyMod_ActionCustom);  // ✅ Now it works!
}
```

### Best Practices

1. **Register All Custom Actions**: Every custom action must be in `ActionConstructor`
2. **Use Mod Prefix**: Prefix actions with your mod name to avoid conflicts
3. **Call super**: Always call `super.RegisterActions(actions)` first
4. **One ActionConstructor**: Only one `ActionConstructor.c` file per mod

### ActionConstructor vs SetActions

| Aspect | ActionConstructor | SetActions |
|--------|------------------|------------|
| **Purpose** | Global action registration | Per-item action assignment |
| **When Called** | Game startup | Item initialization |
| **Scope** | System-wide | Item-specific |
| **Required** | Yes, for custom actions | Yes, to add actions to items |
| **Location** | `classes/useractionscomponent/` | Item class file |

## Registering Actions on Items

After registering actions globally in `ActionConstructor`, add them to items using `SetActions()`:

```c
// In ItemBase or custom item class
override void SetActions()
{
	super.SetActions();
	
	// Add globally registered actions
	AddAction(ActionEatMeat);        // Vanilla action (already registered)
	AddAction(ActionOpen);           // Vanilla action (already registered)
	AddAction(MyMod_ActionCustom);   // Your custom action (registered in ActionConstructor)
}

// Or in modded ItemBase
modded class ItemBase
{
	override void SetActions()
	{
		super.SetActions();
		AddAction(MyMod_ActionCustom);  // Your custom action (must be in ActionConstructor)
	}
}
```

**Important:** Actions added via `AddAction()` must first be registered in `ActionConstructor`. Vanilla actions are already registered, but custom actions must be added to `ActionConstructor`.

## Opening UI Menus from Actions

Actions can open UI menus to display interfaces to players.

### Step 1: Define Menu ID

First, define your menu ID constant (start at 10000+ to avoid conflicts):

**File:** `scripts/3_Game/YourModName/Constants.c`

```c
const int MENU_MYMOD_CUSTOM = 10000;  // Start at 10000+ for custom menus
```

### Step 2: Register Menu in MissionBase

Register your menu in `modded class MissionBase`:

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
			case MENU_MYMOD_CUSTOM:
				menu = new MyMod_CustomMenu;
				break;
		}
		
		return menu;
	}
}
```

### Step 3: Open Menu from Action

Open the menu from your action (client-side only):

```c
class MyMod_ActionOpenMenu : ActionSingleUseBase
{
	void MyMod_ActionOpenMenu()
	{
		m_CommandUID = DayZPlayerConstants.CMD_ACTIONMOD_OPENITEM_ONCE;
		m_Text = "#open_menu";
	}
	
	override void OnExecuteClient(ActionData action_data)
	{
		// Only open menu on client
		if (!g_Game || g_Game.IsDedicatedServer())
			return;
		
		UIManager uiManager = g_Game.GetUIManager();
		if (uiManager && !uiManager.IsMenuOpen(MENU_MYMOD_CUSTOM))
		{
			uiManager.EnterScriptedMenu(MENU_MYMOD_CUSTOM, null);
		}
	}
	
	override void OnExecuteServer(ActionData action_data)
	{
		// Server-side logic if needed
	}
}
```

### Checking if Menu is Open

```c
// Check if menu is open
if (g_Game)
{
	UIManager uiManager = g_Game.GetUIManager();
	if (uiManager && uiManager.IsMenuOpen(MENU_MYMOD_CUSTOM))
	{
		// Menu is open
	}
}

// Close menu
if (g_Game)
{
	UIManager uiManager = g_Game.GetUIManager();
	if (uiManager && uiManager.IsMenuOpen(MENU_MYMOD_CUSTOM))
	{
		uiManager.FindMenu(MENU_MYMOD_CUSTOM).Close();
	}
}
```

**See:** [How to Create UI Menus](How-To-UI-Menus.md) for complete UI menu guide.

## Complete Example: Custom Action

**File:** `scripts/4_world/classes/useractionscomponent/actions/continuous/MyMod_ActionCustom.c`

```c
// Callback class
class MyMod_ActionCustomCB : ActionContinuousBaseCB
{
	override void CreateActionComponent()
	{
		m_ActionData.m_ActionComponent = new CAContinuousTime(5.0);  // 5 second action
	}
}

// Action class
class MyMod_ActionCustom : ActionContinuousBase
{
	void MyMod_ActionCustom()
	{
		m_CallbackClass = MyMod_ActionCustomCB;
		m_CommandUID = DayZPlayerConstants.CMD_ACTIONMOD_CUSTOM;
		m_Text = "#custom_action";
		m_FullBody = false;
		m_StanceMask = DayZPlayerConstants.STANCEMASK_ERECT | DayZPlayerConstants.STANCEMASK_CROUCH;
	}
	
	override void CreateConditionComponents()
	{
		m_ConditionItem = new CCINonRuined;
		m_ConditionTarget = new CCTNone;
	}
	
	override bool ActionCondition(PlayerBase player, ActionTarget target, ItemBase item)
	{
		if (item && item.GetQuantity() > 0)
		{
			return true;
		}
		return false;
	}
	
	override void OnExecuteServer(ActionData action_data)
	{
		// Custom action logic
		Print("[MyMod] Custom action executed");
	}
}
```

## Best Practices

### 1. Use Descriptive Action Names

```c
// ✅ Good - Clear and descriptive
class ActionEatMeat
class ActionBandageTarget
class ActionCraftStoneKnifeEnv

// ❌ Avoid - Too generic
class ActionDo
class ActionUse
```

### 2. Organize Actions in Subfolders

```c
// ✅ Good - Organized by category
actions/continuous/medical/actionbandagetarget.c
actions/interact/vehicles/actionswitchlights.c

// ❌ Avoid - All actions in one folder
actions/actionbandagetarget.c
actions/actionswitchlights.c
```

### 3. Prefix Custom Actions

```c
// ✅ Good - Prefixed to avoid conflicts
class MyMod_ActionCustom

// ❌ Avoid - Generic names
class ActionCustom
```

## Summary

**Actions:**
- **Location:** `4_world/classes/useractionscomponent/actions/`
- **Naming:** `Action[Verb][Object]` (PascalCase)
- **Callback:** `Action[Name]CB`
- **Types:** Continuous, SingleUse, Interact, Instant, Sequential
- **Global Registration:** `ActionConstructor.RegisterActions()` (REQUIRED for custom actions)
- **Item Registration:** `AddAction()` in `SetActions()` method

**Key Rules:**
- All actions go in **4_World** layer
- Use PascalCase for all class names
- Prefix custom actions with your mod prefix
- Organize actions in subfolders by category
- **CRITICAL:** Register custom actions globally in `ActionConstructor` before using in `SetActions()`
- Register actions on items using `SetActions()` method

**Registration Flow:**
1. Create action class in `actions/` folder
2. Register globally in `ActionConstructor.RegisterActions()`
3. Add to items using `SetActions()` and `AddAction()`

---

**Related Guides:**
- [Script Layers Guide](Script-Layers-Guide.md) - Understanding where code belongs
- [How to Create Recipes](How-To-Recipes.md) - Recipe system guide
- [How to Create UI Menus](How-To-UI-Menus.md) - Creating UI menus and layouts
- [Tips: Modded Classes](../Tips/Tips-Modded-Classes.md) - How to extend vanilla classes
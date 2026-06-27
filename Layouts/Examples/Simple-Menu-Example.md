# Simple Menu Example

Complete example of a simple MVC menu implementation in DayZ.

## Overview

This example demonstrates:
- Basic layout file structure
- Controller with ViewBinding
- Menu class setup
- Simple property binding

## File Structure

```
MyMod/
├── gui/
│   └── YourModName/
│       └── GUI/
│           └── layouts/
│               └── simple_menu.layout
└── scripts/
    └── 5_Mission/
        └── YourModName/
            └── GUI/
                ├── MyMod_SimpleMenu.c
                └── MyMod_SimpleMenuController.c
```

## Step 1: Create Layout File

**File:** `gui/YourModName/GUI/layouts/simple_menu.layout`

```xml
FrameWidgetClass SimpleMenu {
 position 0.2 0.2
 size 0.6 0.6
 visible 1
 scriptclass "MyMod_SimpleMenuController"
 {
  PanelWidgetClass background {
   position 0 0
   size 1 1
   color 0.1 0.1 0.1 0.95
   {
    TextWidgetClass title_text {
     position 0.05 0.05
     size 0.9 0.1
     scriptclass "ViewBinding"
     style Bold
     fontsize 24
     {
      ScriptParamsClass {
       Binding_Name "Title"
      }
     }
    }
    
    TextWidgetClass message_text {
     position 0.05 0.2
     size 0.9 0.3
     scriptclass "ViewBinding"
     fontsize 18
     {
      ScriptParamsClass {
       Binding_Name "Message"
      }
     }
    }
    
    ButtonWidgetClass close_button {
     position 0.35 0.7
     size 0.3 0.15
     text "Close"
     scriptclass "ViewBinding"
     {
      ScriptParamsClass {
       Relay_Command "OnCloseButtonClick"
      }
     }
    }
   }
  }
 }
}
```

## Step 2: Create Controller

**File:** `scripts/5_Mission/YourModName/GUI/MyMod_SimpleMenuController.c`

```c
class MyMod_SimpleMenuController: ExpansionViewController
{
	// Properties bound to widgets via ViewBinding
	string Title = "Simple Menu";
	string Message = "Welcome to My Mod!";
	
	// Handle button clicks via Relay_Command
	void OnCloseButtonClick()
	{
		MyMod_SimpleMenu menu = MyMod_SimpleMenu.Cast(GetParent());
		if (menu)
		{
			menu.Close();
		}
	}
}
```

## Step 3: Create Menu Class

**File:** `scripts/5_Mission/YourModName/GUI/MyMod_SimpleMenu.c`

```c
class MyMod_SimpleMenu: ExpansionScriptViewMenu
{
	protected ref MyMod_SimpleMenuController m_Controller;
	
	override string GetLayoutFile()
	{
		return "YourModName/GUI/layouts/simple_menu.layout";
	}
	
	override typename GetControllerType()
	{
		return MyMod_SimpleMenuController;
	}
	
	override Widget Init()
	{
		Widget root = super.Init();
		
		// Get controller instance
		m_Controller = MyMod_SimpleMenuController.Cast(GetController());
		
		if (m_Controller)
		{
			// Initialize controller properties
			m_Controller.Title = "My Simple Menu";
			m_Controller.Message = "This is a simple MVC menu example.";
		}
		
		return root;
	}
	
	MyMod_SimpleMenuController GetMenuController()
	{
		if (!m_Controller)
			m_Controller = MyMod_SimpleMenuController.Cast(GetController());
		
		return m_Controller;
	}
	
	void UpdateMessage(string newMessage)
	{
		MyMod_SimpleMenuController controller = GetMenuController();
		if (controller)
		{
			controller.Message = newMessage;  // Widget updates automatically
		}
	}
}
```

## Step 4: Register Menu

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
			case MENU_MYMOD_SIMPLE:
				menu = new MyMod_SimpleMenu();
				break;
		}
		
		return menu;
	}
}
```

## Step 5: Open Menu

**From Action or Code:**

```c
void OpenSimpleMenu()
{
	if (!g_Game || g_Game.IsDedicatedServer())
		return;
	
	UIManager uiManager = g_Game.GetUIManager();
	if (uiManager && !uiManager.IsMenuOpen(MENU_MYMOD_SIMPLE))
	{
		uiManager.EnterScriptedMenu(MENU_MYMOD_SIMPLE, null);
	}
}
```

## How It Works

### 1. Layout Loads
- `GetLayoutFile()` returns layout path
- System loads layout and creates widgets
- Root widget gets `scriptclass "MyMod_SimpleMenuController"`

### 2. Controller Created
- `GetControllerType()` returns controller class
- System creates `MyMod_SimpleMenuController` instance
- Controller attached to root widget

### 3. ViewBinding Connects
- Widgets with `scriptclass "ViewBinding"` connect to controller
- `Binding_Name "Title"` → `controller.Title`
- `Binding_Name "Message"` → `controller.Message`

### 4. Property Updates
- When `controller.Title` changes → `title_text` widget updates
- When `controller.Message` changes → `message_text` widget updates

### 5. Button Click
- User clicks `close_button`
- `Relay_Command "OnCloseButtonClick"` triggers
- Controller's `OnCloseButtonClick()` method called
- Menu closes

## Extending the Example

### Add Checkbox with Two-Way Binding

**Layout:**
```xml
CheckBoxWidgetClass enable_checkbox {
 position 0.05 0.5
 size 32 32
 scriptclass "ViewBinding"
 {
  ScriptParamsClass {
   Binding_Name "IsEnabled"
   Two_Way_Binding 1
  }
 }
}

TextWidgetClass enable_label {
 position 0.1 0.5
 size 0.8 0.05
 text "Enable Feature"
}
```

**Controller:**
```c
class MyMod_SimpleMenuController: ExpansionViewController
{
	bool IsEnabled = false;
	bool WasEnabled = false;
	
	override void PropertyChanged(string property_name)
	{
		if (property_name == "IsEnabled")
		{
			if (IsEnabled != WasEnabled)
			{
				WasEnabled = IsEnabled;
				
				MyMod_SimpleMenu menu = MyMod_SimpleMenu.Cast(GetParent());
				if (menu)
				{
					menu.OnEnabledChanged(IsEnabled);
				}
			}
		}
	}
}
```

**Menu:**
```c
class MyMod_SimpleMenu: ExpansionScriptViewMenu
{
	void OnEnabledChanged(bool enabled)
	{
		Print("Feature enabled: " + enabled);
		// Update UI based on checkbox state
	}
}
```

### Add Dynamic List

**Layout:**
```xml
GridSpacerWidgetClass items_container {
 position 0.05 0.6
 size 0.9 0.3
 scriptclass "ViewBinding"
 Columns 1
 Rows 100
 {
  ScriptParamsClass {
   Binding_Name "Items"
  }
 }
}
```

**Item Element Layout:**
```xml
FrameWidgetClass item_element {
 scriptclass "MyMod_ItemController"
 {
  TextWidgetClass item_name {
   position 0.05 0
   size 0.9 1
   scriptclass "ViewBinding"
   {
    ScriptParamsClass {
     Binding_Name "Name"
    }
   }
  }
 }
}
```

**Item Controller:**
```c
class MyMod_Item: ObservableObject
{
	string Name = "";
	
	void MyMod_Item(string name)
	{
		Name = name;
	}
}
```

**Menu Controller:**
```c
class MyMod_SimpleMenuController: ExpansionViewController
{
	ref ObservableCollection<ref MyMod_Item> Items = new ObservableCollection<ref MyMod_Item>(this);
}
```

**Menu:**
```c
class MyMod_SimpleMenu: ExpansionScriptViewMenu
{
	void PopulateItems()
	{
		MyMod_SimpleMenuController controller = GetMenuController();
		if (!controller)
			return;
		
		controller.Items.Clear();
		
		controller.Items.Add(new MyMod_Item("Item 1"));
		controller.Items.Add(new MyMod_Item("Item 2"));
		controller.Items.Add(new MyMod_Item("Item 3"));
	}
}
```

## Summary

**Simple Menu Components:**
1. **Layout File** - Defines UI structure with ViewBinding
2. **Controller** - Properties and methods bound to widgets
3. **Menu Class** - Initializes controller and handles logic

**Key Concepts:**
- `scriptclass` on root widget = Controller class
- `ViewBinding` on widgets = Property binding
- `Binding_Name` = Controller property name
- `Relay_Command` = Controller method name

**Next Steps:**
- Add more widgets (checkboxes, listboxes, etc.)
- Add two-way binding for interactive widgets
- Add Observable Collections for dynamic lists
- Connect to modules via ScriptInvoker

---

**Related Guides:**
- [How Layouts Work](../How-Layouts-Work.md) - Understanding layout files
- [How MVC Works](../How-MVC-Works.md) - Controller pattern
- [How Invoke Works](../How-Invoke-Works.md) - Module communication
- [Market Menu Example](Market-Menu-Example.md) - Advanced example

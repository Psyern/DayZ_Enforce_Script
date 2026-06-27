# How MVC Works

Complete guide to understanding the Model-View-Controller (MVC) pattern in DayZ layouts and GUI scripting.

## What is MVC?

MVC separates concerns into three components:

- **Model** - Data and business logic (modules, settings, game state)
- **View** - Visual representation (layout files)
- **Controller** - Connects View to Model (script classes)

```
┌─────────┐      ┌──────────────┐      ┌──────────┐
│  Model  │◄─────│  Controller   │◄─────│   View   │
│ (Data)  │      │  (Script)    │      │ (Layout) │
└─────────┘      └──────────────┘      └──────────┘
     │                  │                    │
     └──────────────────┴────────────────────┘
              User Interaction
```

## MVC Components in DayZ

### Model

The Model represents data and business logic:

- **Modules** - `ExpansionMarketModule`, `CF_ModuleCoreManager`
- **Settings** - `ExpansionSettings`, configuration data
- **Game State** - Player data, inventory, world state
- **Business Logic** - Trading, calculations, validation

**Example:**
```c
class ExpansionMarketModule: CF_ModuleCoreManager
{
	// Model: Data and business logic
	protected ref ExpansionMarketTrader m_TraderMarket;
	protected ref array<ExpansionMarketItem> m_TraderItems;
	
	void ProcessTrade(ExpansionMarketItem item, int quantity)
	{
		// Business logic here
	}
}
```

### View

The View is the visual representation:

- **Layout Files** - `.layout` files defining UI structure
- **Widgets** - Buttons, text, images, panels
- **ViewBinding** - Connections to controller properties

**Example:**
```xml
FrameWidgetClass MyMenu {
 scriptclass "MyMod_MenuController"
 {
  TextWidgetClass title_text {
   scriptclass "ViewBinding"
   {
    ScriptParamsClass {
     Binding_Name "Title"
    }
   }
  }
 }
}
```

### Controller

The Controller connects View to Model:

- **Controller Classes** - Extend `ExpansionViewController` or `ViewController`
- **Properties** - Bound to widgets via ViewBinding
- **Methods** - Handle user interactions
- **PropertyChanged** - React to property changes

**Example:**
```c
class MyMod_MenuController: ExpansionViewController
{
	// Properties bound to widgets
	string Title = "My Menu";
	bool IsEnabled = false;
	
	// React to property changes
	override void PropertyChanged(string property_name)
	{
		if (property_name == "IsEnabled")
		{
			Print("IsEnabled changed to: " + IsEnabled);
		}
	}
	
	// Handle button clicks
	void OnButtonClick()
	{
		Print("Button clicked!");
	}
}
```

## Controller Pattern

### Creating a Controller

**1. Extend Base Controller Class**

```c
class MyMod_MenuController: ExpansionViewController
{
	// Controller properties
}
```

**2. Define Properties**

Properties that match `Binding_Name` in layout files:

```c
class MyMod_MenuController: ExpansionViewController
{
	// String properties
	string Title = "My Menu";
	string PlayerName = "Player";
	
	// Boolean properties
	bool IsEnabled = false;
	bool ShowDetails = true;
	
	// Object properties (for previews)
	Object ItemPreview;
	Object PlayerPreview;
	
	// Observable collections (for dynamic lists)
	ref ObservableCollection<ref MyMod_Item> Items = new ObservableCollection<ref MyMod_Item>(this);
}
```

**3. Handle Property Changes**

```c
class MyMod_MenuController: ExpansionViewController
{
	bool ShowDetails = false;
	bool WasShowingDetails = false;
	
	override void PropertyChanged(string property_name)
	{
		if (property_name == "ShowDetails")
		{
			if (ShowDetails != WasShowingDetails)
			{
				WasShowingDetails = ShowDetails;
				
				// Get menu instance
				MyMod_Menu menu = MyMod_Menu.Cast(GetParent());
				if (menu)
				{
					menu.OnShowDetailsChanged(ShowDetails);
				}
			}
		}
	}
}
```

**4. Handle Relay Commands**

Buttons trigger controller methods via `Relay_Command`:

```c
class MyMod_MenuController: ExpansionViewController
{
	void OnCloseButtonClick()
	{
		MyMod_Menu menu = MyMod_Menu.Cast(GetParent());
		if (menu)
		{
			menu.Close();
		}
	}
	
	void OnBuyButtonClick()
	{
		MyMod_Menu menu = MyMod_Menu.Cast(GetParent());
		if (menu)
		{
			menu.ProcessPurchase();
		}
	}
}
```

## ViewBinding System

### How ViewBinding Works

1. **Layout defines binding:**
```xml
TextWidgetClass title_text {
 scriptclass "ViewBinding"
 {
  ScriptParamsClass {
   Binding_Name "Title"
  }
 }
}
```

2. **Controller defines property:**
```c
class MyMod_MenuController: ExpansionViewController
{
	string Title = "My Menu Title";
}
```

3. **System automatically syncs:**
- When `Title` changes → widget text updates
- When widget text changes (if two-way) → `Title` updates

### One-Way Binding (Default)

Controller → Widget (read-only display):

```xml
TextWidgetClass display_text {
 scriptclass "ViewBinding"
 {
  ScriptParamsClass {
   Binding_Name "DisplayText"
  }
 }
}
```

```c
class MyMod_MenuController: ExpansionViewController
{
	string DisplayText = "Hello World";
	
	void UpdateText()
	{
		DisplayText = "Updated!";  // Widget updates automatically
	}
}
```

### Two-Way Binding

Controller ↔ Widget (interactive widgets):

```xml
CheckBoxWidgetClass checkbox {
 scriptclass "ViewBinding"
 {
  ScriptParamsClass {
   Binding_Name "IsEnabled"
   Two_Way_Binding 1
  }
 }
}
```

```c
class MyMod_MenuController: ExpansionViewController
{
	bool IsEnabled = false;
	
	override void PropertyChanged(string property_name)
	{
		if (property_name == "IsEnabled")
		{
			// Called when checkbox state changes
			Print("IsEnabled: " + IsEnabled);
		}
	}
}
```

### Observable Collections

For dynamic lists (categories, items, etc.):

**Layout:**
```xml
GridSpacerWidgetClass items_container {
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

**Item Layout:**
```xml
FrameWidgetClass item_element {
 scriptclass "MyMod_ItemController"
 {
  TextWidgetClass item_name {
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

**Controller:**
```c
class MyMod_MenuController: ExpansionViewController
{
	ref ObservableCollection<ref MyMod_Item> Items = new ObservableCollection<ref MyMod_Item>(this);
	
	void AddItem(string name)
	{
		MyMod_Item item = new MyMod_Item();
		item.Name = name;
		Items.Add(item);  // Widget updates automatically
	}
	
	void RemoveItem(MyMod_Item item)
	{
		Items.Remove(item);  // Widget updates automatically
	}
}

class MyMod_Item: ObservableObject
{
	string Name = "";
	
	void MyMod_Item()
	{
		// Constructor
	}
}
```

## Menu-Controller Connection

### Menu Class Setup

**1. Define Layout File**

```c
class MyMod_Menu: ExpansionScriptViewMenu
{
	override string GetLayoutFile()
	{
		return "YourModName/GUI/layouts/my_menu.layout";
	}
}
```

**2. Define Controller Type**

```c
class MyMod_Menu: ExpansionScriptViewMenu
{
	override typename GetControllerType()
	{
		return MyMod_MenuController;
	}
}
```

**3. Get Controller Reference**

```c
class MyMod_Menu: ExpansionScriptViewMenu
{
	protected ref MyMod_MenuController m_Controller;
	
	override Widget Init()
	{
		Widget root = super.Init();
		
		// Get controller instance
		m_Controller = MyMod_MenuController.Cast(GetController());
		
		if (m_Controller)
		{
			// Initialize controller properties
			m_Controller.Title = "My Menu";
		}
		
		return root;
	}
}
```

### Controller Access Pattern

```c
class MyMod_Menu: ExpansionScriptViewMenu
{
	protected ref MyMod_MenuController m_Controller;
	
	MyMod_MenuController GetMenuController()
	{
		if (!m_Controller)
			m_Controller = MyMod_MenuController.Cast(GetController());
		
		return m_Controller;
	}
	
	void UpdateTitle(string newTitle)
	{
		MyMod_MenuController controller = GetMenuController();
		if (controller)
		{
			controller.Title = newTitle;  // Widget updates automatically
		}
	}
}
```

## Data Flow Examples

### Example 1: Simple Display

**Model (Module):**
```c
class MyMod_Module: CF_ModuleCoreManager
{
	string GetPlayerName()
	{
		PlayerBase player = GetGame().GetPlayer();
		if (player && player.GetIdentity())
			return player.GetIdentity().GetName();
		return "Unknown";
	}
}
```

**Controller:**
```c
class MyMod_MenuController: ExpansionViewController
{
	string PlayerName = "";
}
```

**Menu:**
```c
class MyMod_Menu: ExpansionScriptViewMenu
{
	protected ref MyMod_MenuController m_Controller;
	protected ref MyMod_Module m_Module;
	
	override Widget Init()
	{
		Widget root = super.Init();
		m_Controller = MyMod_MenuController.Cast(GetController());
		m_Module = MyMod_Module.Cast(CF_ModuleCoreManager.Get(MyMod_Module));
		
		// Update controller from model
		if (m_Controller && m_Module)
		{
			m_Controller.PlayerName = m_Module.GetPlayerName();
		}
		
		return root;
	}
}
```

**View (Layout):**
```xml
TextWidgetClass player_name {
 scriptclass "ViewBinding"
 {
  ScriptParamsClass {
   Binding_Name "PlayerName"
  }
 }
}
```

### Example 2: Interactive Checkbox

**Layout:**
```xml
CheckBoxWidgetClass show_details {
 scriptclass "ViewBinding"
 {
  ScriptParamsClass {
   Binding_Name "ShowDetails"
   Two_Way_Binding 1
  }
 }
}
```

**Controller:**
```c
class MyMod_MenuController: ExpansionViewController
{
	bool ShowDetails = false;
	bool WasShowingDetails = false;
	
	override void PropertyChanged(string property_name)
	{
		if (property_name == "ShowDetails")
		{
			if (ShowDetails != WasShowingDetails)
			{
				WasShowingDetails = ShowDetails;
				
				MyMod_Menu menu = MyMod_Menu.Cast(GetParent());
				if (menu)
				{
					menu.OnShowDetailsChanged(ShowDetails);
				}
			}
		}
	}
}
```

**Menu:**
```c
class MyMod_Menu: ExpansionScriptViewMenu
{
	void OnShowDetailsChanged(bool show)
	{
		// Update UI based on checkbox state
		if (m_DetailsPanel)
		{
			m_DetailsPanel.Show(show);
		}
	}
}
```

### Example 3: Dynamic List

**Layout:**
```xml
GridSpacerWidgetClass items_container {
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
class MyMod_MenuController: ExpansionViewController
{
	ref ObservableCollection<ref MyMod_Item> Items = new ObservableCollection<ref MyMod_Item>(this);
}
```

**Menu:**
```c
class MyMod_Menu: ExpansionScriptViewMenu
{
	protected ref MyMod_MenuController m_Controller;
	
	void PopulateItems()
	{
		if (!m_Controller)
			return;
		
		m_Controller.Items.Clear();
		
		// Add items from model
		array<string> itemNames = GetItemNames();
		foreach (string name : itemNames)
		{
			MyMod_Item item = new MyMod_Item(name);
			m_Controller.Items.Add(item);  // Widget updates automatically
		}
	}
}
```

## Best Practices

### 1. Separate Concerns

```c
// ✅ Good - Controller handles UI state
class MyMod_MenuController: ExpansionViewController
{
	string Title = "";
	bool IsEnabled = false;
}

// ✅ Good - Menu handles business logic
class MyMod_Menu: ExpansionScriptViewMenu
{
	void ProcessPurchase()
	{
		// Business logic here
	}
}

// ❌ Avoid - Don't mix concerns
class MyMod_MenuController: ExpansionViewController
{
	void ProcessPurchase()  // Business logic in controller
	{
		// Should be in menu class
	}
}
```

### 2. Use PropertyChanged for Reactivity

```c
// ✅ Good - React to property changes
override void PropertyChanged(string property_name)
{
	if (property_name == "ShowDetails")
	{
		MyMod_Menu menu = MyMod_Menu.Cast(GetParent());
		if (menu)
		{
			menu.OnShowDetailsChanged(ShowDetails);
		}
	}
}

// ❌ Avoid - Manual updates everywhere
void UpdateShowDetails()
{
	// Manual widget updates
}
```

### 3. Track Previous Values

```c
// ✅ Good - Prevent unnecessary updates
bool ShowDetails = false;
bool WasShowingDetails = false;

override void PropertyChanged(string property_name)
{
	if (property_name == "ShowDetails")
	{
		if (ShowDetails != WasShowingDetails)
		{
			WasShowingDetails = ShowDetails;
			// Update logic here
		}
	}
}
```

### 4. Use Observable Collections for Lists

```c
// ✅ Good - Automatic widget updates
ref ObservableCollection<ref MyMod_Item> Items = new ObservableCollection<ref MyMod_Item>(this);

// ❌ Avoid - Manual list management
array<ref MyMod_Item> Items = new array<ref MyMod_Item>;
```

### 5. Get Controller Safely

```c
// ✅ Good - Safe controller access
MyMod_MenuController GetMenuController()
{
	if (!m_Controller)
		m_Controller = MyMod_MenuController.Cast(GetController());
	
	return m_Controller;
}

// ❌ Avoid - Direct access without check
m_Controller.Title = "Title";  // May be null
```

## Summary

**MVC Pattern:**
- **Model** - Data and business logic (modules, settings)
- **View** - Visual representation (layout files)
- **Controller** - Connects View to Model (script classes)

**Controller Pattern:**
- Extend `ExpansionViewController` or `ViewController`
- Define properties bound to widgets
- Handle `PropertyChanged` for reactivity
- Handle `Relay_Command` for button clicks

**ViewBinding:**
- One-way: Controller → Widget (display)
- Two-way: Controller ↔ Widget (interactive)
- Observable Collections: Dynamic lists

**Best Practices:**
- Separate concerns (UI state vs business logic)
- Use PropertyChanged for reactivity
- Track previous values to prevent unnecessary updates
- Use Observable Collections for lists
- Get controller safely

---

**Related Guides:**
- [How Layouts Work](How-Layouts-Work.md) - Understanding layout files
- [How Invoke Works](How-Invoke-Works.md) - Module-to-menu communication
- [Market Menu Example](Examples/Market-Menu-Example.md) - Complete implementation

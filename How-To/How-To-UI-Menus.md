# How to Create UI Menus

Complete guide to creating UI menus and layouts in DayZ mods.

**Note:** This guide uses `g_Game` consistently (DayZ 1.29+ pattern). For DayZ 1.28, replace `g_Game` with `GetGame()` and remove null checks. See [Tips: g_Game vs GetGame()](../Tips/Tips-g_Game-GetGame.md) for details.

## What are UI Menus?

UI menus are scripted interfaces that display to players. They use layout files (`.layout`) for visual design and EnScript classes for logic.

## UI Menu Structure

### Menu Components

1. **Menu Class** (`UIScriptedMenu`) - Logic in **5_Mission** layer
2. **Layout File** (`.layout`) - Visual design in **gui/** folder
3. **Menu Registration** - Registered in `MissionBase.CreateScriptedMenu()`

## Menu Folder Structure

```
MyMod/
├── scripts/
│   └── 5_Mission/
│       └── YourModName/
│           └── GUI/
│               └── MyMod_CustomMenu.c
└── gui/
    └── YourModName/
        └── GUI/
            └── layouts/
                └── custom_menu.layout
```

**Important:** 
- Menu classes go in **5_Mission** layer
- Layout files go in **gui/** folder (packed into `.pbo`)

## Step 1: Define Menu ID

Define your menu ID constant (start at 10000+ to avoid conflicts):

**File:** `scripts/3_Game/YourModName/Constants.c`

```c
const int MENU_MYMOD_CUSTOM = 10000;  // Start at 10000+ for custom menus
```

**Vanilla Menu IDs** (defined in `scripts/3_Game/constants.c`):
- `MENU_INVENTORY = 11`
- `MENU_MAP = 22`
- `MENU_NOTE = 21`
- `MENU_BOOK = 23`
- `MENU_GESTURES = 25`
- ... (see constants.c for full list)

## Step 2: Create Menu Class

Create your menu class extending `UIScriptedMenu`:

**File:** `scripts/5_Mission/YourModName/GUI/MyMod_CustomMenu.c`

```c
class MyMod_CustomMenu : UIScriptedMenu
{
	protected Widget m_RootWidget;
	protected ButtonWidget m_CloseButton;
	protected TextWidget m_TitleText;
	
	override Widget Init()
	{
		// Load layout file
		if (!g_Game)
			return null;
		
		layoutRoot = g_Game.GetWorkspace().CreateWidgets("YourModName/GUI/layouts/custom_menu.layout");
		
		// Get widget references
		m_RootWidget = layoutRoot;
		m_CloseButton = ButtonWidget.Cast(layoutRoot.FindAnyWidget("btn_close"));
		m_TitleText = TextWidget.Cast(layoutRoot.FindAnyWidget("txt_title"));
		
		// Set up button handlers
		if (m_CloseButton)
		{
			m_CloseButton.m_OnClick.Insert(OnCloseButtonClick);
		}
		
		// Initialize menu data
		UpdateMenu();
		
		return layoutRoot;
	}
	
	override void OnShow()
	{
		super.OnShow();
		// Called when menu is shown
	}
	
	override void OnShow()
	{
		super.OnShow();
		
		// Setup menu when shown
		PlayerBase player = PlayerBase.Cast(g_Game.GetPlayer());
		if (player)
		{
			// Apply blur effect
			PPEffects.SetBlurMenu(0.5);
			
			// Change game focus (disable player movement)
			g_Game.GetInput().ChangeGameFocus(1);
			
			// Show UI cursor
			g_Game.GetUIManager().ShowUICursor(true);
			
			// Hide HUD
			g_Game.GetMission().GetHud().Show(false);
			
			// Disable inputs (except back button)
			TIntArray skip = { UAUIBack };
			ForceDisableInputs(true, skip);
			
			// Set focus to menu
			SetFocus(layoutRoot);
		}
	}
	
	override void OnHide()
	{
		super.OnHide();
		
		// Cleanup when hidden
		PlayerBase player = PlayerBase.Cast(g_Game.GetPlayer());
		if (player)
		{
			// Re-enable inputs
			ForceDisableInputs(false);
			
			// Remove blur
			PPEffects.SetBlurMenu(0);
			
			// Reset game focus
			g_Game.GetInput().ResetGameFocus();
			
			// Hide UI cursor
			g_Game.GetUIManager().ShowUICursor(false);
			
			// Show HUD
			g_Game.GetMission().GetHud().Show(true);
		}
		
		// Close menu
		Close();
	}
	
	// Helper: Disable/enable inputs
	static void ForceDisableInputs(bool state, inout TIntArray skipIDs = null)
	{
		if (!skipIDs)
			skipIDs = new TIntArray;
		
		skipIDs.Insert(UAUIBack);  // Always allow back button
		
		TIntArray inputIDs = new TIntArray;
		GetUApi().GetActiveInputs(inputIDs);
		
		foreach (int inputID : inputIDs)
		{
			if (skipIDs.Find(inputID) == -1)
			{
				GetUApi().GetInputByID(inputID).ForceDisable(state);
			}
		}
	}
	
	override bool OnClick(Widget w, int x, int y, int button)
	{
		if (w == m_CloseButton)
		{
			Close();
			return true;
		}
		
		return super.OnClick(w, x, y, button);
	}
	
	void OnCloseButtonClick(Widget w)
	{
		Close();
	}
	
	void UpdateMenu()
	{
		if (m_TitleText)
		{
			m_TitleText.SetText("My Custom Menu");
		}
	}
	
	override void Close()
	{
		super.Close();
		// Cleanup if needed
	}
}
```

## Step 3: Create Layout File

Create your layout file (`.layout`) in the **gui/** folder:

**File:** `gui/YourModName/GUI/layouts/custom_menu.layout`

```xml
WindowWidgetClass custom_menu {
 position 0.2 0.2
 size 0.6 0.6
 hexactpos 0
 vexactpos 0
 hexactsize 0
 vexactsize 0
 visible 1
 {
  PanelWidgetClass background {
   position 0 0
   size 1 1
   color 0.1 0.1 0.1 0.95
   {
    TextWidgetClass txt_title {
     position 0.05 0.05
     size 0.9 0.1
     text "My Custom Menu"
     fontsize 24
    }
    
    ButtonWidgetClass btn_close {
     position 0.85 0.05
     size 0.1 0.05
     text "Close"
    }
   }
  }
 }
}
```

**Layout File Location:**
- **In mod folder:** `gui/YourModName/GUI/layouts/custom_menu.layout`
- **Packed into:** `gui.pbo` (or separate GUI PBO)
- **Loaded path:** `"YourModName/GUI/layouts/custom_menu.layout"`

## Step 4: Register Menu

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

## Opening Menus

### From Action (Using CallLaterByName)

**Pattern:** Action calls mission method via `CallLaterByName` for delayed execution:

```c
class MyMod_ActionOpenMenu : ActionInteractBase
{
	override void CreateConditionComponents()
	{
		m_ConditionItem = new CCINone;
		m_ConditionTarget = new CCTCursor;
	}
	
	override bool ActionCondition(PlayerBase player, ActionTarget target, ItemBase item)
	{
		if (!target || !target.GetObject())
			return false;
		
		// Check if target is your custom entity
		MyMod_CustomEntity entity = MyMod_CustomEntity.Cast(target.GetObject());
		if (!entity)
			return false;
		
		// Don't allow opening if another menu is open
		if (IsMissionClient() && g_Game.GetUIManager() && g_Game.GetUIManager().GetMenu())
			return false;
		
		return true;
	}
	
	override void OnExecuteClient(ActionData action_data)
	{
		PlayerBase player = PlayerBase.Cast(action_data.m_Player);
		if (!player || !player.GetIdentity())
			return;
		
		// Call mission method with delay (0 = immediate)
		g_Game.GetCallQueue(CALL_CATEGORY_SYSTEM).CallLaterByName(
			g_Game.GetMission(), 
			"OpenMyModMenuDirect", 
			0, 
			false
		);
	}
}
```

**Mission Method (in modded MissionGameplay):**

```c
modded class MissionGameplay
{
	ref MyMod_CustomMenu m_CustomMenu;
	
	void OpenMyModMenuDirect()
	{
		UIScriptedMenu currentMenu = g_Game.GetUIManager().GetMenu();
		
		// Don't open if another menu is open (unless it's our menu)
		if (currentMenu && !currentMenu.IsInherited(MyMod_CustomMenu))
			return;
		
		// Toggle: if already open, close it
		if (currentMenu && currentMenu.IsInherited(MyMod_CustomMenu))
		{
			g_Game.GetUIManager().HideScriptedMenu(currentMenu);
			m_CustomMenu = null;
		}
		else
		{
			// Open menu
			m_CustomMenu = new MyMod_CustomMenu();
			g_Game.GetUIManager().ShowScriptedMenu(m_CustomMenu, null);
		}
	}
}
```

### From Action (Direct Menu Opening)

```c
class MyMod_ActionOpenMenu : ActionSingleUseBase
{
	override void OnExecuteClient(ActionData action_data)
	{
		if (!g_Game || g_Game.IsDedicatedServer())
			return;
		
		UIManager uiManager = g_Game.GetUIManager();
		if (uiManager && !uiManager.IsMenuOpen(MENU_MYMOD_CUSTOM))
		{
			uiManager.EnterScriptedMenu(MENU_MYMOD_CUSTOM, null);
		}
	}
}
```

### From Code

```c
// Open menu
void OpenCustomMenu()
{
	if (!g_Game || g_Game.IsDedicatedServer())
		return;
	
	UIManager uiManager = g_Game.GetUIManager();
	if (uiManager)
	{
		uiManager.EnterScriptedMenu(MENU_MYMOD_CUSTOM, null);
	}
}

// Check if menu is open
bool IsMenuOpen()
{
	if (!g_Game)
		return false;
	
	UIManager uiManager = g_Game.GetUIManager();
	if (uiManager)
	{
		return uiManager.IsMenuOpen(MENU_MYMOD_CUSTOM);
	}
	return false;
}

// Close menu
void CloseCustomMenu()
{
	if (!g_Game)
		return;
	
	UIManager uiManager = g_Game.GetUIManager();
	if (uiManager && uiManager.IsMenuOpen(MENU_MYMOD_CUSTOM))
	{
		uiManager.FindMenu(MENU_MYMOD_CUSTOM).Close();
	}
}
```

## Layout File Storage

### Where Layouts are Stored

**In Mod Folder:**
```
MyMod/
└── gui/
    └── YourModName/
        └── GUI/
            └── layouts/
                └── custom_menu.layout
```

**After Packing:**
- Layouts are packed into `gui.pbo` (or separate GUI PBO)
- Path in code: `"YourModName/GUI/layouts/custom_menu.layout"`

### Loading Layouts

```c
// Load layout (always check g_Game first)
if (!g_Game)
	return null;

Widget layoutRoot = g_Game.GetWorkspace().CreateWidgets("YourModName/GUI/layouts/custom_menu.layout");

// Load layout with parent
Widget childWidget = g_Game.GetWorkspace().CreateWidgets("YourModName/GUI/layouts/child.layout", parentWidget);
```

**Path Rules:**
- Path is relative to `gui/` folder root
- Use forward slashes: `"YourModName/GUI/layouts/custom_menu.layout"`
- No leading slash needed

## Connecting Layout Controls to Code

### Step 1: Find Widgets by Name

**Critical:** Widget names in code must **exactly match** the widget name in the layout file.

```c
override Widget Init()
{
	if (!g_Game)
		return null;
	
	layoutRoot = g_Game.GetWorkspace().CreateWidgets("YourModName/GUI/layouts/menu.layout");
	
	// Find widgets by name (must match layout exactly)
	m_CloseButton = ButtonWidget.Cast(layoutRoot.FindAnyWidget("BttnClose"));
	m_TitleText = TextWidget.Cast(layoutRoot.FindAnyWidget("TitleText"));
	m_PlayerList = TextListboxWidget.Cast(layoutRoot.FindAnyWidget("PlayerList"));
	
	// Always check for null!
	if (!m_CloseButton)
	{
		Print("[ERROR] BttnClose widget not found in layout!");
	}
	
	return layoutRoot;
}
```

**Layout File Widget Name:**
```xml
ButtonWidgetClass BttnClose {  <!-- Name must match: "BttnClose" -->
 text "Close"
}
```

### Step 2: Handle Button Clicks

**Method 1: Using OnClick() (Recommended)**

```c
override bool OnClick(Widget w, int x, int y, int button)
{
	super.OnClick(w, x, y, button);
	
	// Compare widget pointer
	if (w == m_CloseButton)
	{
		Close();
		return true;  // Event handled
	}
	else if (w == m_ActionButton)
	{
		OnActionButtonClick();
		return true;
	}
	
	return false;  // Event not handled
}
```

**Method 2: Using m_OnClick Callback**

```c
override Widget Init()
{
	// ... load layout ...
	
	if (m_CloseButton)
	{
		m_CloseButton.m_OnClick.Insert(OnCloseButtonClick);
	}
	
	return layoutRoot;
}

void OnCloseButtonClick(Widget w)
{
	Close();
}
```

### Step 3: Update Text Widgets

```c
// Set text
void UpdateTitle(string newTitle)
{
	if (m_TitleText)
	{
		m_TitleText.SetText(newTitle);
	}
}

// Get text (if editable)
string GetInputText()
{
	if (m_EditBox)
	{
		return m_EditBox.GetText();
	}
	return "";
}
```

### Step 4: Work with Listboxes

```c
// Add items to list
void PopulatePlayerList(array<string> players)
{
	if (!m_PlayerList)
		return;
	
	m_PlayerList.ClearItems();
	
	foreach (string playerName : players)
	{
		if (playerName != "")
		{
			m_PlayerList.AddItem(playerName, null, 0);
		}
	}
}

// Get selected item
string GetSelectedPlayer()
{
	if (!m_PlayerList)
		return "";
	
	int selectedRow = m_PlayerList.GetSelectedRow();
	if (selectedRow >= 0)
	{
		string playerName;
		m_PlayerList.GetItemText(selectedRow, 0, playerName);
		return playerName;
	}
	return "";
}

// Handle listbox selection in OnClick
override bool OnClick(Widget w, int x, int y, int button)
{
	if (w == m_PlayerList)
	{
		string selected = GetSelectedPlayer();
		Print("Selected: " + selected);
		return true;
	}
	return super.OnClick(w, x, y, button);
}
```

### Step 5: Control Widget Visibility

```c
// Show/hide widgets
void ShowPlayerList(bool show)
{
	if (m_PlayerList)
	{
		m_PlayerList.Show(show);
	}
}

// Check visibility
bool IsPlayerListVisible()
{
	if (m_PlayerList)
	{
		return m_PlayerList.IsVisible();
	}
	return false;
}
```

### Step 6: Control Widget Position and Size

```c
// Set position (relative: 0.0 to 1.0, or absolute pixels)
void SetWidgetPosition(Widget widget, float x, float y)
{
	if (widget)
	{
		widget.SetPos(x, y);
	}
}

// Set size
void SetWidgetSize(Widget widget, float width, float height)
{
	if (widget)
	{
		widget.SetSize(width, height);
	}
}

// Get current size
void GetWidgetSize(Widget widget, out float width, out float height)
{
	if (widget)
	{
		widget.GetSize(width, height);
	}
}

// Set color (ARGB format)
void SetWidgetColor(Widget widget, int color)
{
	if (widget)
	{
		widget.SetColor(color);
	}
}
```

## Menu Widget Types

Common widget types you can use in layouts:

```c
// Text display
TextWidget textWidget = TextWidget.Cast(root.FindAnyWidget("txt_name"));
textWidget.SetText("Hello World");

// Rich text (supports HTML)
RichTextWidget richText = RichTextWidget.Cast(root.FindAnyWidget("rich_text"));
richText.SetText("<b>Bold</b> text");

// Buttons
ButtonWidget button = ButtonWidget.Cast(root.FindAnyWidget("btn_action"));
// Handle clicks in OnClick() method

// Edit boxes (text input)
EditBoxWidget editBox = EditBoxWidget.Cast(root.FindAnyWidget("edit_name"));
string input = editBox.GetText();
editBox.SetText("Default text");

// Multiline edit boxes
MultilineEditBoxWidget multiEdit = MultilineEditBoxWidget.Cast(root.FindAnyWidget("multi_edit"));
multiEdit.SetText("Multi\nline\ntext");

// Listboxes
TextListboxWidget listbox = TextListboxWidget.Cast(root.FindAnyWidget("lst_items"));
listbox.AddItem("Item 1", null, 0);
listbox.AddItem("Item 2", null, 0);
listbox.ClearItems();
int selected = listbox.GetSelectedRow();
string itemText;
listbox.GetItemText(selected, 0, itemText);

// Images
ImageWidget image = ImageWidget.Cast(root.FindAnyWidget("img_icon"));
image.LoadImageFile(0, "YourModName/GUI/textures/icon.edds");

// Checkboxes
CheckBoxWidget checkbox = CheckBoxWidget.Cast(root.FindAnyWidget("chk_enable"));
bool isChecked = checkbox.IsChecked();
checkbox.SetChecked(true);

// Sliders
SliderWidget slider = SliderWidget.Cast(root.FindAnyWidget("slider_value"));
float value = slider.GetCurrent();
slider.SetCurrent(0.5);
```

## Complete Example: Simple Menu

### Menu Class

**File:** `scripts/5_Mission/YourModName/GUI/MyMod_SimpleMenu.c`

```c
class MyMod_SimpleMenu : UIScriptedMenu
{
	protected Widget m_RootWidget;
	protected ButtonWidget m_CloseButton;
	protected TextWidget m_MessageText;
	
	override Widget Init()
	{
		if (!g_Game)
			return null;
		
		layoutRoot = g_Game.GetWorkspace().CreateWidgets("YourModName/GUI/layouts/simple_menu.layout");
		
		m_RootWidget = layoutRoot;
		m_CloseButton = ButtonWidget.Cast(layoutRoot.FindAnyWidget("btn_close"));
		m_MessageText = TextWidget.Cast(layoutRoot.FindAnyWidget("txt_message"));
		
		if (m_CloseButton)
		{
			m_CloseButton.m_OnClick.Insert(OnCloseClick);
		}
		
		if (m_MessageText)
		{
			m_MessageText.SetText("Welcome to My Mod!");
		}
		
		return layoutRoot;
	}
	
	void OnCloseClick(Widget w)
	{
		Close();
	}
	
	override bool OnClick(Widget w, int x, int y, int button)
	{
		if (w == m_CloseButton)
		{
			Close();
			return true;
		}
		return super.OnClick(w, x, y, button);
	}
}
```

### Layout File

**File:** `gui/YourModName/GUI/layouts/simple_menu.layout`

```xml
WindowWidgetClass simple_menu {
 position 0.3 0.3
 size 0.4 0.4
 visible 1
 {
  PanelWidgetClass background {
   position 0 0
   size 1 1
   color 0.2 0.2 0.2 0.9
   {
    TextWidgetClass txt_message {
     position 0.1 0.2
     size 0.8 0.3
     text "Welcome"
     fontsize 20
    }
    
    ButtonWidgetClass btn_close {
     position 0.35 0.6
     size 0.3 0.15
     text "Close"
    }
   }
  }
 }
}
```

### Registration

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
				menu = new MyMod_SimpleMenu;
				break;
		}
		
		return menu;
	}
}
```

## Best Practices

### 1. Always Check for Null

```c
if (!g_Game)
	return;

UIManager uiManager = g_Game.GetUIManager();
if (uiManager && !uiManager.IsMenuOpen(MENU_MYMOD_CUSTOM))
{
	uiManager.EnterScriptedMenu(MENU_MYMOD_CUSTOM, null);
}
```

### 2. Client-Side Only

Menus are client-side only - always check:

```c
if (!g_Game || g_Game.IsDedicatedServer())
	return;

// Open menu code
```

### 3. Check if Menu is Already Open

```c
if (!g_Game)
	return;

UIManager uiManager = g_Game.GetUIManager();
if (uiManager && uiManager.IsMenuOpen(MENU_MYMOD_CUSTOM))
{
	// Menu is already open, don't open again
	return;
}
```

### 4. Clean Up Widgets

```c
override void Close()
{
	super.Close();
	
	// Clean up widget references
	if (m_RootWidget)
	{
		m_RootWidget.Unlink();
		m_RootWidget = null;
	}
}
```

### 5. Use Descriptive Widget Names

```c
// ✅ Good - Clear widget names
ButtonWidget m_CloseButton;
TextWidget m_TitleText;
EditBoxWidget m_PlayerNameInput;

// ❌ Avoid - Generic names
ButtonWidget m_Button1;
TextWidget m_Text1;
```

## Summary

**UI Menus:**
- **Menu Class:** `5_Mission` layer, extends `UIScriptedMenu`
- **Layout Files:** `gui/YourModName/GUI/layouts/` folder
- **Menu ID:** Define in `3_Game/Constants.c` (start at 10000+)
- **Registration:** `modded class MissionBase.CreateScriptedMenu()`
- **Opening:** `g_Game.GetUIManager().EnterScriptedMenu(MENU_ID, null)` (check `g_Game` for null first)

**Key Rules:**
- Menus are **client-side only**
- Always check `g_Game` for null and `!g_Game.IsDedicatedServer()` for client-side code
- Layout paths are relative to `gui/` folder
- Use forward slashes in layout paths
- Start custom menu IDs at 10000+ to avoid conflicts

---

**Related Guides:**
- [How to Create Actions](How-To-Actions.md) - Opening menus from actions
- [Script Layers Guide](Script-Layers-Guide.md) - Understanding where code belongs
- [Tips: g_Game vs GetGame()](../Tips/Tips-g_Game-GetGame.md) - Global access patterns

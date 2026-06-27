# How to Connect Layout Controls to Code

Complete guide to connecting UI layout widgets to your EnScript code.

**Note:** This guide uses `g_Game` consistently (DayZ 1.29+ pattern). For DayZ 1.28, replace `g_Game` with `GetGame()` and remove null checks. See [Tips: g_Game vs GetGame()](../Tips/Tips-g_Game-GetGame.md) for details.

## Overview

Layout files (`.layout`) define the visual structure of your UI. To make them functional, you need to:
1. Find widgets by name
2. Cast them to the correct widget type
3. Handle user interactions (clicks, input, etc.)
4. Update widget properties dynamically

## Finding Widgets

### Basic Pattern

**Critical:** Widget names in code must **exactly match** the widget name in the layout file.

```c
override Widget Init()
{
	// Load layout
	if (!g_Game)
		return null;
	
	layoutRoot = g_Game.GetWorkspace().CreateWidgets("YourModName/GUI/layouts/menu.layout");
	
	// Find widget by name (must match layout exactly)
	m_CloseButton = ButtonWidget.Cast(layoutRoot.FindAnyWidget("BttnClose"));
	
	// Always check for null!
	if (!m_CloseButton)
	{
		Print("[ERROR] BttnClose widget not found!");
		return null;
	}
	
	return layoutRoot;
}
```

**Layout File:**
```xml
ButtonWidgetClass BttnClose {  <!-- Name: "BttnClose" -->
 text "Close"
}
```

### Finding Nested Widgets

If widgets are nested, you can find them from the root:

```c
// Layout structure:
// Root
//   Panel
//     Button (BttnClose)

m_CloseButton = ButtonWidget.Cast(layoutRoot.FindAnyWidget("BttnClose"));
// FindAnyWidget searches recursively, so this works even if nested
```

### Finding Widgets from Parent

```c
// Find from specific parent
Widget panel = layoutRoot.FindAnyWidget("MainPanel");
ButtonWidget button = ButtonWidget.Cast(panel.FindAnyWidget("BttnAction"));
```

## Widget Types and Casting

### Common Widget Types

```c
// Buttons
ButtonWidget button = ButtonWidget.Cast(root.FindAnyWidget("btn_name"));

// Text display
TextWidget text = TextWidget.Cast(root.FindAnyWidget("txt_name"));

// Rich text (HTML support)
RichTextWidget richText = RichTextWidget.Cast(root.FindAnyWidget("rich_name"));

// Text input
EditBoxWidget editBox = EditBoxWidget.Cast(root.FindAnyWidget("edit_name"));

// Multiline text input
MultilineEditBoxWidget multiEdit = MultilineEditBoxWidget.Cast(root.FindAnyWidget("multi_edit"));

// Listboxes
TextListboxWidget listbox = TextListboxWidget.Cast(root.FindAnyWidget("lst_items"));

// Images
ImageWidget image = ImageWidget.Cast(root.FindAnyWidget("img_icon"));

// Checkboxes
CheckBoxWidget checkbox = CheckBoxWidget.Cast(root.FindAnyWidget("chk_enable"));

// Sliders
SliderWidget slider = SliderWidget.Cast(root.FindAnyWidget("slider_value"));

// Generic widget (base class)
Widget widget = root.FindAnyWidget("widget_name");
```

## Handling Button Clicks

### Method 1: OnClick() Handler (Recommended)

```c
class MyMod_Menu : UIScriptedMenu
{
	protected ButtonWidget m_CloseButton;
	protected ButtonWidget m_ActionButton;
	
	override Widget Init()
	{
		if (!g_Game)
		return null;
	
	layoutRoot = g_Game.GetWorkspace().CreateWidgets("YourModName/GUI/layouts/menu.layout");
		
		m_CloseButton = ButtonWidget.Cast(layoutRoot.FindAnyWidget("BttnClose"));
		m_ActionButton = ButtonWidget.Cast(layoutRoot.FindAnyWidget("BttnAction"));
		
		return layoutRoot;
	}
	
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
	
	void OnActionButtonClick()
	{
		Print("Action button clicked!");
	}
}
```

### Method 2: m_OnClick Callback

```c
override Widget Init()
{
	if (!g_Game)
		return null;
	
	layoutRoot = g_Game.GetWorkspace().CreateWidgets("YourModName/GUI/layouts/menu.layout");
	
	m_CloseButton = ButtonWidget.Cast(layoutRoot.FindAnyWidget("BttnClose"));
	
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

## Working with Text Widgets

### Setting Text

```c
// Simple text
void UpdateTitle(string newTitle)
{
	if (m_TitleText)
	{
		m_TitleText.SetText(newTitle);
	}
}

// Rich text (HTML)
void UpdateRichText(string htmlContent)
{
	if (m_RichText)
	{
		m_RichText.SetText(htmlContent);
	}
}
```

### Getting Text from Input

```c
string GetPlayerName()
{
	if (m_EditBox)
	{
		return m_EditBox.GetText();
	}
	return "";
}

// Multiline text
string GetNoteText()
{
	if (m_MultiEdit)
	{
		return m_MultiEdit.GetText();
	}
	return "";
}
```

## Working with Listboxes

### Adding Items

```c
void PopulatePlayerList(array<string> players)
{
	if (!m_PlayerList)
		return;
	
	// Clear existing items
	m_PlayerList.ClearItems();
	
	// Add items
	foreach (string playerName : players)
	{
		if (playerName != "")
		{
			m_PlayerList.AddItem(playerName, null, 0);
		}
	}
}
```

### Getting Selected Item

```c
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
```

### Handling Listbox Selection

```c
override bool OnClick(Widget w, int x, int y, int button)
{
	if (w == m_PlayerList)
	{
		string selected = GetSelectedPlayer();
		if (selected != "")
		{
			Print("Selected player: " + selected);
			OnPlayerSelected(selected);
		}
		return true;
	}
	return super.OnClick(w, x, y, button);
}
```

### Listbox Methods

```c
// Clear all items
m_PlayerList.ClearItems();

// Get item count
int count = m_PlayerList.GetNumItems();

// Remove specific item
m_PlayerList.RemoveItem(0);  // Remove first item

// Get item text from specific row/column
string itemText;
m_PlayerList.GetItemText(0, 0, itemText);  // Row 0, Column 0

// Set selected row programmatically
m_PlayerList.SelectRow(2);
```

## Working with Images

### Loading Images

```c
void LoadIcon()
{
	if (m_IconImage)
	{
		m_IconImage.LoadImageFile(0, "YourModName/GUI/icons/icon.edds");
	}
}
```

### Image Properties

```c
// Set image color (tinting)
m_IconImage.SetColor(ARGB(255, 255, 255, 255));  // White (no tint)

// Get image size
float width, height;
m_IconImage.GetSize(width, height);
```

## Working with Checkboxes

```c
// Get checkbox state
bool IsEnabled()
{
	if (m_Checkbox)
	{
		return m_Checkbox.IsChecked();
	}
	return false;
}

// Set checkbox state
void SetEnabled(bool enabled)
{
	if (m_Checkbox)
	{
		m_Checkbox.SetChecked(enabled);
	}
}

// Handle checkbox change
override bool OnChange(Widget w, int x, int y, bool finished)
{
	if (w == m_Checkbox)
	{
		bool isChecked = CheckBoxWidget.Cast(w).IsChecked();
		OnCheckboxChanged(isChecked);
		return true;
	}
	return super.OnChange(w, x, y, finished);
}
```

## Working with Sliders

```c
// Get slider value (0.0 to 1.0)
float GetSliderValue()
{
	if (m_Slider)
	{
		return m_Slider.GetCurrent();
	}
	return 0.0;
}

// Set slider value
void SetSliderValue(float value)
{
	if (m_Slider)
	{
		m_Slider.SetCurrent(value);
	}
}

// Handle slider change
override bool OnChange(Widget w, int x, int y, bool finished)
{
	if (w == m_Slider)
	{
		float value = SliderWidget.Cast(w).GetCurrent();
		OnSliderChanged(value);
		return true;
	}
	return super.OnChange(w, x, y, finished);
}
```

## Controlling Widget Visibility

```c
// Show widget
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

// Toggle visibility
void TogglePlayerList()
{
	if (m_PlayerList)
	{
		m_PlayerList.Show(!m_PlayerList.IsVisible());
	}
}
```

## Controlling Widget Position and Size

### Position (Relative: 0.0-1.0 or Absolute Pixels)

```c
// Set position (relative coordinates: 0.0 to 1.0)
void SetWidgetPosition(Widget widget, float x, float y)
{
	if (widget)
	{
		widget.SetPos(x, y);
	}
}

// Set position (absolute pixels - requires hexactpos/vexactpos in layout)
void SetWidgetPositionAbsolute(Widget widget, int x, int y)
{
	if (widget)
	{
		widget.SetPos(x, y);
	}
}
```

### Size

```c
// Set size (relative: 0.0-1.0)
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
```

### Color

```c
// Set color (ARGB format: Alpha, Red, Green, Blue)
void SetWidgetColor(Widget widget, int color)
{
	if (widget)
	{
		widget.SetColor(color);
	}
}

// Example colors
const int COLOR_RED = ARGB(255, 255, 0, 0);
const int COLOR_GREEN = ARGB(255, 0, 255, 0);
const int COLOR_BLUE = ARGB(255, 0, 0, 255);
const int COLOR_WHITE = ARGB(255, 255, 255, 255);
const int COLOR_BLACK = ARGB(255, 0, 0, 0);
const int COLOR_TRANSPARENT = ARGB(0, 0, 0, 0);
```

## Complete Example: Interactive Menu

```c
class MyMod_InteractiveMenu : UIScriptedMenu
{
	protected ButtonWidget m_CloseButton;
	protected ButtonWidget m_ActionButton;
	protected TextWidget m_TitleText;
	protected TextListboxWidget m_PlayerList;
	protected EditBoxWidget m_PlayerNameInput;
	protected CheckBoxWidget m_EnableCheckbox;
	protected SliderWidget m_VolumeSlider;
	
	override Widget Init()
	{
		if (!g_Game)
			return null;
		
		layoutRoot = g_Game.GetWorkspace().CreateWidgets("YourModName/GUI/layouts/interactive_menu.layout");
		
		// Find all widgets
		m_CloseButton = ButtonWidget.Cast(layoutRoot.FindAnyWidget("BttnClose"));
		m_ActionButton = ButtonWidget.Cast(layoutRoot.FindAnyWidget("BttnAction"));
		m_TitleText = TextWidget.Cast(layoutRoot.FindAnyWidget("TitleText"));
		m_PlayerList = TextListboxWidget.Cast(layoutRoot.FindAnyWidget("PlayerList"));
		m_PlayerNameInput = EditBoxWidget.Cast(layoutRoot.FindAnyWidget("PlayerNameInput"));
		m_EnableCheckbox = CheckBoxWidget.Cast(layoutRoot.FindAnyWidget("EnableCheckbox"));
		m_VolumeSlider = SliderWidget.Cast(layoutRoot.FindAnyWidget("VolumeSlider"));
		
		// Initialize
		if (m_TitleText)
			m_TitleText.SetText("Interactive Menu");
		
		if (m_PlayerList)
			PopulatePlayerList();
		
		return layoutRoot;
	}
	
	override bool OnClick(Widget w, int x, int y, int button)
	{
		super.OnClick(w, x, y, button);
		
		if (w == m_CloseButton)
		{
			Close();
			return true;
		}
		else if (w == m_ActionButton)
		{
			OnActionButtonClick();
			return true;
		}
		else if (w == m_PlayerList)
		{
			string selected = GetSelectedPlayer();
			if (selected != "")
				Print("Selected: " + selected);
			return true;
		}
		
		return false;
	}
	
	override bool OnChange(Widget w, int x, int y, bool finished)
	{
		if (w == m_EnableCheckbox)
		{
			bool enabled = m_EnableCheckbox.IsChecked();
			Print("Checkbox: " + enabled.ToString());
			return true;
		}
		else if (w == m_VolumeSlider)
		{
			float volume = m_VolumeSlider.GetCurrent();
			Print("Volume: " + volume.ToString());
			return true;
		}
		
		return super.OnChange(w, x, y, finished);
	}
	
	void OnActionButtonClick()
	{
		string playerName = m_PlayerNameInput.GetText();
		if (playerName != "")
		{
			Print("Action for: " + playerName);
			// Do something with playerName
		}
	}
	
	void PopulatePlayerList()
	{
		if (!m_PlayerList)
			return;
		
		m_PlayerList.ClearItems();
		m_PlayerList.AddItem("Player 1", null, 0);
		m_PlayerList.AddItem("Player 2", null, 0);
		m_PlayerList.AddItem("Player 3", null, 0);
	}
	
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
}
```

## Best Practices

### 1. Always Check for Null

```c
// ✅ Good
if (m_CloseButton)
{
	m_CloseButton.Show(true);
}

// ❌ Bad - Will crash if widget not found
m_CloseButton.Show(true);
```

### 2. Use Descriptive Widget Names

```c
// ✅ Good - Clear purpose
ButtonWidget m_CloseButton;
TextWidget m_PlayerNameText;
TextListboxWidget m_OnlinePlayersList;

// ❌ Avoid - Generic names
ButtonWidget m_Button1;
TextWidget m_Text1;
```

### 3. Match Layout Names Exactly

```c
// Layout: ButtonWidgetClass BttnClose
// Code: FindAnyWidget("BttnClose")  ✅

// Layout: ButtonWidgetClass BttnClose
// Code: FindAnyWidget("btn_close")  ❌ Won't find it!
```

### 4. Handle Events Properly

```c
override bool OnClick(Widget w, int x, int y, int button)
{
	super.OnClick(w, x, y, button);  // Always call super first
	
	if (w == m_CloseButton)
	{
		Close();
		return true;  // Return true if handled
	}
	
	return false;  // Return false if not handled
}
```

### 5. Clean Up on Close

```c
override void Close()
{
	// Clear listboxes
	if (m_PlayerList)
		m_PlayerList.ClearItems();
	
	// Reset widgets
	if (m_EditBox)
		m_EditBox.SetText("");
	
	super.Close();
}
```

## Summary

**Key Steps:**
1. **Load layout** - `CreateWidgets("path/to/layout.layout")`
2. **Find widgets** - `FindAnyWidget("WidgetName")` (name must match exactly)
3. **Cast to type** - `ButtonWidget.Cast(...)`
4. **Check for null** - Always validate widget exists
5. **Handle events** - `OnClick()`, `OnChange()`, etc.
6. **Update properties** - `SetText()`, `Show()`, `SetColor()`, etc.

**Common Widget Operations:**
- **Buttons:** Handle in `OnClick()`
- **Text:** `SetText()`, `GetText()`
- **Listboxes:** `AddItem()`, `ClearItems()`, `GetSelectedRow()`, `GetItemText()`
- **Checkboxes:** `IsChecked()`, `SetChecked()`
- **Sliders:** `GetCurrent()`, `SetCurrent()`
- **Visibility:** `Show(true/false)`, `IsVisible()`

---

**Related Guides:**
- [How to Create UI Menus](How-To-UI-Menus.md) - Complete UI menu guide
- [How to Create Actions](How-To-Actions.md) - Opening menus from actions
- [Script Layers Guide](Script-Layers-Guide.md) - Understanding where code belongs

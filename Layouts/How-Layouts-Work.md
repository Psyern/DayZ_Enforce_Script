# How Layouts Work

Complete guide to understanding DayZ layout files, widgets, and visual structure.

## What are Layout Files?

Layout files (`.layout`) are XML-based UI definition files that describe the visual structure of DayZ menus. They define:

- **Widgets** - UI elements (buttons, text, images, panels, etc.)
- **Hierarchy** - Parent-child relationships between widgets
- **Properties** - Position, size, colors, text, images, etc.
- **Bindings** - Connections to script controllers via ViewBinding

## Layout File Structure

### Basic Layout File

```xml
FrameWidgetClass MyMenu {
 position 0.1 0.1
 size 0.8 0.8
 visible 1
 scriptclass "MyMod_MenuController"
 {
  PanelWidgetClass background {
   position 0 0
   size 1 1
   color 0.1 0.1 0.1 0.95
   {
    TextWidgetClass title_text {
     position 0.05 0.05
     size 0.9 0.1
     text "My Menu Title"
     fontsize 24
     scriptclass "ViewBinding"
     {
      ScriptParamsClass {
       Binding_Name "Title"
      }
     }
    }
    
    ButtonWidgetClass close_button {
     position 0.85 0.05
     size 0.1 0.05
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

### Key Components

1. **Root Widget** - Top-level widget (FrameWidgetClass, WindowWidgetClass, etc.)
2. **scriptclass** - Controller class name attached to root widget
3. **Nested Widgets** - Child widgets inside parent widgets
4. **ViewBinding** - Widgets with `scriptclass "ViewBinding"` connect to controller
5. **ScriptParamsClass** - Parameters for ViewBinding (Binding_Name, Relay_Command)

## Widget Types

### Container Widgets

**FrameWidgetClass** - Basic container with frame
```xml
FrameWidgetClass container {
 position 0 0
 size 1 1
}
```

**PanelWidgetClass** - Panel container (no frame)
```xml
PanelWidgetClass panel {
 position 0 0
 size 1 1
 color 0.2 0.2 0.2 0.9
}
```

**WindowWidgetClass** - Window container
```xml
WindowWidgetClass window {
 position 0.2 0.2
 size 0.6 0.6
}
```

**ScrollWidgetClass** - Scrollable container
```xml
ScrollWidgetClass scroller {
 position 0 0.1
 size 1 0.8
 "Scrollbar V" 1
}
```

**GridSpacerWidgetClass** - Grid layout container
```xml
GridSpacerWidgetClass grid {
 Columns 3
 Rows 10
 Padding 5
 Margin 2
}
```

**WrapSpacerWidgetClass** - Wrapping container
```xml
WrapSpacerWidgetClass wrap {
 Padding 5
 Margin 2
 "Size To Content V" 1
}
```

### Display Widgets

**TextWidgetClass** - Text display
```xml
TextWidgetClass text_widget {
 position 0.1 0.1
 size 0.8 0.1
 text "Hello World"
 fontsize 20
 style Bold
}
```

**RichTextWidgetClass** - Rich text with HTML support
```xml
RichTextWidgetClass rich_text {
 position 0.1 0.1
 size 0.8 0.5
 text "<b>Bold</b> text"
 wrap 1
}
```

**ImageWidgetClass** - Image display
```xml
ImageWidgetClass image_widget {
 position 0.1 0.1
 size 64 64
 imageTexture "YourModName/GUI/icons/icon.edds"
 mode blend
}
```

**ItemPreviewWidgetClass** - 3D item preview
```xml
ItemPreviewWidgetClass item_preview {
 position 0.1 0.1
 size 0.3 0.5
 scriptclass "ViewBinding"
 {
  ScriptParamsClass {
   Binding_Name "ItemPreview"
  }
 }
}
```

**PlayerPreviewWidgetClass** - 3D player preview
```xml
PlayerPreviewWidgetClass player_preview {
 position 0.1 0.1
 size 0.3 0.5
 scriptclass "ViewBinding"
 {
  ScriptParamsClass {
   Binding_Name "PlayerPreview"
  }
 }
}
```

### Interactive Widgets

**ButtonWidgetClass** - Clickable button
```xml
ButtonWidgetClass button {
 position 0.1 0.1
 size 0.2 0.1
 text "Click Me"
 scriptclass "ViewBinding"
 {
  ScriptParamsClass {
   Relay_Command "OnButtonClick"
  }
 }
}
```

**EditBoxWidgetClass** - Text input
```xml
EditBoxWidgetClass edit_box {
 position 0.1 0.1
 size 0.3 0.05
 text "Enter text..."
 style ServerBrowser
}
```

**MultilineEditBoxWidgetClass** - Multiline text input
```xml
MultilineEditBoxWidgetClass multi_edit {
 position 0.1 0.1
 size 0.5 0.3
 text "Multi\nline\ntext"
}
```

**CheckBoxWidgetClass** - Checkbox
```xml
CheckBoxWidgetClass checkbox {
 position 0.1 0.1
 size 32 32
 scriptclass "ViewBinding"
 {
  ScriptParamsClass {
   Binding_Name "IsEnabled"
   Two_Way_Binding 1
  }
 }
}
```

**SliderWidgetClass** - Slider control
```xml
SliderWidgetClass slider {
 position 0.1 0.1
 size 0.5 0.05
}
```

**TextListboxWidgetClass** - Listbox
```xml
TextListboxWidgetClass listbox {
 position 0.1 0.1
 size 0.5 0.6
}
```

## Widget Properties

### Position and Size

**Relative Coordinates (0.0 to 1.0)**
```xml
position 0.1 0.1    # 10% from left, 10% from top
size 0.8 0.6        # 80% width, 60% height
```

**Absolute Coordinates (Pixels)**
```xml
hexactpos 1         # Use exact horizontal position
vexactpos 1         # Use exact vertical position
hexactsize 1        # Use exact horizontal size
vexactsize 1        # Use exact vertical size
position 100 50     # 100px from left, 50px from top
size 400 300        # 400px width, 300px height
```

**Alignment**
```xml
halign left_ref     # Horizontal: left, center_ref, right_ref
valign top_ref      # Vertical: top_ref, center_ref, bottom_ref
```

### Visibility and Interaction

```xml
visible 1           # 1 = visible, 0 = hidden
ignorepointer 1      # 1 = ignore mouse, 0 = receive mouse
disabled 0          # 1 = disabled, 0 = enabled
clipchildren 1       # 1 = clip children, 0 = allow overflow
```

### Colors

**RGBA Format (0.0 to 1.0)**
```xml
color 0.1 0.2 0.3 0.9  # Red, Green, Blue, Alpha
```

**ARGB Format (0-255)**
```xml
color ARGB(255, 255, 0, 0)  # Alpha, Red, Green, Blue
```

### Text Properties

```xml
text "Hello World"
fontsize 20
style Bold           # Normal, Bold, etc.
"text halign" center # left, center, right
"text valign" center # top, center, bottom
"exact text" 0       # 0 = auto-wrap, 1 = exact
"text offset" 5 2    # X offset, Y offset
```

### Image Properties

```xml
imageTexture "YourModName/GUI/icons/icon.edds"
image0 "set:iconset image:icon_name"  # Icon set reference
mode blend          # blend, additive, multiply
"src alpha" 1       # Source alpha
"clamp mode" clamp  # clamp, repeat
"stretch mode" stretch_w_h  # stretch_w_h, stretch_w, stretch_h
```

## ViewBinding System

### What is ViewBinding?

ViewBinding automatically syncs controller properties with widget properties. When a controller property changes, bound widgets update automatically.

### Basic ViewBinding

**Layout:**
```xml
TextWidgetClass title_text {
 scriptclass "ViewBinding"
 text "Default"
 {
  ScriptParamsClass {
   Binding_Name "Title"
  }
 }
}
```

**Controller:**
```c
class MyMod_MenuController: ExpansionViewController
{
	string Title = "My Menu Title";  // Widget text updates automatically
}
```

### Two-Way Binding

For checkboxes and other interactive widgets:

**Layout:**
```xml
CheckBoxWidgetClass checkbox {
 scriptclass "ViewBinding"
 {
  ScriptParamsClass {
   Binding_Name "IsEnabled"
   Two_Way_Binding 1  # Widget changes update controller
  }
 }
}
```

**Controller:**
```c
class MyMod_MenuController: ExpansionViewController
{
	bool IsEnabled = false;
	
	override void PropertyChanged(string property_name)
	{
		if (property_name == "IsEnabled")
		{
			// Called when checkbox state changes
			Print("IsEnabled changed to: " + IsEnabled);
		}
	}
}
```

### Relay Commands

Buttons can trigger controller methods:

**Layout:**
```xml
ButtonWidgetClass button {
 scriptclass "ViewBinding"
 {
  ScriptParamsClass {
   Relay_Command "OnButtonClick"
  }
 }
}
```

**Controller:**
```c
class MyMod_MenuController: ExpansionViewController
{
	void OnButtonClick()
	{
		Print("Button clicked!");
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
}
```

## Layout File Location

### In Mod Folder

```
MyMod/
└── gui/
    └── YourModName/
        └── GUI/
            └── layouts/
                └── my_menu.layout
```

### After Packing

- Layouts are packed into `gui.pbo` (or separate GUI PBO)
- Path in code: `"YourModName/GUI/layouts/my_menu.layout"`
- Use forward slashes: `/` not `\`

### Loading Layouts

```c
override string GetLayoutFile()
{
	return "YourModName/GUI/layouts/my_menu.layout";
}
```

## Common Patterns

### Nested Containers

```xml
FrameWidgetClass root {
 {
  PanelWidgetClass header {
   {
    TextWidgetClass title { }
   }
  }
  
  PanelWidgetClass content {
   {
    ScrollWidgetClass scroller {
     {
      GridSpacerWidgetClass items { }
     }
    }
   }
  }
  
  PanelWidgetClass footer {
   {
    ButtonWidgetClass close_button { }
   }
  }
 }
}
```

### Responsive Layout

```xml
FrameWidgetClass menu {
 position 0.1 0.1      # 10% margin
 size 0.8 0.8          # 80% of screen
 {
  PanelWidgetClass header {
   size 1 0.1          # 10% of menu height
  }
  
  PanelWidgetClass content {
   position 0 0.1
   size 1 0.8          # 80% of menu height
  }
  
  PanelWidgetClass footer {
   position 0 0.9
   size 1 0.1          # 10% of menu height
  }
 }
}
```

### Centered Content

```xml
PanelWidgetClass container {
 position 0.2 0.2
 size 0.6 0.6
 halign center_ref
 valign center_ref
 {
  TextWidgetClass centered_text {
   position 0.1 0.1
   size 0.8 0.8
   "text halign" center
   "text valign" center
  }
 }
}
```

## Best Practices

### 1. Use Descriptive Widget Names

```xml
<!-- ✅ Good -->
TextWidgetClass player_name_text { }
ButtonWidgetClass close_menu_button { }

<!-- ❌ Avoid -->
TextWidgetClass text1 { }
ButtonWidgetClass btn1 { }
```

### 2. Organize with Containers

```xml
<!-- ✅ Good - Clear hierarchy -->
PanelWidgetClass header {
 {
  TextWidgetClass title { }
  ButtonWidgetClass close { }
 }
}

<!-- ❌ Avoid - Flat structure -->
TextWidgetClass title { }
ButtonWidgetClass close { }
```

### 3. Use Relative Sizing

```xml
<!-- ✅ Good - Responsive -->
size 0.8 0.6

<!-- ⚠️ Use absolute only when needed -->
hexactsize 1
size 400 300
```

### 4. Group Related Widgets

```xml
PanelWidgetClass button_group {
 {
  ButtonWidgetClass button1 { }
  ButtonWidgetClass button2 { }
  ButtonWidgetClass button3 { }
 }
}
```

### 5. Use ViewBinding for Dynamic Content

```xml
<!-- ✅ Good - Automatic updates -->
TextWidgetClass title {
 scriptclass "ViewBinding"
 {
  ScriptParamsClass {
   Binding_Name "Title"
  }
 }
}

<!-- ❌ Avoid - Manual updates -->
TextWidgetClass title {
 text "Static Text"
}
```

## Summary

**Layout Files:**
- XML-based UI definitions
- Define widgets, hierarchy, and properties
- Use ViewBinding to connect to controllers
- Located in `gui/YourModName/GUI/layouts/`

**Key Concepts:**
- **Widgets** - UI elements (buttons, text, images, etc.)
- **ViewBinding** - Automatic property synchronization
- **Relay Commands** - Button click handlers
- **Observable Collections** - Dynamic lists

**Best Practices:**
- Use descriptive widget names
- Organize with containers
- Use relative sizing for responsiveness
- Use ViewBinding for dynamic content

---

**Related Guides:**
- [How MVC Works](How-MVC-Works.md) - Understanding controllers and ViewBinding
- [How Invoke Works](How-Invoke-Works.md) - Module-to-menu communication
- [Market Menu Example](Examples/Market-Menu-Example.md) - Complete implementation

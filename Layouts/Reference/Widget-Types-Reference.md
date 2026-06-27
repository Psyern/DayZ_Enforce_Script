# Widget Types Reference

Complete reference guide to all widget types available in DayZ layouts.

## Container Widgets

### FrameWidgetClass
Frame container with border/frame styling.

```xml
FrameWidgetClass container {
 position 0 0
 size 1 1
 style rover_sim_black
 {
  # Child widgets
 }
}
```

**Properties:**
- `style` - Frame style (rover_sim_black, UIDefaultPanel, etc.)
- `fixaspect` - Aspect ratio handling (fixwidth, fixheight, inside, outside)

### PanelWidgetClass
Panel container without frame (most common container).

```xml
PanelWidgetClass panel {
 position 0 0
 size 1 1
 color 0.1 0.1 0.1 0.9
 style blank
 {
  # Child widgets
 }
}
```

**Properties:**
- `color` - Background color (RGBA 0.0-1.0)
- `style` - Panel style (blank, UIDefaultPanel, DayZDefaultPanel, etc.)
- `draggable` - Can be dragged (0 or 1)

### WindowWidgetClass
Window container (typically for dialogs).

```xml
WindowWidgetClass window {
 position 0.2 0.2
 size 0.6 0.6
 style rover_sim_black
 {
  # Child widgets
 }
}
```

### ScrollWidgetClass
Scrollable container with scrollbars.

```xml
ScrollWidgetClass scroller {
 position 0 0.1
 size 1 0.8
 "Scrollbar V" 1      # Vertical scrollbar
 "Scrollbar H" 0      # Horizontal scrollbar
 style blank
 {
  # Scrollable content
 }
}
```

**Properties:**
- `"Scrollbar V"` - Vertical scrollbar (0 or 1)
- `"Scrollbar H"` - Horizontal scrollbar (0 or 1)

### GridSpacerWidgetClass
Grid-based layout container with rows and columns.

```xml
GridSpacerWidgetClass grid {
 Columns 3
 Rows 10
 Padding 5
 Margin 2
 "Size To Content H" 1
 "Size To Content V" 1
 content_halign center
 content_valign center
 {
  # Grid items
 }
}
```

**Properties:**
- `Columns` - Number of columns
- `Rows` - Maximum rows (or actual rows)
- `Padding` - Internal padding
- `Margin` - External margin
- `"Size To Content H"` - Auto-size width
- `"Size To Content V"` - Auto-size height
- `content_halign` - Content horizontal alignment
- `content_valign` - Content vertical alignment

### WrapSpacerWidgetClass
Wrapping container that flows content and wraps to next line.

```xml
WrapSpacerWidgetClass wrap {
 Padding 5
 Margin 2
 "Size To Content H" 1
 "Size To Content V" 1
 content_halign center
 content_valign center
 {
  # Wrapping content
 }
}
```

**Properties:**
- `Padding` - Internal padding
- `Margin` - External margin
- `"Size To Content H"` - Auto-size width
- `"Size To Content V"` - Auto-size height

### SpacerWidgetClass
Spacer container (used in some layouts).

```xml
SpacerWidgetClass spacer {
 Padding 5
 Margin 2
}
```

## Display Widgets

### TextWidgetClass
Simple text display widget.

```xml
TextWidgetClass text_widget {
 position 0.1 0.1
 size 0.8 0.1
 text "Hello World"
 fontsize 20
 style Bold
 "text halign" center
 "text valign" center
 "exact text" 0
}
```

**Properties:**
- `text` - Text content
- `fontsize` - Font size
- `style` - Text style (Normal, Bold, etc.)
- `font` - Font name (e.g., "gui/fonts/sdf_MetronBook24")
- `"text halign"` - Horizontal alignment (left, center, right)
- `"text valign"` - Vertical alignment (top, center, bottom)
- `"exact text"` - Exact text mode (0 = wrap, 1 = exact)
- `"text offset"` - Text offset (X Y)
- `"text color"` - Text color (RGBA)
- `"bold text"` - Bold text (0 or 1)
- `"italic text"` - Italic text (0 or 1)
- `"size to text h"` - Auto-size width to text
- `"size to text v"` - Auto-size height to text
- `"text sharpness"` - Text sharpness
- `"exact text size"` - Exact font size
- `"shadow offset"` - Text shadow offset

### RichTextWidgetClass
Rich text widget with HTML support.

```xml
RichTextWidgetClass rich_text {
 position 0.1 0.1
 size 0.8 0.5
 text "<b>Bold</b> text"
 wrap 1
 font "gui/fonts/etelkatextpro22"
 "exact text" 1
}
```

**Properties:**
- `text` - HTML text content
- `wrap` - Text wrapping (0 or 1)
- `font` - Font name
- `"exact text"` - Exact text mode
- `"condense whitespace"` - Condense whitespace (0 or 1)
- `"strip newlines"` - Strip newlines (0 or 1)
- `"text offset"` - Text offset

### HtmlWidgetClass
HTML widget for complex HTML content.

```xml
HtmlWidgetClass html_widget {
 position 0.1 0.1
 size 0.8 0.5
 "exact text" 1
 wrap 1
}
```

**Properties:**
- `wrap` - Text wrapping
- `"exact text"` - Exact text mode

### ImageWidgetClass
Image display widget.

```xml
ImageWidgetClass image_widget {
 position 0.1 0.1
 size 64 64
 imageTexture "YourModName/GUI/icons/icon.edds"
 image0 "set:iconset image:icon_name"
 mode blend
 "src alpha" 1
 "clamp mode" clamp
 "stretch mode" stretch_w_h
}
```

**Properties:**
- `imageTexture` - Direct texture path
- `image0` - Icon set reference ("set:iconset image:icon_name")
- `mode` - Blend mode (blend, additive, multiply)
- `"src alpha"` - Source alpha (0 or 1)
- `"clamp mode"` - Clamp mode (clamp, repeat)
- `"stretch mode"` - Stretch mode (stretch_w_h, stretch_w, stretch_h)
- `"flip u"` - Flip horizontally (0 or 1)
- `"flip v"` - Flip vertically (0 or 1)
- `filter` - Image filtering (0 or 1)
- `nocache` - Disable caching (0 or 1)
- `Mask` - Mask texture path
- `"Transition width"` - Transition width
- `fixaspect` - Aspect ratio (fixwidth, fixheight, inside, outside)

### ItemPreviewWidgetClass
3D item preview widget.

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

**Properties:**
- `scriptclass` - Usually "ViewBinding"
- `Binding_Name` - Controller property name

### PlayerPreviewWidgetClass
3D player preview widget.

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

### MapWidgetClass
Map widget for displaying 2D maps.

```xml
MapWidgetClass map {
 position 0 0
 size 1 1
 {
  # Map markers and overlays
 }
}
```

## Interactive Widgets

### ButtonWidgetClass
Clickable button widget.

```xml
ButtonWidgetClass button {
 position 0.1 0.1
 size 0.2 0.1
 text "Click Me"
 style Default
 scriptclass "ViewBinding"
 {
  ScriptParamsClass {
   Relay_Command "OnButtonClick"
  }
 }
}
```

**Properties:**
- `text` - Button text
- `style` - Button style (Default, Empty, Colorable, InventoryActionMenu, etc.)
- `text_offset` - Text offset (X Y)
- `text_halign` - Text horizontal alignment
- `text_proportion` - Text proportion
- `"text color"` - Text color (RGBA)
- `"disabled text color"` - Disabled text color
- `userID` - User ID for identification
- `scriptclass` - Script class (often "ViewBinding")
- `Relay_Command` - Method name to call on click

### CheckBoxWidgetClass
Checkbox widget.

```xml
CheckBoxWidgetClass checkbox {
 position 0.1 0.1
 size 32 32
 scriptclass "ViewBinding"
 style Expansion_04
 {
  ScriptParamsClass {
   Binding_Name "IsEnabled"
   Two_Way_Binding 1
  }
 }
}
```

**Properties:**
- `style` - Checkbox style (Expansion_04, etc.)
- `scriptclass` - Usually "ViewBinding"
- `Binding_Name` - Controller property name
- `Two_Way_Binding` - Two-way binding (0 or 1)

### EditBoxWidgetClass
Single-line text input widget.

```xml
EditBoxWidgetClass edit_box {
 position 0.1 0.1
 size 0.3 0.05
 text "Enter text..."
 style ServerBrowser
 "limit visible" 0
 "Use default text" 1
 "exact text" 1
}
```

**Properties:**
- `text` - Default/placeholder text
- `style` - Edit box style (ServerBrowser, etc.)
- `"limit visible"` - Limit visible characters (0 or 1)
- `"Use default text"` - Use default text (0 or 1)
- `"exact text"` - Exact text mode

### MultilineEditBoxWidgetClass
Multi-line text input widget.

```xml
MultilineEditBoxWidgetClass multi_edit {
 position 0.1 0.1
 size 0.5 0.3
 text "Multi\nline\ntext"
 lines 15
 "limit visible" 1
}
```

**Properties:**
- `text` - Default text
- `lines` - Number of lines
- `"limit visible"` - Limit visible lines (0 or 1)
- `"exact text"` - Exact text mode

### SliderWidgetClass
Slider control widget.

```xml
SliderWidgetClass slider {
 position 0.1 0.1
 size 0.5 0.05
}
```

**Properties:**
- Standard position/size properties
- Value range: 0.0 to 1.0

### TextListboxWidgetClass
Listbox widget for displaying lists.

```xml
TextListboxWidgetClass listbox {
 position 0.1 0.1
 size 0.5 0.6
}
```

**Properties:**
- Standard position/size properties
- Items added programmatically via code

## Specialized Widgets

### ProgressBarWidgetClass
Progress bar widget (if available).

```xml
ProgressBarWidgetClass progress_bar {
 position 0.1 0.1
 size 0.5 0.05
}
```

## Widget Properties (Common to All)

### Position and Size

```xml
position 0.1 0.1        # Relative position (0.0-1.0)
size 0.8 0.6            # Relative size (0.0-1.0)
hexactpos 1             # Use exact horizontal position
vexactpos 1             # Use exact vertical position
hexactsize 1            # Use exact horizontal size
vexactsize 1            # Use exact vertical size
```

### Alignment

```xml
halign left_ref          # left_ref, center_ref, right_ref
valign top_ref           # top_ref, center_ref, bottom_ref
```

### Visibility and Interaction

```xml
visible 1                # 1 = visible, 0 = hidden
disabled 0               # 1 = disabled, 0 = enabled
ignorepointer 1          # 1 = ignore mouse, 0 = receive mouse
clipchildren 1           # 1 = clip children, 0 = allow overflow
inheritalpha 0           # Inherit alpha from parent (0 or 1)
```

### Colors

```xml
color 0.1 0.2 0.3 0.9    # RGBA format (0.0-1.0)
```

### Priority

```xml
priority 100             # Z-order priority (higher = on top)
```

### Script Classes

```xml
scriptclass "ViewBinding"           # ViewBinding system
scriptclass "MyMod_Controller"      # Custom controller
scriptclass "SizeToChild"           # Auto-size to child
scriptclass "AutoHeightSpacer"      # Auto-height spacer
```

### Styles

Common styles:
- `blank` - No styling
- `Default` - Default style
- `Empty` - Empty style
- `Colorable` - Colorable style
- `UIDefaultPanel` - Default panel style
- `DayZDefaultPanel` - DayZ panel style
- `rover_sim_black` - Black frame style
- `rover_sim_colorable` - Colorable frame style
- `InventoryActionMenu` - Inventory action menu style
- `ServerBrowser` - Server browser style
- `Expansion_01` - Expansion style 1
- `Expansion_04` - Expansion style 4
- `Bold` - Bold text style
- `Normal` - Normal text style
- `EditorPanel` - Editor panel style

## Complete Widget List

### Container Widgets (7 types)
1. **FrameWidgetClass** - Frame container
2. **PanelWidgetClass** - Panel container
3. **WindowWidgetClass** - Window container
4. **ScrollWidgetClass** - Scrollable container
5. **GridSpacerWidgetClass** - Grid layout container
6. **WrapSpacerWidgetClass** - Wrapping container
7. **SpacerWidgetClass** - Spacer container

### Display Widgets (6 types)
8. **TextWidgetClass** - Simple text
9. **RichTextWidgetClass** - Rich text with HTML
10. **HtmlWidgetClass** - HTML widget
11. **ImageWidgetClass** - Image display
12. **ItemPreviewWidgetClass** - 3D item preview
13. **PlayerPreviewWidgetClass** - 3D player preview
14. **MapWidgetClass** - 2D map display

### Interactive Widgets (6 types)
15. **ButtonWidgetClass** - Clickable button
16. **CheckBoxWidgetClass** - Checkbox
17. **EditBoxWidgetClass** - Single-line text input
18. **MultilineEditBoxWidgetClass** - Multi-line text input
19. **SliderWidgetClass** - Slider control
20. **TextListboxWidgetClass** - Listbox

## Widget Usage Examples

### Example 1: Button with Icon

```xml
ButtonWidgetClass icon_button {
 position 0.1 0.1
 size 64 64
 style Empty
 {
  ImageWidgetClass button_icon {
   position 0 0
   size 1 1
   imageTexture "icon.edds"
   mode blend
  }
 }
}
```

### Example 2: Text with Background

```xml
PanelWidgetClass text_background {
 position 0.1 0.1
 size 0.3 0.05
 color 0.1 0.1 0.1 0.8
 {
  TextWidgetClass text {
   position 0.05 0
   size 0.9 1
   text "Label"
   "text valign" center
  }
 }
}
```

### Example 3: Image with Text Overlay

```xml
ImageWidgetClass background_image {
 position 0 0
 size 1 1
 imageTexture "background.edds"
 {
  TextWidgetClass overlay_text {
   position 0.1 0.1
   size 0.8 0.1
   text "Overlay Text"
   "text halign" center
  }
 }
}
```

### Example 4: Checkbox with Label

```xml
PanelWidgetClass checkbox_container {
 position 0.1 0.1
 size 0.3 0.05
 {
  CheckBoxWidgetClass checkbox {
   position 0 0
   size 32 32
   scriptclass "ViewBinding"
   {
    ScriptParamsClass {
     Binding_Name "IsEnabled"
     Two_Way_Binding 1
    }
   }
  }
  TextWidgetClass label {
   position 0.1 0
   size 0.8 1
   text "Enable Feature"
   "text valign" center
  }
 }
}
```

## Summary

**Total Widget Types: 20+**

**Container Widgets:** 7 types
- Frame, Panel, Window, Scroll, GridSpacer, WrapSpacer, Spacer

**Display Widgets:** 7 types
- Text, RichText, Html, Image, ItemPreview, PlayerPreview, Map

**Interactive Widgets:** 6 types
- Button, CheckBox, EditBox, MultilineEditBox, Slider, TextListbox

**Key Properties:**
- Position/size (relative or absolute)
- Alignment (halign, valign)
- Visibility (visible, disabled, ignorepointer)
- Colors (color property)
- Styles (various style options)
- Script classes (ViewBinding, custom controllers)

---

**Related Guides:**
- [How Layouts Work](../How-Layouts-Work.md) - Basic layout concepts
- [Advanced Layout Controls](../Advanced/Advanced-Layout-Controls.md) - Advanced patterns
- [How MVC Works](../How-MVC-Works.md) - Controller patterns

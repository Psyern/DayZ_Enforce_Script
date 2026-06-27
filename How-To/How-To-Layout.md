# How to Write DayZ Layout Files (.layout)

Complete guide to authoring `.layout` files for DayZ UI. Covers syntax, widget classes, properties, positioning, nesting, colors, fonts, and styles.

**Note:** This guide focuses on the `.layout` file format itself. For connecting layouts to EnScript code, see [How to Connect Layout Controls to Code](How-To-Layout-Controls.md). For creating complete UI menus, see [How to Create UI Menus](How-To-UI-Menus.md).

## Overview

Layout files (`.layout`) define the visual structure and appearance of DayZ UI elements. They use a custom text-based format (not XML/JSON) with curly-brace nesting. Each widget is defined by its class, name, properties, and optional children.

**File Location:** `gui/layouts/YourMenu.layout` (packed into `gui.pbo`)

## Basic Syntax

### Widget Declaration

```
WidgetClassName WidgetName {
 property1 value1
 property2 value2
 {
  ChildWidgetClass ChildName {
   property1 value1
  }
 }
}
```

**Rules:**
- Widget class and name on the same line, followed by `{`
- Properties are indented with **one space** (not tabs)
- Children are wrapped in an additional `{ }` block
- Closing `}` on its own line
- No semicolons, no quotes around property names (except special properties)
- String values use double quotes: `text "My Text"`
- Numeric values have no quotes: `position 0.5 0.3`

### Minimal Example

```
PanelWidgetClass MyRoot {
 visible 1
 position 0 0
 size 1 1
 {
  TextWidgetClass MyLabel {
   position 0.1 0.1
   size 0.8 0.05
   text "Hello World"
   font "gui/fonts/metron16"
  }
 }
}
```

## Coordinate System

### Relative vs Absolute Positioning

DayZ layouts support two coordinate modes controlled by flags:

| Flag | Value 0 (Default) | Value 1 |
|------|-------------------|---------|
| `hexactpos` | Position X is relative (0.0-1.0) | Position X is absolute (pixels) |
| `vexactpos` | Position Y is relative (0.0-1.0) | Position Y is absolute (pixels) |
| `hexactsize` | Width is relative (0.0-1.0) | Width is absolute (pixels) |
| `vexactsize` | Height is relative (0.0-1.0) | Height is absolute (pixels) |

**Relative coordinates** (default, recommended): Values from 0.0 to 1.0 represent percentage of the **parent widget's** size.

```
PanelWidgetClass MyPanel {
 position 0.05 0.05
 size 0.9 0.9
 hexactpos 0
 vexactpos 0
 hexactsize 0
 vexactsize 0
}
```

This panel starts at 5% from the left and top edges of its parent, and is 90% of the parent's width and height.

**Absolute coordinates** (pixels): Use when exact pixel dimensions are needed.

```
PanelWidgetClass MyPanel {
 position 50 50
 size 400 300
 hexactpos 1
 vexactpos 1
 hexactsize 1
 vexactsize 1
}
```

**Best Practice:** Always use relative coordinates (all flags = 0) for resolution-independent layouts. Most DayZ mod layouts use relative positioning exclusively.

### Position and Size

```
position X Y     // X = horizontal (0=left, 1=right), Y = vertical (0=top, 1=bottom)
size W H         // W = width, H = height (relative to parent)
```

**Coordinate reference:**
- `position 0 0` = top-left corner of parent
- `position 0.5 0.5` = center of parent (widget's top-left at center)
- `position 1 1` = bottom-right corner of parent (widget fully off-screen)
- `size 1 1` = full parent size
- `size 0.5 0.5` = half parent width, half parent height

### Positioning Children Relative to Parent

Children are always positioned relative to their parent widget:

```
PanelWidgetClass ParentPanel {
 position 0.1 0.1      // 10% from screen edges
 size 0.8 0.8          // 80% of screen
 {
  TextWidgetClass Label {
   position 0.05 0.05  // 5% from ParentPanel's top-left (NOT screen)
   size 0.9 0.1        // 90% of ParentPanel's width, 10% of its height
  }
 }
}
```

## Widget Classes

### Container Widgets

These are used for grouping and layout structure:

#### PanelWidgetClass
The most common container. Can have background color, style, and children.

```
PanelWidgetClass MainPanel {
 visible 1
 color 0.05 0.05 0.05 0.97
 position 0.05 0.05
 size 0.9 0.9
 style DayZDefaultPanel
 {
  // Children go here
 }
}
```

### Text Widgets

#### TextWidgetClass
Single-line text display.

```
TextWidgetClass TitleText {
 visible 1
 color 1 0.5 0 1
 position 0 0.01
 size 1 0.04
 text "#STR_MY_MOD_TITLE"
 font "gui/fonts/metron16bold"
 "exact text" 0
 "text halign" center
 "text valign" center
}
```

**Properties:**
- `text` - Display text. Use `#STR_KEY` for localized strings from stringtable.csv
- `font` - Font path (see Fonts section)
- `"text halign"` - Horizontal alignment: `left`, `center`, `right`
- `"text valign"` - Vertical alignment: `top`, `center`, `bottom`
- `"exact text"` - 0 = allow localization lookup, 1 = display text literally

#### MultilineTextWidgetClass
Multiline text display with word wrapping.

```
MultilineTextWidgetClass DescriptionText {
 visible 1
 color 0.7 0.7 0.7 1
 position 0.03 0.07
 size 0.94 0.4
 text "Line one\nLine two"
 font "gui/fonts/metron14"
}
```

### Input Widgets

#### EditBoxWidgetClass
Single-line text input field.

```
EditBoxWidgetClass EditName {
 position 0.36 0
 size 0.62 0.045
 style Default
 font "gui/fonts/metron16"
 text "default value"
}
```

**Note:** `text` sets the default/placeholder text. In code use `GetText()` and `SetText()`.

### Interactive Widgets

#### ButtonWidgetClass
Clickable button.

```
ButtonWidgetClass btnSave {
 visible 1
 color 0.102 1 0 1
 position 0.02 0.88
 size 0.3 0.06
 style Editor
 text "#STR_MY_MOD_SAVE"
 font "gui/fonts/metron16"
 "no focus" 1
}
```

**Properties:**
- `style Editor` - Gives the button the standard DayZ editor button look
- `"no focus"` - 1 = button does not grab focus (useful for toolbar buttons)

#### CheckBoxWidgetClass
Toggle checkbox with label text.

```
CheckBoxWidgetClass chkEnabled {
 position 0.02 0.46
 size 0.28 0.035
 text " #STR_MY_MOD_ENABLED"
 font "gui/fonts/metron14"
}
```

**Note:** The space before `#STR_` in `text " #STR_..."` adds visual padding between the checkbox and its label text. This is a common DayZ layout convention.

#### SliderWidgetClass
Value slider with min/max range.

```
SliderWidgetClass SliderVolume {
 color 0.1686 0.5804 0.6157 1
 position 0.36 0.3
 size 0.5 0.045
 minimum 0
 maximum 100
 step 1
 current 80
 "fill in" 1
}
```

**Properties:**
- `minimum` - Minimum value
- `maximum` - Maximum value
- `step` - Step increment
- `current` - Initial/default value
- `"fill in"` - 1 = fill the bar from left to current position (visual feedback)

#### XComboBoxWidgetClass
Dropdown/combo box. Items are populated in code.

```
XComboBoxWidgetClass ComboTargetType {
 position 0.24 0.12
 size 0.25 0.035
 style Default
}
```

**Note:** Items are added in code with `XComboBoxWidget.AddItem()`. The layout only defines position and size.

### List Widgets

#### TextListboxWidgetClass
Scrollable list of text items.

```
TextListboxWidgetClass ZoneListBox {
 position 0.02 0.06
 size 0.96 0.82
 hexactpos 0
 vexactpos 0
 hexactsize 0
 vexactsize 0
 style EditorListbox
 font "gui/fonts/metron14"
}
```

**Styles for listboxes:**
- `style Default` - Standard listbox appearance
- `style EditorListbox` - Editor-style listbox (slightly different visual)

## Common Properties Reference

### Universal Properties (All Widgets)

| Property | Type | Description | Example |
|----------|------|-------------|---------|
| `visible` | int | 0 = hidden, 1 = visible | `visible 1` |
| `position` | float float | X Y position (relative or absolute) | `position 0.1 0.2` |
| `size` | float float | Width Height (relative or absolute) | `size 0.8 0.05` |
| `color` | float float float float | R G B A (0.0-1.0 each) | `color 1 0.5 0 1` |
| `hexactpos` | int | 0 = relative X, 1 = absolute X | `hexactpos 0` |
| `vexactpos` | int | 0 = relative Y, 1 = absolute Y | `vexactpos 0` |
| `hexactsize` | int | 0 = relative W, 1 = absolute W | `hexactsize 0` |
| `vexactsize` | int | 0 = relative H, 1 = absolute H | `vexactsize 0` |
| `style` | string | Visual style preset | `style DayZDefaultPanel` |
| `ignorepointer` | int | 0 = accepts mouse events, 1 = pass-through | `ignorepointer 0` |
| `inheritalpha` | int | 1 = inherit alpha from parent | `inheritalpha 1` |

### Text Properties

| Property | Type | Description | Example |
|----------|------|-------------|---------|
| `text` | string | Display text (supports `#STR_` keys) | `text "#STR_MY_LABEL"` |
| `font` | string | Font path | `font "gui/fonts/metron16"` |
| `"text halign"` | string | Horizontal: `left`, `center`, `right` | `"text halign" center` |
| `"text valign"` | string | Vertical: `top`, `center`, `bottom` | `"text valign" center` |
| `"exact text"` | int | 0 = localize, 1 = literal | `"exact text" 0` |

### Slider Properties

| Property | Type | Description | Example |
|----------|------|-------------|---------|
| `minimum` | int/float | Minimum value | `minimum 0` |
| `maximum` | int/float | Maximum value | `maximum 100` |
| `step` | int/float | Step increment | `step 1` |
| `current` | int/float | Initial value | `current 50` |
| `"fill in"` | int | 1 = show fill bar | `"fill in" 1` |

### Button Properties

| Property | Type | Description | Example |
|----------|------|-------------|---------|
| `"no focus"` | int | 1 = no keyboard focus | `"no focus" 1` |

## Colors

Colors use RGBA format with float values from 0.0 to 1.0:

```
color R G B A
```

### Common Color Examples

```
// UI Colors
color 0.05 0.05 0.05 0.97    // Near-black background (slightly transparent)
color 0.08 0.08 0.08 0.9     // Dark panel background
color 0.08 0.08 0.08 1       // Dark panel (opaque)
color 0.1 0.1 0.1 0.5        // Dark semi-transparent overlay
color 0.1 0.1 0.1 0.8        // Dark panel for checkbox rows

// Text Colors
color 0.8 0.8 0.8 1          // Light gray text (standard)
color 0.7 0.7 0.7 1          // Medium gray text (labels)
color 0.6 0.6 0.6 1          // Dim gray text (secondary info)
color 0.5 0.5 0.5 1          // Dark gray text (hints/help)
color 1 1 1 1                 // White text

// Accent Colors
color 1 0.5 0 1              // Orange (titles, highlights)
color 1 0.7 0.3 1            // Warm orange (section headers)
color 0.8 0.6 0.2 1          // Gold (preview titles)
color 0.8 0.8 0.5 1          // Pale yellow (section labels)
color 0.8 0.8 0.2 1          // Yellow (status text)
color 0.5 0.8 1 1            // Light blue (info text)
color 0.5 0.8 0.5 1          // Light green (preview names)
color 0.5 0.9 0.5 1          // Bright green (selected item)

// Button Colors
color 0.102 1 0 1             // Bright green (save/confirm)
color 0 0.8 0 1               // Green (add buttons)
color 0.3 0.6 0.3 1           // Muted green (save rule)
color 0.8 0 0 1               // Red (delete buttons)
color 0.6 0.3 0.3 1           // Muted red (delete)
color 1 0.3 0.3 1             // Bright red (close button)
color 0 0.6 0.9 1             // Blue (save config)
color 0.2 0.6 0.8 1           // Light blue (action buttons)
color 0.3 0.5 0.8 1           // Medium blue (save to server)
color 0.4 0.4 0.6 1           // Muted blue (refresh)
color 0.8 0.5 0 1             // Amber (save to server)
color 0.8 0.6 0.2 1           // Gold (update buttons)
color 0.6 0.4 0.8 1           // Purple (test/preview)
color 0.5 0.3 0.7 1           // Dark purple (special actions)
color 0.6 0.2 0.8 1           // Vivid purple (links/messages)
color 0.3 0.3 0.3 1           // Gray (inactive tabs)
color 0.4 0.4 0.4 1           // Medium gray (separator lines)

// Slider Tint
color 0.1686 0.5804 0.6157 1  // Teal (standard slider fill)
color 1 0.5 0 1                // Orange (alternate slider fill)
```

## Fonts

DayZ provides built-in fonts at various sizes:

```
font "gui/fonts/metron12"         // 12pt - tiny text, hints
font "gui/fonts/metron14"         // 14pt - small text, labels, list items
font "gui/fonts/metron14bold"     // 14pt bold - small section headers
font "gui/fonts/metron16"         // 16pt - standard text, buttons, inputs
font "gui/fonts/metron16bold"     // 16pt bold - titles, section headers
font "gui/fonts/metron24bold"     // 24pt bold - large display (IDs, highlights)
```

**Usage guidelines:**
- **12pt:** Help text, fine print, secondary info
- **14pt:** List items, labels, checkboxes, small buttons, editor fields
- **14pt bold:** Sub-section titles within panels
- **16pt:** Standard UI text, buttons, input fields, combo boxes
- **16pt bold:** Panel titles, main section headers, tab titles
- **24pt bold:** Large highlight displays (e.g., sound ID preview)

## Styles

Styles are preset visual themes for widgets:

```
style DayZDefaultPanel    // Standard panel with border/background
style Editor              // Button style (editor-look button)
style EditorListbox       // Listbox with editor styling
style Default             // Default widget appearance
```

**Where to use:**
- `PanelWidgetClass` -> `style DayZDefaultPanel` (gives border and background)
- `ButtonWidgetClass` -> `style Editor` (gives clickable button appearance)
- `TextListboxWidgetClass` -> `style EditorListbox` or `style Default`
- `EditBoxWidgetClass` -> `style Default`

**Note:** Without a `style`, some widgets may not render visually (e.g., buttons without `style Editor` may appear as invisible click areas).

## Layout Patterns

### Pattern 1: Full-Screen Menu with Tab Navigation

The standard DayZ admin menu pattern: fullscreen root, centered main panel, tab bar, switchable content panels.

```
PanelWidgetClass MenuRoot {
 visible 1
 ignorepointer 0
 position 0 0
 size 1 1
 hexactpos 0
 vexactpos 0
 hexactsize 0
 vexactsize 0
 {
  PanelWidgetClass MainPanel {
   visible 1
   color 0.05 0.05 0.05 0.97
   position 0.05 0.05
   size 0.9 0.9
   style DayZDefaultPanel
   {
    TextWidgetClass TitleText {
     visible 1
     color 1 0.5 0 1
     position 0 0.01
     size 1 0.04
     text "#STR_MY_MOD_TITLE"
     font "gui/fonts/metron16bold"
     "text halign" center
     "text valign" center
    }
    PanelWidgetClass TabPanel {
     visible 1
     color 0.08 0.08 0.08 1
     position 0 0.05
     size 1 0.04
     {
      ButtonWidgetClass btnTab1 {
       visible 1
       color 1 0.5 0 1
       position 0.01 0.1
       size 0.15 0.8
       style Editor
       text "#STR_MY_MOD_TAB_1"
       font "gui/fonts/metron16"
      }
      ButtonWidgetClass btnTab2 {
       visible 1
       color 0.3 0.3 0.3 1
       position 0.17 0.1
       size 0.15 0.8
       style Editor
       text "#STR_MY_MOD_TAB_2"
       font "gui/fonts/metron16"
      }
     }
    }
    PanelWidgetClass Panel1 {
     visible 1
     position 0.01 0.1
     size 0.98 0.82
     {
      // Tab 1 content
     }
    }
    PanelWidgetClass Panel2 {
     visible 0
     position 0.01 0.1
     size 0.98 0.82
     {
      // Tab 2 content (hidden by default, shown via code)
     }
    }
    ButtonWidgetClass btnClose {
     visible 1
     color 1 0.3 0.3 1
     position 0.92 0.01
     size 0.07 0.035
     style Editor
     text "X"
     font "gui/fonts/metron16"
    }
   }
  }
 }
}
```

**Tab switching in code:**
```c
void SwitchTab(int tabIndex)
{
	if (m_Panel1) m_Panel1.Show(tabIndex == 0);
	if (m_Panel2) m_Panel2.Show(tabIndex == 1);

	// Update active tab color
	if (m_BtnTab1) m_BtnTab1.SetColor(tabIndex == 0 ? ARGB(255, 255, 128, 0) : ARGB(255, 77, 77, 77));
	if (m_BtnTab2) m_BtnTab2.SetColor(tabIndex == 1 ? ARGB(255, 255, 128, 0) : ARGB(255, 77, 77, 77));
}
```

### Pattern 2: List + Editor Split Panel

Left panel with a list, right panel with an editor for the selected item.

```
PanelWidgetClass ContentArea {
 position 0.01 0.1
 size 0.98 0.82
 {
  PanelWidgetClass ListPanel {
   visible 1
   color 0.08 0.08 0.08 0.9
   position 0 0
   size 0.25 0.95
   style DayZDefaultPanel
   {
    TextWidgetClass ListTitle {
     visible 1
     color 0.8 0.8 0.8 1
     position 0 0.01
     size 1 0.04
     text "#STR_MY_MOD_ITEM_LIST"
     font "gui/fonts/metron16bold"
     "text halign" center
    }
    TextListboxWidgetClass ItemListBox {
     position 0.02 0.06
     size 0.96 0.82
     style EditorListbox
     font "gui/fonts/metron14"
    }
    ButtonWidgetClass btnAdd {
     visible 1
     color 0 0.8 0 1
     position 0.02 0.9
     size 0.47 0.08
     style Editor
     text "#STR_MY_MOD_ADD"
     font "gui/fonts/metron16"
    }
    ButtonWidgetClass btnDelete {
     visible 1
     color 0.8 0 0 1
     position 0.51 0.9
     size 0.47 0.08
     style Editor
     text "#STR_MY_MOD_DELETE"
     font "gui/fonts/metron16"
    }
   }
  }
  PanelWidgetClass EditorPanel {
   visible 1
   color 0.08 0.08 0.08 0.9
   position 0.26 0
   size 0.74 0.95
   style DayZDefaultPanel
   {
    TextWidgetClass EditorTitle {
     visible 1
     color 0.8 0.8 0.8 1
     position 0 0.01
     size 1 0.03
     text "#STR_MY_MOD_EDITOR"
     font "gui/fonts/metron16bold"
     "text halign" center
    }
    // Editor fields go here
   }
  }
 }
}
```

### Pattern 3: Label + Input Row

Standard form row with label on the left and input on the right.

```
// Text input row
TextWidgetClass LabelName {
 position 0 0
 size 0.35 0.045
 text "#STR_MY_MOD_NAME"
 font "gui/fonts/metron16"
 "text valign" center
}
EditBoxWidgetClass EditName {
 position 0.36 0
 size 0.62 0.045
 style Default
 font "gui/fonts/metron16"
}
```

**Vertical spacing:** Increment the Y position by the row height + gap. Common row heights and gaps:

```
// Row height: 0.045, gap: 0.015, total step: 0.06
// Row 1: Y = 0.00
// Row 2: Y = 0.06
// Row 3: Y = 0.12
// Row 4: Y = 0.18
```

### Pattern 4: Label + Slider + Value Display

```
TextWidgetClass LabelVolume {
 position 0 0.3
 size 0.35 0.045
 text "#STR_MY_MOD_VOLUME"
 font "gui/fonts/metron16"
 "text valign" center
}
SliderWidgetClass SliderVolume {
 color 0.1686 0.5804 0.6157 1
 position 0.36 0.3
 size 0.5 0.045
 minimum 0
 maximum 100
 step 1
 "fill in" 1
}
TextWidgetClass VolumeValue {
 position 0.87 0.3
 size 0.12 0.045
 text "100%"
 font "gui/fonts/metron16"
 "text halign" center
 "text valign" center
}
```

### Pattern 5: Checkbox Panel Row (Toggle with Label)

Styled checkbox inside a framed panel for visual emphasis:

```
PanelWidgetClass EnabledPanel {
 color 0.1 0.1 0.1 0.8
 position 0 0.21
 size 0.98 0.055
 style DayZDefaultPanel
 {
  TextWidgetClass LabelEnabled {
   position 0.02 0
   size 0.75 1
   text "#STR_MY_MOD_ENABLED"
   font "gui/fonts/metron16"
   "text valign" center
  }
  CheckBoxWidgetClass CheckEnabled {
   position 0.85 0.15
   size 0.12 0.7
  }
 }
}
```

### Pattern 6: Inline Checkbox Row (Multiple Checkboxes)

Multiple checkboxes in a horizontal row without panel wrapping:

```
CheckBoxWidgetClass chkPrivate {
 position 0.02 0.46
 size 0.28 0.035
 text " #STR_MY_MOD_PRIVATE_SOUND"
 font "gui/fonts/metron14"
}
CheckBoxWidgetClass chkEnabled {
 position 0.34 0.46
 size 0.22 0.035
 text " #STR_MY_MOD_ENABLED"
 font "gui/fonts/metron14"
}
CheckBoxWidgetClass chkStopOutOfRange {
 position 0.6 0.46
 size 0.36 0.035
 text " #STR_MY_MOD_STOP_OUT_OF_RANGE"
 font "gui/fonts/metron14"
}
```

**Grid layout:** 3 columns at X = 0.02, 0.34, 0.6. New row increments Y by 0.06.

### Pattern 7: Section Header with Separator

```
PanelWidgetClass SeparatorLine {
 color 0.4 0.4 0.4 1
 position 0 0.36
 size 0.98 0.002
}
TextWidgetClass SectionTitle {
 visible 1
 color 0.8 0.6 0.2 1
 position 0 0.38
 size 1 0.04
 text "#STR_MY_MOD_SECTION_TITLE"
 font "gui/fonts/metron16bold"
 "text halign" center
}
```

### Pattern 8: Button Row (Save + Delete Side by Side)

```
ButtonWidgetClass btnSave {
 visible 1
 color 0.3 0.6 0.3 1
 position 0.02 0.88
 size 0.47 0.06
 style Editor
 text "#STR_MY_MOD_SAVE"
 font "gui/fonts/metron16"
}
ButtonWidgetClass btnDelete {
 visible 1
 color 0.6 0.3 0.3 1
 position 0.51 0.88
 size 0.47 0.06
 style Editor
 text "#STR_MY_MOD_DELETE"
 font "gui/fonts/metron16"
}
```

### Pattern 9: Input with Action Button and Preview

Sound ID input with test button and name preview:

```
TextWidgetClass lblSoundID {
 color 0.7 0.7 0.7 1
 position 0.02 0.27
 size 0.2 0.035
 text "#STR_MY_MOD_SOUND_ID"
 font "gui/fonts/metron14"
}
EditBoxWidgetClass editSoundID {
 position 0.24 0.27
 size 0.16 0.035
 style Default
}
ButtonWidgetClass btnTestSound {
 color 0.4 0.6 0.8 1
 position 0.42 0.27
 size 0.16 0.035
 style Editor
 text "#STR_MY_MOD_TEST"
 font "gui/fonts/metron14"
}
TextWidgetClass lblSoundName {
 color 0.5 0.8 0.5 1
 position 0.6 0.27
 size 0.36 0.035
 text ""
 font "gui/fonts/metron14"
}
```

### Pattern 10: Popup/Overlay Panel

A centered popup that overlays the main content. Initially hidden (`visible 0`), shown via code.

```
PanelWidgetClass PopupOverlay {
 visible 0
 color 0.1 0.1 0.15 0.95
 position 0.3 0.05
 size 0.4 0.85
 style DayZDefaultPanel
 {
  TextWidgetClass PopupTitle {
   visible 1
   color 0.8 0.8 0.5 1
   position 0 0.01
   size 1 0.05
   text "#STR_MY_MOD_SELECT_ITEM"
   font "gui/fonts/metron14bold"
   "text halign" center
  }
  TextListboxWidgetClass PopupList {
   visible 1
   position 0.02 0.08
   size 0.96 0.8
   style Default
   font "gui/fonts/metron14"
  }
  ButtonWidgetClass btnApply {
   visible 1
   color 0.3 0.6 0.3 1
   position 0.05 0.93
   size 0.4 0.05
   style Editor
   text "#STR_MY_MOD_APPLY"
   font "gui/fonts/metron14"
  }
  ButtonWidgetClass btnClosePopup {
   visible 1
   color 0.6 0.3 0.3 1
   position 0.55 0.93
   size 0.4 0.05
   style Editor
   text "#STR_MY_MOD_CLOSE"
   font "gui/fonts/metron14"
  }
 }
}
```

## Localized Text (stringtable.csv)

Use `#STR_KEY` references in layout `text` properties to support multiple languages:

**Layout:**
```
text "#STR_MY_MOD_SAVE"
```

**stringtable.csv:**
```csv
"STR_MY_MOD_SAVE","Save","Save","Uložit","Speichern","Сохранить","Zapisz","Salva","Mentés","Guardar","Sauvegarder","儲存","保存","Salvar","保存"
```

Column order: Key, English, English, Czech, German, Russian, Polish, Italian, Hungarian, Spanish, French, Traditional Chinese, Japanese, Portuguese, Chinese

**Best Practice:** Always use `#STR_` keys. Never hardcode text in layouts unless it is purely for development/debugging.

## Common Sizing Guidelines

### Standard Widget Heights (relative)

| Widget Type | Height | Example |
|-------------|--------|---------|
| Tab button | 0.035-0.04 | Tab bar button |
| Text label | 0.035-0.045 | Form label |
| Edit box | 0.035-0.045 | Text input field |
| Checkbox | 0.035-0.05 | Toggle option |
| Slider | 0.035-0.045 | Value slider |
| Small button | 0.035-0.045 | Inline action button |
| Standard button | 0.055-0.08 | Save/Delete/Add buttons |
| Large button | 0.1-0.15 | Prominent action buttons |
| Title text | 0.03-0.04 | Panel titles |
| Separator line | 0.002 | Visual divider |

### Row Spacing

| Spacing Type | Y Increment | Use Case |
|-------------|-------------|----------|
| Tight | 0.05-0.055 | Compact forms (small labels + inputs) |
| Standard | 0.06 | Normal form rows |
| Loose | 0.07-0.08 | Rows with more visual separation |
| Section gap | 0.06-0.08 | Between logical sections |

## Best Practices

### 1. Use Consistent Naming

```
// Prefix by type and context
TextWidgetClass lblVolume          // label
EditBoxWidgetClass editVolume      // edit box
ButtonWidgetClass btnSave          // button
CheckBoxWidgetClass chkEnabled     // checkbox
SliderWidgetClass sliderVolume     // slider
TextWidgetClass txtVolumeValue     // dynamic text display
XComboBoxWidgetClass comboType     // combo box
TextListboxWidgetClass listItems   // listbox
PanelWidgetClass panelSettings     // panel

// Title/status widgets
TextWidgetClass EditorTitle        // PascalCase for standalone titles
TextWidgetClass StatusText         // PascalCase for status displays
```

### 2. Always Set Exact Position Flags

For relative layouts, always include all four flags set to 0:

```
PanelWidgetClass MyPanel {
 position 0.1 0.1
 size 0.8 0.8
 hexactpos 0
 vexactpos 0
 hexactsize 0
 vexactsize 0
}
```

**Note:** You can omit these flags on simple child widgets (labels, inputs, checkboxes) where the default (relative) is assumed. But always include them on root and major container panels for clarity.

### 3. Use Panels for Visual Grouping

Wrap related widgets in a styled panel with a title:

```
PanelWidgetClass OptionsSection {
 color 0.1 0.1 0.1 0.5
 position 0 0.53
 size 0.98 0.37
 style DayZDefaultPanel
 {
  TextWidgetClass OptionsTitle {
   color 1 0.7 0.3 1
   position 0.02 0.02
   size 0.96 0.06
   text "#STR_MY_MOD_OPTIONS"
   font "gui/fonts/metron16bold"
  }
  // Option widgets...
 }
}
```

### 4. Keep Layouts DRY Through Code

Don't duplicate entire widget trees in the layout. If you need multiple identical panels (e.g., for different tabs), define them once and show/hide via code, or create them dynamically with `CreateWidgets()`.

### 5. Test at Multiple Resolutions

Relative coordinates look different at various aspect ratios. Test at:
- 1920x1080 (16:9)
- 2560x1440 (16:9)
- 3440x1440 (21:9 ultrawide)
- 1920x1200 (16:10)

### 6. Leave Space for Future Widgets

When designing forms, leave vertical gaps between sections. If your save button is at Y=0.88, don't pack widgets all the way to Y=0.87. Leave room for new checkboxes or fields without having to shift everything.

## Troubleshooting

### Widget Not Found in Code

- Widget name in `FindAnyWidget("name")` must **exactly match** the layout file name (case-sensitive)
- Check for typos in both layout and code
- `FindAnyWidget` searches recursively from the given root

### Widget Not Visible

- Check `visible 1` is set
- Check parent widget is also visible
- Check `size` is not 0 0
- Check widget is not positioned outside parent bounds
- Check `color` alpha is not 0 (fourth value)

### Button Not Clickable

- Ensure `style Editor` is set (buttons without style may not receive clicks)
- Check parent has `ignorepointer 0` (not 1)
- Check widget size is large enough to click
- Check no invisible widget is overlapping on top

### Text Not Showing

- Ensure `font` is set to a valid path
- Check `text` property has a value
- If using `#STR_` key, verify key exists in stringtable.csv
- Check `color` has non-zero alpha
- Check widget `size` is large enough

### Slider Shows No Fill

- Add `"fill in" 1` property
- Set `minimum` and `maximum` values
- Set initial `current` value if needed

---

**Related Guides:**
- [How to Connect Layout Controls to Code](How-To-Layout-Controls.md) - Widget binding and event handling
- [How to Create UI Menus](How-To-UI-Menus.md) - Complete menu creation workflow
- [EnScript Style Guide](../Tips/EnScript-Style-Guide.md) - Code formatting standards

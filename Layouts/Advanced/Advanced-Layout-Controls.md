# Advanced Layout Controls and UI Patterns

Complete guide to advanced layout controls, accordion patterns, and UI design techniques used in DayZ vanilla and Expansion Market layouts.

## Table of Contents

1. [Container Widgets](#container-widgets)
2. [Size To Content System](#size-to-content-system)
3. [Accordion/Collapsible Patterns](#accordioncollapsible-patterns)
4. [Dropdown Patterns](#dropdown-patterns)
5. [ScrollWidget Patterns](#scrollwidget-patterns)
6. [GridSpacer vs WrapSpacer](#gridspacer-vs-wrapspacer)
7. [Advanced Alignment](#advanced-alignment)
8. [Script Classes](#script-classes)

## Container Widgets

### GridSpacerWidgetClass

Grid-based layout container with rows and columns.

**Properties:**
```xml
GridSpacerWidgetClass container {
 Columns 3        # Number of columns
 Rows 10          # Number of rows (or max rows)
 Padding 5        # Internal padding
 Margin 2         # External margin
 "Size To Content H" 1  # Auto-size horizontally
 "Size To Content V" 1  # Auto-size vertically
 content_halign center  # Content horizontal alignment
 content_valign center  # Content vertical alignment
}
```

**Example:**
```xml
GridSpacerWidgetClass items_grid {
 Columns 2
 Rows 100
 Padding 5
 Margin 2
 "Size To Content V" 1
 {
  ButtonWidgetClass item1 { }
  ButtonWidgetClass item2 { }
  ButtonWidgetClass item3 { }
 }
}
```

### WrapSpacerWidgetClass

Wrapping container that flows content horizontally and wraps to next line.

**Properties:**
```xml
WrapSpacerWidgetClass wrap_container {
 Padding 5
 Margin 2
 "Size To Content H" 1
 "Size To Content V" 1
 content_halign center
 content_valign center
}
```

**Example:**
```xml
WrapSpacerWidgetClass tags_container {
 Padding 5
 Margin 2
 "Size To Content V" 1
 {
  TextWidgetClass tag1 { }
  TextWidgetClass tag2 { }
  TextWidgetClass tag3 { }
 }
}
```

### ScrollWidgetClass

Scrollable container with scrollbars.

**Properties:**
```xml
ScrollWidgetClass scroller {
 position 0 0.1
 size 1 0.8
 "Scrollbar V" 1      # Vertical scrollbar
 "Scrollbar H" 0      # Horizontal scrollbar (optional)
 style blank
 {
  GridSpacerWidgetClass content {
   "Size To Content V" 1
  }
 }
}
```

**Example:**
```xml
ScrollWidgetClass categories_scroller {
 position 0 0.1
 size 1 0.8
 "Scrollbar V" 1
 {
  GridSpacerWidgetClass categories_container {
   Columns 1
   Rows 100
   "Size To Content V" 1
  }
 }
}
```

## Size To Content System

The "Size To Content" system automatically resizes widgets based on their children.

### Size To Content H (Horizontal)

```xml
GridSpacerWidgetClass container {
 "Size To Content H" 1  # Auto-size width to fit content
 Columns 1
 Rows 100
}
```

### Size To Content V (Vertical)

```xml
GridSpacerWidgetClass container {
 "Size To Content V" 1  # Auto-size height to fit content
 Columns 1
 Rows 100
}
```

### Combined Size To Content

```xml
WrapSpacerWidgetClass container {
 "Size To Content H" 1  # Auto-width
 "Size To Content V" 1  # Auto-height
 Padding 5
 Margin 2
}
```

**Real-World Example (Vanilla Inventory):**
```xml
GridSpacerWidgetClass collapsible_container_root {
 "Size To Content V" 1  # Expands/collapses based on content
 Columns 1
 Rows 2
 {
  WrapSpacerWidgetClass header {
   size 1 48
   "Size To Content V" 1
  }
  GridSpacerWidgetClass body {
   visible 1
   "Size To Content H" 1
   "Size To Content V" 1
   Columns 1
   Rows 100
  }
 }
}
```

## Accordion/Collapsible Patterns

### Pattern 1: Vanilla Collapsible Container

**Structure:**
- Header (always visible)
- Body (collapsible content)
- Expand/collapse icons

**Layout:**
```xml
GridSpacerWidgetClass collapsible_container_root {
 "Size To Content V" 1
 Columns 1
 Rows 2
 {
  WrapSpacerWidgetClass header {
   size 1 48
   "Size To Content V" 1
   {
    PanelWidgetClass collapse_button {
     visible 0
     {
      ImageWidgetClass opened {
       visible 1
       image0 "set:dayz_gui image:icon_minus"
      }
      ImageWidgetClass closed {
       visible 0
       image0 "set:dayz_gui image:icon_plus"
      }
     }
    }
   }
  }
  GridSpacerWidgetClass body {
   visible 1  # Toggle this to collapse/expand
   "Size To Content V" 1
   Columns 1
   Rows 100
  }
 }
}
```

**Script Implementation:**
```c
void ToggleCollapse()
{
	if (m_BodyWidget)
	{
		bool isVisible = m_BodyWidget.IsVisible();
		m_BodyWidget.Show(!isVisible);
		
		// Update icons
		if (m_OpenedIcon)
			m_OpenedIcon.Show(!isVisible);
		if (m_ClosedIcon)
			m_ClosedIcon.Show(isVisible);
	}
}
```

### Pattern 2: Expansion Market Category Accordion

**Structure:**
- Category header with expand/collapse icons
- Category items (collapsible)

**Layout:**
```xml
GridSpacerWidgetClass ExpansionMarketCategory {
 scriptclass "ExpansionMarketMenuCategoryController"
 "Size To Content V" 1
 Columns 1
 Rows 1
 {
  PanelWidgetClass category_header_panel {
   size 1 49
   {
    ImageWidgetClass category_expand_icon {
     visible 1  # Show when collapsed
     imageTexture "arrow_64x64.edds"
     "flip v" 0
    }
    ImageWidgetClass category_collapse_icon {
     visible 0  # Show when expanded
     imageTexture "arrow_64x64.edds"
     "flip v" 1
    }
    ButtonWidgetClass category_button {
     scriptclass "ViewBinding"
     {
      ScriptParamsClass {
       Relay_Command "OnCategoryButtonClick"
      }
     }
    }
   }
  }
  WrapSpacerWidgetClass category_items {
   visible 0  # Toggle this
   position 0 50
   "Size To Content V" 1
   scriptclass "ViewBinding"
   {
    ScriptParamsClass {
     Binding_Name "MarketItems"
    }
   }
  }
 }
}
```

**Controller Implementation:**
```c
class ExpansionMarketMenuCategory: ObservableObject
{
	string CategoryName = "";
	bool IsExpanded = false;
	ref ObservableCollection<ref ExpansionMarketItem> MarketItems = new ObservableCollection<ref ExpansionMarketItem>(this);
	
	void OnCategoryButtonClick()
	{
		IsExpanded = !IsExpanded;
		
		// Update visibility in menu
		ExpansionMarketMenu menu = ExpansionMarketMenu.Cast(GetParent().GetParent());
		if (menu)
		{
			menu.OnCategoryToggle(this, IsExpanded);
		}
	}
}
```

**Menu Implementation:**
```c
void OnCategoryToggle(ExpansionMarketMenuCategory category, bool expanded)
{
	// Find category items widget
	Widget categoryElement = FindCategoryElement(category);
	if (categoryElement)
	{
		Widget itemsWidget = categoryElement.FindAnyWidget("category_items");
		if (itemsWidget)
		{
			itemsWidget.Show(expanded);
		}
		
		// Update icons
		ImageWidget expandIcon = ImageWidget.Cast(categoryElement.FindAnyWidget("category_expand_icon"));
		ImageWidget collapseIcon = ImageWidget.Cast(categoryElement.FindAnyWidget("category_collapse_icon"));
		
		if (expandIcon)
			expandIcon.Show(!expanded);
		if (collapseIcon)
			collapseIcon.Show(expanded);
	}
}
```

## Dropdown Patterns

### Pattern 1: Simple Dropdown

**Layout:**
```xml
PanelWidgetClass dropdown_selector {
 visible 0
 color 0.2667 0.2667 0.2667 1
 position 532.24896 11.44
 size 0.29067 41
 {
  TextWidgetClass dropdown_text {
   text "Select Option"
  }
  FrameWidgetClass dropdown_selector_button {
   {
    ImageWidgetClass dropdown_expand_image {
     visible 0  # Show when collapsed
     imageTexture "arrow_64x64.edds"
     "flip v" 1
    }
    ImageWidgetClass dropdown_collapse_image {
     visible 1  # Show when expanded
     imageTexture "arrow_64x64.edds"
    }
   }
  }
  ScrollWidgetClass dropdown_container {
   visible 1
   position 0.00002 1
   size 1 74.14
   "Scrollbar V" 1
   {
    GridSpacerWidgetClass dropdown_content {
     visible 0  # Toggle this
     "Size To Content V" 1
     Columns 1
     Rows 50
     scriptclass "ViewBinding"
     {
      ScriptParamsClass {
       Binding_Name "DropdownElements"
      }
     }
    }
   }
  }
 }
}
```

**Script Implementation:**
```c
void ToggleDropdown()
{
	if (!m_DropdownSelector)
		return;
	
	bool isVisible = m_DropdownSelector.IsVisible();
	
	// Toggle dropdown visibility
	m_DropdownSelector.Show(!isVisible);
	
	if (!isVisible)
	{
		// Show dropdown content
		Widget content = m_DropdownSelector.FindAnyWidget("dropdown_content");
		if (content)
			content.Show(true);
		
		// Update icons
		ImageWidget expandIcon = ImageWidget.Cast(m_DropdownSelector.FindAnyWidget("dropdown_expand_image"));
		ImageWidget collapseIcon = ImageWidget.Cast(m_DropdownSelector.FindAnyWidget("dropdown_collapse_image"));
		
		if (expandIcon)
			expandIcon.Show(false);
		if (collapseIcon)
			collapseIcon.Show(true);
	}
	else
	{
		// Hide dropdown content
		Widget content = m_DropdownSelector.FindAnyWidget("dropdown_content");
		if (content)
			content.Show(false);
		
		// Update icons
		ImageWidget expandIcon = ImageWidget.Cast(m_DropdownSelector.FindAnyWidget("dropdown_expand_image"));
		ImageWidget collapseIcon = ImageWidget.Cast(m_DropdownSelector.FindAnyWidget("dropdown_collapse_image"));
		
		if (expandIcon)
			expandIcon.Show(true);
		if (collapseIcon)
			collapseIcon.Show(false);
	}
}
```

### Pattern 2: Dropdown with Observable Collection

**Layout:**
```xml
GridSpacerWidgetClass dropdown_content {
 scriptclass "ViewBinding"
 "Size To Content V" 1
 Columns 1
 Rows 50
 {
  ScriptParamsClass {
   Binding_Name "DropdownElements"
  }
 }
}
```

**Dropdown Element Layout:**
```xml
PanelWidgetClass ExpansionMarketMenuDropdownElement {
 scriptclass "ExpansionMarketMenuDropdownElementController"
 size 1 36
 {
  TextWidgetClass dropdown_element_text {
   scriptclass "ViewBinding"
   {
    ScriptParamsClass {
     Binding_Name "Text"
    }
   }
  }
  CheckBoxWidgetClass dropdown_element_checkbox {
   scriptclass "ViewBinding"
   {
    ScriptParamsClass {
     Binding_Name "CheckBox"
     Two_Way_Binding 1
    }
   }
  }
 }
}
```

**Controller:**
```c
class ExpansionMarketMenuController: ExpansionViewController
{
	ref ObservableCollection<ref ExpansionMarketMenuDropdownElement> DropdownElements = new ObservableCollection<ref ExpansionMarketMenuDropdownElement>(this);
	
	void PopulateDropdown(array<string> options)
	{
		DropdownElements.Clear();
		
		foreach (string option : options)
		{
			ExpansionMarketMenuDropdownElement element = new ExpansionMarketMenuDropdownElement();
			element.Text = option;
			element.CheckBox = false;
			DropdownElements.Add(element);
		}
	}
}
```

## ScrollWidget Patterns

### Pattern 1: Vertical Scrolling List

```xml
ScrollWidgetClass list_scroller {
 position 0 0.1
 size 1 0.8
 "Scrollbar V" 1
 {
  GridSpacerWidgetClass list_content {
   Columns 1
   Rows 100
   "Size To Content V" 1
   {
    # Items added dynamically
   }
  }
 }
}
```

### Pattern 2: Horizontal Scrolling

```xml
ScrollWidgetClass horizontal_scroller {
 position 0.1 0
 size 0.8 0.2
 "Scrollbar H" 1
 {
  WrapSpacerWidgetClass content {
   "Size To Content H" 1
   {
    # Items flow horizontally
   }
  }
 }
}
```

### Pattern 3: Scroll Indicators

```xml
PanelWidgetClass scroll_indicator_header {
 visible 0  # Show when content scrolls up
 position 0 0
 size 1 0.1
 {
  ImageWidgetClass indicator_icon {
   imageTexture "arrow_64x64.edds"
   "flip v" 0
  }
 }
}

PanelWidgetClass scroll_indicator_footer {
 visible 0  # Show when content scrolls down
 position 0 0.9
 size 1 0.1
 {
  ImageWidgetClass indicator_icon {
   imageTexture "arrow_64x64.edds"
   "flip v" 1
  }
 }
}
```

## GridSpacer vs WrapSpacer

### When to Use GridSpacer

**Use GridSpacer for:**
- Fixed grid layouts (inventory slots, buttons grid)
- Equal-sized items in rows/columns
- Tabular data
- Predictable item sizes

**Example:**
```xml
GridSpacerWidgetClass inventory_grid {
 Columns 5
 Rows 10
 Padding 2
 {
  # 5x10 grid of inventory slots
 }
}
```

### When to Use WrapSpacer

**Use WrapSpacer for:**
- Dynamic content that wraps
- Variable-sized items
- Tags, labels, chips
- Flowing layouts

**Example:**
```xml
WrapSpacerWidgetClass tags_container {
 Padding 5
 Margin 2
 "Size To Content V" 1
 {
  # Tags wrap to next line when needed
 }
}
```

## Advanced Alignment

### Halign and Valign

```xml
WidgetClass widget {
 halign left_ref      # left_ref, center_ref, right_ref
 valign top_ref       # top_ref, center_ref, bottom_ref
}
```

### Content Alignment

```xml
GridSpacerWidgetClass container {
 content_halign center  # left, center, right
 content_valign center  # top, center, bottom
}
```

### Position with Alignment

```xml
ImageWidgetClass icon {
 position 5 0          # Offset from alignment point
 halign right_ref       # Aligns to right, then offset 5px left
 valign center_ref      # Aligns to center
}
```

## Script Classes

### ViewBinding

```xml
WidgetClass widget {
 scriptclass "ViewBinding"
 {
  ScriptParamsClass {
   Binding_Name "PropertyName"
   Two_Way_Binding 1  # Optional
  }
 }
}
```

### SizeToChild

```xml
ImageWidgetClass container {
 scriptclass "SizeToChild"  # Auto-sizes to child content
 {
  GridSpacerWidgetClass content {
   "Size To Content V" 1
  }
 }
}
```

### Custom Script Classes

```xml
FrameWidgetClass custom_widget {
 scriptclass "MyMod_CustomController"
 {
  # Custom controller handles widget behavior
 }
}
```

## Complete Accordion Example

**Layout:**
```xml
GridSpacerWidgetClass accordion_container {
 scriptclass "MyMod_AccordionController"
 "Size To Content V" 1
 Columns 1
 Rows 100
 {
  # Accordion items added dynamically
 }
}
```

**Accordion Item Layout:**
```xml
GridSpacerWidgetClass accordion_item {
 scriptclass "MyMod_AccordionItemController"
 "Size To Content V" 1
 Columns 1
 Rows 2
 {
  PanelWidgetClass accordion_header {
   size 1 50
   {
    TextWidgetClass accordion_title {
     scriptclass "ViewBinding"
     {
      ScriptParamsClass {
       Binding_Name "Title"
      }
     }
    }
    ImageWidgetClass accordion_expand_icon {
     visible 1
     imageTexture "arrow_64x64.edds"
     "flip v" 0
    }
    ImageWidgetClass accordion_collapse_icon {
     visible 0
     imageTexture "arrow_64x64.edds"
     "flip v" 1
    }
    ButtonWidgetClass accordion_toggle_button {
     scriptclass "ViewBinding"
     {
      ScriptParamsClass {
       Relay_Command "OnToggleClick"
      }
     }
    }
   }
  }
  WrapSpacerWidgetClass accordion_content {
   visible 0
   "Size To Content V" 1
   {
    # Content widgets
   }
  }
 }
}
```

**Controller:**
```c
class MyMod_AccordionItem: ObservableObject
{
	string Title = "";
	bool IsExpanded = false;
	
	void OnToggleClick()
	{
		IsExpanded = !IsExpanded;
		
		// Update visibility
		MyMod_AccordionMenu menu = MyMod_AccordionMenu.Cast(GetParent().GetParent());
		if (menu)
		{
			menu.OnAccordionItemToggle(this, IsExpanded);
		}
	}
}
```

**Menu:**
```c
void OnAccordionItemToggle(MyMod_AccordionItem item, bool expanded)
{
	Widget itemWidget = FindAccordionItemWidget(item);
	if (!itemWidget)
		return;
	
	Widget contentWidget = itemWidget.FindAnyWidget("accordion_content");
	if (contentWidget)
		contentWidget.Show(expanded);
	
	ImageWidget expandIcon = ImageWidget.Cast(itemWidget.FindAnyWidget("accordion_expand_icon"));
	ImageWidget collapseIcon = ImageWidget.Cast(itemWidget.FindAnyWidget("accordion_collapse_icon"));
	
	if (expandIcon)
		expandIcon.Show(!expanded);
	if (collapseIcon)
		collapseIcon.Show(expanded);
}
```

## Best Practices

### 1. Use Size To Content for Dynamic Content

```xml
<!-- ✅ Good - Auto-sizes to content -->
GridSpacerWidgetClass container {
 "Size To Content V" 1
 Columns 1
 Rows 100
}

<!-- ❌ Avoid - Fixed size may cut content -->
GridSpacerWidgetClass container {
 size 1 500  # Fixed height
}
```

### 2. Combine ScrollWidget with Size To Content

```xml
<!-- ✅ Good - Scrollable with auto-sizing -->
ScrollWidgetClass scroller {
 "Scrollbar V" 1
 {
  GridSpacerWidgetClass content {
   "Size To Content V" 1
  }
 }
}
```

### 3. Use Observable Collections for Dynamic Lists

```xml
<!-- ✅ Good - Automatic updates -->
GridSpacerWidgetClass container {
 scriptclass "ViewBinding"
 {
  ScriptParamsClass {
   Binding_Name "Items"
  }
 }
}
```

### 4. Toggle Visibility for Accordion/Dropdown

```xml
<!-- ✅ Good - Smooth show/hide -->
WrapSpacerWidgetClass content {
 visible 0  # Toggle this
 "Size To Content V" 1
}
```

### 5. Use Icons for Expand/Collapse States

```xml
<!-- ✅ Good - Clear visual feedback -->
ImageWidgetClass expand_icon {
 visible 1  # When collapsed
}
ImageWidgetClass collapse_icon {
 visible 0  # When expanded
}
```

## Summary

**Key Patterns:**
- **GridSpacer** - Fixed grid layouts
- **WrapSpacer** - Flowing, wrapping content
- **ScrollWidget** - Scrollable containers
- **Size To Content** - Auto-sizing based on children
- **Accordion** - Collapsible sections with headers
- **Dropdown** - Expandable selection menus

**Advanced Techniques:**
- Observable Collections for dynamic lists
- ViewBinding for automatic updates
- Icon toggling for expand/collapse states
- Scroll indicators for long content
- Content alignment and positioning

---

**Related Guides:**
- [How Layouts Work](../How-Layouts-Work.md) - Basic layout concepts
- [How MVC Works](../How-MVC-Works.md) - Controller patterns
- [Market Menu Example](../Examples/Market-Menu-Example.md) - Real-world accordion implementation

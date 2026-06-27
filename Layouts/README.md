# DayZ Layout Development Wiki

Complete guide to understanding and working with DayZ layouts, MVC architecture, and GUI scripting.

## 📚 Quick Index

### 🎨 Core Concepts
- **[How Layouts Work](How-Layouts-Work.md)** - Understanding layout files, widgets, and structure
- **[How MVC Works](How-MVC-Works.md)** - Model-View-Controller pattern in DayZ
- **[How Invoke Works](How-Invoke-Works.md)** - ScriptInvoker and communication patterns

### 🔧 Practical Guides
- **[Layout File Structure](How-Layouts-Work.md#layout-file-structure)** - Creating and organizing layout files
- **[Widget Types](How-Layouts-Work.md#widget-types)** - Available widgets and their properties
- **[ViewBinding System](How-MVC-Works.md#viewbinding-system)** - Connecting widgets to controllers
- **[Controller Pattern](How-MVC-Works.md#controller-pattern)** - Creating and using controllers
- **[ScriptInvoker Communication](How-Invoke-Works.md#scriptinvoker-communication)** - Module-to-menu communication
- **[Open Menu from Action](How-To/How-To-Open-Menu-From-Action.md)** - Connecting actions and keybinds to menus

### 📖 Examples
- **[Market Menu Example](Examples/Market-Menu-Example.md)** - Complete Market menu implementation
- **[Simple Menu Example](Examples/Simple-Menu-Example.md)** - Basic MVC menu setup

### 🎨 Advanced Patterns
- **[Advanced Layout Controls](Advanced/Advanced-Layout-Controls.md)** - Accordion, dropdowns, scroll patterns, and UI design techniques

### 📚 Reference
- **[Widget Types Reference](Reference/Widget-Types-Reference.md)** - Complete list of all widget types (20+ widgets)

## 🎯 What are Layouts?

Layouts (`.layout` files) are XML-based UI definition files that describe the visual structure of DayZ menus. They define:
- **Widgets** - UI elements (buttons, text, images, etc.)
- **Hierarchy** - Parent-child relationships between widgets
- **Properties** - Position, size, colors, text, etc.
- **Bindings** - Connections to script controllers

## 🏗️ MVC Architecture

DayZ uses a **Model-View-Controller** pattern for GUI:

- **Model** - Data and business logic (modules, settings, game state)
- **View** - Layout files (`.layout`) defining visual structure
- **Controller** - Script classes connecting View to Model

```
┌─────────┐      ┌──────────────┐      ┌──────────┐
│  Model  │◄─────│  Controller   │◄─────│   View   │
│ (Data)  │      │  (Script)    │      │ (Layout) │
└─────────┘      └──────────────┘      └──────────┘
```

## 🔄 Communication Flow

1. **Layout loads** → Creates widgets from `.layout` file
2. **Controller created** → Script class attached to layout root
3. **ViewBinding** → Widgets bind to controller properties
4. **User interaction** → Events trigger controller methods
5. **ScriptInvoker** → Modules communicate with menus

## 📁 File Structure

```
MyMod/
├── gui/
│   └── YourModName/
│       └── GUI/
│           └── layouts/
│               └── my_menu.layout          # View (Layout)
└── scripts/
    └── 5_Mission/
        └── YourModName/
            └── GUI/
                ├── MyMod_Menu.c            # View (Menu Class)
                └── MyMod_MenuController.c  # Controller
```

## 🚀 Quick Start

### 1. Create Layout File

**File:** `gui/YourModName/GUI/layouts/my_menu.layout`

```xml
FrameWidgetClass MyMenu {
 scriptclass "MyMod_MenuController"
 {
  TextWidgetClass title_text {
   scriptclass "ViewBinding"
   text "My Menu"
   {
    ScriptParamsClass {
     Binding_Name "Title"
    }
   }
  }
 }
}
```

### 2. Create Controller

**File:** `scripts/5_Mission/YourModName/GUI/MyMod_MenuController.c`

```c
class MyMod_MenuController: ExpansionViewController
{
	string Title = "My Menu";
}
```

### 3. Create Menu Class

**File:** `scripts/5_Mission/YourModName/GUI/MyMod_Menu.c`

```c
class MyMod_Menu: ExpansionScriptViewMenu
{
	protected ref MyMod_MenuController m_Controller;
	
	override string GetLayoutFile()
	{
		return "YourModName/GUI/layouts/my_menu.layout";
	}
	
	override typename GetControllerType()
	{
		return MyMod_MenuController;
	}
	
	override Widget Init()
	{
		Widget root = super.Init();
		m_Controller = MyMod_MenuController.Cast(GetController());
		return root;
	}
}
```

## 📚 Key Concepts

### Layout Files
- XML-based UI definitions
- Widget hierarchy and properties
- ViewBinding connections
- Script class assignments

### Controllers
- Extend `ExpansionViewController` or `ViewController`
- Properties bound to widgets via ViewBinding
- Handle property changes
- Manage data flow

### ScriptInvoker
- Event-based communication
- Module-to-menu messaging
- Decoupled architecture
- Callback patterns

## 🔍 Common Questions

**Q: What's the difference between a layout and a controller?**  
→ Layouts define **what** you see (visual structure). Controllers define **how** it behaves (logic).

**Q: How do widgets connect to code?**  
→ Use `ViewBinding` scriptclass with `Binding_Name` in ScriptParamsClass.

**Q: How do modules communicate with menus?**  
→ Use `ScriptInvoker` to send events from modules to menus.

**Q: How do I open a menu from an action or keybind?**  
→ [Open Menu from Action](How-To/How-To-Open-Menu-From-Action.md) - Complete guide to actions and keybinds.

**Q: Where do layout files go?**  
→ In `gui/YourModName/GUI/layouts/` folder (packed into `gui.pbo`).

**Q: Where do menu classes go?**  
→ In `scripts/5_Mission/YourModName/GUI/` folder.

**Q: What is ViewBinding?**  
→ A system that automatically syncs controller properties with widget properties.

## 📖 Related Documentation

- [DayZ Enforce Script Style Guide](../../../GIT/DAYZ_Enforce%20Script_StyleGuide/Wiki/README.md) - Complete scripting guide
- [How to Create UI Menus](../../../GIT/DAYZ_Enforce%20Script_StyleGuide/Wiki/How-To/How-To-UI-Menus.md) - Basic menu creation
- [How to Connect Layout Controls](../../../GIT/DAYZ_Enforce%20Script_StyleGuide/Wiki/How-To/How-To-Layout-Controls.md) - Widget connection guide

---

**Focus:** This wiki teaches you how layouts, MVC, and GUI scripting work in DayZ.

**Last Updated:** 2025-01-02

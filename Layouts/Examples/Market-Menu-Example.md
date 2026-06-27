# Market Menu Example

Complete example of the Expansion Market menu implementation demonstrating advanced MVC patterns, ScriptInvoker communication, and complex ViewBinding.

## Overview

This example demonstrates:
- Complex layout with nested widgets
- Controller with multiple properties and Observable Collections
- Module-to-menu communication via ScriptInvoker
- Two-way binding for interactive widgets
- Dynamic lists with Observable Collections
- Property change handling

## Architecture

```
┌─────────────────────┐
│ ExpansionMarketModule│ (Model - 4_World)
│  - Business Logic    │
│  - ScriptInvokers    │
└──────────┬──────────┘
           │ ScriptInvoker
           │ (SI_SetTraderInvoker)
           │ (SI_SelectedItemUpdatedInvoker)
           │ (SI_Callback)
           ▼
┌─────────────────────┐
│ ExpansionMarketMenu  │ (View - 5_Mission)
│  - Layout File       │
│  - Widget Management │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│ExpansionMarketMenu   │ (Controller)
│     Controller       │
│  - Properties        │
│  - ViewBinding       │
└─────────────────────┘
```

## Module: ScriptInvoker Setup

**File:** `scripts/4_World/DayZExpansion_Market/Systems/Market/ExpansionMarketModule.c`

```c
class ExpansionMarketModule: CF_ModuleCoreManager
{
	// Static ScriptInvokers for module-to-menu communication
	static ref ScriptInvoker SI_SetTraderInvoker = new ScriptInvoker();
	static ref ScriptInvoker SI_SelectedItemUpdatedInvoker = new ScriptInvoker();
	static ref ScriptInvoker SI_Callback = new ScriptInvoker();
	static ref ScriptInvoker SI_ATMMenuInvoker = new ScriptInvoker();
	
	void ExpansionMarketModule()
	{
		EnableInvokeConnect();
	}
	
	// Invoke when trader is set
	void SetTrader(ExpansionTraderObjectBase trader, bool force = false)
	{
		SI_SetTraderInvoker.Invoke(trader, force);
	}
	
	// Invoke when item is updated from network
	void UpdateSelectedItem(ExpansionMarketItem item)
	{
		SI_SelectedItemUpdatedInvoker.Invoke(item);
	}
	
	// Invoke callback from async operations
	void InvokeCallback(string itemClassName, int result, int option1, int option2, Object object)
	{
		SI_Callback.Invoke(itemClassName, result, option1, option2, object);
	}
}
```

## Controller: Properties and Bindings

**File:** `scripts/5_Mission/DayZExpansion_Market/Market/ExpansionMarketMenu.c` (Controller class)

```c
class ExpansionMarketMenuController: ExpansionViewController
{
	// Observable Collections for dynamic lists
	ref ObservableCollection<ref ExpansionMarketMenuCategory> MarketCategories = new ObservableCollection<ref ExpansionMarketMenuCategory>(this);
	ref ObservableCollection<ref ExpansionMarketMenuDropdownElement> DropdownElements = new ObservableCollection<ref ExpansionMarketMenuDropdownElement>(this);
	ref ObservableCollection<ref ExpansionMarketMenuSkinsDropdownElement> SkinsDropdownElements = new ObservableCollection<ref ExpansionMarketMenuSkinsDropdownElement>(this);
	
	// String properties
	string MarketName;
	string PlayerTotalMoney;
	string MarketItemName;
	string MarketItemDesc;
	string MarketItemTotalSellPrice;
	string MarketItemTotalBuyPrice;
	string MarketItemStockPlayer;
	string MarketItemStockTrader;
	string MarketQuantity;
	string MarketIcon;
	string MarketItemInfoName;
	string CurrencySellIcon;
	string CurrencyBuyIcon;
	
	// Object properties (for previews)
	Object MarketItemPreview;
	Object MarketPlayerPreview;
	
	// Boolean properties (two-way binding)
	bool ShowSellables;
	bool WasShowingSellables;
	bool ShowPurchasables;
	bool WasShowingPurchasables;
	bool IncludeAttachments = true;
	bool WasIncludingAttachments = true;
	
	// Handle property changes
	override void PropertyChanged(string property_name)
	{
		ExpansionMarketMenu menu = ExpansionMarketMenu.Cast(GetParent());
		if (!menu)
			return;
		
		if (property_name == "ShowSellables")
		{
			if (ShowSellables != WasShowingSellables)
			{
				WasShowingSellables = ShowSellables;
				GetExpansionClientSettings().MarketMenuFilterSellableState = ShowSellables;
				menu.SetFilterSellables(ShowSellables);
				menu.UpdateOptionFilterStrings();
				menu.UpdateMarketCategories();
				menu.UpdatePreview();
				menu.UpdateSkinSelectorVisibility();
			}
		}
		else if (property_name == "ShowPurchasables")
		{
			if (ShowPurchasables != WasShowingPurchasables)
			{
				WasShowingPurchasables = ShowPurchasables;
				GetExpansionClientSettings().MarketMenuFilterPurchasableState = ShowPurchasables;
				menu.SetFilterPurchasables(ShowPurchasables);
				menu.UpdateOptionFilterStrings();
				menu.UpdateMarketCategories();
				menu.UpdatePreview();
				menu.UpdateSkinSelectorVisibility();
			}
		}
		else if (property_name == "IncludeAttachments")
		{
			if (IncludeAttachments != WasIncludingAttachments)
			{
				WasIncludingAttachments = IncludeAttachments;
				menu.OnAttachmentsCheckboxStateChange();
			}
		}
	}
}
```

## Menu: ScriptInvoker Subscriptions

**File:** `scripts/5_Mission/DayZExpansion_Market/Market/ExpansionMarketMenu.c` (Menu class)

```c
class ExpansionMarketMenu: ExpansionScriptViewMenu
{
	protected ref ExpansionMarketMenuController m_MarketMenuController;
	protected ref ExpansionMarketModule m_MarketModule;
	
	override string GetLayoutFile()
	{
		return "DayZExpansion/Market/GUI/layouts/market/expansion_market_menu.layout";
	}
	
	override typename GetControllerType()
	{
		return ExpansionMarketMenuController;
	}
	
	override Widget Init()
	{
		Widget root = super.Init();
		
		// Get controller instance
		if (!m_MarketMenuController)
			m_MarketMenuController = ExpansionMarketMenuController.Cast(GetController());
		
		// Get module reference
		if (!m_MarketModule)
			m_MarketModule = ExpansionMarketModule.Cast(CF_ModuleCoreManager.Get(ExpansionMarketModule));
		
		// Subscribe to ScriptInvoker events
		if (m_MarketModule)
		{
			m_MarketModule.SI_SetTraderInvoker.Insert(SetTraderObject);
			m_MarketModule.SI_SelectedItemUpdatedInvoker.Insert(OnNetworkItemUpdate);
			m_MarketModule.SI_Callback.Insert(MenuCallback);
		}
		
		// Initialize controller properties
		if (m_MarketMenuController)
		{
			m_MarketMenuController.MarketName = "UNKNOWN TRADER";
			m_MarketMenuController.PlayerTotalMoney = "0";
		}
		
		return root;
	}
	
	// Handler: Trader object set from module
	void SetTraderObject(ExpansionTraderObjectBase trader, bool force)
	{
		m_TraderObject = trader;
		if (m_TraderObject)
		{
			m_TraderMarket = m_TraderObject.GetMarket();
			
			// Update controller properties (widgets update automatically)
			if (m_MarketMenuController)
			{
				m_MarketMenuController.MarketName = m_TraderMarket.GetName();
				m_MarketMenuController.MarketIcon = m_TraderMarket.GetIcon();
			}
			
			// Update UI
			UpdateTraderUI();
			UpdateMarketCategories();
		}
	}
	
	// Handler: Item updated from network
	void OnNetworkItemUpdate(ExpansionMarketItem item)
	{
		if (item)
		{
			m_SelectedMarketItem = item;
			
			// Update controller properties
			if (m_MarketMenuController)
			{
				m_MarketMenuController.MarketItemName = item.GetDisplayName();
				m_MarketMenuController.MarketItemDesc = item.GetDescription();
				m_MarketMenuController.MarketItemPreview = item.GetItem();
			}
			
			// Update UI
			UpdateItemUI();
		}
	}
	
	// Handler: Callback from async operations
	void MenuCallback(string itemClassName, int result, int option1, int option2, Object object)
	{
		if (result == 1)
		{
			Print("Operation successful: " + itemClassName);
			// Update UI on success
		}
		else
		{
			Print("Operation failed: " + itemClassName);
			// Show error message
		}
	}
	
	override void Close()
	{
		// Unsubscribe from ScriptInvoker events
		if (m_MarketModule)
		{
			m_MarketModule.SI_SetTraderInvoker.Remove(SetTraderObject);
			m_MarketModule.SI_SelectedItemUpdatedInvoker.Remove(OnNetworkItemUpdate);
			m_MarketModule.SI_Callback.Remove(MenuCallback);
		}
		
		super.Close();
	}
}
```

## Layout: ViewBinding Examples

**File:** `gui/DayZExpansion/Market/GUI/layouts/market/expansion_market_menu.layout`

### Example 1: Simple Text Binding

```xml
TextWidgetClass market_text {
 scriptclass "ViewBinding"
 style Bold
 text "UNKNOWN TRADER"
 {
  ScriptParamsClass {
   Binding_Name "MarketName"
  }
 }
}
```

### Example 2: Two-Way Checkbox Binding

```xml
CheckBoxWidgetClass market_item_sellables_checkbox {
 scriptclass "ViewBinding"
 style Expansion_04
 {
  ScriptParamsClass {
   Binding_Name "ShowSellables"
   Two_Way_Binding 1
  }
 }
}
```

### Example 3: Observable Collection Binding

```xml
GridSpacerWidgetClass market_categories_container {
 scriptclass "ViewBinding"
 Padding 0
 Margin 0
 "Size To Content H" 1
 "Size To Content V" 1
 Columns 1
 Rows 100
 {
  ScriptParamsClass {
   Binding_Name "MarketCategories"
  }
 }
}
```

### Example 4: Relay Command Binding

```xml
ButtonWidgetClass market_filter_clear {
 scriptclass "ViewBinding"
 style Empty
 {
  ScriptParamsClass {
   Relay_Command "OnFilterButtonClick"
  }
 }
}
```

### Example 5: Object Preview Binding

```xml
ItemPreviewWidgetClass market_item_preview {
 scriptclass "ViewBinding"
 {
  ScriptParamsClass {
   Binding_Name "MarketItemPreview"
  }
 }
}
```

## Category Element Example

**File:** `gui/DayZExpansion/Market/GUI/layouts/market/expansion_market_menu_category_element.layout`

```xml
FrameWidgetClass category_element {
 scriptclass "ExpansionMarketMenuCategoryController"
 {
  ButtonWidgetClass category_button {
   scriptclass "ViewBinding"
   {
    ScriptParamsClass {
     Relay_Command "OnCategoryButtonClick"
    }
   }
  }
  
  TextWidgetClass category_name {
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

**Category Controller:**

```c
class ExpansionMarketMenuCategory: ObservableObject
{
	string Name = "";
	bool IsSelected = false;
	
	void ExpansionMarketMenuCategory(string name)
	{
		Name = name;
	}
	
	void OnCategoryButtonClick()
	{
		ExpansionMarketMenu menu = ExpansionMarketMenu.Cast(GetParent().GetParent());
		if (menu)
		{
			menu.OnCategorySelected(this);
		}
	}
}
```

## Key Patterns Demonstrated

### 1. Module → Menu Communication

```c
// Module invokes event
ExpansionMarketModule.SI_SetTraderInvoker.Invoke(trader, true);

// Menu subscribes and handles
m_MarketModule.SI_SetTraderInvoker.Insert(SetTraderObject);
```

### 2. Property Change Handling

```c
override void PropertyChanged(string property_name)
{
	if (property_name == "ShowSellables")
	{
		// React to property change
		menu.SetFilterSellables(ShowSellables);
	}
}
```

### 3. Observable Collections

```c
// Controller defines collection
ref ObservableCollection<ref ExpansionMarketMenuCategory> MarketCategories = new ObservableCollection<ref ExpansionMarketMenuCategory>(this);

// Menu adds items
MarketCategories.Add(new ExpansionMarketMenuCategory("Weapons"));

// Widget updates automatically
```

### 4. Two-Way Binding

```xml
CheckBoxWidgetClass checkbox {
 scriptclass "ViewBinding"
 {
  ScriptParamsClass {
   Binding_Name "ShowSellables"
   Two_Way_Binding 1
  }
 }
}
```

### 5. Relay Commands

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

## Summary

**Market Menu Architecture:**
- **Module** - Business logic and ScriptInvokers
- **Menu** - UI management and ScriptInvoker subscriptions
- **Controller** - Properties and ViewBinding
- **Layout** - Widget structure with ViewBinding

**Key Features:**
- ScriptInvoker for module-to-menu communication
- Observable Collections for dynamic lists
- Two-way binding for interactive widgets
- Property change handling for reactivity
- Relay commands for button clicks

**Communication Flow:**
1. Module invokes ScriptInvoker event
2. Menu receives event via subscription
3. Menu updates controller properties
4. ViewBinding updates widgets automatically

---

**Related Guides:**
- [How Layouts Work](../How-Layouts-Work.md) - Understanding layout files
- [How MVC Works](../How-MVC-Works.md) - Controller pattern
- [How Invoke Works](../How-Invoke-Works.md) - ScriptInvoker communication
- [Simple Menu Example](Simple-Menu-Example.md) - Basic example

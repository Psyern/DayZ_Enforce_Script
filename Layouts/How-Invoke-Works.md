# How Invoke Works

Complete guide to understanding ScriptInvoker and communication patterns between modules and menus in DayZ.

## What is ScriptInvoker?

`ScriptInvoker` is DayZ's event system for decoupled communication between modules and menus. It allows:

- **Modules** to send events to menus
- **Menus** to subscribe to module events
- **Decoupled architecture** - modules don't need direct menu references
- **Callback patterns** - async operations with callbacks

## ScriptInvoker Basics

### Creating a ScriptInvoker

**In Module (4_World layer):**
```c
class MyMod_Module: CF_ModuleCoreManager
{
	// Static ScriptInvoker for module-to-menu communication
	static ref ScriptInvoker SI_MenuUpdate = new ScriptInvoker();
	static ref ScriptInvoker SI_MenuCallback = new ScriptInvoker();
	
	void MyMod_Module()
	{
		// Enable invoke connection (required for CF modules)
		EnableInvokeConnect();
	}
}
```

### Invoking Events

**Send event from module:**
```c
class MyMod_Module: CF_ModuleCoreManager
{
	void UpdateMenuData(string data)
	{
		// Invoke event - all subscribers receive it
		SI_MenuUpdate.Invoke(data);
	}
	
	void ProcessRequest(string request, int callbackType)
	{
		// Invoke with callback
		SI_MenuCallback.Invoke(request, callbackType, result);
	}
}
```

### Subscribing to Events

**In Menu (5_Mission layer):**
```c
class MyMod_Menu: ExpansionScriptViewMenu
{
	protected ref MyMod_Module m_Module;
	
	override Widget Init()
	{
		Widget root = super.Init();
		
		// Get module reference
		m_Module = MyMod_Module.Cast(CF_ModuleCoreManager.Get(MyMod_Module));
		
		// Subscribe to events
		if (m_Module)
		{
			m_Module.SI_MenuUpdate.Insert(OnMenuUpdate);
			m_Module.SI_MenuCallback.Insert(OnMenuCallback);
		}
		
		return root;
	}
	
	override void Close()
	{
		// Unsubscribe when menu closes
		if (m_Module)
		{
			m_Module.SI_MenuUpdate.Remove(OnMenuUpdate);
			m_Module.SI_MenuCallback.Remove(OnMenuCallback);
		}
		
		super.Close();
	}
	
	// Event handler methods
	void OnMenuUpdate(string data)
	{
		Print("Menu update received: " + data);
		// Update UI with new data
	}
	
	void OnMenuCallback(string request, int callbackType, string result)
	{
		Print("Callback received: " + result);
		// Handle callback
	}
}
```

## Communication Patterns

### Pattern 1: Module → Menu (One-Way)

**Module sends data to menu:**

```c
// Module (4_World)
class MyMod_Module: CF_ModuleCoreManager
{
	static ref ScriptInvoker SI_DataUpdate = new ScriptInvoker();
	
	void SendDataToMenu(string data)
	{
		SI_DataUpdate.Invoke(data);
	}
}
```

```c
// Menu (5_Mission)
class MyMod_Menu: ExpansionScriptViewMenu
{
	protected ref MyMod_Module m_Module;
	
	override Widget Init()
	{
		Widget root = super.Init();
		m_Module = MyMod_Module.Cast(CF_ModuleCoreManager.Get(MyMod_Module));
		
		if (m_Module)
		{
			m_Module.SI_DataUpdate.Insert(OnDataUpdate);
		}
		
		return root;
	}
	
	void OnDataUpdate(string data)
	{
		// Update UI with data
		MyMod_MenuController controller = GetMenuController();
		if (controller)
		{
			controller.Data = data;
		}
	}
	
	override void Close()
	{
		if (m_Module)
		{
			m_Module.SI_DataUpdate.Remove(OnDataUpdate);
		}
		super.Close();
	}
}
```

### Pattern 2: Module → Menu (With Callback)

**Module sends request, menu responds:**

```c
// Module (4_World)
class MyMod_Module: CF_ModuleCoreManager
{
	static ref ScriptInvoker SI_Request = new ScriptInvoker();
	
	void RequestMenuAction(string action, int option)
	{
		SI_Request.Invoke(action, option);
	}
}
```

```c
// Menu (5_Mission)
class MyMod_Menu: ExpansionScriptViewMenu
{
	protected ref MyMod_Module m_Module;
	
	override Widget Init()
	{
		Widget root = super.Init();
		m_Module = MyMod_Module.Cast(CF_ModuleCoreManager.Get(MyMod_Module));
		
		if (m_Module)
		{
			m_Module.SI_Request.Insert(OnRequest);
		}
		
		return root;
	}
	
	void OnRequest(string action, int option)
	{
		// Handle request from module
		if (action == "UpdateUI")
		{
			UpdateUI();
		}
	}
}
```

### Pattern 3: Menu → Module (Via RPC)

**Menu requests data from module:**

```c
// Menu (5_Mission)
class MyMod_Menu: ExpansionScriptViewMenu
{
	protected ref MyMod_Module m_Module;
	
	void RequestData()
	{
		if (m_Module)
		{
			// Use RPC to request data from server
			GetRPCManager().SendRPC("MyMod_Module", "RequestDataRPC", new Param1<string>("menu_request"), true);
		}
	}
}
```

```c
// Module (4_World)
class MyMod_Module: CF_ModuleCoreManager
{
	[RPC]
	void RequestDataRPC(CallType type, ParamsReadContext ctx, PlayerIdentity sender, Object target)
	{
		Param1<string> data = new Param1<string>("response_data");
		GetRPCManager().SendRPC("MyMod_Menu", "OnDataReceivedRPC", data, true, sender);
	}
}
```

### Pattern 4: Async Operations with Callback

**Module performs async operation, menu handles callback:**

```c
// Module (4_World)
class MyMod_Module: CF_ModuleCoreManager
{
	static ref ScriptInvoker SI_Callback = new ScriptInvoker();
	
	void ProcessAsyncOperation(string itemClassName, int option1, int option2, Object object)
	{
		// Perform async operation (e.g., server request)
		// ...
		
		// Invoke callback when complete
		SI_Callback.Invoke(itemClassName, result, option1, option2, object);
	}
}
```

```c
// Menu (5_Mission)
class MyMod_Menu: ExpansionScriptViewMenu
{
	protected ref MyMod_Module m_Module;
	
	override Widget Init()
	{
		Widget root = super.Init();
		m_Module = MyMod_Module.Cast(CF_ModuleCoreManager.Get(MyMod_Module));
		
		if (m_Module)
		{
			m_Module.SI_Callback.Insert(OnCallback);
		}
		
		return root;
	}
	
	void OnCallback(string itemClassName, int result, int option1, int option2, Object object)
	{
		// Handle callback from async operation
		if (result == 1)
		{
			Print("Operation successful: " + itemClassName);
		}
		else
		{
			Print("Operation failed: " + itemClassName);
		}
	}
}
```

## Real-World Example: Market Module

### Module Defines Invokers

```c
class ExpansionMarketModule: CF_ModuleCoreManager
{
	// Static ScriptInvokers for communication
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
	
	// Invoke when item is updated
	void UpdateSelectedItem(ExpansionMarketItem item)
	{
		SI_SelectedItemUpdatedInvoker.Invoke(item);
	}
	
	// Invoke callback
	void InvokeCallback(string itemClassName, int result, int option1, int option2, Object object)
	{
		SI_Callback.Invoke(itemClassName, result, option1, option2, object);
	}
}
```

### Menu Subscribes to Invokers

```c
class ExpansionMarketMenu: ExpansionScriptViewMenu
{
	protected ref ExpansionMarketModule m_MarketModule;
	
	override Widget Init()
	{
		Widget root = super.Init();
		
		// Get module reference
		m_MarketModule = ExpansionMarketModule.Cast(CF_ModuleCoreManager.Get(ExpansionMarketModule));
		
		// Subscribe to events
		if (m_MarketModule)
		{
			m_MarketModule.SI_SetTraderInvoker.Insert(SetTraderObject);
			m_MarketModule.SI_SelectedItemUpdatedInvoker.Insert(OnNetworkItemUpdate);
			m_MarketModule.SI_Callback.Insert(MenuCallback);
		}
		
		return root;
	}
	
	// Handler: Trader object set
	void SetTraderObject(ExpansionTraderObjectBase trader, bool force)
	{
		m_TraderObject = trader;
		if (m_TraderObject)
		{
			m_TraderMarket = m_TraderObject.GetMarket();
			// Update UI with trader data
			UpdateTraderUI();
		}
	}
	
	// Handler: Item updated from network
	void OnNetworkItemUpdate(ExpansionMarketItem item)
	{
		if (item)
		{
			m_SelectedMarketItem = item;
			// Update UI with item data
			UpdateItemUI();
		}
	}
	
	// Handler: Callback from async operation
	void MenuCallback(string itemClassName, int result, int option1, int option2, Object object)
	{
		if (result == 1)
		{
			Print("Operation successful: " + itemClassName);
		}
		else
		{
			Print("Operation failed: " + itemClassName);
		}
	}
	
	override void Close()
	{
		// Unsubscribe when menu closes
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

## Best Practices

### 1. Always Unsubscribe

```c
// ✅ Good - Clean up subscriptions
override void Close()
{
	if (m_Module)
	{
		m_Module.SI_Update.Remove(OnUpdate);
	}
	super.Close();
}

// ❌ Bad - Memory leak
// Forgot to unsubscribe
```

### 2. Check Module Before Subscribing

```c
// ✅ Good - Safe subscription
override Widget Init()
{
	Widget root = super.Init();
	
	m_Module = MyMod_Module.Cast(CF_ModuleCoreManager.Get(MyMod_Module));
	if (m_Module)
	{
		m_Module.SI_Update.Insert(OnUpdate);
	}
	
	return root;
}

// ❌ Bad - May crash if module not loaded
m_Module.SI_Update.Insert(OnUpdate);
```

### 3. Use Static ScriptInvokers

```c
// ✅ Good - Static for module-wide access
class MyMod_Module: CF_ModuleCoreManager
{
	static ref ScriptInvoker SI_Update = new ScriptInvoker();
}

// ❌ Avoid - Instance invokers harder to access
ref ScriptInvoker m_Update = new ScriptInvoker();
```

### 4. Enable Invoke Connect

```c
// ✅ Good - Required for CF modules
class MyMod_Module: CF_ModuleCoreManager
{
	void MyMod_Module()
	{
		EnableInvokeConnect();
	}
}
```

### 5. Use Descriptive Invoker Names

```c
// ✅ Good - Clear purpose
static ref ScriptInvoker SI_SetTraderInvoker = new ScriptInvoker();
static ref ScriptInvoker SI_SelectedItemUpdatedInvoker = new ScriptInvoker();
static ref ScriptInvoker SI_Callback = new ScriptInvoker();

// ❌ Avoid - Generic names
static ref ScriptInvoker SI_Invoker1 = new ScriptInvoker();
static ref ScriptInvoker SI_Invoker2 = new ScriptInvoker();
```

### 6. Handle Multiple Parameters

```c
// ✅ Good - Clear parameter order
SI_Callback.Invoke(itemClassName, result, option1, option2, object);

// Handler matches parameter order
void OnCallback(string itemClassName, int result, int option1, int option2, Object object)
{
	// Handle callback
}
```

## Common Patterns

### Pattern: Module Initialization

```c
class MyMod_Module: CF_ModuleCoreManager
{
	static ref ScriptInvoker SI_Initialized = new ScriptInvoker();
	
	override void OnInit()
	{
		super.OnInit();
		
		// Notify menus that module is ready
		SI_Initialized.Invoke();
	}
}
```

### Pattern: Settings Update

```c
class MyMod_Module: CF_ModuleCoreManager
{
	static ref ScriptInvoker SI_SettingsUpdated = new ScriptInvoker();
	
	void UpdateSettings()
	{
		// Update settings
		// ...
		
		// Notify menus
		SI_SettingsUpdated.Invoke();
	}
}
```

### Pattern: Data Refresh

```c
class MyMod_Module: CF_ModuleCoreManager
{
	static ref ScriptInvoker SI_DataRefreshed = new ScriptInvoker();
	
	void RefreshData()
	{
		// Refresh data
		// ...
		
		// Notify menus
		SI_DataRefreshed.Invoke();
	}
}
```

## Summary

**ScriptInvoker:**
- Event system for module-to-menu communication
- Decoupled architecture
- Static invokers for module-wide access
- Enable `InvokeConnect` for CF modules

**Communication Patterns:**
- **Module → Menu** - One-way data updates
- **Module → Menu** - With callbacks
- **Menu → Module** - Via RPC
- **Async Operations** - With callbacks

**Best Practices:**
- Always unsubscribe in `Close()`
- Check module before subscribing
- Use static ScriptInvokers
- Enable InvokeConnect
- Use descriptive names
- Handle multiple parameters correctly

---

**Related Guides:**
- [How Layouts Work](How-Layouts-Work.md) - Understanding layout files
- [How MVC Works](How-MVC-Works.md) - Controller pattern
- [Market Menu Example](Examples/Market-Menu-Example.md) - Complete implementation

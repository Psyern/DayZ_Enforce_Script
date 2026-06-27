# Tips: Best Practices

Essential best practices for DayZ modding.

## 1. Use Meaningful Names

```c
// Good
float VehicleSearchRadius;
bool CanStoreWithCargo;

// Avoid
float vsr;
bool cswc;
```

## 2. Keep Methods Focused

Each method should do one thing well:

```c
void SetDefaultLootingBehaviour()
{
	if (Behaviour == "ROAMING")
		LootingBehaviour = "ALL";
	else
		LootingBehaviour = "DEFAULT";
}
```

## 3. Use Constants for Magic Numbers

```c
// Good
const int DEFAULT_MAX_VEHICLES = 10;
MaxStorableVehicles = DEFAULT_MAX_VEHICLES;

// Avoid
MaxStorableVehicles = 10;
```

## 4. Initialize Member Variables

Initialize in constructor or at declaration:

```c
void ExpansionGarageSettings()
{
	EntityWhitelist = new TStringArray;
	m_IsLoaded = false;
}
```

## 5. Document Complex Logic

```c
//! Toggle between hidden and previously set visibility state for each marker category
int visibility;
int previousVisibility;

visibility |= m_MarkerModule.GetVisibility(ExpansionMapMarkerType.SERVER);
previousVisibility |= m_MarkerModule.GetPreviousVisibility(ExpansionMapMarkerType.SERVER);
```

## 6. Use Bitwise Operations Appropriately

```c
layerMask |= PhxInteractionLayers.BUILDING;
layerMask |= PhxInteractionLayers.VEHICLE;
layerMask |= PhxInteractionLayers.TERRAIN;

visibility &= EXPANSION_MARKER_VIS_WORLD;
```

## 7. Leverage Type Safety

```c
// Use enums
eAILootingBehavior behavior;

// Use proper casting
PlayerBase player = PlayerBase.Cast(entity);
if (player)
{
	// Safe to use player
}
```

## 8. Avoid Complex Expressions in Array Assignments

**⚠️ Segfault Risk:**

Some language constructs that work in other languages cause segfaults in EnScript:

```c
class MyClass
{
	bool m_IsInside[3];
	
	// ❌ WRONG - Causes segfault
	void TestBad(int index, vector a, vector b, float distSq)
	{
		m_IsInside[index] = vector.DistanceSq(a, b) <= distSq;
	}
	
	// ✅ CORRECT - Use intermediate variable
	void TestGood(int index, vector a, vector b, float distSq)
	{
		bool isInside = vector.DistanceSq(a, b) <= distSq;
		m_IsInside[index] = isInside;
	}
}
```

**Rule:** For array assignments with complex expressions, use an intermediate variable.

## 9. Be Aware of Integer Limits

**⚠️ Integer Comparison Quirk:**

```c
// int.MIN = -2147483648

1 < int.MIN;      // ⚠️ Returns TRUE in EnScript!
1 < -2147483647;  // ⚠️ Also returns TRUE!

// Always be careful with MIN/MAX integer comparisons
```

## 10. Avoid GetObjectsAtPosition

**Specific use note:** If at all possible, avoid using `GetObjectsAtPosition` and `GetObjectsAtPosition3D` as much as possible. Static arrays on the class you're looking for, triggers, or the GetScene methods are all much better options available.

## 11. Use Proto Methods for Expensive Operations

When available, prefer using proto (native engine) methods over custom script implementations **when the native method performs expensive logic**. Proto methods are faster for complex operations as they run in native code.

```c
// ✅ PREFERRED - Proto method for expensive operations
proto native vector GetPosition();

// ❌ AVOID - Custom reimplementation of expensive native logic
vector GetPosition() 
{
	// Custom expensive calculation
}
```

**Important Note:** There is overhead from calling native methods from script. For simple operations, custom script methods may actually be faster. This is why `proto native CGame GetGame()` was replaced with `DayZGame GetGame() { return g_Game; }` - the script version is faster for simple global access.

**Rule:** Use proto methods when they perform complex/expensive operations. For simple operations (like returning a global), script implementations are often faster.

## 12. Use Obsolete Attribute for Deprecated Code

If someone is writing a mod that will likely be modded/overridden, suggest using the Obsolete attribute on a method/field/variable instead of removing them altogether. The workbench will show a warning when used.

```c
[Obsolete()]
vector GetCustomPosition() 
{
	// Custom expensive calculation
}
```

## 13. Always Check Indexed Documentation First

**❌ CRITICAL: ALWAYS check and reference indexed documentation before implementing any modding feature.** These are the PRIMARY references for DayZ modding and MUST be consulted:

**Main References (ALWAYS check these first):**
- **Tips and Best Practices:** `Tips/` - Essential coding guidelines and patterns
- **How-To Guides:** `How-To/` - Step-by-step implementation guides
- **Frameworks:** `Frameworks/` - Framework documentation (Community Framework, DayZ Expansion, Dabs Framework)
- **DayZ Game API:** `DayZGame/` - DayZ vanilla game API references
- **Community Framework Docs:** `Frameworks/Community-Framework.md` - Complete CF API documentation

**CRITICAL RULE:** Never guess framework APIs or DayZ patterns. Always check these references first to ensure correct implementation.

**Example:** When using Community Framework logging, reference the CF documentation to use the correct syntax:

```c
// ✅ CORRECT - From CF docs
static CF_Log s_Log = new CF_Log("MyMod");
s_Log.Info("Message");

// ❌ WRONG - Don't guess the API
CF_Log.Get("MyMod").Info("Message"); // This doesn't exist!
```

**Rule:** Always check indexed documentation before implementing any feature. Never guess APIs or patterns.

## 14. Check DayZ Vanilla Source Code When Modding

**❌ CRITICAL: ALWAYS check the DayZ vanilla source code at `@1.29_Experimental/1.29` when modding any function or class.** Before implementing modded functionality, examine the vanilla implementation to:

- **Understand available functions and variables** - See what methods and properties are available in the vanilla class
- **Learn proper usage patterns** - Understand how vanilla code uses these functions
- **Avoid reinventing the wheel** - Use existing vanilla functions instead of custom implementations
- **Ensure compatibility** - Understand the vanilla behavior to maintain compatibility

**Example:** When modding `PlayerBase.OnDeath()`, check `@1.29_Experimental/1.29` to see:
- What parameters `OnDeath()` receives
- What member variables are available (e.g., `m_Health`, `m_LastDamageSource`)
- What helper functions exist (e.g., `GetIdentity()`, `GetPosition()`)
- How vanilla code handles death logic

**CRITICAL RULE:** Always reference the vanilla source code implementation before modding any DayZ function or class. This ensures you use the correct API and understand available functionality.

## Performance Considerations

* Aggressive garbage collection destroys function-scope instances immediately
* Use `ref`/`autoptr` on class members to prevent premature collection
* Avoid creating temporary objects in tight loops

---

**Related Guides:**
- [EnScript Style Guide](EnScript-Style-Guide.md) - Complete style reference
- [Tips: Memory Management](Tips-Memory-Management.md) - Memory best practices
- [Tips: Code Structure](Tips-Code-Structure.md) - Code organization
- [Tips: Type System](Tips-Type-System.md) - Type safety
# How to Create Recipes

Complete guide to creating and using recipes (crafting) in DayZ mods.

## What are Recipes?

Recipes define crafting combinations - what items combine to create other items. They handle ingredients, results, and crafting logic.

## Recipe Folder Structure

Recipes belong in the **4_World** layer:

```
scripts/4_world/classes/recipes/
├── recipebase.c                    # Base recipe class
├── cacheobject.c                   # Recipe cache
├── recipes/                        # Recipe implementations (217+ files)
│   ├── recipetest.c
│   └── ... (all recipe files)
└── (plugin recipes manager in plugins/)
```

**Important:** All recipes go in **4_World** layer. See [Script Layers Guide](Script-Layers-Guide.md) for why.

## Recipe Class Naming Conventions

**Pattern:** `Recipe[RecipeName]` (PascalCase, no spaces)

```c
// ✅ CORRECT - Recipe naming
class RecipeTest extends RecipeBase
class RecipeCraftStoneKnife extends RecipeBase
class RecipeCookMeat extends RecipeBase

// ❌ WRONG
class TestRecipe                    // Wrong order
class recipe_craft_knife            // Wrong case and format
class RecipeCraft_Knife             // Wrong separator
```

## Recipe Structure

### Basic Recipe

```c
class RecipeTest extends RecipeBase
{
	override void Init()
	{
		// Recipe name
		m_Name = "Test Recipe";
		
		// Recipe settings
		m_IsInstaRecipe = false;      // Instant or animated
		m_AnimationLength = 1;        // Animation length
		m_Specialty = 0;              // Roughness/precision
		
		// Ingredient conditions
		m_MinDamageIngredient[0] = -1;  // -1 = disable check
		m_MaxDamageIngredient[0] = 3;
		m_MinQuantityIngredient[0] = -1;
		m_MaxQuantityIngredient[0] = -1;
		
		// Ingredients
		InsertIngredient(0, "BloodTestKit");
		m_IngredientAddHealth[0] = 0;
		m_IngredientSetHealth[0] = -1;
		m_IngredientAddQuantity[0] = 0;
		m_IngredientDestroy[0] = true;  // Destroy ingredient
		m_IngredientUseSoftSkills[0] = false;
		
		InsertIngredient(1, "BloodSyringe");
		m_IngredientDestroy[1] = false;  // Keep ingredient
		
		// Results
		AddResult("ResultItem");
		m_ResultSetFullQuantity[0] = false;
		m_ResultSetQuantity[0] = -1;
		m_ResultSetHealth[0] = -1;
		m_ResultInheritsHealth[0] = -1;
		m_ResultInheritsColor[0] = -1;
		m_ResultToInventory[0] = -2;  // -2 = spawn on ground
		m_ResultUseSoftSkills[0] = false;
		m_ResultReplacesIngredient[0] = -1;
	}
	
	override bool CanDo(ItemBase ingredients[], PlayerBase player)
	{
		// Final validation check
		return true;
	}
}
```

## Recipe Properties Explained

### Recipe Settings

```c
m_Name = "Recipe Name";              // Display name
m_IsInstaRecipe = false;            // true = instant, false = animated
m_AnimationLength = 1.0;            // Animation duration
m_Specialty = 0;                    // >0 = roughness, <0 = precision
m_AnywhereInInventory = false;      // Can craft without items in hands
```

### Ingredient Properties

```c
// Ingredient slot 0
InsertIngredient(0, "ItemClassName");
m_MinDamageIngredient[0] = -1;      // -1 = no min damage check
m_MaxDamageIngredient[0] = 3;       // Max damage allowed
m_MinQuantityIngredient[0] = -1;    // -1 = no min quantity check
m_MaxQuantityIngredient[0] = -1;    // -1 = no max quantity check
m_IngredientAddHealth[0] = 0;        // Add health to ingredient
m_IngredientSetHealth[0] = -1;       // -1 = don't set health
m_IngredientAddQuantity[0] = 0;     // Add quantity to ingredient
m_IngredientDestroy[0] = true;      // Destroy ingredient after use
m_IngredientUseSoftSkills[0] = false; // Use soft skills
```

### Result Properties

```c
// Result slot 0
AddResult("ResultItemClassName");
m_ResultSetFullQuantity[0] = false; // true = set to max quantity
m_ResultSetQuantity[0] = -1;        // -1 = don't set quantity
m_ResultSetHealth[0] = -1;          // -1 = don't set health
m_ResultInheritsHealth[0] = -1;     // -1 = don't inherit, >=0 = inherit from ingredient slot
m_ResultInheritsColor[0] = -1;      // -1 = don't inherit, >=0 = inherit from ingredient slot
m_ResultToInventory[0] = -2;        // -2 = spawn on ground, -1 = inventory, >=0 = replace ingredient slot
m_ResultUseSoftSkills[0] = false;   // Use soft skills
m_ResultReplacesIngredient[0] = -1;  // -1 = don't replace, >=0 = replace ingredient slot
m_ResultSpawnDistance[0] = 0.6;     // Distance to spawn from player
```

## Recipe Registration

Recipes must be manually registered in `PluginRecipesManager`. They are **NOT** automatically discovered.

### How to Register Recipes

Create a modded `PluginRecipesManager` class in your **4_World** layer:

**File:** `scripts/4_World/YourModName/PluginRecipesManager.c`

```c
modded class PluginRecipesManager
{
	override void RegisterRecipies()
	{
		super.RegisterRecipies();  // Always call super first!
		
		// Register your custom recipes
		RegisterRecipe(new MyMod_RecipeCustom);
		RegisterRecipe(new MyMod_RecipeCraftItem);
		RegisterRecipe(new MyMod_RecipeCookMeat);
	}
}
```

**Important:**
- **Location:** Must be in **4_World** layer
- **Always call `super.RegisterRecipies()`** first to register vanilla recipes
- **Use `RegisterRecipe(new RecipeClassName)`** to register each recipe
- **File naming:** Can be `PluginRecipesManager.c` or any name you prefer

### Recipe Manager Location

The base recipe manager is in:
- **DayZ 1.28:** `scripts/4_world/plugins/pluginbase/pluginrecipesmanager.c`
- **DayZ 1.29:** `scripts/4_World/Plugins/PluginBase/PluginRecipesManager.c`

You mod this class to add your recipes.

## Complete Example: Custom Recipe

### Step 1: Create Recipe Class

**File:** `scripts/4_World/YourModName/Classes/Recipes/MyMod_RecipeCustom.c`

```c
class MyMod_RecipeCustom extends RecipeBase
{
	override void Init()
	{
		m_Name = "Custom Recipe";
		m_IsInstaRecipe = false;
		m_AnimationLength = 2.0;
		m_Specialty = 0;
		
		// Ingredient 1
		InsertIngredient(0, "Item1");
		m_IngredientDestroy[0] = true;
		
		// Ingredient 2
		InsertIngredient(1, "Item2");
		m_IngredientDestroy[1] = true;
		
		// Result
		AddResult("ResultItem");
		m_ResultToInventory[0] = -1;  // Place in inventory
	}
	
	override bool CanDo(ItemBase ingredients[], PlayerBase player)
	{
		// Final validation check
		return true;
	}
}
```

### Step 2: Register Recipe

**File:** `scripts/4_World/YourModName/PluginRecipesManager.c`

```c
modded class PluginRecipesManager
{
	override void RegisterRecipies()
	{
		super.RegisterRecipies();  // Register vanilla recipes first
		
		// Register your custom recipe
		RegisterRecipe(new MyMod_RecipeCustom);
	}
}
```

**That's it!** Your recipe will now be available in-game.

## Best Practices

### 1. Follow Recipe Naming Pattern

```c
// ✅ Good - Clear recipe name
class RecipeCraftStoneKnife
class RecipeCookMeat

// ❌ Avoid - Generic names
class Recipe1
class RecipeTest
```

### 2. Prefix Custom Recipes

```c
// ✅ Good - Prefixed to avoid conflicts
class MyMod_RecipeCustom

// ❌ Avoid - Generic names
class RecipeCustom
```

### 3. Use Descriptive Names

```c
// ✅ Good - Clear what it does
class RecipeCraftStoneKnife
class RecipeCookMeat
class RecipeRepairClothing

// ❌ Avoid - Unclear
class Recipe1
class RecipeCraft
```

## Summary

**Recipes:**
- **Location:** `4_World/YourModName/Classes/Recipes/`
- **Naming:** `Recipe[Name]` (PascalCase)
- **Base:** `RecipeBase`
- **Registration:** Must manually register in `modded class PluginRecipesManager`

**Key Rules:**
- All recipes go in **4_World** layer
- Use PascalCase for all class names
- Prefix custom recipes with your mod prefix
- Use descriptive names

---

**Related Guides:**
- [Script Layers Guide](Script-Layers-Guide.md) - Understanding where code belongs
- [How to Create Actions](How-To-Actions.md) - Action system guide
- [Tips: Modded Classes](Tips-Modded-Classes.md) - How to extend vanilla classes
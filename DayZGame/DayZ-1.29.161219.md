# DayZ 1.29.161219 - Experimental Version

Documentation for DayZ Standalone experimental version 1.29.161219 scripts and modding.

## Version Information

- **Version:** 1.29.161219
- **Status:** Experimental
- **Script Location:** `C:\Users\Administrator\Desktop\Mod Repositories\scripts - 1.29` (P-drive when extracted)

## Script Structure

DayZ 1.29 experimental uses a similar structure to 1.28 with potential updates:

**Actual Structure (when extracted):**
```
C:\Users\Administrator\Desktop\Mod Repositories\scripts - 1.29/              # P-drive location
├── 1_Core/              # Core engine systems (note: capital C)
├── 2_GameLib/           # Game library
├── 3_Game/              # Game logic (note: capital G)
├── 4_World/             # World entities
├── 5_Mission/           # Mission logic
└── config.cpp           # Main configuration
```

**Notes:**
- Layer directories use capital letters (1_Core, 3_Game) compared to 1.28 (1_core, 3_game)
- When DayZ is extracted, scripts are located at `C:\Users\Administrator\Desktop\Mod Repositories\scripts - 1.29` (P-drive)

## Key Differences from 1.28

### Directory Naming

- **1.28:** `1_core`, `3_game` (lowercase after number)
- **1.29:** `1_Core`, `3_Game` (capital letter after number)

This is likely a standardization of naming conventions.

### RPC System

RPC enum location:
- **Location:** `3_Game/Enums/ERPCs.c` (capital E in Enums, capital E in ERPCs)
- **Enum:** `ERPCs` (same as 1.28)

RPC functionality remains largely the same, but check for new RPC values.

### File Naming Conventions

Some files may use different naming:
- **1.28:** `staticdefinesdoc.c` (lowercase)
- **1.29:** `staticDefinesDoc.c` (PascalCase)

This suggests a move toward more consistent PascalCase naming.

### Global Variable Access

DayZ 1.29 uses `g_Game` global variable directly:

- **1.28:** Uses `GetGame()` function: `GetGame().IsDedicatedServer()`
- **1.29:** Uses `g_Game` global: `g_Game.IsDedicatedServer()`

**Important:** In DayZ 1.29+, always check for null when using `g_Game`:

```c
// DayZ 1.29+ pattern
if (!g_Game)
	return;

if (g_Game.IsDedicatedServer())
{
	// Server code
}
```

See [Tips: g_Game vs GetGame()](Tips-g_Game-GetGame.md) for detailed information.

## Key Systems

### Core Systems

Similar to 1.28 but check for:
- Updated proto methods
- New base classes
- Enhanced module system
- Improved RPC handling

### Updated Classes

Check for updates to:
- PlayerBase
- ItemBase
- EntityAI
- Module system classes

## Modding Reference

### Compatibility Considerations

When developing for 1.29 experimental:

1. **Check Layer Naming:** Use capital letters if modding 1.29
2. **Test RPC Values:** New RPCs may have been added
3. **Verify Class Changes:** Check for updated method signatures
4. **Test Thoroughly:** Experimental versions may have bugs

### Example Config for 1.29

```cpp
class CfgMods
{
	class MyMod
	{
		dir = "MyMod";  // Mod identifier (not workspace folder structure)
		// ... config ...
		class defs
		{
			class gameScriptModule
			{
				value = "";  // Usually blank - entry point is rarely needed
				files[] = {"MyMod/scripts/3_Game"};  // No namespace - starts with mod name
				// Note: Still uses "3_Game" in path, not "3_Core"
			};
		};
	};
};
```

**Important:** 
- The `dir` and `files[]` paths must start with the mod name, not a workspace namespace folder
- Namespace folders (like `MyNamespace/`) are only for workspace organization and don't exist in the packaged mod
- Clients and servers will only see the mod structure starting from the mod name

## Development Tips

### Testing on Experimental

1. **Backup:** Always backup mods before testing on experimental
2. **Version Check:** Verify mod compatibility with 1.29
3. **Watch for Changes:** Monitor for breaking changes in updates
4. **Report Issues:** Help improve DayZ by reporting bugs

### Finding Changes

To identify what changed from 1.28 to 1.29:

1. Compare file structures
2. Check for new RPC values
3. Look for new classes or methods
4. Review updated method signatures

## Important Files

### Core System Files
- `1_Core/` - Core script functionality
- `3_Game/Enums/ERPCs.c` - RPC enum definitions
- `3_Game/` - Game logic and modules

### World Entity Files
- `4_World/Classes/` - Base classes
- `4_World/Entities/` - Entity implementations

## Experimental Features

Experimental versions may include:
- New gameplay features
- Updated systems
- Bug fixes
- Performance improvements
- API changes

**Always check patch notes and test thoroughly!**

## Additional 1.29 Update Notes

Newly reported 1.29 changes with direct scripting/modding impact:

### Script and API Additions

- `Physics.GetGeomName` was added to retrieve a geometry name by index.
- `Transport` now exposes physics functions directly so vehicles can be put to sleep from script.
- `EntityAI.SetRequiredSimulation` was added to explicitly control active simulation state for custom `Transport` vehicles.
- `Pawn.GetOwnerState` and `Pawn.GetNextMove` were added for preloading data during character `CommandHandler` logic.
- `ILT_TEMP` was added as a new inventory location type for temporary desynced item limbo until the server sends the correct location.
- Action and inventory junctures can now carry user data.

### Engine and Behavior Changes Relevant to Mods

- `EntityType` can now be used in script per config class definition, allowing caching of shared variables and expensive initialization logic.
- Animation System received an initial Enfusion 2021 update, so animation-related assumptions should be revalidated.
- Physics filtered results no longer report collisions without an interaction layer.
- Prioritized use of `g_Game` instead of `GetGame()` was confirmed again at engine level.
- Several scripted functions were optimized, including `Object::IsAnyInherited`.
- Inactive physics bodies no longer tick `EOnSimulate` / `EOnPostSimulate`.
- Vehicle headlights were partially moved from script to native code.
- Gizmo API was improved and moved to `GizmoApi`.
- Unnecessary native calls with negative performance impact were removed.

### Fixes With Modding Impact

- Fixed crash when using `SurfaceUnderObjectByBone` on some animals.
- Fixed crashes related to geometry element usage on skeleton components.
- Fixed several `DebugText` crashes.
- Fixed `DayZPlayer.IsAuthority` returning `false` in singleplayer.
- Fixed `Inventory.CreateEntity` not respecting the flip flag.
- Fixed `Transport` not being able to gather inputs.
- Fixed `Transport` lacking dynamic collision resolution outside car-specific behavior.
- Fixed vehicles sending players to random coordinates on exit when geometry was invalid.
- Fixed vehicles still being controllable in freecam while `Player move in FrCam` was disabled.
- Fixed script assignment of `typename` for base types such as `string` and `int`.
- Fixed some cases where assigning quoted vectors to variables failed.
- Fixed forcing impossible nested spawns from script, such as barrel-in-barrel.
- Fixed Workbench local method variable corruption in plugins.
- Fixed `make avoidGrassUpdate` not transferring correctly during binarization.
- Fixed Gizmo rotation being reversed in some cases.

### Practical Modding Notes

- Recheck any custom vehicle scripts that rely on script-side sleep/wake, lights, or collision behavior.
- Recheck any inventory edge-case handling around desync, temporary locations, and custom juncture payloads.
- If you maintain diagnostic/admin tooling, retest `DebugText`, Gizmo helpers, and `IsAuthority` assumptions.
- If you use skeleton/geometry helpers or `SurfaceUnderObjectByBone`, revisit old crash workarounds because engine behavior has changed.
- Prefer `g_Game` patterns consistently and treat remaining `GetGame()` usage as legacy migration work.

## Compatibility Notes

### For Modders

- **Backward Compatibility:** Mods for 1.28 may need updates for 1.29
- **Forward Compatibility:** Don't rely on experimental features for stable releases
- **Testing:** Test mods on both versions if supporting multiple

### Breaking Changes

Check official DayZ patch notes for:
- Removed methods or classes
- Changed method signatures
- New required parameters
- Updated RPC system

## Script Layer Details

Same structure as 1.28 but with updated implementations:

### 1_Core Layer
- Core engine systems
- Proto method definitions
- Base utilities

### 2_GameLib Layer
- Game library abstractions
- Shared utilities
- Input and menu management

### 3_Game Layer
- Game initialization
- Modules and managers
- Settings and configuration

### 4_World Layer
- Entity base classes
- Item and vehicle classes
- World objects

### 5_Mission Layer
- Mission initialization
- Player scripts
- GUI systems

## Related Documentation

- [DayZ 1.28.161464](DayZ-1.28.161464.md) - Compare with stable version
- [Basic Mod Structure](Basic-Mod-Structure.md)
- [How to Use RPC](../How-To/How-To-RPC.md)
- [EnScript Style Guide](EnScript-Style-Guide.md)

## Notes

- Experimental versions are subject to change
- Always test mods thoroughly
- Watch for updates and breaking changes
- Consider maintaining compatibility with stable version

---

**Note on Script Locations:**
- When DayZ is extracted, scripts are located at `C:\Users\Administrator\Desktop\Mod Repositories\scripts - 1.29` (P-drive)
- You may organize extracted scripts into folders like `DayZ_1.29.161219` for your own version tracking

**Last Updated:** Updated with additional 1.29 engine, physics, transport, inventory, and scripting notes
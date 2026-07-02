---
name: enforce-script
description: Use when writing, reviewing, or debugging Enforce Script (EnScript) code for DayZ / Enfusion engine — mods, modded classes, RPC, script layers (1_Core/3_Game/4_World/5_Mission), config.cpp, UI layouts/widgets, ref/memory issues, or compiler errors like "Broken expression (missing ';'?)" and "Multiple declaration"
---

# Enforce Script / Enfusion (DayZ)

## Overview

The authoritative reference is this wiki (the DAYZ_Enforce-Script repository).
All doc paths below are relative to the repository root.
If you install this skill user-level (outside the repo), prefix every path with
the absolute location of your local clone.
**Consult the matching guide before implementing — never guess DayZ or framework APIs.**

## Hard Rules (compiler- or save-data-critical — always apply)

| Rule | Wrong | Right |
|---|---|---|
| No ternary operator | `x = c ? a : b;` | `if/else` |
| One variable per declaration | `int a, b, c;` | `int a;` `int b;` `int c;` |
| Never `GetGame()` (DayZ 1.29+) | `GetGame().IsDedicatedServer()` | `if (g_Game && g_Game.IsDedicatedServer())` |
| No `?.`, `??`, lambdas, `var`, `auto`, generics | — | explicit types + null checks |
| Function calls on ONE line | multi-line arg lists | single line, or extract variables |
| No redeclaring a name in nested scope | `int v;` then `int v;` in `if` | declare once, reuse |
| `ref` only on member vars / typedefs | `ref` in params, returns, locals | `ref MyClass m_Obj;` members only |
| No `delete` for managed objects | `delete obj;` | set to `null`, GC handles it |
| `modded class` never has `: Parent` | `modded class ItemBase : X` | `modded class ItemBase` |
| `override` + `super` call in overrides | missing either | `override void Fn(){ super.Fn(); ... }` |
| Prefix members in modded classes | `m_Flag` | `m_MyMod_Flag` (mod prefix) |
| Layer order: lower never references higher | `3_Game` using `PlayerBase` | move code up or use RPC/invokers |
| RPC enum values start ≥ 10000 | vanilla range collision | `MYMOD_RPC_START = 10000` |
| `Param6<...>` read as ONE object | `ctx.Read(field)` per field | `ctx.Read(data)` once |
| Existing mod's `requiredVersion` is frozen | changing it | keep as-is (ModStorage data loss); new mods: `0.1` |
| Tabs for indentation | spaces | tabs |

Full ruleset: `Frameworks/Safe-AI-CodingPrompt.md`. Style: `Tips/EnScript-Style-Guide.md`.

## Task Routing

| Task | Read first |
|---|---|
| New mod / config.cpp / requiredAddons | `How-To/How-To-Create-Mod.md`, `How-To/Basic-Mod-Structure.md` |
| Which layer does code go in? | `How-To/Script-Layers-Guide.md` |
| Server↔client data (RPC) | `How-To/How-To-RPC.md`; CF variant: `Frameworks/Community-Framework/CF/How-To-CF-RPC.md` |
| `ref` / leaks / segfaults | `Tips/Tips-Memory-Management.md`, `Tips/Tips-Common-Pitfalls.md` |
| modded class / override behavior | `Tips/Tips-Modded-Classes.md`, `Tips/Tips-Override-Keyword.md` |
| Compiler errors, weird crashes | `Tips/Tips-Common-Pitfalls.md` |
| UI menus / layouts / widgets / MVC | `Layouts/README.md`, `Layouts/Reference/Widget-Types-Reference.md`, `How-To/How-To-UI-Menus.md` |
| Player actions (hold/continuous) | `How-To/How-To-Actions.md` |
| Crafting recipes | `How-To/How-To-Recipes.md` |
| JSON profile settings + validation | `How-To/How-To-Profile-Settings.md`, `How-To/How-To-Validate-Config-Data.md` |
| Enums / logging / module singletons | `How-To/How-To-Enums.md`, `How-To/How-To-Logger.md`, `How-To/Module-System.md` |
| CF logging/notifications/ModStorage | `Frameworks/Community-Framework/CF/` |
| Expansion / Dabs integration | `Frameworks/DayZ-Expansion.md`, `Frameworks/Dabs-Framework.md` |
| DayZ 1.29 breaking changes | `DayZGame/DayZ-1.29.161219.md`, `DME_129_Audit_Prompt.md` |

## Verifying APIs

Before using an unfamiliar method, verify it exists — in this order:
1. DayZ Expansion source (local clone if available, else https://github.com/salutesh/DayZ-Expansion-Scripts)
2. Community Framework source (local clone if available, else https://github.com/Arkensor/DayZ-CommunityFramework)
3. Dabs Framework source (local clone if available, else https://github.com/InclementDab/DayZ-Dabs-Framework)
4. Vanilla: https://dayzexplorer.zeroy.com/index.html

If a signature can't be verified, say so instead of guessing.

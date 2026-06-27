# DME Custom Events - DayZ 1.29 Kompatibilitaets-Audit (Multi-Agent Orchestration)

## Auftrag

Fuehre ein vollstaendiges Code-Audit des Mods **DME_Custom_Events** durch, um alle Fehlerquellen und Inkompatibilitaeten mit **DayZ 1.29+** zu finden. Nutze dafuer parallele Sub-Agenten. Aendere in Phase 1 und 2 NICHTS - nur analysieren und reporten.

## Referenz-Wissensbasis (autoritativ)

Alle Regeln und Patterns stammen aus:
`C:\Users\Administrator\Desktop\Mod Repositories\DAYZ_Enforce-Script-main\`

Lies bei Bedarf die Dateien in diesen Unterordnern:
- `Tips/` - Alle EnScript Best Practices und Common Pitfalls
- `How-To/` - RPC, Actions, Menus, ModStorage, Profile Settings, etc.
- `Frameworks/` - CF, Expansion, Dabs Framework Referenz
- `DayZGame/DayZ-1.29.161219.md` - Breaking Changes 1.29

## Mod-Verzeichnis (Ziel des Audits)

`C:\Users\Administrator\Desktop\DME_Custom_Events\DME_Custom_Events\scripts\`

---

## Phase 1: Parallele Analyse - Starte diese 5 Agenten GLEICHZEITIG

---

### Agent 1: GetGame() -> g_Game Migration

**Aufgabe:** Finde ALLE `GetGame()` Aufrufe und alle unzuverlaessigen Client/Server-Checks im gesamten Mod.

**Regeln (aus Tips/Tips-g_Game-GetGame.md):**
- `GetGame()` ist DEPRECATED - darf NICHT mehr verwendet werden ab DayZ 1.29
- Ersetze JEDEN `GetGame()` Aufruf mit `g_Game`
- Vor JEDER `g_Game` Nutzung MUSS ein Null-Check stehen: `if (!g_Game) return;`
- Bei inline-Nutzung: `if (g_Game && g_Game.IsDedicatedServer())`
- Bei mehrfacher Nutzung in einer Funktion: einmal am Anfang pruefen, dann sicher verwenden
- `IsClient()` und `IsServer()` sind UNZUVERLAESSIG - NUR `IsDedicatedServer()` verwenden

**Suche nach:**
```
GetGame()
GetGame().
DayZGame game = GetGame()
GetGame().IsClient()
GetGame().IsServer()
g_Game.IsClient()    <- auch das ist unzuverlaessig
g_Game.IsServer()    <- auch das ist unzuverlaessig
.IsClient()
.IsServer()
```

**Fuer jede Fundstelle zeige:**
1. Datei + Zeilennummer + aktuellen Code
2. Korrigierten Code mit `g_Game` + passendem Null-Check
3. Falls `IsClient()` -> `!IsDedicatedServer()` ersetzen
4. Falls `IsServer()` -> `IsDedicatedServer()` ersetzen

**Korrektur-Beispiele:**
```c
// VORHER (falsch):
if (GetGame().IsDedicatedServer()) { ... }
GetGame().RPC(null, RPC_ID, data, false);
GetGame().GetCallQueue(CALL_CATEGORY_GAMEPLAY).CallLater(MyFunc, 1000, false);

// NACHHER (korrekt):
if (!g_Game) return;
if (g_Game.IsDedicatedServer()) { ... }
g_Game.RPC(null, RPC_ID, data, false);
g_Game.GetCallQueue(CALL_CATEGORY_GAMEPLAY).CallLater(MyFunc, 1000, false);

// VORHER (unzuverlaessig):
if (GetGame().IsClient()) { ... }
if (GetGame().IsServer()) { ... }

// NACHHER (zuverlaessig):
if (!g_Game.IsDedicatedServer()) { ... }   // Client-Check
if (g_Game.IsDedicatedServer()) { ... }    // Server-Check
```

---

### Agent 2: EnScript Syntax & Common Pitfalls

**Aufgabe:** Pruefe den gesamten Mod auf verbotene Syntax und bekannte EnScript-Fallen.

**Pruefe auf diese Fehler (aus Tips/Tips-Common-Pitfalls.md, Tips/Tips-Code-Structure.md):**

1. **Ternary-Operator** (`? :`) - EnScript unterstuetzt KEINEN Ternary. Suche nach Pattern `= expr ? val1 : val2`. Jede Fundstelle -> if-else Ersatz zeigen.

2. **Multi-Variable-Deklarationen** - `int a, b, c;` verursacht Compile-Fehler. Jede Variable muss einzeln deklariert werden.

3. **Mehrzeilige Funktionsaufrufe** - Funktionsaufrufe duerfen NICHT ueber mehrere Zeilen gebrochen werden. Parameter in temporaere Variablen extrahieren wenn noetig.

4. **`delete` Keyword** - NIEMALS verwenden. Stattdessen `= null` setzen.

5. **`auto` Keyword** - Nicht unterstuetzt in vanilla EnScript.

6. **Optional Chaining `?.`** - Nicht unterstuetzt.

7. **Null Coalescing `??`** - Nicht unterstuetzt.

8. **Lambdas** - Nicht unterstuetzt.

9. **Leere `#ifdef/#endif` Bloecke** - Verursachen Segfaults. Muessen mindestens ein Statement enthalten.

10. **Variablen-Redekleration** - Gleicher Variablenname in verschachteltem Scope verursacht "Variable already declared" Fehler.

**Fuer jede Fundstelle zeige:** Datei + Zeile + aktueller Code + korrigierter Code.

---

### Agent 3: Memory Management, ref & Crash-Risiken

**Aufgabe:** Pruefe `ref` Verwendung, Memory-Patterns und bekannte Crash-Ursachen.

**Regeln (aus Tips/Tips-Memory-Management.md, Tips/Tips-Common-Pitfalls.md, Tips/Tips-Best-Practices.md):**

**ref-Regeln:**
- `ref` auf Member-Variablen: PFLICHT fuer Objekt-Referenzen (verhindert vorzeitige GC)
- `ref` auf Parameter: VERBOTEN (Compile-Fehler)
- `ref` auf Return-Types: VERBOTEN (Compile-Fehler)
- `ref` auf lokale Variablen: VERBOTEN (Compile-Fehler)
- `delete obj;` VERBOTEN - verwende `obj = null;`

**Crash-Risiken:**
- Komplexe Ausdruecke direkt in Array-Index-Zuweisung -> Segfault:
  ```c
  // FALSCH (Segfault):
  m_Array[i] = SomeFunc() <= value;
  // RICHTIG:
  bool temp = SomeFunc() <= value;
  m_Array[i] = temp;
  ```
- `GetObjectsAtPosition` / `GetObjectsAtPosition3D` -> NIEMALS verwenden
- Null-Pointer nach `.Cast()` ohne Check
- Null-Pointer nach `GetParent()`, `FindAttachment()`, `GetIdentity()` ohne Check
- `new` in tight Loops -> GC-Druck, Variable ausserhalb deklarieren
- `g_Game.GetCallQueue()` kann null sein - immer pruefen
- `1 < int.MIN` gibt TRUE in EnScript - Integer-Grenzwert-Vergleiche vermeiden

**Fuer jede Fundstelle zeige:** Datei + Zeile + Problem + Fix-Vorschlag.

---

### Agent 4: RPC System & Netzwerk-Korrektheit

**Aufgabe:** Pruefe alle RPC-Implementierungen auf Korrektheit und 1.29-Kompatibilitaet.

**Regeln (aus How-To/How-To-RPC.md, Frameworks/Community-Framework/CF/How-To-CF-RPC.md):**

**Native DayZ RPC:**
- MUSS `g_Game.RPC()` verwenden (NICHT `GetGame().RPC()`)
- RPC Enum-Werte MUESSEN >= 10000 sein (Vanilla-Konflikte vermeiden)
- Client-Handler MUSS in `5_Mission/MissionGameplay.c` registriert sein
- Server-Handler MUSS in `5_Mission/MissionServer.c` registriert sein
- Ohne diese Registrierung scheitern RPCs STILL (kein Fehler, einfach nichts passiert)
- `CallType` MUSS im Handler geprueft werden

**CF RPC (falls verwendet):**
- MUSS `CF_GetRPCManager().SendRPC()` verwenden - NICHT `g_Game.RPC()`
- Handler-Signatur: `void Handler(Param4<CallType, ParamsReadContext, PlayerIdentity, Object> data)`
- Param6 etc. MUSS als ganzes Objekt gelesen werden: `ctx.Read(typedParam)` - NICHT Feld fuer Feld

**Server/Client Checks in RPC-Handlern:**
- NIEMALS `IsClient()` oder `IsServer()`
- IMMER `IsDedicatedServer()` oder CallType-Check
- Menus/UI duerfen nur Client-seitig geoeffnet werden

**Null-Checks:**
- `g_Game` vor jedem RPC-Call
- `PlayerIdentity` vor Zugriff
- Alle RPC-Parameter validieren

**Dateien mit hoher Prioritaet:**
```
5_Mission/DME_ZedWave_RPC.c
5_Mission/DME_CodeBreaker_RPC.c
5_Mission/DME_HeliEvent_RPC.c
5_Mission/MissionGameplay.c
5_Mission/MissionServer.c
4_World/DME_HeliEvent_RPCHelper.c
```

---

### Agent 5: Modded Classes, Script Layer & config.cpp

**Aufgabe:** Pruefe Mod-Struktur, Klassen-Definitionen und Konfiguration.

**Modded Class Regeln (aus Tips/Tips-Modded-Classes.md, Tips/Tips-Override-Keyword.md):**
- `modded class X` darf KEINE Vererbungssyntax haben (`: SomeClass`)
- `override` ist PFLICHT auf allen ueberschriebenen Methoden
- `super.Method()` muss ZUERST aufgerufen werden (besonders bei Serialisierung)
- Member-Variablen in modded Classes MUESSEN Mod-Prefix haben: `m_DME_VarName`
- Fehlende `override` erzeugt Warning: "FIX-ME: Overriding function but not marked as override"

**Naming-Regeln (aus Tips/Tips-Prefixes-Naming.md):**
- Alle neuen Klassen MUESSEN Prefix haben: `DME_ClassName`
- Alle Enums MUESSEN Prefix haben: `EDME_EnumName`
- JSON Config-Klassen: Member-Variablen NICHT prefixen (JSON Key Matching!)
- RPC Enum-Werte >= 10000

**Script Layer Regeln (aus How-To/Script-Layers-Guide.md):**
- `modded class ItemBase/PlayerBase/EntityAI` MUSS in 4_World
- `MissionGameplay/MissionServer` MUSS in 5_Mission
- `UIScriptedMenu` MUSS in 5_Mission
- Actions MUESSEN in 4_World
- Constants/Configs gehoeren in 3_Game
- Niedrigere Layer duerfen NICHT hoehere referenzieren

**config.cpp Regeln (aus How-To/Basic-Mod-Structure.md):**
- `requiredVersion` muss `0.1` sein
- `requiredAddons[]` nur `"DZ_Data"`
- `files[]` Pfade muessen mit tatsaechlicher Ordnerstruktur uebereinstimmen
- Pfade in config.cpp: einfache Backslashes (KEINE doppelten)

**Action-Registrierung (aus How-To/How-To-Actions.md):**
- Jede Custom Action MUSS in `modded class ActionConstructor.RegisterActions()` registriert sein
- Fehlende Registrierung -> Action funktioniert nicht (kein Fehler, einfach nichts)
- `super.RegisterActions(actions)` muss ZUERST aufgerufen werden

**Pruefe:**
- `config.cpp` im Mod-Root
- Alle `modded class` Definitionen
- Alle `override` Methoden
- Action-Registrierung in den relevanten Dateien
- Layer-Violations (Code in falscher Schicht)

---

## Phase 2: Zusammenfuehrung & Priorisierte Fix-Liste

Nachdem alle 5 Agenten fertig sind, erstelle:

### 1. Zusammenfassung
- Anzahl Probleme pro Agent/Kategorie
- Betroffene Dateien auflisten

### 2. Priorisierte Fix-Liste
Sortiert nach Schwere:

| Prio | Kategorie | Beschreibung |
|------|-----------|-------------|
| P0 | Compile-Fehler | Blockiert Build komplett (Ternary, Multi-Var, delete, auto) |
| P1 | Segfault/Crash | Runtime-Crashes (leere ifdef, Array-Segfault, fehlende Null-Checks) |
| P2 | 1.29 Breaking | GetGame() Migration, IsClient/IsServer Ersatz |
| P3 | Silent Failures | Fehlende Action/RPC-Registrierung, RPC-Handler-Fehler |
| P4 | Best Practices | ref-Korrekturen, Naming, Override, Layer-Fixes |

### 3. Frage den User
- Sollen alle Fixes automatisch angewendet werden?
- Oder Kategorie fuer Kategorie mit Review dazwischen?
- Oder nur bestimmte Prioritaeten?

---

## Phase 3: Fixes anwenden (NUR nach User-Bestaetigung)

Wende Fixes in dieser Reihenfolge an:
1. P0: Compile-Fehler beheben
2. P1: Crash-Risiken eliminieren
3. P2: GetGame() -> g_Game Migration (wichtigste 1.29-Aenderung)
4. P3: Silent Failures fixen
5. P4: Best Practices anwenden

Nutze parallele Agents (max 5) um mehrere Dateien gleichzeitig zu fixen.
Nach allen Fixes: `git diff` zeigen zur finalen Review.

---

## Absolute Regeln fuer ALLE Agents

1. **Phase 1+2: NUR LESEN** - Keine Dateien aendern
2. **Vollstaendige Fundstellen**: Datei + Zeile + aktueller Code + vorgeschlagener Fix
3. **Keine Logik-Aenderungen**: Nur syntaktische/API-Korrekturen
4. **Referenz-Dateien nutzen**: Bei Unsicherheit die Dateien in `DAYZ_Enforce-Script-main/` lesen
5. **Keine falschen Positiven**: Nur echte, verifizierte Probleme melden
6. **Kontext beachten**: Code im Kontext der umgebenden Funktion lesen, nicht isoliert
7. **Bestehende Funktionalitaet erhalten**: Keine neuen Features, kein Refactoring
8. **Server/Client Replikation**: Pruefen ob Aenderungen die Authority-Regeln brechen koennten

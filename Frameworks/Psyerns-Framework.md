# Psyerns Framework

Psyerns Framework is a **standalone, dependency-free HTTP / Webhook framework** for DayZ Standalone. It is built entirely on the engine-native `RestApi`, so it needs **no external programs, companion services, or root server access** — everything runs inside the DayZ server process.

**⚠️ Server-side:** All transport and REST logic runs on the dedicated server. The framework is **optional to depend on** — other mods integrate against it with `#ifdef Psyerns_Framework` and degrade gracefully when it is absent.

## Overview

**Location:** `Mods/Psyerns_Framework/`

Psyerns Framework provides:
- HTTP client (GET / POST) over the native `RestApi`
- Async request queue with adaptive rate limiting and retries
- Discord webhook embeds
- WordPress REST API integration
- A safe, fluent JSON builder
- A single unified JSON config file with version auto-upgrade and auto-generated API keys
- Live config reload in-game (F9)
- File + RPT logging with automatic secret masking
- Drop-in REST modules: whitelist, player lookup, server status, kill feed, zone alerts, Discord events, quest webhooks, leaderboard export
- Optional mod integrations (Auction House, Terje Skills) behind `#ifdef`

## Structure

```
Psyerns_Framework/
├── config.cpp
├── mod.cpp
├── data/                                   # default config seeds, modded_inputs.xml (F9)
└── scripts/
    ├── 3_Game/Psyerns_Framework/
    │   ├── Logging/        PF_Logger.c
    │   ├── RPC/            PF_RPCConstants.c
    │   ├── Utils/          PF_HttpArguments.c, PF_JsonBuilder.c
    │   ├── Web/
    │   │   ├── PF_WebClient.c, PF_WebRequest.c, PF_WebResponse.c
    │   │   ├── Config/         PF_WebConfig.c, PF_WebEndpoint.c
    │   │   ├── Notifications/  PF_ServerNotifications.c
    │   │   ├── Payload/        PF_JsonPayload.c, PF_DiscordPayload.c, PF_WordPressPayload.c
    │   │   ├── Queue/          PF_WebQueue.c, PF_WebQueueItem.c
    │   │   ├── RestCallback/   PF_RestCallback.c
    │   │   └── WebApi/         PF_WebApiBase.c, PF_DiscordWebhook.c, PF_WordPressApi.c
    │   ├── REST/
    │   │   ├── Base/        PF_RestBase.c
    │   │   ├── Config/      PF_RestConfig.c
    │   │   ├── Discord/     PF_DiscordIntegration.c
    │   │   ├── Leaderboard/ PF_LeaderboardReader.c, PF_LeaderboardExport.c
    │   │   ├── PlayerLookup/PF_PlayerLookup.c
    │   │   ├── ServerStatus/PF_ServerStatus.c
    │   │   ├── TopGames/    PF_TopGamesVoteService.c
    │   │   └── Whitelist/   PF_WhitelistManager.c
    │   └── Integrations/    AuctionHouse/ (PF_AH_*), TerjeSkills/ (PF_Terje*)
    ├── 4_World/Psyerns_Framework/
    │   ├── PF_WebQueueProcessor.c          # pumps the queue on MissionServer.OnUpdate
    │   └── REST/  PF_KillFeedHook.c, Alerts/PF_AlertSystem.c, KillFeed/PF_KillFeedManager.c, Quests/PF_QuestWebhook.c
    └── 5_Mission/Psyerns_Framework/
        ├── PF_MissionInit.c                # server boot, config load, queue start
        ├── PF_MissionClient.c              # client F9 reload request
        └── PF_RestInit.c                   # wires up enabled REST modules
```

## Key Features

### Web Client & Requests

`PF_WebClient` is a singleton wrapping the engine `RestApi`. It caches one `RestContext` per base URL and dispatches `PF_WebRequest` objects. `PF_WebRequest` is a fluent builder (default method is **POST**, default header `application/json`):

```c
PF_WebClient client = PF_WebClient.GetInstance();

PF_WebRequest req = new PF_WebRequest();
req.SetUrl("https://api.example.com");
req.SetEndpoint("/data");
req.SetBody("{\"key\":\"value\"}");
req.Post();   // or .Get();

client.Send(req);
```

| Method | Purpose |
|--------|---------|
| `PF_WebClient.GetInstance()` | Get the singleton client |
| `client.Send(PF_WebRequest)` | Dispatch a request (async via `RestContext`) |
| `client.GetRestContext(baseUrl)` | Get / create a cached `RestContext` |
| `req.SetUrl / SetEndpoint / SetBody / SetHeader` | Configure the request (fluent) |
| `req.Post() / req.Get()` | Set HTTP method |

### JSON Builder

`PF_JsonBuilder` builds JSON safely with automatic string escaping — never hand-concatenate JSON:

```c
PF_JsonBuilder b = PF_JsonBuilder.Begin();
b.Add("name", "PlayerOne");
b.AddInt("kills", 15);
b.AddBool("online", true);
string json = b.Build();
// {"name":"PlayerOne","kills":15,"online":true}
```

| Method | Adds |
|--------|------|
| `Add(key, string)` | Escaped string value |
| `AddInt / AddFloat(key, n)` | Numeric value |
| `AddBool(key, bool)` | `true` / `false` |
| `AddArray(key, array<string>)` | String array |
| `AddObject(key, PF_JsonBuilder)` | Nested object |
| `AddRaw(key, rawJson)` | Pre-built JSON (no escaping) |
| `Build()` | Returns the final `{...}` string |

### Discord Webhooks

`PF_DiscordWebhook` (a `PF_WebApiBase` subclass) sends rich embeds. Build them with `PF_DiscordPayload` / `PF_DiscordEmbed`:

```c
PF_DiscordWebhook discord = new PF_DiscordWebhook("WEBHOOK_ID", "WEBHOOK_TOKEN");

PF_DiscordPayload payload = new PF_DiscordPayload();
payload.username = "My DayZ Server";

PF_DiscordEmbed embed = payload.CreateEmbed();
embed.SetTitle("Server Status");
embed.SetDescription("The server is online.");
embed.SetColor(3066993);
embed.AddField("Players", "42/60", true);
embed.AddField("Uptime", "12h 34m", true);

discord.Send(payload);
```

Quick one-liner without building a payload:

```c
discord.SendSimple("Title", "Description text", 3066993); // color = decimal RGB
```

### WordPress REST API

`PF_WordPressApi` posts data to a WordPress site (the included companion plugin), auto-appending the `api_key` query argument:

```c
PF_WordPressApi wordpress = new PF_WordPressApi("https://mysite.com/wp-json/psyern/v1", "MY_KEY");

PF_WordPressPayload payload = new PF_WordPressPayload();
payload.generatedAt = "2026-03-24T12:00:00Z";
payload.totalPlayers = 42;
wordpress.UploadLeaderboard(payload);   // POST /upload?api_key=...

wordpress.Ping();                        // GET /ping?api_key=...  (connection test)
```

### Custom API Targets

Extend `PF_WebApiBase` to add your own REST target. The base handles the `RestContext` and exposes `Post(endpoint, data)` / `Get(endpoint)`:

```c
class MyMod_Api : PF_WebApiBase
{
	void MyMod_Api(string baseUrl)
	{
		m_BaseUrl = baseUrl;
		m_RestContext = m_Rest.GetRestContext(m_BaseUrl);
		m_RestContext.SetHeader("application/json");
	}

	void PushEvent(string json)
	{
		Post("/events", json);
	}
}
```

### Async Queue & Rate Limiting

All outbound HTTP is processed through `PF_WebQueue`, pumped by `PF_WebQueueProcessor` on `MissionServer.OnUpdate`:

- Requests are queued and sent one at a time
- Adaptive rate limiting (interval shrinks on success, backs off on failure)
- Failed requests retried up to `DefaultRetryCount`
- Per-endpoint minimum interval via `RateLimitMs`

### Logging

`PF_Logger` is a static logger writing to both the server RPT and a dated file. Debug output automatically **masks `api_key=` secrets**:

```c
PF_Logger.Init(PF_WebConfig.GetInstance().EnableDebugLogging);

PF_Logger.Log("Always shown");
PF_Logger.Error("Always shown, marked [ERROR]");
PF_Logger.Debug("Only when EnableDebugLogging = true");
```

- Log file: `$profile:Psyerns_Framework\Logs\PF_Log_YYYY-MM-DD.log`
- All entries prefixed `[Psyerns Framework]` and timestamped

### Unified Config

`PF_WebConfig` is a single JSON-backed singleton with **version auto-upgrade** and **auto-generated API keys**:

```c
PF_WebConfig cfg = PF_WebConfig.GetInstance();

PF_WebEndpoint wp = cfg.GetEndpoint("WordPress");
if (cfg.IsEndpointEnabled("Discord"))
{
	// Discord endpoint is active
}

if (cfg.IsAdmin(steam64Id))
{
	// player is a configured admin
}
```

- Path: `$profile:DeadmansEcho\PsyernsFramework\PsyernsFrameworkConfig.json`
- `CURRENT_VERSION = 4` — on load, missing fields are added and `ConfigVersion` is bumped, then the file is re-saved (existing values preserved)
- Empty / placeholder `ApiKey` values are replaced with a generated `pf-xxxx…` key on startup; the `WordPress` and `Leaderboard` endpoints **share one key**
- `PF_WebConfig.Reload()` re-reads the file at runtime (used by the F9 reload)

### Live Config Reload (F9)

Admins listed in `AdminIDs` can reload the config in-game without a restart. The client sends a reload request to the server over the framework's RPC channel; the server calls `PF_WebConfig.Reload()` and confirms via chat. The RPC identifiers live in `PF_RPCConstants`:

```c
const string PF_RPC_CHANNEL          = "Psyerns_Framework";
const string PF_RPC_RELOAD_REQUEST   = "PF_ReloadRequest";
const string PF_RPC_RELOAD_RESPONSE  = "PF_ReloadResponse";
```

The F9 keybind is defined in `data/modded_inputs.xml` (rebindable under DayZ Settings → Controls → **PF** tab).

### REST Modules

Each module is toggled by an `Enable*` flag in the config and wired up by `PF_RestInit` when enabled:

| Module | Class | Purpose |
|--------|-------|---------|
| Whitelist | `PF_WhitelistManager` | Check / add / remove via REST |
| Player Lookup | `PF_PlayerLookup` | Player data + online status queries |
| Server Status | `PF_ServerStatus` | Periodic status push to WordPress |
| Kill Feed | `PF_KillFeedManager` (+ `PF_KillFeedHook`) | PvP/PvE kill events to webhook URLs |
| Discord Events | `PF_DiscordIntegration` | Connect / disconnect / kill embeds |
| Alerts | `PF_AlertSystem` | Zone-based trigger webhooks |
| Quests | `PF_QuestWebhook` | Expansion quest completion (`#ifdef`) |
| Leaderboard | `PF_LeaderboardReader` / `PF_LeaderboardExport` | Reads tracking JSONs, POSTs to WordPress |

## Key Classes

### Transport
- `PF_WebClient` — singleton HTTP client over `RestApi`
- `PF_WebRequest` / `PF_WebResponse` — request / response models
- `PF_WebApiBase` — base class for REST targets (`Post` / `Get`)
- `PF_WebQueue` / `PF_WebQueueItem` / `PF_WebQueueProcessor` — async send queue

### Payloads & Utils
- `PF_JsonBuilder` — safe fluent JSON builder
- `PF_HttpArguments` — query-string builder (`api_key`, etc.)
- `PF_DiscordPayload` / `PF_DiscordEmbed` / `PF_DiscordEmbedField` — Discord embeds
- `PF_WordPressPayload` — WordPress upload payload

### API Targets
- `PF_DiscordWebhook` — Discord webhook sender
- `PF_WordPressApi` — WordPress REST client

### Infrastructure
- `PF_WebConfig` / `PF_WebEndpoint` — unified config + endpoints
- `PF_Logger` — static file + RPT logger with secret masking
- `PF_RPCConstants` — RPC channel + reload message names

## Using Psyerns Framework

### Optional Dependency

Other mods integrate **behind a preprocessor guard** so they still load when the framework is absent:

```c
#ifdef Psyerns_Framework
PF_WebClient client = PF_WebClient.GetInstance();
// ... send requests
#endif
```

### Integration Points

1. Use `PF_WebClient.GetInstance()` to send generic requests.
2. Extend `PF_WebApiBase` for a custom REST target.
3. Read endpoint config with `PF_WebConfig.GetInstance().GetEndpoint("name")`.
4. Build JSON with `PF_JsonBuilder` (never concatenate by hand).
5. Use `PF_RPCConstants` for the channel when communicating between server and client scripts.

## Best Practices

### 1. Build JSON with PF_JsonBuilder

```c
// ✅ Good - escaped, safe
string json = PF_JsonBuilder.Begin().Add("name", playerName).AddInt("score", score).Build();

// ❌ Avoid - breaks on quotes / special chars
string json = "{\"name\":\"" + playerName + "\"}";
```

### 2. Guard integration with #ifdef

```c
// ✅ Good - degrades gracefully when the framework isn't loaded
#ifdef Psyerns_Framework
PF_WebClient.GetInstance().Send(req);
#endif
```

### 3. Read endpoints from config, don't hard-code URLs

```c
// ✅ Good
PF_WebEndpoint ep = PF_WebConfig.GetInstance().GetEndpoint("WordPress");

// ❌ Avoid - hard-coded, not admin-configurable
string url = "https://mysite.com/wp-json/psyern/v1";
```

### 4. Log with PF_Logger, not Print

```c
// ✅ Good - file + RPT, secret masking, debug gating
PF_Logger.Debug("Sending to " + url);

// ❌ Avoid
Print("Sending to " + url);
```

## Configuration Reference

The single config file lives at `$profile:DeadmansEcho\PsyernsFramework\PsyernsFrameworkConfig.json`.

| Field | Default | Purpose |
|-------|---------|---------|
| `ConfigVersion` | `4` | Internal version for auto-upgrade (do not edit) |
| `EnableDebugLogging` | `false` | Verbose debug to log + RPT |
| `DefaultRetryCount` | `3` | Retries for failed requests |
| `QueueMaxSize` | `100` | Max queued requests |
| `ServerName` | `"DayZ Server"` | Name shown in notifications/embeds |
| `Endpoints[]` | WordPress, Discord, Leaderboard, TopGames | Per-target `Name` / `BaseUrl` / `ApiKey` / `Enabled` / `RateLimitMs` |
| `AdminIDs[]` | — | Steam64 IDs allowed to F9-reload |
| `Enable*` flags | `false` | Toggle each REST / notification module |

> **Auto-upgrade & key generation** are handled in `PF_WebConfig.Load()` — see [How to Create Profile Settings](../How-To/How-To-Profile-Settings.md) and [How to Validate Config Data](../How-To/How-To-Validate-Config-Data.md).

## Related Documentation

- [How to Create Profile Settings](../How-To/How-To-Profile-Settings.md) — config saved to the profile folder
- [How to Validate Config Data](../How-To/How-To-Validate-Config-Data.md) — validating JSON values
- [How to Create a Logger](../How-To/How-To-Logger.md) — logging patterns
- [How to Use RPC](../How-To/How-To-RPC.md) — RPC between server and client
- [DayZ Expansion](DayZ-Expansion.md) — quest system consumed by `PF_QuestWebhook` (`#ifdef`)

## Notes

- Standalone and **zero-dependency** — built only on the engine `RestApi`
- All transport runs server-side; the outbound queue is pumped on `MissionServer.OnUpdate`
- Integration is **optional** via `#ifdef Psyerns_Framework`
- The config is a single self-healing JSON file (auto-upgrade + auto-generated keys)
- Optional WordPress companion plugins consume the REST API and Discord webhooks

---

**Location:** `Mods/Psyerns_Framework/`  
**Author:** [Psyern](https://steamcommunity.com/profiles/76561198043039918/) · Community: [Deadmans Echo](https://deadmansecho.com)

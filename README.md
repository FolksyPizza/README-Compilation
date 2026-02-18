# PizzaSMP Server Configuration

This README documents the current live server configuration.

Last updated: 2026-02-18

## Canonical Command + Plugin Reference

- Full organized inventory is now maintained in:
  - `docs/COMPLETE_PLUGIN_COMMAND_REFERENCE.md`
- This includes:
  - plugin descriptions
  - plugin-to-plugin dependency/soft-dependency links
  - command chapters per plugin
  - MyCommand custom command mappings (`/forgive`, `/unpunish`, `/bancheck`)
  - LibertyBans alias command coverage from live config
  - full Essentials root command list
- Regenerate after plugin/config command changes:
```bash
python tools/generate_plugin_reference.py
```

## 0.5 Staff Command Hub + Plugin Admin Gate (2026-02-18)

- Added custom help hubs:
  - `/pizzaadmintools help`
  - `/pizzateams help`
  - `/pizzamenus help`
  - `/pizzahome help`
  - `/pizzabans help`
  - `/pizzachatguard help`
- Added branded plugin list command:
  - `/pizzaplugins` (staff-only)
  - outputs branded stack names (PizzaTeamsGUI, PizzaTeams, PizzaMenus, PizzaAC, PizzaBans, PizzaHome, etc.)
- Added plugin/admin command gate:
  - `/pl`, `/plugins`, `/lp`, `/luckperms`, `/version`, `/ver`, `/about`, `/paper`, `/timings`
  - blocked for `default` and `pizza+`
  - available to staff with `pizzasmp.pluginadmin`
- ChatGuard message update:
  - blocked mild message text is now `Message Blocked By Filter.`
- ChatGuard severe-word coverage expanded:
  - added many extra variant spellings in `instant_ban_words.txt`
  - added fuzzy severe matching (near-miss typo/incomplete detection) in `PizzaChatGuard` code
- Added warning-clear utility:
  - `/clearallwarnings [player]`
  - with no player: clears all PizzaChatGuard warning entries
  - with player: clears that player warning entry
- Moderation command ownership unified:
  - `/punish`, `/unpunish`, `/forgive`, `/bancheck` are now owned by `PunishDrop`.
  - `commands.yml` aliases force these roots to `punishdrop:*` namespace.
  - MyCommand moderation definitions were removed from active command config (`plugins/MyCommand/commands/punish.yml` now `{}`).
  - `PunishDrop` unban flow now includes short retry handling to resolve punish/unpunish race timing.
- Ban-screen consistency fix:
  - `PunishDrop` now writes the same PizzaSMP-format ban message into fallback Bukkit name-bans.
  - reconnect/join attempts now keep consistent ban-screen text instead of switching to mismatched formatting.

## 0.4 Chat Warning Clear Command (2026-02-17)

- Added `PizzaChatGuard` staff command:
  - `/clearwarnings <player>`
  - aliases: `/clearwarn`, `/clearwarns`
- Purpose: clear a player's accumulated chat warnings so isolated mistakes can be forgiven without forced escalation.
- Permission node:
  - `pizzasmp.chatguard.clearwarnings`
- Granted to groups:
  - `owner`, `dev`, `sradmin`, `admin`, `srmod`

### Staff Moderation Commands (Current)

- `/punish <player> <duration> <reason...>`
- `/unpunish <player>`
- `/forgive <player>`
- `/bancheck <player>`
- `/freeze <player>`
- `/unfreeze <player>`
- `/clearwarnings <player>`
- `/clearallwarnings [player]`

## 0.3 Auth + Ban Format Update (2026-02-17)

- `start.sh` now warms DNS/auth hosts (`api.minecraftservices.com`, `sessionserver.mojang.com`) before Java startup.
- JVM DNS cache tuning updated for auth stability:
  - `-Dnetworkaddress.cache.ttl=1800`
  - `-Dnetworkaddress.cache.negative.ttl=1`
  - `-Dsun.net.inetaddr.ttl=1800`
  - `-Dsun.net.inetaddr.negative.ttl=1`
- Removed JVM DNS nameserver override flags (`1.1.1.1`, `8.8.8.8`) because this host cannot reliably reach public DNS directly; server now uses system resolver path.
- Auth warmup is now strict-fail: if Mojang auth DNS/endpoints fail preflight, `start.sh` aborts instead of starting Paper in a broken auth state.
- 2026-02-18 DNS fix: removed JVM hosts override file usage (`.java-hosts-auth` + `-Djdk.net.hosts.file`) because it blocked non-Mojang domain lookups (notably Geyser skin API).
- Java runtime now relies on system DNS plus startup preflight and conservative JVM DNS cache flags.
- LibertyBans English time fragments were adjusted to remove week/month/year display in favor of day/hour/minute style for new messages.

## 0.1 WebUI Ops Update (2026-02-16)

- WebUI HTTP request spam logging is disabled in `webapp/server.py` (`ApiHandler.log_message` no-op).
- New reliable PaperMC start action is available via WebUI API:
  - `POST /api/server/action` with `{"action":"start_reliable"}`
  - retries startup and verifies running state before reporting success.
- Console behavior in WebUI:
  - main + host console panes now auto-scroll to latest lines on refresh.
  - stop/restart actions clear console panes to blank before fresh logs return.

## 0.2 PunishDrop Kick Timing Update (2026-02-16)

- `PunishDrop` now kicks the target immediately after ban apply during `/punish`.
- Removed next-tick scheduled kick path so online targets are disconnected instantly when command succeeds.
- 2026-02-18 enforcement hardening:
  - `PunishDrop` now applies an immediate Bukkit name-ban first, then mirrors to LibertyBans.
  - This removes the first-offense rejoin window (ban screen without actual deny state).

## 0. Stability Hotfix (2026-02-12)

- Rebuilt and redeployed:
  - `plugins/PunishDrop-1.0.0.jar`
  - `plugins/PizzaAdminTools.jar`
- `/punish` path stabilized:
  - LibertyBans command syntax normalized (`/ban <player> [time] <reason>`)
  - extra save flush after inventory clear before ban command
  - exact temporary unban scheduler retained
- WebUI ban backend migrated to LibertyBans-log parsing:
  - `webapp/server.py` ban action now sends `/punish`
  - active/archive status now derives from parsed `LibertyBans` events in `logs/latest.log`
- Ban disconnect text cleanup:
  - `PizzaAdminTools` now sanitizes pre-login ban messages to strip appeal footer lines
  - removes lines beginning with `Appeal Your Punishment`, `Website:`, and `Discord:`
- LuckPerms model normalized:
  - `default` -> `sethome.maxhomes.2=true`, `sethome.maxhomes.5=false`
  - `pizza+` and staff+ -> `sethome.maxhomes.5=true`, `sethome.maxhomes.2=false`
  - chat-filter enforcement now applies to all groups
  - removed `default` wildcard deny (`*=false`) to prevent global command lockouts
  - reaffirmed `owner` and `dev` wildcard grants (`*=true`, plus `minecraft.command.*`, `bukkit.command.*`, `paper.command.*`)
  - fixed `dev` inheritance conflict by clearing old `staff/default` parent chain and inheriting from `owner` for full command access
- PvP restoration:
  - `plugins/Essentials/config.yml` -> `protect.disable.pvp: true` so player-vs-player damage is allowed
- Command routing hardening:
  - `commands.yml` now aliases these to `PizzaAdminTools` namespace to avoid plugin command collisions:
  - `/freeze`, `/unfreeze`, `/menu`, `/guide`, `/discord`, `/homes`, `/gtp`
- WebUI upgraded to tabbed cloud-panel layout:
  - first screen requires server selection, then opens tabs
  - sections for `Home`, `Overview`, `Servers`, `Console`, `Players`, `Bans`, `Backups`, `Properties`, `Files`, `Users`
  - removed `Appearance` and `Settings` tabs
  - added APIs for backups, properties editing, file browser, and LuckPerms user/group view
  - added `Servers` section for provisioning and managing additional Paper instances under `/ssd/PaperMC`
  - managed server metadata now tracked in `webapp/data/managed_servers.json`
  - provisioning now supports selecting local server jar version and plugin templates
  - per-managed-server plugin enable/disable and properties editing available in WebUI
  - `Properties` tab now edits raw `server.properties` with save + reload
  - `Files` tab now supports browse/open/edit/delete with optional reload-on-save
  - backups table now shows size in GB
  - chat parser cleanup: only UUID-backed real player chat is included in WebUI chat logs

## 1. Core Networking (No Proxy)

| Component | Port | Protocol | Mode |
|---|---:|---|---|
| Java server | `25569` | TCP | Direct (no proxy) |
| Geyser Bedrock listener | `25568` | UDP | Direct to Paper |

Configured in:

- `server.properties`
  - `server-port=25569`
  - `online-mode=true`
  - `prevent-proxy-connections=true`
- `config/paper-global.yml`
  - `proxies.velocity.enabled=false`

## 2. Geyser and Floodgate

### `plugins/Geyser-Spigot/config.yml`

- `bedrock.port: 25568`
- `bedrock.clone-remote-port: false`
- `java.auth-type: floodgate`
- `gameplay.command-suggestions: false`
- `notify-on-new-bedrock-update: false`

### `plugins/floodgate/config.yml`

- `username-prefix: "."`
- `metrics.enabled: false`

## 3. Public Server Hardening

### 3.1 Update checks and telemetry disabled

- `config/paper-global.yml` -> `update-checker.enabled: false`
- `plugins/Essentials/config.yml` -> `update-check: false`
- `plugins/PlaceholderAPI/config.yml` -> `check_updates: false`
- `plugins/PlaceholderAPI/config.yml` -> `cloud_enabled: false`
- `plugins/Vault/config.yml` -> `update-check: false`
- `plugins/ViaVersion/config.yml` -> `check-for-updates: false`
- `plugins/MyCommand/config.yml` -> `USE_THE_UPDATER: false`
- `plugins/Maintenance/config.yml` -> `update-checks: false`
- `plugins/Punishment/config.yml` -> `updates: false`
- `plugins/Punishment/config.yml` -> `metrics: false`
- `plugins/PaperMC/config.yml` -> `metrics.enabled: false`
- `plugins/Geyser-Spigot/config.yml` -> `notify-on-new-bedrock-update: false`
- `plugins/floodgate/config.yml` -> `metrics.enabled: false`
- `plugins/bStats/config.yml` -> `enabled: false`
- `plugins/PluginMetrics/config.yml` -> `opt-out: true`

### 3.2 Anti-noise and safety hardening

- `bukkit.yml` -> `query-plugins: false`
- `server.properties` -> `broadcast-console-to-ops=false`
- `server.properties` -> `broadcast-rcon-to-ops=false`
- `plugins/LuckPerms/config.yml` -> `broadcast-received-log-entries: false`
- `plugins/LuckPerms/config.yml` -> `commands-allow-op: false`

### 3.3 MyCommand command-surface reduction

In `plugins/MyCommand/config.yml`:

- `BLOCK_LISTENER: false`
- `ITEM_LISTENER: false`
- `SIGN_LISTENER: false`
- `INVENTORY_LISTENER: false`
- `RUN_COMMANDS.SENT_BY_CHAT: false`

Command packs:

- Disabled sample pack:
  - `plugins/MyCommand/commands/examples.yml` -> `plugins/MyCommand/commands/.disabled/examples.yml`
- Active command files:
  - `plugins/MyCommand/commands/punish.yml`
    - Provides only `/unpunish`, `/forgive`, `/bancheck`

### 3.4 Join/quit and message suppression

In `plugins/Essentials/config.yml`:

- `allow-silent-join-quit: true`
- `custom-join-message: "none"`
- `custom-quit-message: "none"`
- `custom-new-username-message: "none"`
- `hide-join-quit-messages-above: 0`
- `delay-motd: -1`

Additional:

- `plugins/Essentials/motd.txt` is empty (0 bytes)
- `plugins/DisableJoinMessage.jar` is active in `plugins/`

## 4. Punishment System Policy

Goal: use **LibertyBans** as the single authoritative punishment backend.

### 4.1 Backend policy

1. Legacy plugin is disabled:
   - `plugins/Punishment-2.0.jar.disabled`
2. `/punish` is handled by `PunishDrop` (plugin command), not MyCommand.
3. MyCommand wrappers are used for:
   - `/unpunish` -> `unban`
   - `/forgive` -> `unban`
   - `/bancheck` -> `history`
4. Web UI and staff tooling should call `/punish <player> <duration> <reason...>` directly.
5. CleanerX and GrimAC escalation commands should target LibertyBans `ban` syntax.
6. Web UI ban history now comes from `webapp/data/webui.db` (`ban_events`), sourced from parsed `LibertyBans` lines.

### 4.2 Staff command formats

- `/punish <player> <duration> <reason...>`
- `/unpunish <player>`
- `/forgive <player>`
- `/bancheck <player>`

Examples:

- `/punish codex 30d Cheating in PvP`
- `/punish codex 45s Combat logging`
- `/unpunish codex Appeal accepted`
- `/bancheck codex`

### 4.3 Chat guard and logging

- `plugins/PizzaChatGuard.jar` is the single chat enforcement source and intercepts modern Paper chat (`AsyncChatEvent`), legacy chat fallback (`AsyncPlayerChatEvent`), and private-message commands (`/msg`, `/tell`, `/w`, `/whisper`, `/dm`, `/pm`, `/m`).
- Policy:
  - Mild list (`warn_words.txt` and mild tokens from `banned_words.txt`): first offense warning, second offense 15-day ban.
  - Severe list (`instant_ban_words.txt`): first offense immediate 30-day ban.
  - Team-chat command flows are blocked on violation; severe terms still trigger punishment.
- Blocked messages are cancelled entirely and are not replaced with `***` in public chat.
- Full chat history is persisted under `logs/chatguard/players/<player>/<YYYY-MM-DD>.log`.
- Flagged content is written to:
  - `logs/chatguard/flags/<player>.log`
  - `logs/chat-filter-flagged.log` (aggregate file used by WebUI)

Temporary duration handling:

- LibertyBans does not accept `s` durations directly.
- PunishDrop normalizes temporary bans to a LibertyBans-safe minute duration and schedules exact unban at true expiry.
- Result: `/punish <player> 5s <reason>` works as a real ~5 second ban.

## 5. Commands and Access

### 5.1 Active command providers

From `PizzaAdminTools`:

- `/menu` (opens DeluxeMenus main menu)
- `/guide` (opens guide menu)
- `/discord`
- `/homes`
- `/gtp`
- `/freeze <player>`
- `/unfreeze <player>`

From `SetHome`:

- `/home`

From `PunishDrop`:

- `/punish`

From `MyCommand`:

- `/unpunish`
- `/forgive`
- `/bancheck`

### 5.2 Command access policy

| Command | Allowed Groups | Denied Groups | Notes |
|---|---|---|---|
| `/menu` | `owner`, `dev`, `co-owner`, `sradmin`, `admin`, `srmod`, `mod`, `pizza+`, `staff`, `default` | - | PizzaAdminTools command that opens `pizzasmp_menu` |
| `/homes` | `owner`, `dev`, `co-owner`, `sradmin`, `admin`, `srmod`, `mod`, `pizza+`, `staff`, `default` | - | PizzaAdminTools homes GUI command |
| `/guide` | `owner`, `dev`, `co-owner`, `sradmin`, `admin`, `srmod`, `mod`, `pizza+`, `default` | - | Public server guide + linking info |
| `/discord` | `owner`, `dev`, `co-owner`, `sradmin`, `admin`, `srmod`, `mod`, `pizza+`, `default` | - | Sends Discord invite message |
| `/home` (SetHome backend + PizzaAdminTools GUI wrapper) | `owner`, `co-owner`, `sradmin`, `admin`, `srmod`, `mod`, `pizza+`, `dev`, `staff`, `default` | - | `/home` opens homes GUI; `/home <slot|name>` teleports with countdown |
| `/gtp <player> [home-name|home-index]` | `owner`, `dev`, `co-owner`, `sradmin`, `admin` | `srmod`, `mod`, `pizza+`, `staff`, `default` | Teleport to another player's SetHome home; no home arg uses first home |
| `/freeze <player>` | `owner`, `dev` | everyone else | Freeze an online player (movement/command lock) |
| `/unfreeze <player>` | `owner`, `dev` | everyone else | Remove freeze state from an online player |
| `/punish <player> <duration> <reason...>` | `owner`, `dev`, `co-owner`, `sradmin`, `admin`, `srmod`, `mod` | `pizza+`, `default` | Staff tempban entrypoint |
| `/unpunish <player>` | `owner`, `dev`, `co-owner`, `sradmin`, `admin`, `srmod`, `mod` | `pizza+`, `default` | Unban command |
| `/forgive <player>` | `owner`, `dev`, `co-owner`, `sradmin`, `admin`, `srmod`, `mod` | `pizza+`, `default` | Alias-style unban |
| `/bancheck <player>` | `owner`, `dev`, `co-owner`, `sradmin`, `admin`, `srmod`, `mod` | `pizza+`, `default` | History lookup |

### 5.3 Installed Plugin Map (Purpose + Commands)

Notes:

- This maps currently installed JAR plugins in `plugins/`.
- Full command list is in `## 7. Full Command Inventory (166 Commands)`.

| Plugin (JAR) | Purpose | Commands owned / provided |
|---|---|---|
| `PizzaAdminTools.jar` | PizzaSMP custom admin/player UX glue (menus, homes wrapper, moderation helpers) | `/gtp`, `/homes`, `/menu`, `/guide`, `/discord`, `/freeze`, `/unfreeze` |
| `PunishDrop-1.0.0.jar` | Staff punishment flow + short-duration tempban normalization for LibertyBans | `/punish` |
| `MyCommand.jar` | Legacy/custom wrappers and aliases | `/unpunish`, `/forgive`, `/bancheck` (from MyCommand command files) |
| `SetHomeGUI.jar` | Home backend and teleport/home UX | `/home` |
| `DeluxeMenus.jar` | GUI framework for custom menus | `/deluxemenus` |
| `LibertyBans.jar` | Core punishment backend (ban/mute/warn/kick) | backend moderation commands (see full inventory) |
| `LuckPerms-Bukkit-5.5.11.jar` | Permissions and groups | `/luckperms` |
| `Vault.jar` | Economy/chat/permission API bridge | `/vault-info`, `/vault-convert` |
| `PlaceholderAPI.jar` | Placeholder expansion API | `/placeholderapi` |
| `Plan.jar` | Analytics/dashboard plugin | `/plan` |
| `TAB.jar` | Tablist/nametag formatting | `/tab` |
| `EssentialsX-2.21.2.jar` | Core utility/teleport/economy/server commands | many Essentials commands (see full inventory) |
| `EssentialsXChat-2.21.2.jar` | Essentials chat formatting module | `/toggleshout` |
| `BetterTeams.jar` | Team management | `/team`, `/tc` and related team commands |
| `BetterTeamsGUI.jar` | BetterTeams GUI frontend | `/teams` |
| `Chunky-Bukkit-1.4.40.jar` | Chunk pre-generation | `/chunky` |
| `ViaVersion-5.5.1.jar` | Cross-version protocol compatibility | `/viaversion` |
| `ViaBackwards-5.5.1.jar` | Backward protocol support for older clients | no direct operator command expected |
| `Geyser-Spigot.jar` | Bedrock <-> Java bridge | managed primarily via config/runtime (no primary staff command required) |
| `floodgate-spigot.jar` | Bedrock account linking/identity integration | `/linkaccount` flow |
| `GrimAC.jar` | Anti-cheat | plugin-managed; staff actions use configured punish pipeline |
| `PizzaChatGuard.jar` | Custom chat moderation + auto-punish workflow | chat-event driven (no primary chat command) |
| `Maintenance-4.3.0.jar` | Maintenance mode controls | `/maintenance` |
| `CleanerX.jar` | Cleanup/maintenance utility plugin | plugin-specific controls (verify in plugin docs/config) |
| `amethyst.jar` | Utility tools plugin | `/tools` |
| `rtp-1.5.jar` | Random teleport feature | `/rtp` |

## 6. LuckPerms Nodes for New Commands

Current command permission nodes:

- `pizzasmp.menu` (enabled for player/staff groups)
- `pizzasmp.homes` (enabled for player/staff groups)
- `pizzasmp.guide` (enabled for player/staff groups)
- `pizzasmp.discord` (enabled for player/staff groups)
- `pizzasmp.freeze` (owner/dev only)
- `pizzasmp.unfreeze` (owner/dev only)
- `sethome.use` (enabled for `default`, `pizza+`, `mod`, `srmod`, `admin`, `sradmin`, `co-owner`, `staff`, `dev`, `owner`)
- `sethome.maxhomes.2` (`default`)
- `sethome.maxhomes.5` (`pizza+`, `mod`, `srmod`, `admin`, `sradmin`, `co-owner`, `dev`, `owner`, `staff`)
- `sethome.admin` (`admin`, `sradmin`, `co-owner`, `dev`, `owner`)
- `pizzasmp.gtp` (`admin`, `sradmin`, `co-owner`, `dev`, `owner`)
- `pizzasmp.punish` (staff-only)
- `mycommand.unpunish` (staff-only)
- `mycommand.forgive` (staff-only)
- `mycommand.bancheck` (staff-only)
- wildcard admin access:
  - `owner` -> `*=true`
  - `dev` -> `*=true`
  - `owner` -> `minecraft.command.*`, `bukkit.command.*`, `paper.command.*`
  - `dev` -> `minecraft.command.*`, `bukkit.command.*`, `paper.command.*`

Staff groups for punishment wrappers:

- `owner`, `dev`, `co-owner`, `sradmin`, `admin`, `srmod`, `mod`

Non-staff denied for punishment wrappers:

- `pizza+`, `default`

Homes overlap policy:

- Essentials homes nodes are explicitly false for player ranks to avoid dual-home-system confusion:
  - `essentials.home`
  - `essentials.sethome`
  - `essentials.delhome`

Homes tier mapping applied:

- `default` -> `sethome.maxhomes.2`
- `pizza+` and all staff ranks -> `sethome.maxhomes.5`

## 7. Full Command Inventory (166 Commands)

Generated from installed plugin manifests (`plugin.yml`) and active MyCommand command files.

```text
/afk
/antioch
/anvil
/back
/backup
/balance
/balancetop
/ban
/bancheck
/banip
/beezooka
/bigtree
/book
/bottom
/break
/broadcast
/broadcastworld
/burn
/cartographytable
/chunky
/clearinventory
/clearinventoryconfirmtoggle
/compass
/condense
/createkit
/customtext
/delhome
/deljail
/delkit
/delwarp
/depth
/discord
/disposal
/eco
/editsign
/enchant
/enderchest
/essentials
/exp
/ext
/feed
/fireball
/firework
/fly
/forgive
/gamemode
/gc
/getpos
/give
/god
/grindstone
/hat
/heal
/help
/helpop
/home
/ice
/ignore
/info
/invsee
/item
/itemdb
/itemlore
/itemname
/jails
/jump
/kick
/kickall
/kill
/kit
/kitreset
/kittycannon
/lightning
/list
/loom
/luckperms
/mail
/maintenance
/me
/more
/motd
/msg
/msgtoggle
/mute
/near
/nick
/nuke
/pay
/payconfirmtoggle
/paytoggle
/ping
/placeholderapi
/playtime
/potion
/powertool
/powertooltoggle
/ptime
/punish
/pweather
/r
/realname
/recipe
/remove
/renamehome
/repair
/rest
/rtoggle
/rules
/seen
/sell
/sethome
/setjail
/settpr
/setwarp
/setworth
/showkit
/skull
/smithingtable
/socialspy
/spawner
/spawnmob
/speed
/stonecutter
/sudo
/suicide
/tab
/tempban
/tempbanip
/thunder
/time
/togglejail
/toggleshout
/tools
/top
/tp
/tpa
/tpaall
/tpacancel
/tpaccept
/tpahere
/tpall
/tpauto
/tpdeny
/tphere
/tpo
/tpoffline
/tpohere
/tppos
/tpr
/tptoggle
/tree
/unban
/unbanip
/unlimited
/unpunish
/vanish
/vault-convert
/vault-info
/viaversion
/warp
/warpinfo
/weather
/whois
/workbench
/world
/worth
```

## 8. LuckPerms Command Reference

### 8.1 Set wrapper permissions

```text
lp group <group> permission set pizzasmp.punish true
lp group <group> permission set mycommand.unpunish true
lp group <group> permission set mycommand.bancheck true
lp group <group> permission set mycommand.forgive true
lp group <group> permission set pizzasmp.menu true
lp group <group> permission set pizzasmp.homes true
lp group <group> permission set pizzasmp.guide true
lp group <group> permission set pizzasmp.discord true
lp group <group> permission set sethome.use true
lp group owner permission set mycommand.forgive true
lp group co-owner permission set mycommand.forgive true
lp group sradmin permission set mycommand.forgive true
lp group admin permission set mycommand.forgive true
lp group srmod permission set mycommand.forgive true
lp group mod permission set mycommand.forgive true
lp group pizza+ permission set mycommand.forgive false
lp group default permission set mycommand.forgive false
```

### 8.2 Deny direct Punishment backend ban command

```text
lp group <group> permission set punishment.ban false
```

### 8.3 Verify permissions

```text
lp group <group> permission check pizzasmp.punish
lp group <group> permission check mycommand.unpunish
lp group <group> permission check mycommand.bancheck
lp group <group> permission check mycommand.forgive
lp group <group> permission check punishment.ban
```

## 9. Runtime Operations

Start server:

```bash
./start.sh
```

Stop from console:

```text
stop
```

## 10. Staff Web Panels and UUID Continuity

### 10.1 Web panels

- Main WebUI: `http://127.0.0.1:3030`
- Plan Player Analytics: `http://127.0.0.1:8804`
- LibertyBans Web: `http://127.0.0.1:8090`

Screen sessions:

- `server` (Paper server)
- `webui` (PizzaSMP WebUI)
- `libertyweb` (Liberty-Bans-Web)

### 10.2 Plan analytics data collection

In `plugins/Plan/config.yml`:

- `Webserver.Internal_IP: 127.0.0.1`
- `Webserver.Port: 8804`
- `Webserver.Security.CORS.Allow_origin: "http://127.0.0.1:3030"`
- `Webserver.Security.IP_whitelist.Enabled: true`
- `Data_gathering.Geolocations: true`
- `Data_gathering.Accept_GeoLite2_EULA: true`
- `Data_gathering.Join_addresses.Enabled: true`

### 10.3 Floodgate account linking policy

In `plugins/floodgate/config.yml`:

- `player-link.enabled: true`
- `player-link.allowed: true`
- `player-link.enable-global-linking: true`

Player-facing instruction:

- Run `/linkaccount` and complete linking at `link.geysermc.org` before switching between Bedrock and Java to keep one inventory/progress.

### 10.4 UUID migration tool

Scripts:

- `tools/migrate_uuid.sh`
- `tools/migrate_player_uuid` (wrapper name)

Usage:

```bash
tools/migrate_player_uuid <world_dir> <old_uuid> <new_uuid>
```

Example:

```bash
tools/migrate_player_uuid world 11111111-2222-3333-4444-555555555555 aaaaaaaa-bbbb-cccc-dddd-eeeeeeeeeeee
```

Behavior:

- Validates both UUIDs
- Backs up source and destination files under `backups/uuid-migrations/...`
- Copies `playerdata`, `advancements`, `stats` for overworld + sibling dimensions (`_nether`, `_the_end`) if present
- Copies `plugins/Essentials/userdata/<uuid>.yml` if present

### 10.5 Liberty-Bans-Web database requirement

- `webapps/libertybans-web/` is running on `127.0.0.1:8090`.
- Liberty-Bans-Web requires LibertyBans external SQL (`MARIADB`/`MYSQL`) for full live queries.
- Current `plugins/LibertyBans/sql.yml` uses `rdms-vendor: 'HSQLDB'`, which causes file-lock errors while the server is online.
- Keep HSQLDB for now unless you have a reachable MySQL/MariaDB endpoint and credentials ready.

## 11. Guide + LP Wiring

### 11.1 /guide command

- `/guide` is registered by `PizzaAdminTools` (not MyCommand).
- Permission node: `pizzasmp.guide`.
- Command action: opens `pizzasmp_guide` DeluxeMenus menu.

### 11.2 LuckPerms groups (current)

- Gameplay groups used: `default`, `pizza+`, `mod`, `srmod`, `admin`, `sradmin`, `co-owner`, `staff`, `dev`, `owner`.
- Public command nodes (`pizzasmp.menu`, `pizzasmp.homes`, `pizzasmp.guide`, `pizzasmp.discord`) are enabled for all gameplay groups.
- Punish nodes (`pizzasmp.punish`, `mycommand.unpunish`, `mycommand.forgive`, `mycommand.bancheck`) are enabled for staff moderation groups only.
- Chat filter enforcement is applied to all groups, including `dev` and `owner`.

### 11.3 Homes permissions and behavior

- `default` -> `sethome.maxhomes.2=true`, `sethome.maxhomes.5=false`.
- `pizza+` and staff+ -> `sethome.maxhomes.5=true`, `sethome.maxhomes.2=false`.
- Homes GUI includes a same-menu Team Home entry (`WHITE_BANNER`) and Team Home set action (dye button).
- Clicking a home/team-home teleport closes the inventory first, then runs 5s countdown teleport.
- Admin homes browse/teleport nodes:
  - `sethome.admin=true`
  - `pizzasmp.gtp=true`
  - Enabled on: `admin`, `sradmin`, `co-owner`, `dev`, `owner`.

### 11.4 Combat teleport restrictions

- Combat tag applies on player-vs-player damage (including projectiles).
- While tagged, `/rtp`, `/home`, `/homes`, and `/gtp` teleports are blocked unless player has `pizzasmp.combat.bypass`.
- Restriction logic is implemented in:
  - `plugins/PizzaAdminTools.jar`
  - `tools/pizzaadmintools/src/dev/pizzasmp/admin/PizzaAdminTools.java`

## 2026-02-11 Patch Notes (Commands + Homes + LP)

- `PizzaAdminTools` upgraded to `v1.1.0` and now registers:
  - `/gtp`
  - `/homes`
  - `/menu`
  - `/guide`
  - `/discord`
- Verified in console help output that these commands are valid and registered.
- Homes flow now targets a single-menu UX for `/home` and `/homes` with rank-based slot limits.
- LuckPerms matrix normalized:
  - `default` has 2 homes.
  - `pizza+` and staff groups have 5 homes.
  - `admin`, `sradmin`, `co-owner`, `dev`, `owner` have other-player home access via `sethome.admin` + `pizzasmp.gtp`.
- Public command nodes for menu/guide/homes/discord are explicitly enabled for all gameplay groups.
- Added `STAFFCMD.md` and refreshed all active markdown operational docs.
- Homes GUI redesign finalized:
  - `/home` and `/homes` both open one single homes GUI.
  - 5 bed slots shown; `default` can use 2, all higher ranks can use 5.
  - Dye icon is directly below each bed slot.
  - Dyes are per-slot colored, and switch to gray during delete confirmation state.
  - Admin-level ranks can browse target players in the same GUI and teleport to their homes.
- LuckPerms inheritance cleanup applied:
  - Removed bad inheritance: `default -> pizza+`
  - Removed bad inheritance: `owner -> dev`
  - This fixed owner/dev prefix bleed and incorrect inherited command denies.

## 0.3 Security + Moderation + Layout Update (2026-02-16)

- Chat moderation enforcement is now centralized in `PizzaChatGuard`:
  - blocked words cancel the full message (nothing sent to public chat)
  - severe words/slurs: instant `/punish <player> 30d PizzaSMP Chat Policy Violation`
  - mild words: configurable 1-3 warning chances, then `/punish <player> <days>d PizzaSMP Chat Policy Violation`
  - warning counts persist across restarts in `plugins/PizzaChatGuard/warnings.yml`
- Duplicate severe-chat punish path was removed from `PizzaAdminTools` to avoid double punish races.
- WebUI now has account auth and tenant access controls:
  - signup/login/activate/logout endpoints
  - cookie sessions
  - per-server membership checks for host/server/file/plugin/property endpoints
  - invite-code based server access sharing
- LibertyBans visible English prefix branding updated to:
  - `PizzaSMP Moderation »`
- Added layout standard + audit script:
  - `docs/SERVER_LAYOUT.md`
  - `tools/layout_audit.sh`

## 0.4 Auth Server Stability Mitigation (2026-02-16)

To reduce intermittent `Authentication servers are down` join failures:

- set `enforce-secure-profile=false` in `server.properties`
- `start.sh` now performs a Mojang auth endpoint preflight before launch
- JVM startup now pins IPv4 preference and short DNS cache TTLs for auth endpoint reliability

## 0.5 WebUI Uptime + Resource Monitor (2026-02-16)

- WebUI overview now surfaces live server/runtime metrics:
  - server uptime
  - startup ETA
  - CPU usage/load
  - RAM usage
  - disk usage
- Startup ETA is now API-backed and linked to server start request markers (instead of frontend-only countdown).

## 0.6 AntiCheat Punishment Update (2026-02-16)

- `plugins/GrimAC/punishments.yml` now issues punishments via `/punish` for offense tiers:
  - less-severe thresholds: `15d`
  - severe/repeated thresholds: `30d`
- Legacy `kick/tempban` lines in key Grim sections were replaced with the same PizzaSMP moderation pipeline used elsewhere.

## 12. Plugin Inventory and Command Ownership

This is the current operational plugin matrix. Command names remain unchanged.

| Plugin | Responsibility | Player/Staff Commands |
|---|---|---|
| PizzaAdminTools | PizzaSMP custom admin utilities, homes/menu glue, freeze tools | `/gtp`, `/homes`, `/menu`, `/guide`, `/discord`, `/freeze`, `/unfreeze` |
| PizzaChatGuard | Chat/PM/team chat filtering, warning ladder, auto-punish escalation | no direct command |
| PunishDrop | Punish wrapper, inventory drop+clear, LibertyBans-safe duration conversion | `/punish` |
| SetHome | Home data backend + teleport engine | `/home` (frontend via PizzaAdminTools UX) |
| MyCommand | Custom wrappers only | `/unpunish`, `/forgive`, `/bancheck` |
| LibertyBans | Punishment backend and disconnect screen/prefix language | backend for `/punish` flow |
| LuckPerms | Permissions and group model | `lp ...` (staff/admin console usage) |
| DeluxeMenus | GUI framework for menu/guide/homes screens | opened via `/menu`, `/guide`, `/homes` |
| BetterTeams + BetterTeamGUI | Team system and team chat/home support | `/team`, `/tc` |
| Essentials + EssentialsChat | Core utility/chat formatting | Essentials command surface |
| RTP plugin | Random teleport functionality | `/rtp` |
| GrimAC | Anticheat | Grim command surface (staff use) |
| TAB | Tablist/scoreboard formatting | config-driven |
| Vault | Permissions/economy/chat bridge | none |
| PlaceholderAPI | Placeholder expansion layer | `/papi ...` (admin) |
| Plan | Analytics web/reporting | `/plan ...` (admin) |
| ViaVersion + ViaBackwards | Protocol compatibility layer | none |
| Geyser-Spigot + floodgate | Bedrock connectivity and account bridge | `/linkaccount` and Bedrock join bridge |
| Maintenance | Maintenance mode tooling | maintenance commands (staff) |
| Chunky | Chunk pre-generation | `/chunky ...` |

Notes:
- Third-party updateable plugins remain separate and are not merged.
- Branding is display-layer only (messages/UI/docs), not command namespace changes.

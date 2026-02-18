# PizzaSMP Agent Handoff Guide

Last updated: 2026-02-16

## 1) Environment

- Server software: Paper 1.21.8
- Root path: `/ssd/PaperMC/PizzaSMP`
- Main world folders:
  - `world`
  - `world_nether`
  - `world_the_end`
- Runtime screen sessions:
  - `server` (Minecraft)
  - `webui` (custom web app on port 3030)
  - `libertyweb` (Liberty-Bans-Web on port 8090)

## 2) Critical Rule For Future Agents

- Update markdown documentation every `3-6` user queries while active work is ongoing.
- At minimum, keep these files current:
  - `README.md`
  - `docs/COMMANDS.md`
  - `docs/AGENT_RUNBOOK.md`
  - `docs/NEXT_AGENT_TASKS.md`
  - `STAFFCMD.md`
  - `webapp/README.md`
  - `WEBUI.md`

## 3) Current Command Stack

- SetHome is the home backend (`/home` data + countdown teleport).
- `PizzaAdminTools` now provides registered commands:
  - `/gtp`
  - `/homes`
  - `/menu`
  - `/guide`
  - `/discord`
  - `/freeze`
  - `/unfreeze`
- `PunishDrop` provides `/punish`.
- MyCommand only provides:
  - `/unpunish`
  - `/forgive`
  - `/bancheck`

## 4) Homes UX (Current)

- `/home` and `/homes` open a single custom homes menu (no extra nested home menus).
- Default group is limited to 2 homes; Pizza+ and staff groups are set to 5 homes.
- Each home slot uses a bed icon with a dye icon directly below it.
- Delete is a same-slot two-click confirm flow, with gray dye used for pending state.
- Admin-level groups can browse/teleport to other players’ homes.
- Combat-tag restrictions remain active for `/rtp`, `/home`, `/homes`, and home teleports.

## 5) Key Permission Model (Verified)

- Public/new command nodes available to all normal groups:
  - `pizzasmp.menu`
  - `pizzasmp.homes`
  - `pizzasmp.guide`
  - `pizzasmp.discord`
- Cross-player home access:
  - `sethome.admin` + `pizzasmp.gtp` true for `admin`, `sradmin`, `co-owner`, `dev`, `owner`
- Freeze moderation access:
  - `pizzasmp.freeze` + `pizzasmp.unfreeze` for `owner`, `dev`
- Full-access override:
  - `owner` -> `*=true`
  - `dev` -> `*=true`
- Home limits:
  - `default`: `sethome.maxhomes.2=true`, `sethome.maxhomes.5=false`
  - `pizza+` and staff+: `sethome.maxhomes.5=true`, `sethome.maxhomes.2=false`

## 6) Reality Check: "All users became dev"

- Verified false.
- Latest LP export (`plugins/LuckPerms/luckperms-2026-02-11-20-40.json.gz`) shows a mixed set of groups.
- Some stale UUID-only records still exist from earlier offline/fake entries; do not mass-delete without confirmation.
- Additional fix applied:
  - Removed `default -> pizza+` inheritance.
  - Removed `owner -> dev` inheritance.
  - Owner prefix and command access now resolve correctly from owner group without dev bleed-through.

## 7) Operational Commands

- Reload MyCommand: `mycmd-reload commands`
- Reload DeluxeMenus: `dm reload`
- Reload SetHome: `home reload`
- Export LuckPerms snapshot: `lp export`

## 8) Punish Notes

- LibertyBans does not parse `s` suffix directly in `/ban`.
- PunishDrop normalizes temporary bans to LibertyBans-safe minute format, then schedules exact unban at the true expiry time.
- `/punish <player> 5s <reason>` now resolves as a real ~5 second punishment (not permanent).
- PunishDrop now flushes saves after inventory clear before applying the ban command to reduce inventory dupe risk.

## 9) WebUI Ban Source

- `webapp/server.py` now sends `/punish` for ban actions.
- Active/archive bans come from parsed `LibertyBans` lines in `logs/latest.log`, stored in `webapp/data/webui.db` (`ban_events` table).
- Stale active bans are reconciled by unban events and superseded-ban cleanup logic.

## 10) Build + Deploy PizzaAdminTools

```bash
CLASSPATH=$(find libraries -type f -name '*.jar' | tr '\n' ':' | sed 's/:$//')
mkdir -p tools/pizzaadmintools/bin
javac -cp "$CLASSPATH" -d tools/pizzaadmintools/bin tools/pizzaadmintools/src/dev/pizzasmp/admin/PizzaAdminTools.java
jar --create --file plugins/PizzaAdminTools.jar -C tools/pizzaadmintools/bin . -C tools/pizzaadmintools/src plugin.yml
```

Then restart Paper in `screen -S server`.

## 11) Current Priority Notes (2026-02-16)

- `PizzaChatGuard` is the single chat enforcement source:
  - blocks full message on violations (no `***` chat leak)
  - enforces mild (warn -> 15d) and severe (instant 30d) ladders
  - covers global chat, PM commands, and team-chat command paths
- WebUI backend now:
  - suppresses HTTP access spam logging
  - returns explicit port conflict error when `3030` is already in use
  - safely handles dropped sockets (reduces noisy fetch failures)
- WebUI action label is `Start` in the UI (backend action remains `start_reliable`).

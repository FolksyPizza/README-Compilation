# PizzaSMP WebUI Runbook

Last updated: 2026-02-18

## Related Server Docs

- Canonical plugin + command reference:
  - `docs/COMPLETE_PLUGIN_COMMAND_REFERENCE.md`
- Staff quick command summary:
  - `STAFFCMD.md`

## 2026-02-17 Updates

- Ban duration input in WebUI now accepts only `d`, `h`, and `m` units (plus permanent aliases).
- Ban/archive tables now display durations in day/hour/minute format (no weeks/seconds display).
- Paper startup now includes Mojang auth DNS warmup and stronger JVM DNS cache settings to reduce transient auth verification failures.
- Startup/auth reliability fix: removed JVM DNS nameserver overrides to `1.1.1.1/8.8.8.8` and switched to strict preflight failure if Mojang auth endpoints are unreachable.

## 2026-02-18 Ops note

- In-game plugin/admin command visibility is now staff-gated by `pizzasmp.pluginadmin`.
- Staff can use `/pizzaplugins` for the branded plugin stack list.
- Ban-screen consistency fix:
  - reconnect attempts now preserve the same PizzaSMP ban-screen layout (`You Have Been Banned From The PizzaSMP`, `Duration`, `Reason`).

## 2026-02-16 Updates

- Main action buttons use `Start` (wired to reliable startup backend action) for PaperMC startup.
- Browser-side action failures now show clear API connectivity messaging.
- Console panes auto-scroll to newest output and are cleared to blank on stop/restart.
- Added `Simple Mode` toggle in top bar:
  - beginner-focused tab set
  - down-server console gate with one-click start
  - startup loading indicator with ETA

## Overview

WebUI now uses a server-picker-first cloud-panel layout:

- first screen: select a server
- then open tabbed control panel for that server

- `Home` (server picker + icon management)
- `Overview`
- `Servers`
- `Plugins`
- `Console`
- `Players`
- `Bans`
- `Backups`
- `Properties`
- `Files`
- `Users`

Major UX upgrade applied:

- Redesigned visual system (cleaner hosting-panel look with stronger contrast and spacing).
- Sticky console input bars and improved terminal readability.
- Auto-follow toggles for both main console and selected managed-server console.
- Header-level `Sync Now` action for full panel refresh.
- Toast notifications for action success/failure feedback.
- Improved sidebar runtime status widget.

Default URL: `http://0.0.0.0:3030`

## Runtime

- Screen session: `webui`
- Start:
```bash
cd /ssd/PaperMC/PizzaSMP
python3 webapp/server.py
```
- Restart:
```bash
screen -S webui -X quit || true
screen -dmS webui bash -lc 'cd /ssd/PaperMC/PizzaSMP && python3 webapp/server.py'
```

If startup reports `port 3030 is already in use`, stop old WebUI process/session and start `webui` again.

## Core Files

- Backend: `webapp/server.py`
- Frontend: `webapp/static/index.html`
- Frontend logic: `webapp/static/app.js`
- Frontend style: `webapp/static/styles.css`
- Data DB: `webapp/data/webui.db`
- Chat interaction file: `chat-interactions.jsonl`

## API Endpoints

- Health/status:
- `GET /api/health`
- `GET /api/server/status`
- `GET /api/stats`
- `GET /api/server/logs?lines=`
- Server control:
- `POST /api/server/action` (`start|stop|restart|save`)
- `POST /api/server/command`
- `POST /api/server/announce`
- `GET /api/server/properties`
- `POST /api/server/properties`
- Player/chat:
- `GET /api/players?q=&limit=&real=1|0`
- `GET /api/players/<uuid>`
- `GET /api/chat/events?limit=&q=`
- `GET /api/chat/file?lines=`
- `GET /api/userlogs/list?uuid=&name=`
- `GET /api/userlogs/read?uuid=&name=&file=&lines=`
- `GET /api/chat/flagged?lines=`
- Bans:
- `GET /api/bans?q=&active=1|0&limit=`
- `GET /api/bans/archive?q=&limit=`
- `POST /api/bans` (sends `/punish`)
- `POST /api/bans/archive/purge`
- Admin links:
- `GET /api/admin/links`
- New panel data:
- `GET /api/backups?limit=`
- `POST /api/backups/create`
- `GET /api/files?path=&limit=`
- `GET /api/lp/users?limit=`
- Multi-server host management:
- `GET /api/host/servers`
- `POST /api/host/servers`
- `POST /api/host/server/action`
- `POST /api/host/server/command`
- `GET /api/host/server/logs?id=<server_id>&lines=`
- `GET /api/host/server/icon?id=<server_id>`
- `POST /api/host/server/icon` (`server_id`, PNG `data_url`)
- `GET /api/host/server/properties/raw?id=<server_id>`
- `POST /api/host/server/properties/raw`
- `GET /api/host/server/files?id=<server_id>&path=&limit=`
- `GET /api/host/server/file?id=<server_id>&path=`
- `POST /api/host/server/file`
- `POST /api/host/server/file/delete`

Operator workflow improvements:

- Home tab now provides server cards (icon/status/port) and one-click selection.
- Selected server carries over to servers/console/properties/plugin actions.
- Server icon can be changed directly in UI (PNG upload).
- Console auto-scrolls to newest lines.
- Console auto-follow is always enabled by default:
  - scrolling up pauses follow
  - scrolling back near bottom resumes follow automatically
- Important action results are shown both inline and as toasts.
- Plugin template UX upgrades:
  - search box for templates
  - `Select Recommended`, `Select All`, `Clear` quick actions
  - selected-count indicator
  - readable plugin cards (name + jar + size + recommended badge)
- Dedicated Plugins tab:
  - server-scoped plugin catalog
  - plugin detail panel with description
  - install (from templates), enable, disable actions

## Multi-Server Provisioning

The `Servers` tab can now provision new managed servers in `/ssd/PaperMC/<server_id>`.

Provisioned server scaffold includes:

- `paper.jar` copied from current PizzaSMP root
- `start.sh` with configurable `Xms` / `Xmx`
- `eula.txt` (`eula=true`)
- baseline `server.properties` with:
- `pvp=true`
- `spawn-protection=0`
- auto-allocated `server-port` (or user-provided port)
- selectable server jar/version from local discovered jars (`paper*`, `folia*`, `purpur*`)
- optional plugin template copy during provisioning (from `PizzaSMP/plugins/*.jar`)

Server-level controls now also include:

- per-server `server.properties` edit/save
- per-server plugin list with enable/disable toggles (`.jar` <-> `.jar.disabled`)
- install selected plugin templates into existing managed server

Managed server metadata is stored in:

- `webapp/data/managed_servers.json`

## Chat Logging Notes

Chat data is parsed from `logs/latest.log` into `chat_events` and mirrored to `chat-interactions.jsonl`.

Supported formats:

- `<player> message`
- Essentials-like sender lines only when the sender resolves to a known UUID-backed player

Current filtering:

- only UUID-backed real players are kept in WebUI chat feeds
- system/plugin console lines are excluded from chat history views
- `/api/chat/file` now returns only valid player-chat JSONL entries

If chat appears empty, send one in-game chat message and refresh `Players -> Chat Logs`.

## Per-user and Flagged Files

- Per-user daily logs are written to:
  - `logs/user-events/<uuid>/YYYY-MM-DD.log`
  - `logs/user-events/by-name/<name>/YYYY-MM-DD.log`
- Flagged-only log is written to:
  - `logs/chat-filter-flagged.log`
- Players tab now supports:
  - loading a user’s daily files
  - opening an individual daily file
  - viewing the single flagged-chat log file

## Security Notes

- WebUI currently binds to `0.0.0.0:3030`.
- Default external links now target:
  - Plan: `http://jetson-nano:8804`
  - LibertyBans Web: `http://jetson-nano:8090`
- Recommended for remote exposure: reverse proxy + auth.
- Ban actions and console actions execute directly in `screen -S server`.

## 2026-02-16 auth + tenancy

- WebUI now requires sign-in for all `/api/*` except `/api/health` and auth endpoints.
- New auth endpoints:
  - `POST /api/auth/signup`
  - `POST /api/auth/activate`
  - `POST /api/auth/login`
  - `POST /api/auth/logout`
  - `GET /api/auth/me`
- Access controls:
  - host/server/file/plugin/property APIs now require membership on target server
  - server invite codes support grant/redeem/revoke flows

## 2026-02-16 uptime/resource monitoring

Overview now includes resource monitoring cards for:

- Server Uptime
- Startup ETA
- CPU
- RAM
- Disk

Data is sourced from server APIs (`/api/server/status`, `/api/stats`, `/api/system/metrics`).

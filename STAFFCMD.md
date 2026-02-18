# PizzaSMP Staff Commands

Last updated: 2026-02-18

## Canonical Reference

- Full plugin command catalog and plugin descriptions:
  - `docs/COMPLETE_PLUGIN_COMMAND_REFERENCE.md`

## WebUI note (2026-02-16)

- Use **Start** in WebUI for PaperMC startup.
- If WebUI actions fail with browser `NetworkError`, restart WebUI screen session (`webui`) and retry.

## Latest fixes

- `owner` and `dev` now have confirmed full command access (including vanilla command permissions).
- `dev` inheritance was corrected to avoid command blocks from lower-rank explicit denies.
- PvP damage restored by Essentials protect config fix.

## Moderation

- `/punish <player> <duration> <reason...>`
- `/unpunish <player>`
- `/forgive <player>`
- `/bancheck <player>`
- `/freeze <player>`
- `/unfreeze <player>`
- `/clearwarnings <player>` (aliases: `/clearwarn`, `/clearwarns`)
- `/clearallwarnings [player]` (aliases: `/clearallwarns`, `/clearwarningsall`)

Notes:

- `/punish` is served by `PunishDrop` and bans through LibertyBans.
- `/unpunish`, `/forgive`, and `/bancheck` are also served by `PunishDrop` (single moderation command owner).
- Immediate `/forgive` after `/punish` is now race-safe (PunishDrop retries unban briefly until backend punish state settles).
- Ban screen on reconnect is standardized:
  - `You Have Been Banned From The PizzaSMP`
  - `Duration: ...`
  - `Reason`
- Duration supports `d/h/m` and combo values (`1d12h`, `2h30m`).
- Online-target punish drops and clears inventory before ban.
- `/punish` temporary durations are normalized for LibertyBans and exact unban is scheduled by PunishDrop.
- `/clearwarnings` clears a player's PizzaChatGuard warning counter (used to de-escalate after isolated language violations).
- `/clearallwarnings` clears all PizzaChatGuard warning entries when run without args, or one player when a name is provided.

## Staff Plugin/Admin Visibility

- Plugin/admin commands are staff-only via `pizzasmp.pluginadmin`:
  - `/pl`, `/plugins`, `/lp`, `/luckperms`, `/version`, `/ver`, `/about`, `/paper`, `/timings`
- `default` and `pizza+` are explicitly denied plugin/admin command access.
- Staff groups with `pizzasmp.pluginadmin`:
  - `owner`, `dev`, `sradmin`, `admin`, `srmod`

## Staff Help Hubs

- `/pizzaadmintools help`
- `/pizzateams help`
- `/pizzamenus help`
- `/pizzahome help`
- `/pizzabans help`
- `/pizzaplugins`
- `/pizzachatguard help`

## Homes and Teleport

- `/home` or `/homes` opens the single homes GUI.
- `/home <slot|name>` teleports directly with the 5-second countdown.
- Homes GUI now includes Team Home (`WHITE_BANNER`) in the same inventory.
- `/gtp <player> [home-name|home-index]` teleports to other-player homes (admin-capable groups only).

Homes limits:

- `default`: 2 slots
- `pizza+` and staff+: 5 slots

## Utility

- `/menu`
- `/guide`
- `/discord`

## Required nodes (staff side)

- `pizzasmp.punish`
- `mycommand.punish`
- `mycommand.unpunish`
- `mycommand.forgive`
- `mycommand.bancheck`
- `pizzasmp.freeze` (owner/dev)
- `pizzasmp.unfreeze` (owner/dev)
- `sethome.use`
- `sethome.maxhomes.5`

Admin-capable extras:

- `sethome.admin`
- `pizzasmp.gtp`
- `pizzasmp.combat.bypass`

Chat filter enforcement policy:

- ChatGuard moderation is enforced for all player groups (including `owner` and `dev`).

## 2026-02-16 chat filter enforcement

- Mild profanity:
  - blocked + warning, with configurable warning chances (`1-3`) before ban
- Severe slurs/profanity:
  - first offense = instant policy ban
- Blocked messages are fully canceled and never posted to global chat.
- PM commands are also enforced (`/msg`, `/tell`, `/w`, `/whisper`, `/dm`, `/pm`, `/m`).

## 2026-02-16 operations note

- Use WebUI Overview for quick health checks: uptime, startup ETA, CPU, RAM, disk.
- Command surfaces remain unchanged.

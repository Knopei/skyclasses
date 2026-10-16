# 1v1 Class Duel — Project Docs

A Minecraft 1v1 PvP game written entirely in **Skript**. Two players draft three classes each, fight in a shared arena over a 30-life pool, level their classes up mid-match, and contest jungle mobs and a major boss for XP and extra lives.

| Document | Read it for |
|---|---|
| [DESIGN.md](DESIGN.md) | Game design: match flow, classes, resources, leveling, objectives, progression, balance tooling |
| [TECHNICAL.md](TECHNICAL.md) | Implementation: stack, file layout, systems, variable conventions, Skript pitfalls, resource pack, admin commands |

## Scope

These docs cover the **core game** (`00-config.sk` to `20-combat.sk`), the **14 playable class files**, and the **Executioner resource pack** (`ea_pack`).

Not covered: `21-prototype-core.sk` and the prototype boss files. They are experimental and still changing.

## Stack

| Component | Version / note |
|---|---|
| Server | Paper 1.21.11 (Minehut-hosted) |
| Skript | 2.13.2 |
| Addons | skript-reflect, SkBee |
| Resource pack | `ea_pack.zip` (Executioner ultimate visuals) |

## Quick start (admin)

1. Copy the `.sk` files into `plugins/Skript/scripts/` and run `/sk reload all`.
2. Make sure the `lobby` and `arena_1` worlds exist and the coordinates in `00-config.sk` match your map.
3. Host `ea_pack.zip` and require it through `server.properties` (see [TECHNICAL.md](TECHNICAL.md#12-executioner-resource-pack)).
4. Two players run `/duel`. To test without recording stats, use `/testduel` instead.

## Roster at a glance

| Category | Classes |
|---|---|
| Juggernaut | Basher, Frankenstein, Paladin, Demon |
| Striker | Mighty, Rapid, Viking, Zeus, Executioner |
| Skirmisher | Ghost, Ninja, Reaper, Tempest, Cyberpunk |

> Written against the files as they stood on 2026-10-15. Some behaviours (item components, yaw conventions, `pack_format`) were verified by lint and review rather than a full in-game test; those are flagged in TECHNICAL.md.

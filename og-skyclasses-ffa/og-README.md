# SkyClasses FFA (original)

The original SkyClasses game: a class-based free-for-all PvP game for Minecraft, written entirely in **Skript** by me. Players pick a class, fight with class-specific weapons and abilities, and climb the leaderboards.

This folder is kept exactly as I wrote it, as the human-written baseline for the later 1v1 duel rewrite in [`../skyclasses-duel/`](../skyclasses-duel/).

## What's in the game

- **Classes:** each class has its own weapon, a tactical and an ultimate ability, and a set of perks. Class files live in [`classes/`](classes/).
- **Resources:** abilities cost mana and stamina (`mana.sk`, `stamina.sk`).
- **Combat:** custom damage handling and death/respawn logic (`damage.sk`, `death.sk`).
- **Loadouts:** class selection and class info menus (`class-select.sk`, `class-info.sk`, `class-items.sk`, `class-func.sk`), consumables (`consume.sk`) and general perks (`generalperk.sk`).
- **Mobs:** summoned and boss mobs (`summon.sk`).
- **Progression:** leaderboards (`leaderboards.sk`).
- **World:** game rules and server setup (`gamerule.sk`), plus shared helpers (`useful.sk`).

## Classes

Basher, Cyberpunk, Demon, Frankenstein, Ghost, Hunter, Marksman, Mighty, Ninja, Paladin, Pirate, Poseidon, Rapid, Reaper, Tempest, Viking, Warden, Zeus.

## Stack

Skript, running on a Minecraft server (Minehut).

## Running it

Copy the `.sk` files and the `classes/` folder into `plugins/Skript/scripts/` and run `/sk reload all`. Don't run this alongside the duel rewrite, because the two scripts overlap.

## Relationship to the duel rewrite

The 1v1 duel game reuses ideas from this project, such as the class system, resources and pregame draft, but it is a separate game built with AI assistance. See the [root README](../README.md) for how the two are separated.

# SkyClasses

The original SkyClasses game: a class-based free-for-all PvP game for Minecraft, written entirely in **Skript** by me. Players pick a class, fight with class-specific weapons and abilities, and climb the leaderboards.

This `main` branch holds only my original, human-written code.

## Repo layout

| Path | What it is |
|---|---|
| [`og-skyclasses-ffa/`](og-skyclasses-ffa/) | The original FFA game, exactly as I wrote it |
| [`og-skyclasses-ffa/classes/`](og-skyclasses-ffa/classes/) | One script per playable class |

## What's in the game

- **Classes:** each class has its own weapon, a tactical and an ultimate ability, and a set of perks.
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

Copy the `.sk` files and the `classes/` folder from `og-skyclasses-ffa/` into `plugins/Skript/scripts/` and run `/sk reload all`.

## The AI-assisted duel rewrite

I later built a separate 1v1 duel game that reuses ideas from this project, such as the class system, resources and pregame draft. That work was done with AI assistance, and it lives on the [`ai-experimenting` branch](https://github.com/Knopei/skyclasses/tree/ai-experimenting) so it stays separate from my original code. The tag `v0-human-original` marks the original code before any of it.

# SkyClasses

A Minecraft class-based PvP project written entirely in **Skript**. This repo shows two stages of it: my original hand-written game, and a later 1v1 duel rewrite built with AI assistance.

## Repo layout

| Folder | What it is | Who wrote it |
|---|---|---|
| [`og-skyclasses-ffa/`](og-skyclasses-ffa/) | The original free-for-all SkyClasses game, untouched | Me (human-written) |
| [`skyclasses-duel/`](skyclasses-duel/) | The 1v1 persistent-duel rewrite: draft, arena, leveling, bosses, stats | AI-assisted, directed and tested by me |
| `skyclasses-duel/classes/` | The 14 playable class files | AI-assisted |
| `skyclasses-duel/texture pack/` | `ea_pack.zip`, the resource pack for the Executioner ultimate visual | AI-assisted |
| [`docs/`](docs/) | Design and technical documentation for the duel game | AI-assisted |

The duel rewrite reuses ideas and code patterns from the original, such as the pregame draft, but it is a separate game and does not run together with the original files.

## How to tell human and AI work apart

- **Branches:** `main` holds only my original code. `ai-experimenting` adds the AI-assisted rewrite on top of it.
- **Tag:** `v0-human-original` marks the original code before any AI-assisted changes.
- **Commit labels:** `[human]` for my own work, `[AI]` for AI-written work, `[human+AI]` for work we did together. In the AI-assisted work, I set the design and decisions, tested everything in-game and reported the bugs, and the AI wrote and fixed the code.

## Docs

Start with [`docs/README.md`](docs/README.md). It links to the design document (match flow, classes, balance) and the technical document (file map, systems, known Skript pitfalls).

## Running it

1. Put the `.sk` files from `skyclasses-duel/` (including `classes/`) into `plugins/Skript/scripts/` and run `/sk reload all`.
2. Host `skyclasses-duel/texture pack/ea_pack.zip` as a server resource pack. It is not a script, so keep it out of the scripts folder.
3. See [`docs/TECHNICAL.md`](docs/TECHNICAL.md) for the required worlds, addons and configuration.

Requires Paper 1.21.x, Skript 2.13+, skript-reflect and SkBee.

> Only run one of the two games on a server at a time. Their scripts overlap and will conflict.

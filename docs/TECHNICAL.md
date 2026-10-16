# Technical Document

## 1. Stack and deployment

| Item | Value |
|---|---|
| Server | Paper 1.21.11 on Minehut |
| Skript | 2.13.2 |
| Addons | skript-reflect (Java calls), SkBee (NBT, boss bars, fastboard) |
| Worlds | `lobby` (draft structure), `arena_1` (permanent pre-built map, never regenerated) |

Deploy by copying the `.sk` files to `plugins/Skript/scripts/` and running `/sk reload all`. Files load alphabetically, which is why the core files are numbered. Tunable values live in `00-config.sk`.

## 2. File map

| File | Responsibility |
|---|---|
| `00-config.sk` | All tunables: life pool, XP thresholds, cooldown formula, boss stats, arena and lobby coordinates, secret door locations |
| `01-utils.sk` | Shared helpers: gamemode commands, `/hset`, `/mana`, `/stamina`, attacker tracking, floating damage numbers, projectile trails, mob activate/deactivate, boss leash |
| `02-class-data.sk` | Class names, categories, weapons, attack damage and speed, mana costs, cooldowns, lore/display items |
| `03-resources.sk` | Mana and stamina storage, regen ticker, `staminaCHECK`/`tacCHECK`/`ultCHECK`, action bar HUD, sneak-dash |
| `04-leveling.sk` | `classLevel`, `addClassKill`, `maxHealth`, death cooldown, cooldown bossbars, sidebar leveling lines, level tags |
| `05-class-menu.sk` | `classSelect`, in-match "Your Classes" menu |
| `06-perks-general.sk` | The 7 general perks and their in-match picker |
| `07-consumables.sk` | Consumables, delivery on respawn/kill, `stackItem`, grenades |
| `08-damage.sk` | `dmgUpdate` pipeline, per-class `*CHECK` hooks, `mobMeleeCHECK`, `dmgStats` |
| `09-death-match.sk` | Death handling, kill credit and rewards, life pool, respawn flow, spawn protection |
| `10-match-manager.sk` | `/duel`, `/testduel`, `/endduel`, `/unstuck`, match start/end, sidebar, disconnect handling |
| `11-mini-bosses.sk` | Marauder and Crawler mobs |
| `12-major-boss.sk` | Iron Golem, hole/platform, reward |
| `13-mastery.sk` | Mastery score and tiers |
| `14-stats.sk` | Persistent stats, `/stats`, match summary, `/matchhistory`, `/resetstats` |
| `15-arena-manager.sk` | Gamerules, arena setup/teardown (static arena) |
| `16-world-rules.sk` | Environment locks, chat format, block protection, secret doors, launch pad |
| `17-draft.sk` | Ban deck, coinflip, snake draft, starting class menu |
| `18-class-browser.sk` | `/class` browser, class info, per-class default selectors |
| `19-balance-stats.sk` | `/balancestats` aggregate dashboard |
| `20-combat.sk` | Combat state, opponent bossbar, nametag hiding |
| 14 class files | `viking`, `demon`, `frankenstein`, `ghost`, `basher`, `mighty`, `ninja`, `paladin`, `rapid`, `reaper`, `zeus`, `cyberpunk`, `tempest`, `executioner` (`.sk`) |
| `ea_pack/`, `ea_pack.zip` | Executioner resource pack |

## 3. Class identifiers

| ID | Class | ID | Class |
|---|---|---|---|
| VBA | Viking | NS (NK katana form) | Ninja |
| DS | Demon | PGS | Paladin |
| FH | Frankenstein | RB (RD dagger form) | Rapid |
| GD | Ghost | RS | Reaper |
| BS | Basher | ZT | Zeus |
| MH | Mighty | CB | Cyberpunk |
| TS | Tempest | EA | Executioner |

`{menu::classlist::*}` lists the classes the draft and menus use. Per-class data is keyed by ID, for example `{weapons::MH::TAC-COST}` and `{class-name::MH}`.

## 4. Global architecture

### State model

All state is stored in Skript variables, keyed by player UUID. Persistent variables survive restarts; per-match variables are reset by `resetMatchStats`, `resetPlayerLeveling` and the cleanup in `endMatch`/`forceEndMatchWorker`.

| Prefix | Meaning |
|---|---|
| `{inMatch::uuid}`, `{matchOpponent::uuid}` | Match membership |
| `{class::uuid}` | Currently equipped class ID |
| `{loadout::uuid::1..3}` | The three drafted classes |
| `{lifepool::uuid}` | Lives remaining |
| `{mana::uuid::curr/max}`, `{stamina::...}` | Resources (×10 and ×300 scaling) |
| `{classxp::uuid::CLASS::kills}` | XP per class |
| `{godlike-ULT-USE::uuid::CLASS}` | An ultimate is active |
| `{godlike-ULT-CANCEL::uuid::CLASS}` | Cancel flag set on death |
| `{match*::uuid}` | Current-match stats |
| `{testMatch}` | Global flag: skip persistent/balance stats |
| `{config::...}` | Tunables |

### Event flow of a hit

1. Bukkit damage event → class files and `01-utils.sk` record the attacker (`updateAttacker`, `metadata "attacker"`).
2. `dmgUpdate(attacker, victim, dmg, cause)` in `08-damage.sk` runs: attacker-class branches (Executioner ultimate/tactical, Rapid, Viking, etc.), victim-class branches (Basher shield, Frankenstein shield), fire/wither attribution, reductions (lightning ×0.4 for Zeus, ×0.6 otherwise; `playerDefense` percent), fall damage zeroed, `showDMG`, `dmgStats`, `combatCheck`.
3. The returned damage is applied to the event.
4. On death, `09-death-match.sk` runs: stats, `giveKillReward`, `addClassKill`, life-pool decrement, lead tracking, win check, then `handleRespawnFlow`.

Per-class hooks follow the pattern `xxCHECK` (for example `vbaCHECK`, `bsCHECK`, `fhULT`) called from the pipeline.

### Ability gating

`tacCHECK`, `ultCHECK` and `staminaCHECK` (`03-resources.sk`) validate level, mana, cooldown and stamina before an ability runs, and they apply perk modifiers (Spell Caster mana reduction, `ultCooldownFor` cooldown reduction). Ultimates set `{godlike-ULT-USE::uuid::CLASS}` and poll `{godlike-ULT-CANCEL::...}` so a death ends them immediately.

## 5. Tick loops

| Loop | Interval | Purpose |
|---|---|---|
| Resource regen and action bar | 5 ticks | Mana +10 internal, stamina +7.5 internal |
| Respawn pickup watcher | 5 ticks | Applies `pendingClassSelect` on a fresh execution |
| Combat expiry and bars | 5 ticks | Rolling 5 s combat timer |
| Projectile trails | 1 tick | `01-utils.sk` |
| Executioner mark watcher | 1 s | Slows victims at ≤ threshold health |
| Mini/major boss attack and leash loops | per mob | Telegraphed attacks, leashing |

## 6. Match lifecycle (functions)

`/duel` → `startMatch` → `runClassDraft` → `enterArena` → fighting → `endMatch` (or `forceEndMatch`).

- `runClassDraft` is deliberately **one inline sequence** (no wait-containing sub-functions called from a loop); earlier versions desynced picks and bossbars.
- `enterArena` calls `setupArena`, sets lives and spawns, calls `classSelect` for each player's starting class, and starts boss timers.
- `endMatch` records stats (unless `{testMatch}`), snapshots the match, cleans up, calls `teardownArena`, waits 2.5 s, then opens the summary GUI (opening it earlier is closed by the teleport).
- `forceEndMatch` is split into a wrapper and a worker because of Skript issue #7100 (a function with both `wait` and `return`).

### Respawn flow

`handleRespawnFlow` freezes the player (invisibility, flight allowed to avoid fall jitter, `on player move` cancels positional movement), waits `respawn::menu-delay`, opens the class menu, polls every 5 ticks for a selection or the 10 s timeout, then stores the choice in `{pendingClassSelect}`. The 5-tick watcher applies it with `classSelect`, because `classSelect` uses `return` internally and is unsafe to call from this delayed context.

## 7. Damage, health and economy details

| Topic | Rule |
|---|---|
| Max health | `9 + juggernaut + Healthy Boy + Frankenstein Perk 1` hearts, set in `maxHealth` |
| Kill reward | Golden apple plus selected consumable; killer fully healed, stamina refill, bad effects removed |
| Mob melee | `mobMeleeCHECK` replaces default damage with random 0.5–1.0 unless the mob has `specialAttack` metadata |
| Stats | `dmgStats` counts only hits under 10 damage, ignores void/suffocation, and skips prototype entities (`protoRole` metadata) |
| Executioner TAC | `ea-tacBuff` flag consumed in `dmgUpdate`, damage ×1.55 |
| Executioner ULT | Level from `ea-chargeLevel`; 2.5/5/7.5 flat or %-max-health with Perk 2; cap 10 |
| Shared defense | `{playerDefense::uuid}` is a percent used by Reinforced Dash and Death's Toll (the larger wins; each clears only its own value) |

## 8. Objectives implementation

- **Mini-bosses** — spawned at four fixed locations from `{config::arena::miniboss-spawn::N}`. Tagged with `minibossLoc` metadata. Passive via `mobDeactivate` (Paper `setAware(false)`), activated on first player hit. Slowness from `{config::mob::move-slowness-level}`. Marauder wears an unbreakable chainmail helmet so it doesn't burn.
- **Major boss** — `isMajorBoss` metadata, a hole/platform filled and cleared with barriers and a collapse animation on death, reward rolled at first hit and shown on a bossbar. `forcedDespawn` metadata prevents reward on timeout.
- **Secret doors** — tripwire-hook driven; `resetSecretDoors` forces them open at match start and end.

## 9. Persistent stats

`initializePlayerStats` in `14-stats.sk` is the single place that initialises every persistent stat (used on first join and `/resetstats`). Stats exist at overall and per-class level (games, wins, losses, kills, deaths, damage, XP by phase, comeback, best-game records). Mastery reads `class-games`, `class-wins`, `class-kill`, `class-death`.

Balance variables (`balance-*`) are global and never keyed to a player. Abilities tag their own damage with `{balance-abilitySource::uuid}` ("TAC"/"ULT"/"PASSIVE") immediately before dealing it; DoT sources are tagged on the **victim** (`balance-fireSource`) and bridged in `dmgUpdate`.

## 10. Items and NBT (1.21.5+)

- Item components are set with `add nbt compound of "{...}" to nbt of {item}`.
- Tooltip hiding: `minecraft:tooltip_display:{hidden_components:[...]}`; `attribute_modifiers` is a plain list.
- An invalid component ID breaks item creation entirely.
- Custom model: `minecraft:item_model` (see §12).

## 11. Skript and skript-reflect pitfalls (project conventions)

| Pitfall | Convention |
|---|---|
| Java calls reject Skript numbers | Add `+ 0.0` to coerce to double |
| `delete {entity}` | Only clears the variable; use `kill`/`.remove()` |
| Nested loops of the same type | `loop-value` is ambiguous; use counters or different loop types |
| Variable durations | `"%n% seconds" parsed as a timespan` |
| No `mod` operator | Use arithmetic workarounds |
| Function calls inside `{var::%...%}` indexes | Assign to a variable first |
| `wait` inside functions | Control returns to the caller immediately |
| Editing damage after a `wait` | Not possible; do it before any wait |
| `%` in strings | Write `%%` |
| Parenthesised conditions with `or` | Use flag variables |
| `wait` plus `return` in one function | Split worker/wrapper (Skript #7100) |
| Spawning a specific entity type | `loc.getWorld().spawnEntity(loc, EntityType.X)` |
| Imports | `import x as Alias` works |

## 12. Executioner resource pack

### Contents

```
ea_pack/
  pack.mcmeta                      pack_format 75
  assets/executioner/
    items/ult_axe_1..3.json        item definitions → models
    models/item/ult_axe_1..3.json  voxel models
    textures/item/ult_axe_1..3.png
    textures/item/ult_axe_3.png.mcmeta   animation {frametime 3, interpolate true}
```

Level 3's texture is a 32×128 animated sheet that pulses. `pack_format: 75` is assumed for 1.21.11 and has not been verified; if the client reports an incompatible pack, update it.

### Item selection

`eaFxItem(level)` returns an item whose `minecraft:item_model` component is `executioner:ult_axe_<level>`.

### Display entity

`eaUltFx(p, level)` runs once per tick while Death's Toll is active:

- Spawns an `ITEM_DISPLAY` and mounts it with `addPassenger` (teleport fallback). Mounting removes follow lag.
- Billboard FIXED; rotation is `rawYaw + {ea-fx::yaw-offset}` where raw yaw comes from `getLocation().getYaw()`.
- Scale per level: `scale1/2/3` (defaults 1.1 / 1.8 / 2.6), raise animation `raise-ticks` 8, lift 0.15.
- `eaFxTransform(d, ty, scale, roll, ticks)` builds `Transformation(Vector3f, Quaternionf, Vector3f, Quaternionf)` with `interpolation_duration`; roll sign is multiplied by `{ea-fx::swing}`.
- Level 2: anvil sound and dust. Level 3: glow (custom colour via `org.bukkit.Color`), flame burst, embers.
- `eaUltFxEnd(p, hit)`: a hit slams forward (roll −80, 4 ticks); a miss tips back (75, 8 ticks).
- `eaUltFxKill(p)` plus `on quit` and `on death of player` clean up the display. A reload sweeps stray displays.
- `{ea-useAxeFx}` (true by default) switches back to the legacy particle semicircles (`eaDrawSemicircle`).

### Tuning and testing (op only)

| Command | Purpose |
|---|---|
| `/eatest [hit\|miss\|old]` | Plays only the visuals: no cost, cooldown or damage. `old` plays the particle version. |
| `/eafx yaw\|swing\|lift\|scale1\|scale2\|scale3 <number>` | Adjust the live values. The calibrated defaults are yaw 270 and swing −1. |

### Hosting on Minehut

Host `ea_pack.zip` at a direct-download URL, then set `resource-pack`, `resource-pack-sha1` and `require-resource-pack=true` in the server properties. The SHA-1 of the current pack is `2a80176d477363c97a57780165bed74d094e896e`; regenerate it whenever the zip changes. Players who decline are kicked when the pack is required.

Pack source is generated by two scripts (texture and voxel model generators); the handle is 2 px wide (x 7–8) and the blade is 1 px wide, centred on the handle.

## 13. Arena and world

Gamerules are set once on load using the Minecraft 1.21.11 snake_case names (for example `fire_spread_radius_around_player` takes a number, not a boolean). The map is static, so `setupArena` is only a safety cleanup and `teardownArena` returns players to the lobby. `16-world-rules.sk` locks weather, hunger, block break/place, drops, swap-hand, interaction with workbenches and pots, and manages the launch pad (slime block) and `{top::}` block protection.

## 14. Admin commands

| Command | Access | Purpose |
|---|---|---|
| `/duel`, `/testduel` | players | Queue for a match (`/testduel` records no stats) |
| `/class` | players | Class browser and per-class default consumable/perk |
| `/stats [player]`, `/matchhistory` | players | Stats and last 7 matches |
| `/stack` | players | Stack consumables |
| `/endduel` | op | Force-end the current duel, no stats |
| `/unstuck <player>` | op | Clear respawn state and send to lobby |
| `/balancestats`, `/resetbalancestats` | op | Open or wipe the balance dashboard |
| `/resetstats [player]` | op | Reinitialise a player's stats |
| `/testmastery <win rate %> <k/d>` | op | Preview a mastery tier |
| `/hset`, `/mana`, `/stamina`, `/gms`, `/gmsp`, `/gmc`, `/gma` | op | Debug helpers |
| `/top` | op | Builder-mode toggle (opt out of block protection) |
| `/disabletrapdoor` | op | World utility |
| `/eatest`, `/eafx` | op | Executioner visual testing |

## 15. Known risks and untested items

- The Executioner axe visual, item component, `pack_format` 75 and Skript's yaw convention were validated by review and user calibration, not exhaustive in-game testing.
- Double-to-float conversion in the `Transformation`/`Vector3f` constructors relies on skript-reflect coercion.
- Only one match can run at a time; state such as `{testMatch}`, `{firstBloodClaimed}` and `{matchStartTime}` is global.
- Some comments in the scripts still mention older class counts ("11 classes", "13 classes"); the live roster is 14.

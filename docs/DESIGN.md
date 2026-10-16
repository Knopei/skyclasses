# Design Document

## 1. Vision

A fast, readable 1v1 class-based PvP duel. Each match is a series of short fights (target 8–12 s each) inside a longer match with a shared life pool. Depth comes from three layers:

1. **Drafting** — which three classes you bring.
2. **In-match growth** — each class levels up independently through kills and objectives.
3. **Resource play** — mana and stamina limit how often you can use abilities.

## 2. Match flow

1. **Queue** — `/duel` (or `/testduel`). The first player waits; the second joins and the match starts.
2. **Draft** (in the `lobby` world)
   - 3 classes are auto-banned (not in `/testduel`). Bans come from a shuffle-bag deck, and a class banned in one game is held out of the next game's bans.
   - A coinflip decides who picks first.
   - Snake draft, 3 picks each. Pick timer is 10 s; an unmade pick is auto-assigned. A "Random Class" option is available.
   - A short window to choose which drafted class to start as (defaults to your first pick). Consumable and perk come from each class's saved defaults.
3. **Arena** — players spawn at opposite sides of `arena_1`, each with 30 lives, and mini-boss and major boss timers start.
4. **Fighting** — every death costs a life. The killer is credited only if they are the registered opponent.
5. **Respawn** — you are frozen, invisible and invulnerable at the top-centre of the map for 3 s, then pick one of your three classes (auto-assigned after 10 s to the first class not on cooldown). You respawn with 2 s spawn protection.
6. **End** — a player at 0 lives loses. A disconnect before first blood cancels the match with no stats; after first blood it is a forfeit and a real win/loss. A summary GUI opens for both players after the return to lobby.

## 3. Resources

| Resource | Max | Regeneration | Used for |
|---|---|---|---|
| Health | 9 hearts base | — | Juggernaut +1, Healthy Boy +1, Frankenstein Perk 1 +1 |
| Mana | 100 (displayed) | 1 per 5 ticks (~4/s) | Tacticals and ultimates |
| Stamina | 3 bars (displayed) | 7.5 internal units per 5 ticks | Sneak-dash, Basher's Bash, Ninja dashes |

Internally mana is stored ×10 and stamina ×300. A Mana Potion restores 50 mana and a Stamina Potion refills stamina completely. Respawning refunds 50 mana, and a kill grants stamina (+1 bar, or a full refill above 2 bars).

**Tactical (right-click)** costs mana and has no cooldown. **Ultimate (drop key)** costs mana and has a cooldown, and unlocks at class level 1.

## 4. Class categories

Each category grants a level 0 passive on top of the class kit.

| Category | Identity | Level 0 buff |
|---|---|---|
| **Juggernaut** | Durability, outlasting | +1 max health |
| **Striker** | Steady damage, pressure | Melee damage 5 instead of 4.5 (2.5 hearts instead of 2.25) |
| **Skirmisher** | Mobility, positioning | No fall stun (plus a level 2 passive: direct-hit tactical damage +0.25) |

Fall damage is disabled for everyone.

## 5. Classes

Costs are mana. Ultimate cooldown in seconds.

| Class | Cat. | Weapon | Passive | Tactical (cost) | Ultimate (cost / CD) |
|---|---|---|---|---|---|
| Viking | Striker | Axe | Little Rage: more damage/attack speed at low health | Bullseye (45): slowing arrow, speed on hit | Big Rage (55 / 80) |
| Demon | Juggernaut | Netherite sword | Life Steal | Blood Beam (45): damage + heal | Blood Harvest (55 / 70): auto beams at nearby enemies |
| Frankenstein | Juggernaut | Mace | Hammer Down | Leap (50): damaging, slowing landing | It's Alive! (60 / 90): electric shield, reflects projectiles |
| Ghost | Skirmisher | Golden sword | Quick Feet: permanent speed | Lunge (55): directional dash | Ghost (60 / 90): invisibility, speed, big %-health hit |
| Basher | Juggernaut | Shield | Stamina Recharge (Bash refunds stamina) | Shield (30 per second): channeled block | Shield Quake (55 / 70): ground slams, knock-up, stamina lock |
| Mighty | Striker | Mace | Hammer Down | Explode (55): AOE + knock-up, 20% max health capped at 4 | Mighty Leap (60 / 90): leap and slam |
| Ninja | Skirmisher | Nether star / katana | Improved Stamina | Shuriken Throw (50): three stars | Dragon Blade (60 / 80): speed and jump boost |
| Paladin | Juggernaut | Iron sword | Believer: regen below 5 hearts | Holy Shield (60): absorption | Holy Radiance (50 / 90): buff self, debuff enemies in an area |
| Rapid | Striker | Bow / dagger | Longshot: every 3rd melee hit fires a 25% max-health arrow | Sprint (50): declining speed | Rapid Bow (60 / 90): machine-gun arrows |
| Reaper | Skirmisher | Netherite hoe | Reaping Slash: wither every 3rd hit | Grapple (20 miss / 40 / 50 / 60 by range) | Soul Collector (60 / 70): wither and blindness sweep |
| Zeus | Striker | Trident | Pitch Fork: extended reach every 4th hit | Empowered Throw (45): lightning at impact | Thunderstorm (60 / 80) |
| Cyberpunk | Skirmisher | Diamond sword | Mana Breach: below half mana, steal enemy mana | Malicious Dart (45): disables stamina | Cyber Glitch (45 / 12 per charge, 2 charges): phase-dash through terrain |
| Tempest | Skirmisher | Iron hoe | Airborne: speed bursts | Air Burst (45): wind charge, 1.25 direct damage | Tornado (60 / 80): damaging directional tornado |
| Executioner | Striker | Netherite axe | Executioner's Mark | Cleaving Rush (55) | Death's Toll (55 / 80) |

### Executioner in detail

- **Executioner's Mark** — damage that leaves an enemy at 3 hearts or fewer (4 with Perk 1) applies repeating Slowness 2 until they heal above 3 hearts or die. Triggers from melee, the buffed tactical hit, the ultimate and grenades.
- **Cleaving Rush** — dash forward; the next hit deals **+55%** damage.
- **Death's Toll** — a 6 s window: 3 s of charging (level 1 instantly, level 2 at 1.5 s, level 3 at 3 s) plus 3 s of grace to land the hit. You take 20% less damage throughout and are slowed (Slowness 2) until full charge. The next melee hit consumes the charge: **2.5 / 5 / 7.5** damage by level. Perk 2 turns that into 25 / 50 / 75% of the target's max health, floored at the flat values. Damage is hard-capped at 10.
- **Visual** — a 3D axe over the player's head grows and reddens per charge level and glows red at max. See [TECHNICAL.md §12](TECHNICAL.md#12-executioner-resource-pack).

## 6. Leveling

Levels are per class, reset each match, and persist through death. XP is counted in "kills".

| Level | Threshold | Unlocks |
|---|---|---|
| 1 | 10 | Ultimate |
| 2 | 50 | Perk 1 |
| 3 | 100 | Perk 2 |

| Source | XP |
|---|---|
| Player kill | 10 |
| Major boss kill | 10 |
| Marauder (mini-boss) | 7 |
| Crawler (each of 3) | 2 |

**Death cooldown** (a class you died on is locked for swapping): `3 + 3 × class level + 2 × major bosses claimed`, capped at 30 s.

### Perks by class

| Class | Perk 1 (Lv 2) | Perk 2 (Lv 3) |
|---|---|---|
| Viking | Joyful to Furious: sharper health-scaling damage | Victim Chase: tactical on-hit speed 2 → 3 during ultimate |
| Demon | Extended Siphon: longer beam range | Vampire: more lifesteal in ultimate |
| Frankenstein | The Biggest Boy: +1 max health | Undying: stronger absorption shield |
| Ghost | Lunge Master: cheaper tactical | Supportive Ghost: cleanse and regen in ultimate |
| Basher | Resilient Basher: bigger stamina refund | Unsteady Grounds: ultimate 1.5 → 1.75 per tick, stamina pause 1.25 → 1.75 s |
| Mighty | Grenade Protection: less grenade damage | Fire Pit: fire ring on ultimate landing |
| Ninja | Stamina Master: farther stamina dashes | Fiery Shurikens: ignite during ultimate |
| Paladin | Elder Shield: shield lasts 3 s (from 2) | Supreme Radiance: larger radius |
| Rapid | Long Shot: 30% (from 25%) at 7.5+ blocks | Overheat: arrows ignite late in ultimate |
| Reaper | Extended Grapple: more range, better pull | Soul Collector: bonus damage/heal vs blinded enemies |
| Zeus | Weight Reduction: speed burst after tactical hit | Lightning Speed: faster ultimate telegraph |
| Cyberpunk | Stamina Lockout: longer dart lockout | Overclock: faster ultimate charge recovery |
| Tempest | Lingering Winds: Airborne speed 2 → 3 s | Category 5: Tornado travels farther |
| Executioner | Wider Mark: trigger at ≤ 4 hearts | Execution: %-max-health ultimate damage |

## 7. Loadout systems

### General perks (one active per class)

Healthy Boy (+1 heart), Stamina Recovery (+20% regen, speed 2 at full stamina), Reinforced Dash (−40% damage while dashing), Spell Caster (−15 ultimate mana, −10% cooldown for ultimates of 50 s or more), Enchanted Consumables (+0.5 heart on every consumable), Nimble Feet (no fall stun, brief speed 2), Quick Fuse (faster grenades, bonus damage to mobs and bosses by class level).

### Consumables (one selected per class)

Each respawn gives **2 golden apples plus 1 of the selected consumable**. A kill gives a golden apple plus your selected consumable (no duplicates).

| Item | Effect |
|---|---|
| Golden Apple | Heal 1 heart, regeneration 2 for 5 s |
| Health Potion | Heal 3 hearts |
| Golden Pepper | Speed 2 and regeneration 2 for 5 s |
| Golden Melon | Cleanse negatives, regeneration 2 for 5 s |
| Chorus Fruit | Brief invisibility, regeneration, +0.5 heart |
| Mana Potion | +50 mana |
| Stamina Potion | Full stamina |

Every class also gets **grenades** (up to 3, one every 30 s): 2 s fuse, 3 block radius.

Defaults are set per class in `/class` and can be overridden in-match during the respawn menu for that game only.

## 8. Objectives

| Objective | Details |
|---|---|
| **Mini-bosses** (4) | Two Marauders (12 HP zombie, two telegraphed AOE attacks) and two Crawler packs (3 cave spiders, 4 HP each). Respawn 30 s after the group clears. |
| **Major boss** | Iron Golem, 30 HP, up to 5 spawns every 3 minutes, 2 minute lifetime plus 30 s if still engaged. Three telegraphed attacks: Fortify, Ground Slam, Rock Throw. Reward is **+1 life**, previewed on a bossbar after the first hit. |
| **Secret doors** | Four tripwire-hook doors that reset at match start. |

Mobs are passive until hit and leash back to their spawn (5 blocks for mini-bosses, 7 for the golem, healing 10% on reset). Their touch damage is deliberately weak (0.5–1.0); only telegraphed attacks hit hard.

## 9. Combat state

For 5 s after dealing or taking damage with your opponent or the major boss, you are "in combat". Your opponent sees a bossbar with your class, level and health. Vanilla nametags are hidden unless you are in combat. Floating level tags are available but disabled (`{config::nametags-enabled}`).

## 10. Progression and stats

- **Mastery** per class: `score = win rate × 700 + min(K/D, 2) × 150`. A tier also needs a minimum games played.

| Tier | Min score | Min games |
|---|---|---|
| Trash | 0 | 0 |
| Rookie | 350 | 1 |
| Bronze | 375 | 3 |
| Silver | 400 | 3 |
| Gold | 450 | 3 |
| Diamond | 525 | 5 |
| Master | 600 | 10 |
| Grandmaster | 650 | 15 |
| Predator | 675 | 20 |

- A win only counts toward class mastery if you had 3+ combined kills and deaths on that class in the match.
- **Lifetime stats** (`/stats`): record, combat, damage, jungle, XP by phase (early < 5 min, mid < 10 min, late), personal records, comeback stats (7+ life lead).
- **Match summary** and the last 7 matches (`/matchhistory`).
- `/testduel` plays a full match without touching lifetime or balance stats.

## 11. Balance tooling

`/balancestats` (op) is an aggregate dashboard: ability usage and hit rate, damage by source (tactical/ultimate/melee), time to first kill/death, matchup win rates, draft and carry rates, level vs lives left, consumable and perk pick rates with K/D. All counters are event-driven with no tick cost.

## 12. Recent balance changes

- Mighty tactical cost 50 → **55** mana.
- Executioner tactical bonus +60% → **+55%**.
- Tempest Tornado tick damage 1.5 → **1.25** hearts.

## 13. Open items

- Single-match-at-a-time design. Multi-arena support is noted in `10-match-manager.sk`.
- Class browser ratings (difficulty, melee, range, toughness, crowd control, mobility) are hand-set flavour data.

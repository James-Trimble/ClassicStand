---
name: project_plan_storage_upgrades
description: "Agreed but UNIMPLEMENTED design (Direction A, per-ingredient preservation upgrades) plus a softening of the base overnight spoilage rates, to make spoilage manageable. Decided 2026-08-21."
metadata:
  node_type: memory
  type: project
---

Plan for the todo item **"Add storage or preservation upgrades the player can buy to slow water evaporation, salt clumping, and ice melting."** Chosen 2026-08-21: **Direction A (per-ingredient upgrades)** PLUS **softening the base spoilage rates** (both agreed). **Not yet coded.** From the [[project_design_feedback]] spoilage bullet ("spoilage is too fast"). Config-driven per [[project_data_driven_config]].

## Baseline (current overnight spoilage)
In `calculate_new_day_conditions` (`extrafuncts.nvgt:506–555`), on each new day where `daynumber > 1`, every perishable loses a random fraction of its **current stock** (floored), converted into a spoiled item added to inventory (recyclable). Rates are **hardcoded**:
- **Ice**: `random_float(0.5, 1.0)` — **50–100%, brutal** (stockpiling ice is impossible).
- **Lemons**: 0.1–0.3.
- **Salt**: 0.1–0.3.
- **Water**: 0.05–0.15.
Spoilage is already announced via `dlgmessage` (accessible). Melted/rotted/etc. items go to inventory to discard/recycle (existing behavior, keep).

## Chosen design

**Part 1 — soften the base rates (config rebalance).** Move the four rates out of hardcode into config (new `[spoilage]` section, min/max fraction per ingredient) and ship **softened defaults**, ice especially (e.g. down to roughly 0.2–0.4 range). Exact numbers TBD at impl. Moving them to config also enables the upgrade math and modder tuning.

**Part 2 — per-ingredient preservation upgrades (Direction A).** One permanent, saved upgrade per perishable, each **tiered I/II/III** with growing reduction:
- **Icebox** → ice
- **Pantry** → lemons
- **Sealed jars** → salt
- **Covered jugs** → water

Each owned tier multiplies down that ingredient's overnight loss fraction: in the spoilage code, `fraction *= (1 - reduction_for(ingredient))`, where the reduction comes from the owned tier (config-tunable per tier). You invest where your pain is (ice first).

**Buying them:** a shop with **one-time tier purchases** (own tier N to unlock tier N+1) — distinct from the existing ingredient/napkin/poster stores, which are quantity buys. Config-driven costs/reductions via a `.store`-style file + parser, but the parser needs an **"owned tier" concept** the current store parsers don't have.

**New state:** a per-ingredient storage tier (`ice_storage_level`, `lemon_storage_level`, `salt_storage_level`, `water_storage_level` or similar), saved in the save file, applied in the overnight spoilage loop.

**Accessibility:** overnight spoilage is already spoken; upgrade purchases confirmed by speech; the shop menu reads each upgrade's current tier and its effect.

## Decisions
- **Softened base spoilage rates — DECIDED 2026-08-22** (fresh nightly loss %, before any upgrade; all move to config): **ice 15–30%** (was 50–100%), **lemons 5–15%** (was 10–30%), **salt 5–15%** (was 10–30%), **water 5–10%** (was 5–15%).

- **Scope — DECIDED 2026-08-22: Option A, 4 upgrades, overnight spoilage only.** Icebox (ice), Pantry (lemons), Sealed jars (salt), Covered jugs (water). NO sugar upgrade and NO cold-freeze-event coverage — sugar only spoils via the `ingredient_freeze_sugar` cold event, which stays an unpreventable weather risk. (Confirmed sugar is NOT in the overnight spoilage loop; salt is in both the loop and its cold event, but upgrades only touch the loop.)
- **Tiers — DECIDED 2026-08-22: 4 tiers each, reductions 20% / 40% / 60% / 80%** of the nightly loss (owning a higher tier supersedes lower). At tier IV an ingredient loses only 20% of its base rate.

- **Shop placement & name — DECIDED 2026-08-22:** a new market station **"Preservation shop"** at **x37, y26–27** (1×2 tile, north of the cleaning station at 37,25; in the gap column between ingredients x32–36 and poster shop x38–42). Handler `preservemenu`. Map lines: `menu 37 37 26 27 preservemenu` + `zone 37 37 26 27 Preservation shop. Press enter to buy storage upgrades.`

- **Per-tier costs — DECIDED 2026-08-22: $5 / $20 / $40 / $60** (uniform across all four upgrades; flat one-time prices, config-tunable). Menu lists each upgrade's next buyable tier, **sorted cheapest-first**; a maxed (tier IV) upgrade drops off / shows maxed. Buying all four to tier IV cuts each ingredient's nightly loss to ~20% of base (small trickle, never zero — the 80% cap keeps a reason to keep selling).

## Open (implementation detail, handle at build time)
- Store/parser format for **one-time owned-tier purchases** (own tier N unlocks N+1). Likely a `.store`-style file listing the four upgrades with tier reductions + costs, a parser tracking the owned tier per upgrade (saved per game), and applying `loss *= (1 - reduction)` in the overnight spoilage loop. Base spoilage rates move fully to config too.

## Status: FULLY PLANNED, not yet built (dev paused the build 2026-08-22 for an unrelated task).

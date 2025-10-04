# `Battle Realms: 21 Turns to Build an Empire`

# Game Design & Implementation Specification (DRAFT)

## Table of Contents

1. [Vision & Philosophy](#vision--philosophy)
2. [System Overview Map](#system-overview-map)
3. [Core Game Loop](#core-game-loop)
4. [Turn Phase State Machine](#turn-phase-state-machine)
5. [Sector System](#sector-system)
6. [Labor System](#labor-system)
7. [Resource Economy](#resource-economy)
8. [Feedback Systems (Approval & Pollution)](#feedback-systems)
9. [Research System](#research-system)
10. [Market Systems](#market-systems)
11. [Victory Conditions & Scoring](#victory-conditions--scoring)
12. [Data Structures](#data-structures)
13. [Formulas & Calculations Reference](#formulas--calculations-reference)
14. [UI Requirements](#ui-requirements)

---

## Vision & Philosophy

### What We're Building

**A strategy game as a bounded challenge, not an endless treadmill.**

Modern games are designed as expensive time-sinks: endless progression systems, daily login rewards, fear-of-missing-out mechanics, and pay-to-skip timers. They extract time and money while teaching nothing of value.

Classical games were low-risk playgrounds for developing strategic thinking: chess, go, poker, and 1990s BBS door games like Barren Realms Elite. They had clear boundaries, definitive endings, and taught genuine skills—resource optimization, long-term planning, reading complex systems.

**Battle Realms is a return to classical game design with modern accessibility.**

### Core Statement

**"You have 21 turns to build an empire."**

This single constraint defines everything:
- **Bounded scope** - Not endless, has a clear end
- **Resource scarcity** - Fixed budget of turns
- **Strategic depth** - Must optimize within constraints
- **Definitive outcome** - Score is final
- **Replayability** - Master through repetition

### Design Pillars

**1. Constraints Create Value**
- Fixed 21 turns (no more, no less)
- 5-minute timer per turn (real time pressure)
- Finite resources (sectors, population capacity)
- Decisions have lasting consequences

**2. Strategic Depth Over Content Volume**
- Few systems, deep interactions
- Emergent complexity from simple rules
- Multiple viable strategies
- High skill ceiling, clear skill expression

**3. Respect for Player Intelligence**
- No tutorial hand-holding (learn by doing)
- No optimal path (discover strategies)
- No pay-to-win (pure skill competition)
- No artificial engagement hooks (the game is the hook)

### What Players Learn

- **Resource management under genuine scarcity** - You can't have everything, must prioritize
- **Long-term planning with irreversible decisions** - Early choices cascade through 21 turns
- **System optimization with feedback loops** - Approval/Pollution create dynamic constraints
- **Risk assessment and recovery** - When to commit, when to pivot
- **Pattern recognition and meta-gaming** - Discovering optimal strategies through iteration

### Success Criteria

**For MVP, we succeed if:**
1. Players voluntarily complete 3+ runs (replayability)
2. Completion rate >80% (turns 1-21, not abandoning mid-run)
3. Timer hit rate <30% (5 minutes is sufficient)
4. Score variance shows skill expression (top 10% < 2.5× median)
5. Players report "I want to try a different strategy" (emergent depth)

**We prove:** Bounded challenges with meaningful decisions can compete with endless progression for player engagement.

---

## System Overview Map

```
┌─────────────────────────────────────────────────────────────────┐
│                         GAME LOOP (21 TURNS)                    │
│                                                                 │
│  Turn Start → 6 Phases → Turn Resolution → Next Turn / End     │
│               (5 min)     (Server-side)                         │
└─────────────────────────────────────────────────────────────────┘
                                   ↓
┌─────────────────────────────────────────────────────────────────┐
│                      CORE RESOURCE ECONOMY                      │
│                                                                 │
│  Credits ←──────┐                                               │
│  Fuel   ←──────┼─── PRODUCTION (each turn)                     │
│  Research Points├─── Workers × Skill × (1 - Pollution Penalty) │
│  Population ←───┘    (except RP, not affected by pollution)    │
│                                                                 │
│  All stored without limits, can deficit (consequences apply)   │
└─────────────────────────────────────────────────────────────────┘
                                   ↓
┌─────────────────────────────────────────────────────────────────┐
│                        SECTOR SYSTEM                            │
│                                                                 │
│  4 Sector Types (owned by player):                             │
│  ┌────────────────┬──────────┬───────────┬────────────────┐    │
│  │ Gas Giant      │ Habitat  │ Trade Stn │ Research Colony│    │
│  ├────────────────┼──────────┼───────────┼────────────────┤    │
│  │ Produces: Fuel │ Provides │ Produces  │ Produces: RP   │    │
│  │ Cap: 50 pop    │ Capacity │ Credits   │ Cap: 75 pop    │    │
│  │ Pollutes: +0.2 │ 200 pop  │ Cap: 100  │ Cleans: -0.2   │    │
│  │ Maint: 100 Cr  │ Cleans:  │ Pollutes: │ Maint: 200 Cr  │    │
│  │                │ -0.1     │ +0.05     │ + 100 Fuel     │    │
│  │                │ Maint:   │ Maint:    │                │    │
│  │                │ 200 Cr + │ 100 Cr    │                │    │
│  │                │ 100 Fuel │           │                │    │
│  └────────────────┴──────────┴───────────┴────────────────┘    │
│                                                                 │
│  Market: 100 sectors total (shared pool across all types)      │
│  Price increases as supply decreases (1× → 3× over 100→0)      │
│  Can buy/sell anytime (Phase 3), sell at 80% of market price   │
└─────────────────────────────────────────────────────────────────┘
                                   ↓
┌─────────────────────────────────────────────────────────────────┐
│                        LABOR SYSTEM                             │
│                                                                 │
│  Population assigned to 3 roles:                               │
│  ┌──────────────┬────────────────┬──────────────────────┐      │
│  │ Extractors   │ Traders        │ Researchers          │      │
│  ├──────────────┼────────────────┼──────────────────────┤      │
│  │ Work in:     │ Work in:       │ Work in:             │      │
│  │ Gas Giants   │ Trade Stations │ Research Colonies    │      │
│  │              │                │                      │      │
│  │ Produce:     │ Produce:       │ Produce:             │      │
│  │ 6 Fuel/turn  │ 1,200 Cr/turn  │ 0.25 RP/turn         │      │
│  │              │                │                      │      │
│  │ Affected by  │ Affected by    │ NOT affected by      │      │
│  │ pollution    │ pollution      │ pollution            │      │
│  └──────────────┴────────────────┴──────────────────────┘      │
│                                                                 │
│  Skill System:                                                  │
│  • Start at 50% efficiency when assigned                       │
│  • Increase +5% per turn (cap: 100% after 10 turns)            │
│  • RESET to 50% when reassigned to different role              │
│  • Workers assigned beyond sector capacity produce nothing     │
│                                                                 │
│  Path Dependency: Early assignments compound, switching costly │
└─────────────────────────────────────────────────────────────────┘
                                   ↓
┌─────────────────────────────────────────────────────────────────┐
│                    FEEDBACK LOOPS (Dynamic Constraints)         │
│                                                                 │
│  APPROVAL RATING (0-100%)                                       │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │ Affects: Population growth rate                          │  │
│  │                                                          │  │
│  │ Increases from:              Decreases from:            │  │
│  │ • Credit surplus (+1)        • Credit deficit (-2)      │  │
│  │ • Low density <70% (+1)      • Fuel deficit (-2/turn)   │  │
│  │ • Successful expansion (+1)  • High density >85% (-1)   │  │
│  │ • All maintenance paid (+1)  • Very high >95% (-3)      │  │
│  │                              • Failed maintenance (-5)   │  │
│  │                              • Pop. decline (-2)         │  │
│  │                              • High pollution >60% (-2)  │  │
│  │                                                          │  │
│  │ Critical Threshold: <30% = 10% revolt chance per turn   │  │
│  │ Revolt: -20% population, -10% credits, reset to 40%     │  │
│  └──────────────────────────────────────────────────────────┘  │
│                                                                 │
│  POLLUTION (0-100%)                                             │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │ Affects: Production efficiency (Credits & Fuel only)     │  │
│  │                                                          │  │
│  │ Penalty Curve:                                          │  │
│  │ • 0-20%: No penalty                                     │  │
│  │ • 21-40%: -5% production                                │  │
│  │ • 41-60%: -15% production                               │  │
│  │ • 61-80%: -30% production                               │  │
│  │ • 81-100%: -50% production (economy choking)            │  │
│  │                                                          │  │
│  │ Generation: +0.2/turn per Gas Giant, +0.05 per Trade Stn│  │
│  │ Reduction: -0.1/turn per Habitat, -0.2 per Research Col │  │
│  │ Natural Decay: -1% of current pollution per turn        │  │
│  └──────────────────────────────────────────────────────────┘  │
│                                                                 │
│  POPULATION GROWTH                                              │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │ Formula: base_rate × approval_factor × capacity_factor   │  │
│  │                                                          │  │
│  │ base_rate = 5% max                                       │  │
│  │ approval_factor = (approval - 50) / 50                   │  │
│  │   (negative if approval < 50 → emigration)              │  │
│  │ capacity_factor:                                         │  │
│  │   < 70% density: 1.0× (plenty of room)                  │  │
│  │   70-90% density: 1.0× (normal)                         │  │
│  │   90-95% density: 0.5× (crowded, slowed)                │  │
│  │   > 95% density: 0.0× (at capacity, no growth)          │  │
│  │                                                          │  │
│  │ Calculated and applied during Turn Resolution           │  │
│  └──────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────┘
                                   ↓
┌─────────────────────────────────────────────────────────────────┐
│                      RESEARCH SYSTEM                            │
│                                                                 │
│  Single Technology: TERRAFORM SECTOR                            │
│                                                                 │
│  Unlock: 500 RP (one-time cost)                                │
│  Effect: Change any owned sector to any other type             │
│                                                                 │
│  Upgrade Progression (10 levels):                              │
│  ┌─────────┬──────────────┬────────────────┬────────────────┐  │
│  │ Level   │ RP Cost      │ Terraform Cost │ Total RP Spent │  │
│  ├─────────┼──────────────┼────────────────┼────────────────┤  │
│  │ 1       │ 500          │ 50,000 Cr      │ 500            │  │
│  │ 2       │ +200         │ 45,000 Cr      │ 700            │  │
│  │ 3       │ +300         │ 40,000 Cr      │ 1,000          │  │
│  │ 4       │ +400         │ 35,000 Cr      │ 1,400          │  │
│  │ 5       │ +500         │ 30,000 Cr      │ 1,900          │  │
│  │ ...     │ ...          │ ...            │ ...            │  │
│  │ 10 (MAX)│ +1,000       │ 5,000 Cr       │ 5,900          │  │
│  └─────────┴──────────────┴────────────────┴────────────────┘  │
│                                                                 │
│  Terraform Mechanics:                                           │
│  • Costs Credits (based on level), instant transformation      │
│  • Does NOT reassign workers automatically                     │
│  • Workers remain in old role, produce nothing until manually  │
│    reassigned (which resets their skill to 50%)                │
│  • Cannot terraform offline sectors (must pay maintenance)     │
│                                                                 │
│  Strategic Uses:                                                │
│  • Fix early mistakes (bought wrong sector types)              │
│  • Adapt to strategy pivots mid-game                           │
│  • Optimize sector mix in late game (turns 16-21)              │
│  • Manage pollution (convert Gas Giants to cleaner types)      │
└─────────────────────────────────────────────────────────────────┘
                                   ↓
┌─────────────────────────────────────────────────────────────────┐
│                    TURN RESOLUTION (Server-Side)                │
│                                                                 │
│  Execution Order (critical for deterministic results):         │
│                                                                 │
│  1. CALCULATE PRODUCTION                                        │
│     • Count workers per role × base production × skill         │
│     • Apply pollution penalty (Credits & Fuel only)            │
│     • Cap by sector capacity (excess workers produce nothing)  │
│     • Add resources to player totals                           │
│                                                                 │
│  2. APPLY MAINTENANCE COSTS                                     │
│     • Deduct Credits/Fuel for each online sector               │
│     • If insufficient: mark sectors offline, accumulate debt   │
│                                                                 │
│  3. UPDATE APPROVAL RATING                                      │
│     • Calculate deltas from all sources (see Feedback Loops)   │
│     • Clamp to 0-100 range                                     │
│     • Check for revolt (if <30%: RNG roll based on run_id +    │
│       turn_number seed, 10% chance)                            │
│     • If revolt: apply penalties, set flag (revolt_occurred)   │
│                                                                 │
│  4. UPDATE POLLUTION                                            │
│     • Sum generation (Gas Giants, Trade Stations)              │
│     • Sum reduction (Habitats, Research Colonies)              │
│     • Apply natural decay (1% of current)                      │
│     • Clamp to 0-100 range                                     │
│                                                                 │
│  5. CALCULATE POPULATION GROWTH                                 │
│     • Use approval rating and capacity factor                  │
│     • Apply growth/decline delta                               │
│     • Floor at 0 (cannot go negative)                          │
│     • Check if <50: trigger Game Over flag                     │
│                                                                 │
│  6. UPDATE WORKER SKILLS                                        │
│     • For each labor group still assigned: +5% skill (max 100%)│
│     • Newly reassigned groups already reset to 50% in Phase 4  │
│                                                                 │
│  7. SAVE STATE SNAPSHOT                                         │
│     • Store turn_history entry with all values                 │
│     • Calculate and cache networth                             │
│                                                                 │
│  8. INCREMENT TURN COUNTER                                      │
│     • If turn < 21: prepare for next turn                      │
│     • If turn == 21: calculate final score, submit leaderboard │
│                                                                 │
│  9. CHECK GAME OVER CONDITIONS                                  │
│     • Population < 50: Empire Collapsed                        │
│     • Turn == 21: Calculate Final Score                        │
└─────────────────────────────────────────────────────────────────┘
                                   ↓
┌─────────────────────────────────────────────────────────────────┐
│                  VICTORY CONDITIONS & SCORING                   │
│                                                                 │
│  BASE NETWORTH (sum of all assets at Turn 21):                 │
│  credits + (fuel × 10) + sector_market_values +                │
│  (population × 100) + (research_points × 50)                   │
│                                                                 │
│  FINAL SCORE = base_networth × efficiency × sustainability ×   │
│                mastery                                          │
│                                                                 │
│  EFFICIENCY MULTIPLIER (1.0 - 1.5×):                            │
│  Based on productive turns (turns with networth increase)      │
│  = 1.0 + (productive_turns / 21) × 0.5                         │
│                                                                 │
│  SUSTAINABILITY MULTIPLIER (0.5 - 1.2×):                        │
│  Based on final pollution and approval                         │
│  = [(100 - pollution) × 0.006] + [approval × 0.006]           │
│  Penalizes dirty/unstable empires, rewards clean/stable        │
│                                                                 │
│  MASTERY MULTIPLIER (1.0 - 2.0×):                               │
│  Based on strategic execution:                                 │
│  • Terraform unlock timing: (21 - unlock_turn)/21 × 30 points │
│  • Terraform level: level × 3 points                           │
│  • Average worker skill: avg_skill × 0.4 points                │
│  • Terraform usage: min(num_uses × 5, 20) points               │
│  = 1.0 + (total_points / 100)                                  │
│                                                                 │
│  Winner: Highest Final Score                                   │
│  Tiebreaker: Higher base networth, then earlier completion     │
└─────────────────────────────────────────────────────────────────┘
```

### Key Feedback Loops

**1. Economic Snowball (Positive Feedback)**
```
More Credits → Buy more Trade Stations → Assign more Traders →
More Credits → (repeat) → Exponential growth

Constraint: Sector market depletion (rising prices slow expansion)
Constraint: Population capacity (need Habitats to grow workers)
Constraint: Pollution (reduces Credit production efficiency)
```

**2. Pollution Death Spiral (Negative Feedback)**
```
High Pollution → Lower production → Less Credits →
Can't buy Research Colonies → Pollution increases →
(repeat) → Economy collapses

Escape: Early investment in Research Colonies (prevention)
Escape: Terraform Gas Giants to cleaner types (cure)
```

**3. Population Density Trap (Negative Feedback)**
```
High Density → Lower Approval → Population decline →
Less workers → Less production → Can't afford Habitats →
Density stays high → (repeat)

Escape: Build Habitats preemptively (before density crisis)
Escape: Leave population unassigned (reserve capacity buffer)
```

**4. Skill Lock-In (Path Dependency)**
```
Assign 200 Traders Turn 1 → 10 turns to reach 100% skill →
Locked into Trade Station strategy (switching resets to 50%)

Trade-off: Early commitment vs. late flexibility
Strategic decision: When to pivot? (cost = 10 turns of skill growth)
```

### Strategic Archetypes

**Rush Strategy (Terraform Early)**
- Assign heavy Researchers Turn 1 (200+ workers)
- Unlock terraform by Turn 8-10
- Sacrifice early economic growth for flexibility
- High Mastery multiplier (early unlock, high level)
- Risk: Lower base networth (fewer Credits accumulated)

**Economic Strategy (Terraform Late/Never)**
- Assign heavy Traders Turn 1 (300+ workers)
- Maximize Credit production early
- Buy many sectors while prices low
- High base networth, low Mastery multiplier
- Risk: Locked into initial sector choices, high pollution

**Balanced Strategy (Mid-Game Terraform)**
- Mixed assignments (150 Traders, 150 Extractors, 50 Researchers)
- Unlock terraform Turn 12-15
- Moderate networth, moderate multipliers
- Adapt based on Turn 10-12 state
- Risk: Neither specialized advantage

**Sustainability Focus (Pollution Management)**
- Buy Research Colonies early
- Keep pollution <40% (avoid heavy penalties)
- High Sustainability multiplier
- Lower production (Research Colonies expensive)
- Risk: Slower growth, may lose to Economic players

### Critical Decision Points

**Turn 1: Initial Allocation**
- Must buy Habitats (population capacity)
- Must buy income sectors (Trade Stations or Gas Giants)
- Sets trajectory for entire run (path dependency)

**Turn 7-10: Commitment Phase**
- Workers approaching max skill (90-100%)
- Switching becomes very expensive
- Must decide: Stay course or pivot?

**Turn 12-15: Mid-Game Pivot**
- Terraform typically unlocks here (Balanced strategy)
- First chance to correct early mistakes
- Sector market 50% depleted (prices rising)

**Turn 16-21: Optimization Phase**
- Last chance for meaningful changes
- Terraform Level 5-7 (cheap transforms)
- Focus: Maximize multipliers for scoring

### Common Failure Modes

**1. No Habitats → Population Starvation**
- Buy only Trade Stations Turn 1
- Population cannot grow (no capacity)
- Approval tanks, population declines
- Game Over by Turn 5-8

**2. Skill Reset Cascade**
- Reassign workers multiple times
- All workers stuck at 50% skill
- Production never scales
- Low final score (poor Mastery multiplier)

**3. Pollution Choking**
- Buy many Gas Giants (need Fuel for maintenance)
- Pollution hits 60-80% (-30% production penalty)
- Cannot afford Research Colonies (already in deficit)
- Death spiral, cannot recover

**4. Market Timing Miss**
- Wait until Turn 10+ to buy sectors
- Prices 2-3× higher (market depleted)
- Credits spent on expensive sectors
- Lower final networth (fewer sectors owned)

**5. Maintenance Cascade Failure**
- Over-expand without income buffer
- One turn of negative Credits
- Cannot pay maintenance → sectors offline
- Approval tanks → population declines → production drops
- Positive feedback loop to failure

### Design Intent Summary

**Constraints Create Interesting Choices:**
- 21 turns (not 20, not 25 - specific enough to plan)
- 5-minute timer (pressure without panic)
- Skill reset on reassignment (commitment matters)
- Market depletion (early expansion advantage)
- Pollution/Approval trade-offs (cannot optimize both)

**Emergent Complexity:**
- No dominant strategy (Rock-Paper-Scissors balance)
- Multiple viable paths to victory
- Early mistakes are recoverable (terraform unlock)
- Skill ceiling visible but distant (optimization depth)

**Replayability Sources:**
- Different strategic archetypes
- Timing variations (when to unlock/pivot)
- Optimization challenges (maximize each multiplier)
- Leaderboard competition (beat top scores)
- Self-improvement (beat personal best)

This is not a game about grinding or paying. It's a game about **thinking**, **planning**, and **optimizing within constraints**. The 21-turn structure makes every run a complete strategic puzzle with a definitive solution - but finding that solution requires skill, experience, and adaptation.

---

## Core Game Loop

### Game Structure

**One Run = 21 Turns**

Each turn consists of:
1. **Production Report** (view-only, 30s)
2. **Maintenance Payment** (required, 30s)
3. **Sector Market** (buy/sell/terraform, 1-2min)
4. **Labor Assignment** (reassign population, 1-2min)
5. **Research** (unlock/upgrade terraform, 30s)
6. **End Turn Summary** (preview next turn, 30s)

Total turn time: **~5 minutes** (enforced by timer)

When timer expires: Turn auto-completes with no changes applied (all actions cancelled)

### Session Structure

**Players can complete turns at any pace:**
- All 21 turns in 2 hours (speedrun style)
- 1-3 turns per day over a week (casual)
- Variable pacing (life happens)

**No regeneration:** Turns don't refill over time. Once you start a run with 21 turns, those are all you get.

**When all 21 turns are used:**
- Final score calculated
- Submitted to leaderboard
- Run complete (can start new run)

### Meta Loop

**Run → Score → Learn → Iterate**

```
Start Run (21 turns available)
    ↓
Play Turn 1-21 (5 minutes each)
    ↓
View Final Score + Analysis
    ↓
Compare to Leaderboard
    ↓
Identify Optimization Opportunities
    ↓
Start New Run (different strategy)
```

**Goal:** Players discover emergent strategies through iteration, not through reading guides.

---

## Turn Phase State Machine

### State Diagram

```
[TURN START]
    ↓
┌─────────────────────────────────────┐
│ Phase 1: PRODUCTION REPORT          │
│ - Display last turn's production    │
│ - Display any events/changes        │
│ - Show current resource totals      │
│ - Read-only, no actions             │
│ Duration: 30s (or player continues) │
└─────────────────────────────────────┘
    ↓
┌─────────────────────────────────────┐
│ Phase 2: MAINTENANCE PAYMENT        │
│ - Auto-calculate costs              │
│ - Deduct from resources if able     │
│ - If insufficient: prompt selection │
│ - Apply consequences if unpaid      │
│ Duration: 30s                       │
└─────────────────────────────────────┘
    ↓
┌─────────────────────────────────────┐
│ Phase 3: SECTOR MARKET              │
│ - View available sectors            │
│ - Buy sectors (if affordable)       │
│ - Sell owned sectors                │
│ - Terraform sectors (if unlocked)   │
│ Duration: 1-2min                    │
└─────────────────────────────────────┘
    ↓
┌─────────────────────────────────────┐
│ Phase 4: LABOR ASSIGNMENT           │
│ - View current assignments          │
│ - Reassign workers (loses skill)    │
│ - View warnings about consequences  │
│ Duration: 1-2min                    │
└─────────────────────────────────────┘
    ↓
┌─────────────────────────────────────┐
│ Phase 5: RESEARCH                   │
│ - View RP total                     │
│ - Unlock Terraform (500 RP)         │
│ - Upgrade Terraform (200-1000 RP)   │
│ Duration: 30s                       │
└─────────────────────────────────────┘
    ↓
┌─────────────────────────────────────┐
│ Phase 6: END TURN SUMMARY           │
│ - Preview next turn production      │
│ - Show warnings (deficits, etc)     │
│ - Confirm or cancel changes         │
│ Duration: 30s                       │
└─────────────────────────────────────┘
    ↓
[TURN END - Process turn resolution]
    ↓
[Option to Start Next Turn OR game ends if turn 21]
```

### Timer Behavior

**Timer starts when turn begins: 5:00 countdown**

**During each phase:**
- Timer continues counting
- Can skip ahead to next phase anytime
- Cannot go back to previous phases

**When timer reaches 0:00:**
- Current phase freezes
- All pending changes cancelled
- Turn ends with no modifications applied
- Next turn begins with previous state

**Critical:** Timer expiration = "did nothing this turn" (harsh penalty)

### Phase 1: Production Report (Detailed)

**Display format:**

```
═══════════════════════════════════
        TURN X/21 - PRODUCTION
═══════════════════════════════════

RESOURCES GAINED:
+ 180,000 Credits (from 150 Traders @ 80% skill)
+ 1,200 Fuel (from 200 Extractors @ 70% skill)
+ 25 Research Points (from 100 Researchers @ 65% skill)

POPULATION CHANGE:
+ 18 Population (Approval: 72%, Density: 85%)
  Total Population: 1,268 / 1,500 capacity

APPROVAL: 72% (Stable)
+ Won last turn's market timing (+1)
+ Positive credit balance (+1)
- High density 85% (-1)

POLLUTION: 34% (-5% production penalty)
+ Gas Giants generating +1.0/turn
- Natural decay -0.5/turn
= Net +0.5/turn

WARNINGS:
⚠ Fuel deficit projected next turn (-300)
⚠ Approaching density cap (85%)

[CONTINUE] ▶
```

**Data displayed:**
- Each resource type gained/lost
- Source attribution (which workers produced what)
- Current skill levels
- Population change with explanation
- Approval rating with delta breakdown
- Pollution level with delta breakdown
- Warnings for projected problems

**No actions available - just information**

### Phase 2: Maintenance Payment (Detailed)

**Maintenance costs calculated:**

For each sector type you own:
```
Gas Giant: 100 Credits/sector
Habitat: 200 Credits + 100 Fuel/sector
Trade Station: 100 Credits/sector
Research Colony: 200 Credits + 100 Fuel/sector
```

**Display format:**

```
═══════════════════════════════════
        MAINTENANCE PAYMENT
═══════════════════════════════════

YOUR SECTORS:
Gas Giants (×5): 500 Credits
Habitats (×6): 1,200 Credits + 600 Fuel
Trade Stations (×4): 400 Credits
Research Colonies (×1): 200 Credits + 100 Fuel
───────────────────────────────────
TOTAL COST: 2,300 Credits + 700 Fuel

CURRENT BALANCE:
Credits: 245,000 ✓
Fuel: 1,800 ✓

AFTER PAYMENT:
Credits: 242,700 ✓
Fuel: 1,100 ✓

[AUTO-PAY ALL] ✓
[SELECT WHICH SECTORS TO PAY] ⚙
```

**If insufficient resources:**

```
⚠ INSUFFICIENT RESOURCES

You cannot afford full maintenance.
You must choose which sectors to pay.

Unpaid sectors:
- Go offline (produce nothing)
- Cannot be sold until paid
- Lose 1 Approval per unpaid sector

[SELECT WHICH TO PAY]
```

**Consequences of failed maintenance:**
- Sector goes offline (produces nothing)
- Workers assigned to offline sectors produce nothing
- -1 Approval per unpaid sector
- Sector cannot be sold/terraformed until debt paid
- Debt accumulates (must pay back-maintenance to reactivate)

### Phase 3: Sector Market (Detailed)

**Two distinct markets:**

**A. Sector Purchase/Sale Market**
```
═══════════════════════════════════
           SECTOR MARKET
═══════════════════════════════════

AVAILABLE FOR PURCHASE: 73/100

Gas Giant: 4,200 Credits [BUY]
  - Produces: Fuel (base 6/turn per worker)
  - Capacity: 50 population
  - Maintenance: 100 Credits/turn

Habitat: 5,800 Credits [BUY]
  - Produces: Population capacity
  - Capacity: 200 population
  - Maintenance: 200 Credits + 100 Fuel/turn

Trade Station: 3,500 Credits [BUY]
  - Produces: Credits (base 1,200/turn per worker)
  - Capacity: 100 population
  - Maintenance: 100 Credits/turn

Research Colony: 4,000 Credits [BUY]
  - Produces: Research Points (base 0.25/turn per worker)
  - Capacity: 75 population
  - Maintenance: 200 Credits + 100 Fuel/turn

YOUR SECTORS (15 total):
Gas Giants (×5) [SELL for 3,200 each]
Habitats (×6) [SELL for 4,400 each]
Trade Stations (×4) [SELL for 2,800 each]
Research Colonies (×1) [SELL for 3,200 each]

[CONTINUE] ▶
```

**B. Terraform (if unlocked)**
```
═══════════════════════════════════
      TERRAFORM YOUR SECTORS
═══════════════════════════════════

Terraform Technology: Level 3
Cost per Terraform: 40,000 Credits

SELECT SECTOR TO TRANSFORM:
[Gas Giant #1] - 50 Extractors assigned
[Gas Giant #2] - 150 Extractors assigned
[Gas Giant #3] - EMPTY ⭐ (recommended)
[Gas Giant #4] - 200 Extractors assigned
[Gas Giant #5] - 100 Extractors assigned

TRANSFORM TO:
○ Gas Giant (no change)
● Trade Station
○ Habitat
○ Research Colony

COST: 40,000 Credits

⚠ Workers will remain assigned but produce
   nothing until reassigned to matching work.

[CONFIRM TERRAFORM] ✓
[CANCEL] ✗
```

**Rules:**
- Can buy as many sectors as you can afford
- Can sell any sector you own (at 80% of current market price)
- Cannot sell offline sectors (must pay maintenance first)
- Terraform requires: tech unlocked + sufficient credits + owned sector
- Terraforming does NOT reassign workers (manual step)

### Phase 4: Labor Assignment (Detailed)

```
═══════════════════════════════════
        POPULATION: 1,268
═══════════════════════════════════

Capacity: 1,500 (85% full)
⚠ High density reduces Approval

CURRENT ASSIGNMENTS:

EXTRACTORS: 200 (⚠ 90% skill)
  Producing: 1,200 Fuel/turn
  Required sectors: Gas Giants (you have 5)
  [-50] [-10] [+10] [+50]
  ⚠ Reassigning resets to 50% skill!

TRADERS: 150 (⚠ 80% skill)
  Producing: 180,000 Credits/turn
  Required sectors: Trade Stations (you have 4)
  [-50] [-10] [+10] [+50]
  ⚠ Reassigning resets to 50% skill!

RESEARCHERS: 100 (⚠ 65% skill)
  Producing: 25 RP/turn
  Required sectors: Research Colonies (you have 1)
  [-50] [-10] [+10] [+50]
  ⚠ Reassigning resets to 50% skill!

UNASSIGNED: 818
  Not producing anything
  
SECTOR CAPACITY CHECK:
✓ Extractors: 200 / 250 max (5 Gas Giants)
✓ Traders: 150 / 400 max (4 Trade Stations)
⚠ Researchers: 100 / 75 max (1 Research Colony)
  Excess: 25 workers producing nothing!

[OPTIMIZE AUTO] 🤖
[CONFIRM CHANGES] ✓
[CANCEL] ✗
```

**Rules:**
- Total assigned ≤ total population
- Assigned to sector type ≤ sector capacity
- Excess workers produce nothing (warning shown)
- Reassigning workers resets skill to 50%
- Skill increases by +5% per turn, caps at 100%
- Unassigned workers don't gain/lose skill

**Auto-Optimize button behavior:**
- Removes excess workers from over-capacity sectors
- Suggests balanced allocation based on sector mix
- Still requires player confirmation (not forced)

### Phase 5: Research (Detailed)

**Before unlock:**
```
═══════════════════════════════════
            RESEARCH
═══════════════════════════════════

Research Points: 350 / 500
████████████████░░░░ (70%)

TERRAFORM SECTOR TECHNOLOGY
Status: 🔒 LOCKED

Unlock Cost: 500 RP (need 150 more)

Effect when unlocked:
• Change any sector to any other type
• Initial cost: 50,000 Credits/terraform
• Upgradeable to reduce cost

Researchers producing: 25 RP/turn
Estimated unlock: Turn 13

[CONTINUE] ▶
```

**After unlock, before max level:**
```
═══════════════════════════════════
            RESEARCH
═══════════════════════════════════

Research Points: 150 RP available

TERRAFORM SECTOR TECHNOLOGY
Status: ✅ UNLOCKED
Current Level: 3
Terraform Cost: 40,000 Credits

UPGRADE TO LEVEL 4
Cost: 400 RP (need 250 more)
New Cost: 35,000 Credits/terraform
Savings: 5,000 Credits per use

Researchers producing: 30 RP/turn
Estimated upgrade: Turn 16

UPGRADE PATH:
Level 3 → 4: 400 RP (40k → 35k Credits)
Level 4 → 5: 500 RP (35k → 30k Credits)
Level 5 → 6: 600 RP (30k → 25k Credits)
...
Level 10 (MAX): 5,000 Credits/terraform

[CONTINUE] ▶
```

**After max level:**
```
═══════════════════════════════════
            RESEARCH
═══════════════════════════════════

Research Points: 2,450 RP (unused)

TERRAFORM SECTOR TECHNOLOGY
Status: ✅ MAX LEVEL (10)
Terraform Cost: 5,000 Credits

You have mastered terraform technology.
(No further upgrades available)

[CONTINUE] ▶
```

### Phase 6: End Turn Summary (Detailed)

```
═══════════════════════════════════
         END TURN SUMMARY
═══════════════════════════════════

CHANGES THIS TURN:
✓ Bought 2 Trade Stations (-7,000 Credits)
✓ Terraformed Gas Giant → Habitat (-40,000 Credits)
✓ Reassigned 100 Extractors → Traders (skill reset to 50%)

PROJECTED NEXT TURN:

PRODUCTION:
+ 190,000 Credits (includes new Traders @ 50% skill)
+ 600 Fuel (reduced from reassignment)
+ 30 Research Points

MAINTENANCE:
- 2,500 Credits (2 new sectors)
- 800 Fuel

NET RESULT:
+ 187,500 Credits ✓
- 200 Fuel ⚠ (will deficit)

APPROVAL: 72% → 71% (-1 from high density)
POLLUTION: 34% → 35% (+1%)

WARNINGS:
⚠ Fuel deficit next turn
⚠ 100 Traders at only 50% skill (lost 30%)

CURRENT NETWORTH: 2,450,000 (#8 on leaderboard)

[CONFIRM END TURN] ✓
[CANCEL - MAKE MORE CHANGES] ✗
```

**If player clicks CONFIRM:**
- All changes applied
- Turn counter increments (X/21 → X+1/21)
- Turn resolution runs (server-side)
- Next turn begins with Phase 1 (Production Report)

**If player clicks CANCEL:**
- Returns to Phase 3 (Sector Market)
- Can continue modifying
- Timer continues counting

**If timer expires during this phase:**
- All changes CANCELLED
- No actions applied
- Turn ends with no modifications

---

## Sector System

### Sector Types (4 Total)

Each sector has:
- **Type** - Gas Giant, Habitat, Trade Station, Research Colony
- **Population Capacity** - Max workers it can host
- **Production** - What it generates per turn
- **Maintenance Cost** - Required payment each turn
- **Pollution Generation** - Environmental impact

### Gas Giant

**Purpose:** Fuel production

**Attributes:**
```
Population Capacity: 50
Base Production: 6 Fuel per worker per turn
Maintenance Cost: 100 Credits/turn
Pollution Generation: +0.2 per sector per turn
```

**Production Formula:**
```
fuel_produced = workers_assigned × 6 × (worker_skill / 100) × (1 - pollution_penalty)

Example:
50 workers @ 80% skill, 30% pollution (-3% penalty):
= 50 × 6 × 0.80 × 0.97
= 232.8 Fuel/turn
```

**Strategic Role:**
- Essential early (need Fuel for Habitat maintenance)
- Diminishing returns (only need so much Fuel)
- Pollution generator (trade-off)
- Terraform target (convert excess to Trade Stations)

### Habitat

**Purpose:** Population capacity

**Attributes:**
```
Population Capacity: 200
Base Production: None (provides capacity only)
Maintenance Cost: 200 Credits + 100 Fuel/turn
Pollution Reduction: -0.1 per sector per turn (life support scrubbing)
```

**Capacity Formula:**
```
total_capacity = sum(habitat_capacity) + sum(other_sector_capacity)

Example:
6 Habitats (1,200) + 5 Gas Giants (250) + 4 Trade Stations (400) = 1,850 capacity
```

**Strategic Role:**
- Required for population growth
- Reduces pollution (side benefit)
- Expensive maintenance (Credits + Fuel)
- Density management (need enough but not too many)

### Trade Station

**Purpose:** Credit generation

**Attributes:**
```
Population Capacity: 100
Base Production: 1,200 Credits per worker per turn
Maintenance Cost: 100 Credits/turn
Pollution Generation: +0.05 per sector per turn (minimal)
```

**Production Formula:**
```
credits_produced = workers_assigned × 1,200 × (worker_skill / 100) × (1 - pollution_penalty)

Example:
100 workers @ 90% skill, 25% pollution (-2% penalty):
= 100 × 1,200 × 0.90 × 0.98
= 105,840 Credits/turn
```

**Strategic Role:**
- Primary income source
- Scales with worker count + skill
- Low maintenance cost (self-funding)
- Core of economic snowball strategies

### Research Colony

**Purpose:** Research Point generation

**Attributes:**
```
Population Capacity: 75
Base Production: 0.25 RP per worker per turn
Maintenance Cost: 200 Credits + 100 Fuel/turn
Pollution Reduction: -0.2 per sector per turn (research includes environmental tech)
```

**Production Formula:**
```
rp_produced = workers_assigned × 0.25 × (worker_skill / 100)

Example:
75 workers @ 100% skill:
= 75 × 0.25 × 1.0
= 18.75 RP/turn
```

**Note:** Research Points are NOT affected by pollution penalty (intellectual work)

**Strategic Role:**
- Unlocks Terraform technology
- Reduces pollution (important side effect)
- Expensive maintenance (high opportunity cost)
- Rush strategy vs. economic strategy trade-off

### Sector Attributes Table

| Attribute | Gas Giant | Habitat | Trade Station | Research Colony |
|-----------|-----------|---------|---------------|-----------------|
| **Capacity** | 50 | 200 | 100 | 75 |
| **Production** | 6 Fuel/worker | Capacity only | 1,200 Credits/worker | 0.25 RP/worker |
| **Maintenance** | 100 Cr | 200 Cr + 100 Fuel | 100 Cr | 200 Cr + 100 Fuel |
| **Pollution** | +0.2/turn | -0.1/turn | +0.05/turn | -0.2/turn |
| **Affected by Pollution?** | Yes | N/A | Yes | No |

### Sector Ownership Rules

**Initial state (Turn 1):**
- Player owns 0 sectors
- Market has 100 sectors available
- Player must buy initial sectors Turn 1

**Acquisition:**
- Purchase from Sector Market (Phase 3)
- Price determined by market supply curve

**Disposal:**
- Sell back to Sector Market (Phase 3)
- Receive 80% of current market price
- Cannot sell offline sectors (must pay maintenance first)

**Transformation:**
- Terraform (if tech unlocked)
- Costs credits based on Terraform level
- Does NOT automatically reassign workers

**Capacity constraints:**
- No hard limit on sectors owned
- Practical limit: maintenance cost + population capacity
- Strategic limit: sector market supply (100 total across all players in future multiplayer)

---

## Labor System

### Labor Types (3 Total)

Each labor type has:
- **Role** - What sector type they work in
- **Base Production** - Output per worker at 100% skill
- **Starting Skill** - Efficiency when first assigned
- **Skill Growth** - How quickly they improve
- **Skill Cap** - Maximum efficiency

### Extractors

**Role:** Work in Gas Giants, produce Fuel

**Attributes:**
```
Base Production: 6 Fuel per worker per turn
Starting Skill: 50%
Skill Growth: +5% per turn
Skill Cap: 100%
```

**Production Formula:**
```
fuel_output = num_extractors × 6 × (skill / 100) × (1 - pollution_penalty)

Skill progression example:
Turn 1: Assign 100 extractors → 50% skill → 300 Fuel/turn
Turn 2: → 55% skill → 330 Fuel/turn
Turn 3: → 60% skill → 360 Fuel/turn
...
Turn 11: → 100% skill → 600 Fuel/turn
```

**Strategic Considerations:**
- High initial output (Fuel needed early)
- Fast to max skill (10 turns to 100%)
- Affected by pollution (trade-off with Gas Giants)
- Often over-assigned early, under-assigned late (terraform play)

### Traders

**Role:** Work in Trade Stations, produce Credits

**Attributes:**
```
Base Production: 1,200 Credits per worker per turn
Starting Skill: 50%
Skill Growth: +5% per turn
Skill Cap: 100%
```

**Production Formula:**
```
credit_output = num_traders × 1,200 × (skill / 100) × (1 - pollution_penalty)

Skill progression example:
Turn 1: Assign 200 traders → 50% skill → 120,000 Credits/turn
Turn 2: → 55% skill → 132,000 Credits/turn
Turn 3: → 60% skill → 144,000 Credits/turn
...
Turn 11: → 100% skill → 240,000 Credits/turn
```

**Strategic Considerations:**
- Primary income source (Credits buy everything)
- Scales linearly with worker count
- Long-term investment (takes 10 turns to maximize)
- Core of "economic snowball" strategy

### Researchers

**Role:** Work in Research Colonies, produce Research Points

**Attributes:**
```
Base Production: 0.25 RP per worker per turn
Starting Skill: 50%
Skill Growth: +5% per turn
Skill Cap: 100%
NOT affected by pollution
```

**Production Formula:**
```
rp_output = num_researchers × 0.25 × (skill / 100)

Skill progression example:
Turn 1: Assign 100 researchers → 50% skill → 12.5 RP/turn
Turn 2: → 55% skill → 13.75 RP/turn
Turn 3: → 60% skill → 15 RP/turn
...
Turn 11: → 100% skill → 25 RP/turn
```

**Strategic Considerations:**
- Unlocks Terraform (500 RP = critical threshold)
- Not affected by pollution (intellectual work)
- Low per-worker output (need many workers OR time)
- High opportunity cost (workers not producing Credits/Fuel)
- Rush vs. Economic strategy pivot point

### Labor Mechanics

**Assignment:**
- Players assign population to labor types
- Total assigned ≤ total population
- Assigned to type ≤ sector capacity for that type
- Excess workers produce nothing (warning shown)

**Skill System:**
```
Initial Assignment:
worker_skill = 50%

Each Turn (if still assigned):
if worker_skill < 100%:
    worker_skill = min(worker_skill + 5%, 100%)

On Reassignment:
worker_skill = 50% (reset)
```

**Example Skill Progression:**
```
Turn 1: Assign 100 workers as Traders → 50% skill
Turn 2-11: Workers remain Traders → skill increases to 100%
Turn 12: Reassign 50 to Researchers → those 50 reset to 50% skill
Turn 13-22: Those 50 Researchers increase to 100% skill
```

**Critical Rule: Reassignment resets skill**

This creates:
- **Path dependency** (early choices matter)
- **Switching cost** (pivoting is expensive)
- **Strategic commitment** (must plan ahead)

**Sector Capacity Constraint:**
```
If workers assigned > sector capacity:
    excess_workers = assigned - capacity
    effective_workers = capacity
    excess_workers produce nothing
    warning displayed
```

**Example:**
```
You have: 1 Research Colony (75 capacity)
You assign: 100 Researchers
Result: Only 75 produce RP, 25 produce nothing
Warning: "25 Researchers exceed capacity!"
```

### Population Dynamics

**Population grows/shrinks based on Approval Rating:**

```
population_delta = base_growth × (approval - 50) / 50 × capacity_factor

Where:
base_growth = 0.05 (5% max growth rate)
approval = current approval rating (0-100)
capacity_factor = (current_pop / total_capacity)

If approval > 50: positive growth
If approval < 50: negative growth (emigration)
If approval < 30: risk of revolt
```

**Capacity Factor:**
```
if capacity_factor < 0.7:
    # Plenty of room, growth encouraged
    capacity_factor = 1.0
elif capacity_factor < 0.9:
    # Getting crowded, normal growth
    capacity_factor = 1.0
elif capacity_factor < 0.95:
    # Very crowded, slowed growth
    capacity_factor = 0.5
else:
    # At capacity, emigration pressure
    capacity_factor = 0.0 (no growth possible)
```

**Example Calculations:**

**Scenario A: Healthy Growth**
```
Current Population: 1,000
Total Capacity: 1,500 (67% full)
Approval: 75%

population_delta = 0.05 × (75 - 50) / 50 × 1.0
                = 0.05 × 0.5 × 1.0
                = 0.025 (2.5% growth)

New Population = 1,000 × 1.025 = 1,025
```

**Scenario B: Overcrowding**
```
Current Population: 1,400
Total Capacity: 1,500 (93% full)
Approval: 70%

population_delta = 0.05 × (70 - 50) / 50 × 0.5
                = 0.05 × 0.4 × 0.5
                = 0.01 (1% growth)

New Population = 1,400 × 1.01 = 1,414
```

**Scenario C: Decline**
```
Current Population: 1,200
Total Capacity: 1,500
Approval: 35%

population_delta = 0.05 × (35 - 50) / 50 × 1.0
                = 0.05 × -0.3 × 1.0
                = -0.015 (-1.5% decline)

New Population = 1,200 × 0.985 = 1,182
```

### Unassigned Population

**Unassigned workers:**
- Do not produce anything
- Do not gain or lose skill
- Still consume resources (counted in maintenance indirectly)
- Still contribute to density calculations
- Can be assigned at any time

**Strategic use:**
- Reserve for future expansion
- Avoid skill reset when terraform-ing (assign after transformation)
- Flexibility for pivots

---

## Resource Economy

### Primary Resources (4 Total)

**1. Credits**
```
Purpose: Universal currency
Starting Amount: 50,000
Gained From: Traders (Trade Stations)
Spent On: Buying sectors, maintenance, terraform
Cannot Go Negative: Yes (blocks purchases)
```

**2. Fuel**
```
Purpose: Maintenance resource (Habitats, Research Colonies)
Starting Amount: 500
Gained From: Extractors (Gas Giants)
Spent On: Maintenance costs
Cannot Go Negative: No (can deficit)
```

**3. Population**
```
Purpose: Workers / labor force
Starting Amount: 500
Gained From: Growth (based on Approval + Capacity)
Lost From: Decline (low Approval), Revolt
Cannot Go Negative: No (but practical minimum ~100)
```

**4. Research Points**
```
Purpose: Unlock/upgrade Terraform technology
Starting Amount: 0
Gained From: Researchers (Research Colonies)
Spent On: Unlock Terraform (500 RP), Upgrades (200-1000 RP)
Cannot Go Negative: Yes (blocks unlocks)
Carries Over: Yes (unused RP persist)
```

### Resource Storage

**No storage limits** - all resources can accumulate infinitely

**Rationale:**
- Simplifies early game (no "build storage" chores)
- Allows economic snowball strategies
- Credits accumulation is part of optimization puzzle
- No artificial caps = no cap-related exploits

### Resource Deficit Handling

**Credits Deficit:**
```
if credits < cost_of_action:
    action_blocked = True
    show_error("Insufficient Credits")
```
Cannot go negative. Purchases/actions blocked.

**Fuel Deficit:**
```
if fuel < maintenance_cost:
    fuel = negative_value
    approval -= 2 per turn while negative
    show_warning("Fuel Deficit - Approval declining")
```
Can go negative. Penalizes Approval.

**Population Deficit:**
```
if population < 100:
    approval -= 5 per turn
    show_critical_warning("Population Crisis")
    
if population < 50:
    game_over("Empire Collapsed - Population too low")
```

Can decline naturally. Below thresholds = penalties.

**Research Points Deficit:**
```
if rp < unlock_cost:
    unlock_blocked = True
    show_message("Need {remaining} more RP")
```
Cannot go negative. Unlocks blocked.

### Starting Resources (Turn 1)

```
Credits: 50,000
Fuel: 500
Population: 500
Research Points: 0
Sectors Owned: 0
Approval: 70% (neutral starting point)
Pollution: 0% (clean slate)
```

**Forced Actions Turn 1:**
- Must buy at least 1 Habitat (for population capacity)
- Should buy income sectors (Trade Stations)
- Should buy Fuel sectors (Gas Giants)
- Must assign population to start production

**Typical Turn 1 strategy:**
```
Buy: 3 Habitats (17,400 Cr), 3 Trade Stations (10,500 Cr), 2 Gas Giants (8,400 Cr)
Cost: 36,300 Credits
Remaining: 13,700 Credits

Assign:
200 → Traders (50% skill = 120,000 Credits/turn)
150 → Extractors (50% skill = 450 Fuel/turn)
50 → Researchers (50% skill = 6.25 RP/turn)
100 → Unassigned (reserve)

Total Capacity: 1,050 (Habitats 600 + TS 300 + GG 100 + RC 50)
Population: 500/1,050 (48% density - good)
```

---

## Feedback Systems

### Approval Rating

**Purpose:** Represents population happiness, affects growth rate

**Starting Value:** 70% (neutral)

**Range:** 0% - 100%

**Effects:**
```
if approval > 50:
    positive population growth
    
if approval < 50:
    negative population growth (emigration)
    
if approval < 30:
    10% chance per turn: REVOLT EVENT
    - Lose 20% of population
    - Lose 10% of credits
    - Approval resets to 40%
```

### Approval Delta Calculation (Per Turn)

**Formula:**
```
approval_delta = sum(all_approval_modifiers)

approval_new = clamp(approval_old + approval_delta, 0, 100)
```

**Positive Modifiers:**

| Condition | Approval Change | Notes |
|-----------|----------------|-------|
| Credits surplus | +1 | If credits increased this turn |
| Low density (<70%) | +1 | Plenty of living space |
| Successful expansion | +1 | If bought sectors this turn |
| No deficits | +1 | All maintenance paid, no negative resources |

**Negative Modifiers:**

| Condition | Approval Change | Notes |
|-----------|----------------|-------|
| Credits deficit | -2 | If credits decreased this turn |
| Fuel deficit | -2 per turn | While fuel < 0 |
| High density (>85%) | -1 | Overcrowding |
| Very high density (>95%) | -3 | Severe overcrowding |
| Failed maintenance | -5 per unpaid sector | Sectors offline |
| Population decline | -2 | If population decreased |
| High pollution (>60%) | -2 | Environmental dissatisfaction |

**Example Turn Calculation:**

```
Turn 10 State:
Approval: 68%

This Turn Events:
+ Credits increased (surplus): +1
+ Low density (65%): +1
- High pollution (64%): -2
- Population declined: -2

approval_delta = +1 +1 -2 -2 = -2
approval_new = 68 + (-2) = 66%
```

**Critical Thresholds:**

```
Approval >= 80: "Thriving" - max growth rate
Approval 60-79: "Stable" - normal growth
Approval 40-59: "Discontent" - slowed growth
Approval 30-39: "Unrest" - decline + revolt risk
Approval < 30: "Revolt Imminent" - 10% chance/turn
```

### Pollution

**Purpose:** Environmental degradation, reduces production efficiency

**Starting Value:** 0% (clean)

**Range:** 0% - 100%

**Effects:**
```
pollution_penalty = pollution_level / 100 × production_modifier

Production Penalty Table:
0-20%: 0% penalty (no effect)
21-40%: -5% production
41-60%: -15% production
61-80%: -30% production
81-100%: -50% production (severe choking)
```

**Applied To:**
- Fuel production (Extractors)
- Credit production (Traders)
- NOT applied to Research Points (intellectual work immune)

### Pollution Delta Calculation (Per Turn)

**Formula:**
```
pollution_delta = pollution_generation - pollution_reduction - natural_decay

pollution_new = clamp(pollution_old + pollution_delta, 0, 100)
```

**Pollution Generation:**

| Source | Pollution/Turn | Notes |
|--------|---------------|-------|
| Gas Giant | +0.2 per sector | Heavy extraction |
| Trade Station | +0.05 per sector | Commercial activity |

**Pollution Reduction:**

| Source | Reduction/Turn | Notes |
|--------|---------------|-------|
| Habitat | -0.1 per sector | Life support scrubbing |
| Research Colony | -0.2 per sector | Environmental tech research |

**Natural Decay:**
```
natural_decay = current_pollution × 0.01

(1% of current pollution naturally dissipates per turn)
```

**Example Turn Calculation:**

```
Turn 8 State:
Pollution: 42%

Current Sectors:
5 Gas Giants: +1.0
4 Trade Stations: +0.2
6 Habitats: -0.6
1 Research Colony: -0.2

Natural Decay: 42 × 0.01 = -0.42

pollution_delta = +1.0 +0.2 -0.6 -0.2 -0.42 = -0.02
pollution_new = 42 + (-0.02) = 41.98% ≈ 42%
```

**Strategic Implications:**

**Early Game (Turns 1-7):**
- Pollution low (few sectors)
- Can ignore pollution penalties
- Focus on economic growth

**Mid Game (Turns 8-15):**
- Pollution rising (many Gas Giants for fuel)
- -5% to -15% production penalty emerges
- Must start balancing (buy Research Colonies)

**Late Game (Turns 16-21):**
- Pollution critical decision point
- High pollution (-30% penalty) vs. terraform cost
- Research Colonies become valuable (reduce pollution + generate RP)

**Critical Thresholds:**

```
Pollution < 20%: "Clean" - no penalty
Pollution 20-40%: "Moderate" - small penalty
Pollution 40-60%: "High" - significant penalty
Pollution 60-80%: "Severe" - major penalty + Approval loss
Pollution > 80%: "Critical" - choking economy
```

### Feedback Loop Interactions

**Approval ↔ Pollution:**
```
High Pollution (>60%) → -2 Approval/turn
Low Approval (<50%) → Population decline
Population decline → -2 Approval (feedback)
Low Approval → Less production (fewer workers)
Less production → Harder to fix problems
(Death spiral possible)
```

**Approval ↔ Density:**
```
High Density (>85%) → -1 to -3 Approval/turn
Low Approval → Slower growth
Slower growth → Easier to maintain density
(Self-regulating feedback)
```

**Pollution ↔ Production:**
```
High Pollution → Lower production (-30%)
Lower production → Less Credits/Fuel
Less Credits → Can't buy Research Colonies
Can't reduce Pollution → Stuck
(Requires strategic planning to avoid)
```

---

## Research System

### Terraform Sector Technology

**Only research available in MVP**

### Unlock Requirements

**Initial State:** Locked

**Unlock Cost:** 500 Research Points (one-time)

**Effect When Unlocked:**
- Enables "Terraform" action in Sector Market phase
- Initial terraform cost: 50,000 Credits per use
- Can transform any owned sector to any other type

### Upgrade Progression

**After unlock, can upgrade through 10 levels:**

| Level | RP Cost (Cumulative) | Terraform Cost | RP Investment Total |
|-------|---------------------|----------------|---------------------|
| 1 (Unlock) | 500 RP | 50,000 Cr | 500 RP |
| 2 | +200 RP | 45,000 Cr | 700 RP |
| 3 | +300 RP | 40,000 Cr | 1,000 RP |
| 4 | +400 RP | 35,000 Cr | 1,400 RP |
| 5 | +500 RP | 30,000 Cr | 1,900 RP |
| 6 | +600 RP | 25,000 Cr | 2,500 RP |
| 7 | +700 RP | 20,000 Cr | 3,200 RP |
| 8 | +800 RP | 15,000 Cr | 4,000 RP |
| 9 | +900 RP | 10,000 Cr | 4,900 RP |
| 10 (MAX) | +1,000 RP | 5,000 Cr | 5,900 RP |

**Formula:**
```
terraform_cost = max(50000 - (level - 1) × 5000, 5000)

upgrade_cost(level) = level × 100 RP
```

### Research Point Generation

**Only source:** Researchers working in Research Colonies

**Formula:**
```
rp_per_turn = num_researchers × 0.25 × (skill / 100)

Example:
100 Researchers @ 80% skill = 100 × 0.25 × 0.8 = 20 RP/turn
```

**Accumulation:**
- RP never spent except on Terraform unlocks/upgrades
- Unused RP carries over indefinitely
- No cap on RP accumulation

### Strategic Timing

**Unlock Timing Analysis:**

**Turn 8-10 (Rush Strategy):**
```
Starting Turn 1:
- Assign 200 Researchers immediately
- Build 3 Research Colonies (capacity 225)
- Researchers @ 50% skill = 25 RP/turn

Turn 1: 25 RP
Turn 2: 28 RP (55% skill) = 53 total
Turn 3: 30 RP (60% skill) = 83 total
...
Turn 9: 50 RP (100% skill) = 500 total

Unlocked by Turn 9-10
```

**Turn 13-15 (Balanced Strategy):**
```
Starting Turn 1:
- Assign 100 Researchers
- Build 1-2 Research Colonies
- Researchers @ 50% skill = 12.5 RP/turn

Turn 1-10: Accumulate ~200 RP
Turn 10: Reassign more to Research (150 total)
Turn 11-14: Accumulate remaining 300 RP

Unlocked by Turn 13-15
```

**Turn 18+ (Economic Strategy):**
```
Starting Turn 1:
- Assign 50 Researchers
- Build 1 Research Colony
- Focus on Credits/Fuel growth

Turn 1-15: Slow accumulation ~150 RP
Turn 15: Reassign more to Research
Turn 16-20: Rush remaining RP

Unlocked by Turn 18+ (may not have time to use much)
```

### Terraform Mechanics

**Requirements to Terraform:**
- Technology unlocked (500 RP spent)
- Own the sector to transform
- Have sufficient Credits (based on tech level)
- Sector not offline (maintenance paid)

**Process:**
1. Select owned sector
2. Choose new sector type
3. Pay credit cost
4. Transformation instant

**Important:** Terraform does NOT reassign workers
```
Example:
You have: Gas Giant with 100 Extractors assigned
You terraform: Gas Giant → Trade Station
Result: Trade Station with 100 Extractors assigned (producing nothing)
Action Required: Manually reassign Extractors → Traders
```

**Strategic Uses:**

**1. Fix Early Mistakes**
```
Turn 3: Bought 6 Gas Giants (seemed like good idea)
Turn 12: Realize I have too much Fuel, need Credits
Turn 13: Terraform unlocked
Turn 14: Transform 3 Gas Giants → Trade Stations
Result: Corrected sector imbalance
```

**2. Adapt to Strategy Pivot**
```
Turn 1-10: Economic build (many Trade Stations)
Turn 11: See leader has research rush (terraform early)
Turn 12: Realize I need to catch up
Turn 13: Transform 2 Trade Stations → Research Colonies
Turn 14-18: Rush terraform technology
Result: Late pivot attempt
```

**3. Late-Game Optimization**
```
Turn 18: Terraform Level 7 (cheap transforms)
Turn 18-21: Transform sectors to perfect ratio
- Convert excess Gas Giants → Trade Stations
- Convert 1 Trade Station → Research Colony (pollution)
Result: Optimized sector mix for final turns
```

**4. Pollution Management**
```
Turn 15: Pollution at 65% (-30% production penalty)
Turn 16: Transform 2 Gas Giants → Research Colonies
Result: -0.4 pollution generation, +0.4 reduction = net -0.8/turn
Turn 17-21: Pollution drops to manageable levels
```

---

## Market Systems

### Sector Market

**Purpose:** Buy/sell sectors, limited supply decreases over time

**Starting State (Turn 1):**
```
Total Available: 100 sectors
Sectors Sold: 0
Base Price: 3,000 Credits
```

**Price Formula:**
```
sector_price(type, sectors_remaining) = base_price(type) × price_multiplier(sectors_remaining)

Where:
base_price(Gas Giant) = 3,000
base_price(Habitat) = 4,000
base_price(Trade Station) = 2,500
base_price(Research Colony) = 3,500

price_multiplier = 1 + (100 - sectors_remaining) / 100 × 2

Simplified:
price_multiplier = 1 + (sectors_sold / 50)
```

**Price Progression Example:**

| Sectors Remaining | Multiplier | Gas Giant | Habitat | Trade Station | Research Colony |
|-------------------|-----------|-----------|---------|---------------|-----------------|
| 100 | 1.0× | 3,000 | 4,000 | 2,500 | 3,500 |
| 90 | 1.2× | 3,600 | 4,800 | 3,000 | 4,200 |
| 75 | 1.5× | 4,500 | 6,000 | 3,750 | 5,250 |
| 50 | 2.0× | 6,000 | 8,000 | 5,000 | 7,000 |
| 25 | 2.5× | 7,500 | 10,000 | 6,250 | 8,750 |
| 10 | 2.8× | 8,400 | 11,200 | 7,000 | 9,800 |
| 0 | 3.0× | 9,000 | 12,000 | 7,500 | 10,500 |

**Selling Back:**
```
sell_price = current_market_price × 0.8

(20% loss on resale - prevents buy/sell cycling)
```

**Market Depletion:**
```
As players buy sectors across turns:
sectors_remaining decreases → prices increase

This creates:
- Urgency (buy sectors early when cheap)
- Scarcity (can't buy unlimited sectors)
- Market timing decisions (when to expand?)
```

**Turn-by-Turn Typical Depletion (Single Player):**

| Turn | Sectors Remaining | Notes |
|------|-------------------|-------|
| 1 | 100 → 92 | Initial purchases (8 sectors) |
| 2 | 92 → 85 | Expansion (7 sectors) |
| 3-5 | 85 → 75 | Steady growth (10 sectors) |
| 6-10 | 75 → 60 | Moderate expansion (15 sectors) |
| 11-15 | 60 → 50 | Slowing (prices rising) |
| 16-21 | 50 → 40 | Late game (expensive, selective buying) |

**Strategic Implications:**

**Early Buying Advantage:**
```
Turn 1: Gas Giant costs 3,000
Turn 15: Gas Giant costs 6,500
Savings: 3,500 Credits per sector

Player who buys 10 sectors early saves 35,000 Credits
```

**Market Timing:**
```
Question: Buy sectors now or wait for more Credits?

Early: Cheap prices, but limited income
Late: Higher income, but expensive prices

Optimal: Balance (buy some early, some mid-game)
```

---

## Victory Conditions & Scoring

### Victory Condition

**After Turn 21: Player with highest SCORE wins**

Score is calculated from final Networth with multipliers for efficiency, sustainability, and strategy.

### Networth Calculation

**Base Networth Formula:**
```
networth = 
    credits +
    (fuel × 10) +
    (sum of all sector values) +
    (population × 100) +
    (research_points × 50)
```

**Component Breakdown:**

**Credits:**
```
credits_value = current_credits

(Credits count 1:1 in Networth)
```

**Fuel:**
```
fuel_value = current_fuel × 10

(Fuel valued at 10 Credits equivalent)
```

**Sectors:**
```
sector_value = current_market_price_of_each_sector

(Sectors valued at what you could sell them for)

Example:
5 Gas Giants @ 6,500 = 32,500
6 Habitats @ 8,000 = 48,000
4 Trade Stations @ 5,000 = 20,000
1 Research Colony @ 7,000 = 7,000
Total Sector Value: 107,500
```

**Population:**
```
population_value = current_population × 100

(Each population unit worth 100 Credits)
```

**Research Points:**
```
rp_value = current_research_points × 50

(Unused RP have value)
```

**Example Networth Calculation:**
```
Turn 21 Final State:
Credits: 850,000
Fuel: 2,500
Sectors: 16 (valued at 110,000)
Population: 2,200
Research Points: 450 (unused)

Networth = 850,000 + (2,500 × 10) + 110,000 + (2,200 × 100) + (450 × 50)
         = 850,000 + 25,000 + 110,000 + 220,000 + 22,500
         = 1,227,500
```

### Score Multipliers

**Final Score Formula:**
```
final_score = base_networth × efficiency_mult × sustainability_mult × mastery_mult

Where multipliers are:
efficiency_mult = 1.0 to 1.5 (based on turn usage)
sustainability_mult = 0.5 to 1.2 (based on pollution/approval)
mastery_mult = 1.0 to 2.0 (based on strategy execution)
```

### Efficiency Multiplier

**Purpose:** Reward players who finish strong, don't waste turns

**Formula:**
```
# Count "productive turns" (turns where networth increased)
productive_turns = count(turns where networth_delta > 0)

efficiency_mult = 1.0 + (productive_turns / 21) × 0.5

Range: 1.0× (no productive turns) to 1.5× (all 21 productive)
```

**Example:**
```
Player A: 21/21 turns increased networth = 1.5× multiplier
Player B: 18/21 turns increased networth = 1.43× multiplier
Player C: 14/21 turns increased networth = 1.33× multiplier
```

### Sustainability Multiplier

**Purpose:** Reward clean/stable empires, penalize pollution/instability

**Formula:**
```
sustainability_score = (100 - final_pollution) × 0.006 + (final_approval × 0.006)

sustainability_mult = sustainability_score / 100

Range: 0.5× (100% pollution, 0% approval) to 1.2× (0% pollution, 100% approval)
```

**Example:**
```
Player A: 20% pollution, 85% approval
= (80 × 0.006) + (85 × 0.006) = 0.48 + 0.51 = 0.99× mult

Player B: 65% pollution, 45% approval
= (35 × 0.006) + (45 × 0.006) = 0.21 + 0.27 = 0.48× mult (harsh penalty!)

Player C: 5% pollution, 92% approval
= (95 × 0.006) + (92 × 0.006) = 0.57 + 0.55 = 1.12× mult (rewarded!)
```

### Mastery Multiplier

**Purpose:** Reward strategic execution (terraform usage, tech timing, skill management)

**Formula:**
```
mastery_score = 0

# Terraform unlock timing (earlier = better)
if terraform_unlocked:
    unlock_bonus = (21 - unlock_turn) / 21 × 30
    mastery_score += unlock_bonus

# Terraform upgrade level (higher = better)
terraform_level_bonus = terraform_level × 3
mastery_score += terraform_level_bonus

# Worker skill optimization (fewer reassignments = better)
avg_skill = average(all_worker_skills)
skill_bonus = avg_skill × 0.4
mastery_score += skill_bonus

# Terraform usage (used terraform effectively)
terraform_usage_bonus = min(num_terraforms × 5, 20)
mastery_score += terraform_usage_bonus

mastery_mult = 1.0 + (mastery_score / 100)

Range: 1.0× (no strategy) to 2.0× (perfect execution)
```

**Example:**
```
Player A (Terraform Rush Master):
- Unlocked Turn 9: (21-9)/21 × 30 = 17.14
- Terraform Level 8: 8 × 3 = 24
- Avg Skill 85%: 85 × 0.4 = 34
- 8 Terraforms: 8 × 5 = 40 (capped at 20)
= 17.14 + 24 + 34 + 20 = 95.14 → 1.95× mult

Player B (Economic Focus):
- Unlocked Turn 16: (21-16)/21 × 30 = 7.14
- Terraform Level 2: 2 × 3 = 6
- Avg Skill 92%: 92 × 0.4 = 36.8
- 2 Terraforms: 2 × 5 = 10
= 7.14 + 6 + 36.8 + 10 = 59.94 → 1.60× mult

Player C (No Terraform):
- Never unlocked: 0
- Level 0: 0
- Avg Skill 78%: 78 × 0.4 = 31.2
- 0 Terraforms: 0
= 31.2 → 1.31× mult
```

### Final Score Examples

**Example 1: Terraform Rush Master**
```
Base Networth: 2,800,000
Efficiency: 1.45× (20/21 productive turns)
Sustainability: 1.08× (15% pollution, 88% approval)
Mastery: 1.95× (terraform master)

Final Score = 2,800,000 × 1.45 × 1.08 × 1.95
            = 8,525,160
```

**Example 2: Economic Powerhouse**
```
Base Networth: 4,200,000
Efficiency: 1.48× (21/21 productive turns)
Sustainability: 0.68× (58% pollution, 62% approval - dirty growth)
Mastery: 1.60× (good execution, late terraform)

Final Score = 4,200,000 × 1.48 × 0.68 × 1.60
            = 6,781,824
```

**Example 3: Balanced Strategy**
```
Base Networth: 3,500,000
Efficiency: 1.40× (18/21 productive turns)
Sustainability: 0.95× (28% pollution, 75% approval)
Mastery: 1.70× (solid execution)

Final Score = 3,500,000 × 1.40 × 0.95 × 1.70
            = 7,863,500
```

**Key Insight:** Highest Networth doesn't always win. Strategic execution matters.

### Leaderboard Categories

**1. Overall High Score**
- Single best final score
- All-time record

**2. Highest Networth**
- Best base networth (before multipliers)
- Pure economic achievement

**3. Fastest Terraform Unlock**
- Earliest turn terraform was unlocked
- Research rush achievement

**4. Cleanest Empire**
- Lowest final pollution at 2M+ networth
- Sustainability achievement

**5. Most Efficient**
- Highest efficiency multiplier
- Perfect turn usage

**Display Format:**
```
╔═══════════════════════════════════╗
║         LEADERBOARD               ║
╠═══════════════════════════════════╣
║                                   ║
║ #1  PlayerName    8,525,160       ║
║     └ Networth: 2.8M              ║
║     └ Terraform: Turn 9           ║
║     └ Strategy: Rush Master       ║
║                                   ║
║ #2  OtherPlayer  7,863,500        ║
║     └ Networth: 3.5M              ║
║     └ Terraform: Turn 12          ║
║     └ Strategy: Balanced          ║
║                                   ║
║ #3  ThirdPlayer  6,781,824        ║
║     └ Networth: 4.2M (highest!)   ║
║     └ Terraform: Turn 16          ║
║     └ Strategy: Economic          ║
║                                   ║
║ ...                               ║
║                                   ║
║ #47 YOU          2,450,000        ║
║     └ Networth: 2.4M              ║
║     └ Terraform: Turn 19          ║
║     └ Strategy: Learning          ║
║                                   ║
║ [VIEW FULL LEADERBOARD]           ║
║ [ANALYZE TOP STRATEGIES]          ║
║ [PLAY AGAIN]                      ║
╚═══════════════════════════════════╝
```

---

## Data Structures

### Player State Object

```typescript
interface PlayerState {
  // Identity
  player_id: string;
  player_name: string;
  run_id: string; // Unique per run
  
  // Turn tracking
  current_turn: number; // 1-21
  turn_start_time: timestamp; // For 5-min timer
  
  // Resources
  credits: number;
  fuel: number;
  population: number;
  research_points: number;
  
  // Sectors
  sectors: Sector[];
  
  // Labor
  labor_assignments: {
    extractors: LaborGroup;
    traders: LaborGroup;
    researchers: LaborGroup;
  };
  
  // Feedback systems
  approval_rating: number; // 0-100
  pollution_level: number; // 0-100
  
  // Research
  terraform_unlocked: boolean;
  terraform_level: number; // 0-10
  
  // History (for analytics)
  turn_history: TurnSnapshot[];
  
  // Calculated values (cached)
  total_capacity: number;
  networth: number;
}

interface Sector {
  sector_id: string;
  type: 'gas_giant' | 'habitat' | 'trade_station' | 'research_colony';
  is_online: boolean; // False if maintenance unpaid
  maintenance_debt: number; // Accumulated unpaid maintenance
}

interface LaborGroup {
  num_workers: number;
  average_skill: number; // 50-100
  turns_assigned: number; // For skill progression
}

interface TurnSnapshot {
  turn_number: number;
  networth: number;
  credits: number;
  fuel: number;
  population: number;
  approval: number;
  pollution: number;
  actions_taken: Action[];
}

interface Action {
  type: 'buy_sector' | 'sell_sector' | 'terraform' | 'reassign_labor' | 'research';
  details: object;
  timestamp: timestamp;
}
```

### Market State Object

```typescript
interface MarketState {
  run_id: string;
  
  // Sector market
  sectors_remaining: number; // 100 → 0
  
  // Price curves (calculated from sectors_remaining)
  gas_giant_price: number;
  habitat_price: number;
  trade_station_price: number;
  research_colony_price: number;
}
```

### Game Configuration Object

```typescript
interface GameConfig {
  // Constants
  MAX_TURNS: 21;
  TURN_TIMER_SECONDS: 300; // 5 minutes
  
  // Starting resources
  STARTING_CREDITS: 50000;
  STARTING_FUEL: 500;
  STARTING_POPULATION: 500;
  STARTING_RP: 0;
  
  // Starting feedback
  STARTING_APPROVAL: 70;
  STARTING_POLLUTION: 0;
  
  // Sector attributes
  SECTOR_TYPES: {
    gas_giant: SectorAttributes;
    habitat: SectorAttributes;
    trade_station: SectorAttributes;
    research_colony: SectorAttributes;
  };
  
  // Market
  INITIAL_SECTOR_SUPPLY: 100;
  SECTOR_BASE_PRICES: {
    gas_giant: 3000;
    habitat: 4000;
    trade_station: 2500;
    research_colony: 3500;
  };
  SELL_PRICE_MULTIPLIER: 0.8;
  
  // Research
  TERRAFORM_UNLOCK_COST: 500;
  TERRAFORM_BASE_COST: 50000;
  TERRAFORM_MAX_LEVEL: 10;
  
  // Formulas (could be functions or constants)
  POLLUTION_PENALTY_CURVE: number[];
  APPROVAL_GROWTH_FORMULA: string;
}

interface SectorAttributes {
  capacity: number;
  production_per_worker: number;
  maintenance_credits: number;
  maintenance_fuel: number;
  pollution_delta: number;
}
```

---

## Formulas & Calculations Reference

### Production Calculations

**Fuel Production:**
```typescript
function calculateFuelProduction(
  num_extractors: number,
  avg_skill: number,
  num_gas_giants: number,
  pollution_level: number
): number {
  const capacity = num_gas_giants * 50;
  const effective_workers = Math.min(num_extractors, capacity);
  const pollution_penalty = getPollutionPenalty(pollution_level);
  
  return effective_workers * 6 * (avg_skill / 100) * (1 - pollution_penalty);
}
```

**Credit Production:**
```typescript
function calculateCreditProduction(
  num_traders: number,
  avg_skill: number,
  num_trade_stations: number,
  pollution_level: number
): number {
  const capacity = num_trade_stations * 100;
  const effective_workers = Math.min(num_traders, capacity);
  const pollution_penalty = getPollutionPenalty(pollution_level);
  
  return effective_workers * 1200 * (avg_skill / 100) * (1 - pollution_penalty);
}
```

**Research Point Production:**
```typescript
function calculateRPProduction(
  num_researchers: number,
  avg_skill: number,
  num_research_colonies: number
): number {
  const capacity = num_research_colonies * 75;
  const effective_workers = Math.min(num_researchers, capacity);
  
  // NOT affected by pollution
  return effective_workers * 0.25 * (avg_skill / 100);
}
```

### Pollution Penalty

```typescript
function getPollutionPenalty(pollution_level: number): number {
  if (pollution_level <= 20) return 0.0;
  if (pollution_level <= 40) return 0.05;
  if (pollution_level <= 60) return 0.15;
  if (pollution_level <= 80) return 0.30;
  return 0.50;
}
```

### Population Growth

```typescript
function calculatePopulationGrowth(
  current_population: number,
  total_capacity: number,
  approval_rating: number
): number {
  const base_growth_rate = 0.05; // 5% max
  
  // Capacity factor
  const density = current_population / total_capacity;
  let capacity_factor = 1.0;
  if (density >= 0.95) capacity_factor = 0.0;
  else if (density >= 0.90) capacity_factor = 0.5;
  else if (density >= 0.70) capacity_factor = 1.0;
  else capacity_factor = 1.0; // Plenty of room
  
  // Approval factor
  const approval_factor = (approval_rating - 50) / 50;
  
  // Growth delta
  const growth_rate = base_growth_rate * approval_factor * capacity_factor;
  const population_delta = current_population * growth_rate;
  
  return Math.floor(population_delta);
}
```

### Approval Delta

```typescript
function calculateApprovalDelta(
  current_state: PlayerState,
  previous_state: PlayerState
): number {
  let delta = 0;
  
  // Credit surplus/deficit
  if (current_state.credits > previous_state.credits) delta += 1;
  if (current_state.credits < previous_state.credits) delta -= 2;
  
  // Fuel deficit
  if (current_state.fuel < 0) delta -= 2;
  
  // Density
  const density = current_state.population / current_state.total_capacity;
  if (density < 0.70) delta += 1;
  if (density > 0.85 && density < 0.95) delta -= 1;
  if (density >= 0.95) delta -= 3;
  
  // Population change
  if (current_state.population < previous_state.population) delta -= 2;
  
  // Pollution
  if (current_state.pollution_level > 60) delta -= 2;
  
  // Unpaid maintenance
  const unpaid_sectors = current_state.sectors.filter(s => !s.is_online).length;
  delta -= unpaid_sectors * 5;
  
  return delta;
}
```

### Pollution Delta

```typescript
function calculatePollutionDelta(
  sectors: Sector[],
  current_pollution: number
): number {
  let generation = 0;
  let reduction = 0;
  
  // Generation
  const gas_giants = sectors.filter(s => s.type === 'gas_giant' && s.is_online).length;
  const trade_stations = sectors.filter(s => s.type === 'trade_station' && s.is_online).length;
  generation += gas_giants * 0.2;
  generation += trade_stations * 0.05;
  
  // Reduction
  const habitats = sectors.filter(s => s.type === 'habitat' && s.is_online).length;
  const research_colonies = sectors.filter(s => s.type === 'research_colony' && s.is_online).length;
  reduction += habitats * 0.1;
  reduction += research_colonies * 0.2;
  
  // Natural decay
  const natural_decay = current_pollution * 0.01;
  reduction += natural_decay;
  
  return generation - reduction;
}
```

### Skill Progression

```typescript
function updateWorkerSkills(labor_group: LaborGroup): void {
  if (labor_group.average_skill < 100) {
    labor_group.average_skill = Math.min(
      labor_group.average_skill + 5,
      100
    );
  }
  labor_group.turns_assigned += 1;
}

function resetWorkerSkills(labor_group: LaborGroup): void {
  labor_group.average_skill = 50;
  labor_group.turns_assigned = 0;
}
```

### Sector Market Pricing

```typescript
function calculateSectorPrice(
  base_price: number,
  sectors_remaining: number
): number {
  const price_multiplier = 1 + ((100 - sectors_remaining) / 50);
  return Math.floor(base_price * price_multiplier);
}

function calculateSellPrice(current_market_price: number): number {
  return Math.floor(current_market_price * 0.8);
}
```

### Terraform Cost

```typescript
function getTerraformCost(terraform_level: number): number {
  return Math.max(50000 - (terraform_level - 1) * 5000, 5000);
}

function getUpgradeCost(current_level: number): number {
  return (current_level + 1) * 100;
}
```

### Networth Calculation

```typescript
function calculateNetworth(state: PlayerState, market: MarketState): number {
  let networth = 0;
  
  // Credits
  networth += state.credits;
  
  // Fuel
  networth += state.fuel * 10;
  
  // Sectors (valued at current market price)
  for (const sector of state.sectors) {
    const base_price = SECTOR_BASE_PRICES[sector.type];
    const market_price = calculateSectorPrice(base_price, market.sectors_remaining);
    networth += market_price;
  }
  
  // Population
  networth += state.population * 100;
  
  // Research Points
  networth += state.research_points * 50;
  
  return networth;
}
```

### Final Score Calculation

```typescript
function calculateFinalScore(state: PlayerState, history: TurnSnapshot[]): number {
  const base_networth = state.networth;
  
  // Efficiency multiplier
  const productive_turns = history.filter(t => t.networth > (history[t.turn_number - 2]?.networth || 0)).length;
  const efficiency_mult = 1.0 + (productive_turns / 21) * 0.5;
  
  // Sustainability multiplier
  const sustainability_score = 
    (100 - state.pollution_level) * 0.006 + 
    state.approval_rating * 0.006;
  const sustainability_mult = sustainability_score / 100;
  
  // Mastery multiplier
  let mastery_score = 0;
  
  // Terraform unlock timing
  if (state.terraform_unlocked) {
    const unlock_turn = history.find(t => t.actions_taken.some(a => a.type === 'research' && a.details.unlocked)).turn_number;
    mastery_score += (21 - unlock_turn) / 21 * 30;
  }
  
  // Terraform level
  mastery_score += state.terraform_level * 3;
  
  // Average worker skill
  const avg_skill = (
    state.labor_assignments.extractors.average_skill +
    state.labor_assignments.traders.average_skill +
    state.labor_assignments.researchers.average_skill
  ) / 3;
  mastery_score += avg_skill * 0.4;
  
  // Terraform usage
  const terraform_count = history.reduce((sum, t) => 
    sum + t.actions_taken.filter(a => a.type === 'terraform').length, 0);
  mastery_score += Math.min(terraform_count * 5, 20);
  
  const mastery_mult = 1.0 + (mastery_score / 100);
  
  // Final score
  return Math.floor(base_networth * efficiency_mult * sustainability_mult * mastery_mult);
}
```

## UI Requirements

### Platform & Technical Constraints

**Target Devices:**
- iOS 15+ (iPhone 11 and newer)
- Android 11+ (released 2020+)
- Minimum screen size: 5.5" diagonal
- Portrait orientation only (simpler design, one-handed play)

**Performance Targets:**
- Turn submission: <2 seconds
- Screen transitions: <300ms
- Timer updates: 60fps (smooth countdown)
- No loading spinners for phase transitions

**Connectivity:**
- Offline-capable for viewing current state
- Sync required only for turn submission
- Graceful handling of connection loss during turn

---

### Design Principles

**1. Information Density Over Minimalism**
- Show all critical data on screen (no hiding behind menus)
- Use every pixel efficiently (this is a strategy game, not social media)
- Scrolling is acceptable if it reduces taps

**2. No Confirmation Dialogs Except End Turn**
- All actions during turn are provisional (can be undone)
- Only "End Turn" is permanent (requires confirmation)
- Timer expiration = cancels all changes (harsh but clear)

**3. Visual Hierarchy Through Typography & Color**
```
Critical Info: 18-24pt, bold, red/green
Primary Info: 14-16pt, medium, white/gray
Secondary Info: 12-14pt, regular, gray
Tertiary Info: 10-12pt, light, dark gray
```

**4. Consistent Iconography**
```
💰 Credits
⛽ Fuel
👥 Population
🔬 Research Points
🌍 Sectors
⏱️ Timer
⚠️ Warning
✓ Success
✗ Error/Cancel
```

**5. Real-time Preview**
- Show "After this turn" projections constantly
- Update projections as player makes changes
- No surprises at turn end

---

### Color Palette

**Core Colors:**
```css
--bg-primary: #0a0e17 (dark space blue)
--bg-secondary: #151b28 (slightly lighter)
--bg-tertiary: #1f2937 (card backgrounds)

--text-primary: #f9fafb (white)
--text-secondary: #9ca3af (light gray)
--text-tertiary: #6b7280 (medium gray)

--accent-primary: #3b82f6 (blue - neutral actions)
--accent-success: #10b981 (green - positive)
--accent-warning: #f59e0b (orange - caution)
--accent-danger: #ef4444 (red - negative)

--border-default: #374151 (subtle borders)
--border-focus: #3b82f6 (active elements)
```

**Resource Colors:**
```css
--credits: #fbbf24 (gold)
--fuel: #06b6d4 (cyan)
--population: #8b5cf6 (purple)
--research: #ec4899 (pink)
```

**Status Colors:**
```css
--online: #10b981 (green)
--offline: #6b7280 (gray)
--deficit: #ef4444 (red)
--surplus: #10b981 (green)
```

---

### Screen Layouts

### 1. Turn Start Screen (Phase 1: Production Report)

**Layout:**
```
┌─────────────────────────────────────┐
│ ┌─────────────────────────────────┐ │ ← Status bar (fixed)
│ │ TURN 14/21        [3:47] ⏱️     │ │
│ │ Networth: 2.4M (+180k) ↗️       │ │
│ └─────────────────────────────────┘ │
│                                     │
│ ┌─────────────────────────────────┐ │ ← Scrollable content
│ │ PRODUCTION REPORT               │ │
│ │                                 │ │
│ │ 💰 CREDITS: +180,000            │ │
│ │   Trade Stations (150 @ 80%)    │ │
│ │   = 144,000 base × 0.95 (poll)  │ │
│ │                                 │ │
│ │ ⛽ FUEL: +1,200                 │ │
│ │   Gas Giants (200 @ 70%)        │ │
│ │   = 840 base × 0.95 (poll)      │ │
│ │                                 │ │
│ │ 🔬 RESEARCH: +25 RP             │ │
│ │   Research Colonies (100 @ 65%) │ │
│ │                                 │ │
│ │ 👥 POPULATION: +18              │ │
│ │   Approval: 72% (Stable)        │ │
│ │   Density: 85% (High)           │ │
│ │   Total: 1,268 / 1,500          │ │
│ │                                 │ │
│ ├─────────────────────────────────┤ │
│ │ APPROVAL: 72% (→ 71%)           │ │
│ │ ✓ Credit surplus (+1)          │ │
│ │ ⚠️ High density (-1)           │ │
│ │                                 │ │
│ │ POLLUTION: 34% (-5% production) │ │
│ │ Gas Giants: +1.0/turn           │ │
│ │ Research Colonies: -0.2/turn    │ │
│ │ Natural decay: -0.34/turn       │ │
│ │ Net: +0.46/turn → 35%           │ │
│ │                                 │ │
│ ├─────────────────────────────────┤ │
│ │ ⚠️ WARNINGS                     │ │
│ │ • Fuel deficit next turn (-300) │ │
│ │ • Approaching density cap       │ │
│ └─────────────────────────────────┘ │
│                                     │
│ ┌─────────────────────────────────┐ │ ← Fixed bottom button
│ │      [CONTINUE] ▶️             |  │
│ └─────────────────────────────────┘ │
└─────────────────────────────────────┘
```

**Interactions:**
- Tap anywhere on content to scroll
- Tap "Continue" to proceed to Phase 2
- No other actions available
- Timer continues counting (visible in status bar)

**Information Priority:**
1. **Critical**: Timer, Turn counter, Warnings
2. **Primary**: Resource production totals
3. **Secondary**: Source attribution (which workers)
4. **Tertiary**: Formula breakdown (why this number)

---

### 2. Maintenance Payment Screen (Phase 2)

**Layout:**
```
┌─────────────────────────────────────┐
│ ┌─────────────────────────────────┐ │
│ │ TURN 14/21        [3:15] ⏱️     │ │
│ │ Networth: 2.4M                  │ │
│ └─────────────────────────────────┘ │
│                                     │
│ ┌─────────────────────────────────┐ │
│ │ MAINTENANCE PAYMENT             │ │
│ │                                 │ │
│ │ YOUR SECTORS:                   │ │
│ │ Gas Giants (×5)    500 💰       │ │
│ │ Habitats (×6)      1,200 💰     │ │
│ │                    600 ⛽       │ │
│ │ Trade Stations (×4) 400 💰      │ │
│ │ Research Colonies (×1)          │ │
│ │                    200 💰       │ │
│ │                    100 ⛽       │ │
│ │ ─────────────────────────────   │ │
│ │ TOTAL:            2,300 💰      │ │
│ │                    700 ⛽       │ │
│ │                                 │ │
│ │ CURRENT BALANCE:                │ │
│ │ 💰 245,000  ✓                   │ │
│ │ ⛽ 1,800    ✓                   │ │
│ │                                 │ │
│ │ AFTER PAYMENT:                  │ │
│ │ 💰 242,700  ✓                   │ │
│ │ ⛽ 1,100    ✓                   │ │
│ │                                 │ │
│ └─────────────────────────────────┘ │
│                                     │
│ ┌─────────────────────────────────┐ │
│ │      [AUTO-PAY ALL] ✓           │ │
│ └─────────────────────────────────┘ │
└─────────────────────────────────────┘
```

**If Insufficient Resources:**
```
┌─────────────────────────────────────┐
│ ┌─────────────────────────────────┐ │
│ │ TURN 14/21        [3:15] ⏱️     │ │
│ │ Networth: 2.4M                  │ │
│ └─────────────────────────────────┘ │
│                                     │
│ ┌─────────────────────────────────┐ │
│ │ ⚠️ INSUFFICIENT RESOURCES       │ │
│ │                                 │ │
│ │ Cannot afford full maintenance  │ │
│ │ You must select sectors to pay  │ │
│ │                                 │ │
│ │ Unpaid sectors will:            │ │
│ │ • Go offline (no production)    │ │
│ │ • Reduce Approval (-1 each)     │ │
│ │ • Accumulate debt               │ │
│ │                                 │ │
│ │ SELECT SECTORS TO PAY:          │ │
│ │                                 │ │
│ │ ☑ Gas Giants (×5)  500 💰       │ │ ← Tap to toggle
│ │ ☑ Habitats (×6)    1,200 💰     │ │
│ │                    600 ⛽        │ │
│ │ ☐ Trade Stations (×4) 400 💰    │ │ ← Unchecked (skip)
│ │ ☑ Research (×1)    200 💰       │ │
│ │                    100 ⛽        │ │
│ │                                 │ │
│ │ SELECTED COST:    1,900 💰      │ │
│ │                    700 ⛽        │ │
│ │ BALANCE AFTER:     300 💰  ✓    │ │
│ │                    0 ⛽     ✓    │ │
│ │                                 │ │
│ │ UNPAID: 4 Trade Stations        │ │
│ │ Effect: -4 Approval, offline    │ │
│ └─────────────────────────────────┘ │
│                                     │
│ ┌─────────────────────────────────┐ │
│ │  [CONFIRM PARTIAL PAYMENT] ⚠️   │ │
│ └─────────────────────────────────┘ │
└─────────────────────────────────────┘
```

**Interactions:**
- Auto-pay if sufficient (tap to proceed)
- If insufficient: checkboxes for each sector type
- Live update of "Balance After"
- Red highlight if still can't afford selected
- Must manually confirm partial payment

---

### 3. Sector Market Screen (Phase 3)

**Layout (Two Tabs):**

**Tab 1: Buy/Sell**
```
┌─────────────────────────────────────┐
│ ┌─────────────────────────────────┐ │
│ │ TURN 14/21        [2:48] ⏱️      │ │
│ │ 💰 242,700  ⛽ 1,100  🔬 350 RP  │ │
│ └─────────────────────────────────┘ │
│                                     │
│ [BUY/SELL] [TERRAFORM] ← Tabs      │
│                                     │
│ ┌─────────────────────────────────┐ │
│ │ SECTOR MARKET                   │ │
│ │ Available: 73/100               │ │
│ │                                 │ │
│ │ ┌─────────────────────────────┐ │ │
│ │ │ 🌫️ GAS GIANT    4,200 💰    │ │ │
│ │ │ Produces: 6 ⛽/worker        │ │ │
│ │ │ Capacity: 50 👥             │ │ │
│ │ │ Maint: 100 💰/turn          │ │ │
│ │ │                             │ │ │
│ │ │ You own: 5                  │ │ │
│ │ │ [BUY +1] [SELL -1 @ 3,360]  │ │ │
│ │ └─────────────────────────────┘ │ │
│ │                                 │ │
│ │ ┌─────────────────────────────┐ │ │
│ │ │ 🏠 HABITAT      5,800 💰    │ │ │
│ │ │ Capacity: 200 👥            │ │ │
│ │ │ Maint: 200 💰 + 100 ⛽/turn │ │ │
│ │ │ Reduces pollution: -0.1     │ │ │
│ │ │                             │ │ │
│ │ │ You own: 6                  │ │ │
│ │ │ [BUY +1] [SELL -1 @ 4,640]  │ │ │
│ │ └─────────────────────────────┘ │ │
│ │                                 │ │
│ │ ┌─────────────────────────────┐ │ │ ← Scrollable
│ │ │ 💼 TRADE STATION 3,500 💰   │ │ │
│ │ │ Produces: 1,200 💰/worker   │ │ │
│ │ │ Capacity: 100 👥            │ │ │
│ │ │ Maint: 100 💰/turn          │ │ │
│ │ │                             │ │ │
│ │ │ You own: 4                  │ │ │
│ │ │ [BUY +1] [SELL -1 @ 2,800]  │ │ │
│ │ └─────────────────────────────┘ │ │
│ │                                 │ │
│ │ ┌─────────────────────────────┐ │ │
│ │ │ 🔬 RESEARCH COLONY 4,000 💰 │ │ │
│ │ │ Produces: 0.25 🔬/worker    │ │ │
│ │ │ Capacity: 75 👥             │ │ │
│ │ │ Maint: 200 💰 + 100 ⛽/turn │ │ │
│ │ │ Reduces pollution: -0.2     │ │ │
│ │ │                             │ │ │
│ │ │ You own: 1                  │ │ │
│ │ │ [BUY +1] [SELL -1 @ 3,200]  │ │ │
│ │ └─────────────────────────────┘ │ │
│ └─────────────────────────────────┘ │
│                                     │
│ ┌─────────────────────────────────┐ │
│ │ Changes: +1 Habitat, -1 Gas     │ │
│ │ Cost: +1,600 💰 this turn       │ │
│ │      [CONTINUE] ▶️               │ │
│ └─────────────────────────────────┘ │
└─────────────────────────────────────┘
```

**Tab 2: Terraform (if unlocked)**
```
┌─────────────────────────────────────┐
│ ┌─────────────────────────────────┐ │
│ │ TURN 14/21        [2:20] ⏱️      │ │
│ │ 💰 242,700  ⛽ 1,100  🔬 350 RP  │ │
│ └─────────────────────────────────┘ │
│                                     │
│ [BUY/SELL] [TERRAFORM] ← Tabs      │
│                                     │
│ ┌─────────────────────────────────┐ │
│ │ TERRAFORM YOUR SECTORS          │ │
│ │                                 │ │
│ │ Tech Level: 3                   │ │
│ │ Cost: 40,000 💰 per terraform   │ │
│ │                                 │ │
│ │ SELECT SECTOR:                  │ │
│ │                                 │ │
│ │ ┌─────────────────────────────┐ │ │
│ │ │ ○ Gas Giant #1              │ │ │
│ │ │   150 Extractors assigned   │ │ │
│ │ └─────────────────────────────┘ │ │
│ │ ┌─────────────────────────────┐ │ │
│ │ │ ○ Gas Giant #2              │ │ │
│ │ │   100 Extractors assigned   │ │ │
│ │ └─────────────────────────────┘ │ │
│ │ ┌─────────────────────────────┐ │ │ ← Scrollable
│ │ │ ● Gas Giant #3  ⭐ EMPTY    │ │ │ ← Selected
│ │ │   Recommended (no workers)  │ │ │
│ │ └─────────────────────────────┘ │ │
│ │ ┌─────────────────────────────┐ │ │
│ │ │ ○ Gas Giant #4              │ │ │
│ │ │   200 Extractors assigned   │ │ │
│ │ └─────────────────────────────┘ │ │
│ │                                 │ │
│ │ TRANSFORM TO:                   │ │
│ │ ○ Gas Giant (no change)         │ │
│ │ ● Trade Station ← Selected      │ │
│ │ ○ Habitat                       │ │
│ │ ○ Research Colony               │ │
│ │                                 │ │
│ │ ⚠️ Workers remain assigned but   │ │
│ │   produce nothing until         │ │
│ │   reassigned in Labor phase     │ │
│ │                                 │ │
│ │ Cost: 40,000 💰                 │ │
│ │ [TERRAFORM] 🌍                  │ │
│ └─────────────────────────────────┘ │
│                                     │
│ ┌─────────────────────────────────┐ │
│ │ Changes: 1 terraform queued     │ │
│ │      [CONTINUE] ▶️              │ │
│ └─────────────────────────────────┘ │
└─────────────────────────────────────┘
```

**Interactions:**
- Tap [BUY +1] to purchase (instant, deducts credits)
- Tap [SELL -1] to sell (instant, adds credits)
- Multiple purchases/sales allowed
- Bottom summary shows net changes
- Can undo by buying/selling back
- Terraform: radio buttons for selection
- Cannot terraform offline sectors (grayed out)
- "EMPTY" sectors highlighted (recommended targets)

**If Terraform Not Unlocked:**
```
[TERRAFORM] tab shows:
┌─────────────────────────────────┐
│ 🔒 TERRAFORM LOCKED             │
│                                 │
│ Unlock at 500 🔬 RP             │
│ Current: 350 RP (need 150 more) │
│                                 │
│ Assign more Researchers in      │
│ Labor phase to unlock faster    │
└─────────────────────────────────┘
```

---

### 4. Labor Assignment Screen (Phase 4)

**Layout:**
```
┌─────────────────────────────────────┐
│ ┌─────────────────────────────────┐ │
│ │ TURN 14/21        [1:45] ⏱️     │ │
│ │ 👥 1,268 / 1,500 (85% density) │ │
│ └─────────────────────────────────┘ │
│                                     │
│ ┌─────────────────────────────────┐ │
│ │ POPULATION ASSIGNMENT           │ │
│ │                                 │ │
│ │ ⚠️ High density (85%) reduces   │ │
│ │    Approval. Consider buying    │ │
│ │    Habitats or leave unassigned │ │
│ │                                 │ │
│ │ ┌─────────────────────────────┐ │ │
│ │ │ ⛽ EXTRACTORS    [   200]   │ │ │
│ │ │ ⚠️ 90% skill (10 turns)     │ │ │
│ │ │                             │ │ │
│ │ │ Produces: 1,200 ⛽/turn     │ │ │
│ │ │ Capacity: 250 (5 Gas Giants)│ │ │
│ │ │                             │ │ │
│ │ │ [-50] [-10] [+10] [+50]     │ │ │
│ │ │                             │ │ │
│ │ │ ⚠️ Reassigning resets to    │ │ │
│ │ │    50% skill!               │ │ │
│ │ └─────────────────────────────┘ │ │
│ │                                 │ │
│ │ ┌─────────────────────────────┐ │ │
│ │ │ 💰 TRADERS      [   150]    │ │ │
│ │ │ ⚠️ 80% skill (7 turns)      │ │ │
│ │ │                             │ │ │
│ │ │ Produces: 180,000 💰/turn   │ │ │
│ │ │ Capacity: 400 (4 TS)        │ │ │
│ │ │                             │ │ │
│ │ │ [-50] [-10] [+10] [+50]     │ │ │
│ │ │                             │ │ │
│ │ │ ⚠️ Reassigning resets to    │ │ │
│ │ │    50% skill!               │ │ │
│ │ └─────────────────────────────┘ │ │
│ │                                 │ │
│ │ ┌─────────────────────────────┐ │ │
│ │ │ 🔬 RESEARCHERS  [   100]    │ │ │
│ │ │ ⚠️ 65% skill (4 turns)      │ │ │
│ │ │                             │ │ │
│ │ │ Produces: 25 🔬 RP/turn     │ │ │
│ │ │ ⚠️ Excess: 25 (Cap: 75)     │ │ │
│ │ │                             │ │ │
│ │ │ [-50] [-10] [+10] [+50]     │ │ │
│ │ │                             │ │ │
│ │ │ ⚠️ 25 workers over capacity │ │ │
│ │ │    (producing nothing!)     │ │ │
│ │ └─────────────────────────────┘ │ │
│ │                                 │ │
│ │ ┌─────────────────────────────┐ │ │
│ │ │ UNASSIGNED     [   818]     │ │ │
│ │ │ Not producing anything      │ │ │
│ │ └─────────────────────────────┘ │ │
│ └─────────────────────────────────┘ │
│                                     │
│ ┌─────────────────────────────────┐ │
│ │ [OPTIMIZE AUTO] 🤖              │ │
│ │      [CONTINUE] ▶️              │ │
│ └─────────────────────────────────┘ │
└─────────────────────────────────────┘
```

**Interactions:**
- Tap [-50][-10][+10][+50] to adjust quickly
- Live update of "Produces" based on new assignment
- Red warning if over capacity
- Orange warning if reassigning skilled workers
- Auto-Optimize button suggests balanced allocation
- All changes provisional until End Turn

**If Over Capacity:**
```
⚠️ 25 workers over capacity (producing nothing!)
Suggestion: Reassign to other roles or buy more Research Colonies
```

**If About to Reset High-Skill Workers:**
```
⚠️ Removing 50 Traders @ 90% skill
   If reassigned elsewhere, they reset to 50% skill
   Loss: 48,000 💰/turn production
Are you sure?
```

**Auto-Optimize Behavior:**
- Removes excess workers from over-capacity sectors
- Suggests balanced allocation based on sector ownership
- Shows before/after comparison
- Requires confirmation (doesn't auto-apply)

---

### 5. Research Screen (Phase 5)

**Before Unlock:**
```
┌─────────────────────────────────────┐
│ ┌─────────────────────────────────┐ │
│ │ TURN 14/21        [1:20] ⏱️      │ │
│ │ 🔬 350 RP                        │ │
│ └─────────────────────────────────┘ │
│                                     │
│ ┌─────────────────────────────────┐ │
│ │ RESEARCH                        │ │
│ │                                 │ │
│ │ 🔬 350 / 500 RP                 │ │
│ │ ████████████████░░░░ (70%)      │ │
│ │                                 │ │
│ │ ┌─────────────────────────────┐ │ │
│ │ │ 🌍 TERRAFORM SECTOR         │ │ │
│ │ │ Status: 🔒 LOCKED           │ │ │
│ │ │                             │ │ │
│ │ │ Unlock at: 500 RP           │ │ │
│ │ │ (need 150 more)             │ │ │
│ │ │                             │ │ │
│ │ │ Effect when unlocked:       │ │ │
│ │ │ • Change sector types       │ │ │
│ │ │ • Initial: 50,000 💰 each   │ │ │
│ │ │ • Upgradeable (10 levels)   │ │ │
│ │ │                             │ │ │
│ │ │ Why unlock this?            │ │ │
│ │ │ → Fix early mistakes        │ │ │
│ │ │ → Adapt to changing needs   │ │ │
│ │ │ → Optimize late game        │ │ │
│ │ │ → Manage pollution          │ │ │
│ │ └─────────────────────────────┘ │ │
│ │                                 │ │
│ │ Researchers producing:          │ │
│ │ 25 RP/turn                      │ │
│ │                                 │ │
│ │ Estimated unlock: Turn 18       │ │
│ │                                 │ │
│ │ 💡 Assign more Researchers in   │ │
│ │    Labor phase to unlock faster │ │
│ └─────────────────────────────────┘ │
│                                     │
│ ┌─────────────────────────────────┐ │
│ │      [CONTINUE] ▶️              │ │
│ └─────────────────────────────────┘ │
└─────────────────────────────────────┘
```

**After Unlock, Before Max Level:**
```
┌─────────────────────────────────────┐
│ ┌─────────────────────────────────┐ │
│ │ TURN 18/21        [0:55] ⏱️     │ │
│ │ 🔬 650 RP available             │ │
│ └─────────────────────────────────┘ │
│                                     │
│ ┌─────────────────────────────────┐ │
│ │ RESEARCH                        │ │
│ │                                 │ │
│ │ ┌─────────────────────────────┐ │ │
│ │ │ 🌍 TERRAFORM SECTOR         │ │ │
│ │ │ Status: ✅ UNLOCKED         │ │ │
│ │ │ Current Level: 3            │ │ │
│ │ │ Cost: 40,000 💰/terraform   │ │ │
│ │ └─────────────────────────────┘ │ │
│ │                                 │ │
│ │ ┌─────────────────────────────┐ │ │
│ │ │ ⬆️ UPGRADE TO LEVEL 4       │ │ │
│ │ │                             │ │ │
│ │ │ Cost: 400 RP                │ │ │
│ │ │ 🔬 650 RP (✓ can afford)    │ │ │
│ │ │                             │ │ │
│ │ │ New Terraform Cost:         │ │ │
│ │ │ 40,000 💰 → 35,000 💰       │ │ │
│ │ │ Savings: -5,000 per use     │ │ │
│ │ │                             │ │ │
│ │ │ Break-even: 2 terraforms    │ │ │
│ │ │ (10,000 💰 saved = 400 RP)  │ │ │
│ │ │                             │ │ │
│ │ │ [UPGRADE NOW] ⬆️            │ │ │
│ │ └─────────────────────────────┘ │ │
│ │                                 │ │
│ │ UPGRADE PATH:                   │ │
│ │ Level 3 → 4: 400 RP (35k 💰)   │ │
│ │ Level 4 → 5: 500 RP (30k 💰)   │ │
│ │ Level 5 → 6: 600 RP (25k 💰)   │ │
│ │ ...                             │ │
│ │ Level 10 (MAX): 5,000 💰        │ │
│ │                                 │ │
│ │ Total to MAX: 4,500 more RP    │ │
│ │ Estimated: 12 more turns       │ │
│ │ (won't reach MAX this run)     │ │
│ └─────────────────────────────────┘ │
│                                     │
│ ┌─────────────────────────────────┐ │
│ │      [CONTINUE] ▶️              │ │
│ └─────────────────────────────────┘ │
└─────────────────────────────────────┘
```

**At Max Level:**
```
┌─────────────────────────────────┐
│ 🌍 TERRAFORM SECTOR             │
│ Status: ✅ MAX LEVEL (10)       │
│ Cost: 5,000 💰/terraform        │
│                                 │
│ You have mastered terraform     │
│ technology. No further upgrades │
│ available.                      │
│                                 │
│ Unused RP: 2,450                │
│ (no other use in this version)  │
└─────────────────────────────────┘
```

**Interactions:**
- Tap [UPGRADE NOW] to spend RP
- Instant upgrade (no confirmation needed)
- Can upgrade multiple levels in one turn if enough RP
- Show break-even analysis (helps decision)
- Show time-to-MAX estimate

---

### 6. End Turn Summary Screen (Phase 6)

**Layout:**
```
┌─────────────────────────────────────┐
│ ┌─────────────────────────────────┐ │
│ │ TURN 14/21        [0:30] ⏱️      │ │
│ │ Networth: 2.4M                  │ │
│ └─────────────────────────────────┘ │
│                                     │
│ ┌─────────────────────────────────┐ │
│ │ END TURN SUMMARY                │ │
│ │                                 │ │
│ │ CHANGES THIS TURN:              │ │
│ │ ✓ Bought 2 Trade Stations       │ │
│ │   Cost: -7,000 💰               │ │
│ │ ✓ Terraformed Gas Giant #3      │ │
│ │   → Habitat                     │ │
│ │   Cost: -40,000 💰              │ │
│ │ ✓ Reassigned 100 Extractors     │ │
│ │   → Traders (skill 90% → 50%)   │ │
│ │                                 │ │
│ │ ─────────────────────────────   │ │
│ │                                 │ │
│ │ PROJECTED NEXT TURN:            │ │
│ │                                 │ │
│ │ 💰 CREDITS: +165,000            │ │
│ │   (includes new Traders @ 50%)  │ │
│ │   ⚠️ Down from 180k (skill loss) │ │
│ │                                 │ │
│ │ ⛽ FUEL: +600                    │ │
│ │   ⚠️ Down from 1,200 (reassign)  │ │
│ │                                 │ │
│ │ 🔬 RESEARCH: +30 RP              │ │
│ │   (no change)                   │ │
│ │                                 │ │
│ │ 👥 POPULATION: +12               │ │
│ │   (slowed by high density)      │ │
│ │                                 │ │
│ │ ─────────────────────────────   │ │
│ │                                 │ │
│ │ MAINTENANCE:                    │ │
│ │ -2,700 💰 (2 new sectors)       │ │
│ │ -800 ⛽                          │ │
│ │                                 │ │
│ │ NET RESULT:                     │ │
│ │ +162,300 💰 ✓                   │ │
│ │ -200 ⛽ ⚠️ (will deficit)        │ │
│ │                                 │ │
│ │ ─────────────────────────────   │ │
│ │                                 │ │
│ │ FEEDBACK:                       │ │
│ │ Approval: 72% → 70% (-2)        │ │
│ │   ⚠️ High density penalty        │ │
│ │   ⚠️ Fuel deficit penalty        │ │
│ │                                 │ │
│ │ Pollution: 34% → 33% (-1)       │ │
│ │   ✓ New Habitat reduces poll.   │ │
│ │                                 │ │
│ │ ─────────────────────────────   │ │
│ │                                 │ │
│ │ ⚠️ WARNINGS:                     │ │
│ │ • Fuel deficit (-200)           │ │
│ │ • 100 Traders lost 40% skill    │ │
│ │ • Approval declining            │ │
│ │                                 │ │
│ │ 💡 SUGGESTIONS:                  │ │
│ │ • Buy Gas Giants for Fuel       │ │
│ │ • Buy Habitats to reduce density│ │
│ └─────────────────────────────────┘ │
│                                     │
│ ┌─────────────────────────────────┐ │
│ │ [BACK - MAKE CHANGES] ←         │ │
│ │ [CONFIRM END TURN] ✓            │ │
│ └─────────────────────────────────┘ │
└─────────────────────────────────────┘
```

**Confirmation Dialog (when tapping CONFIRM):**
```
┌─────────────────────────────────┐
│ ⚠️ CONFIRM END TURN              │
│                                 │
│ You are ending Turn 14 with:    │
│ • 2 sectors purchased           │
│ • 1 terraform applied           │
│ • 100 workers reassigned        │
│                                 │
│ Once confirmed, you cannot      │
│ undo these actions.             │
│                                 │
│ Proceed?                        │
│                                 │
│ [CANCEL] ✗    [CONFIRM] ✓       │
└─────────────────────────────────┘
```

**Interactions:**
- Shows all changes made this turn
- Projects next turn's production
- Highlights negative consequences (red)
- Highlights positive outcomes (green)
- [BACK] returns to Phase 3 (Sector Market) - can continue modifying
- [CONFIRM] requires explicit confirmation dialog
- Timer reaching 0:00 = auto-cancel (harsh penalty)

---

### 7. Turn Resolution

When the turn ends (either via explicit CANCEL/CONFIRM or via timer), the "End Turn Summary" screen should be updated to reflect this fact and then offer a "Start Next Turn" button to allow the user to explicitly start a new 5-minute timer.

At the end of turn 21 the button says "Game Summary" and take you to the Final Score Screen instead.

---

### 8. Final Score Screen (After Turn 21)

**Layout:**
```
┌─────────────────────────────────────┐
│ ┌─────────────────────────────────┐ │
│ │ ╔═══════════════════════════════╗│
│ │ ║ YOU HAD 21 TURNS TO BUILD AN  ║│
│ │ ║          EMPIRE.              ║│
│ │ ╚═══════════════════════════════╝│
│ └─────────────────────────────────┘ │
│                                     │
│ ┌─────────────────────────────────┐ │
│ │ YOUR FINAL SCORE                │ │
│ │                                 │ │
│ │ ┌─────────────────────────────┐ │ │
│ │ │ 7,863,500 (#12)             │ │ │
│ │ │ ⭐⭐⭐                      │ │ │
│ │ └─────────────────────────────┘ │ │
│ │                                 │ │
│ │ BASE NETWORTH:                  │ │
│ │ 3,500,000                       │ │
│ │ ├ Credits: 850,000              │ │
│ │ ├ Fuel: 25,000 (2,500 × 10)    │ │
│ │ ├ Sectors: 110,000              │ │
│ │ ├ Population: 220,000 (2,200)  │ │
│ │ └ Research: 22,500 (450 RP)    │ │
│ │                                 │ │
│ │ MULTIPLIERS:                    │ │
│ │ × 1.40  Efficiency              │ │
│ │   (18/21 productive turns)      │ │
│ │ × 0.95  Sustainability          │ │
│ │   (28% pollution, 75% approval) │ │
│ │ × 1.70  Mastery                 │ │
│ │   (Terraform Turn 12, Level 5)  │ │
│ │                                 │ │
│ │ ACHIEVEMENTS:                   │ │
│ │ ✓ Empire Builder (2M+ Networth)│ │
│ │ ✓ Tech Pioneer (Terraform <15) │ │
│ │ ✓ Stable Ruler (Approval >70%) │ │
│ └─────────────────────────────────┘ │
│                                     │
│ ┌─────────────────────────────────┐ │
│ │ LEADERBOARD POSITION: #12/247   │ │
│ │                                 │ │
│ │ #1  TopPlayer     8,525,160     │ │
│ │ #2  SecondPlace   8,102,450     │ │
│ │ ...                             │ │
│ │ #11 JustAhead     7,901,200     │ │
│ │ #12 YOU           7,863,500 ⬅️  │ │
│ │ #13 BelowYou      7,650,300     │ │
│ │                                 │ │
│ │ You're in the top 5%!           │ │
│ └─────────────────────────────────┘ │
│                                     │
│ ┌─────────────────────────────────┐ │
│ │ [VIEW FULL STATS] 📊             │ │
│ │ [COMPARE TO #1] 👑               │ │
│ │ [PLAY AGAIN] 🔄                  │ │
│ └─────────────────────────────────┘ │
└─────────────────────────────────────┘
```

**Expanded Stats View:**
```
┌─────────────────────────────────────┐
│ DETAILED STATISTICS                 │
│                                     │
│ RESEARCH:                           │
│ Terraform Unlocked: Turn 12         │
│ Final Terraform Level: 5            │
│ Terraforms Used: 6                  │
│ ├ Gas Giant → Trade Station: 3      │
│ ├ Trade Station → Habitat: 2        │
│ └ Habitat → Research Colony: 1      │
│                                     │
│ SECTORS:                            │
│ Final Count: 16                     │
│ ├ Gas Giants: 3                     │
│ ├ Habitats: 7                       │
│ ├ Trade Stations: 5                 │
│ └ Research Colonies: 1              │
│ Peak Sectors: 18 (Turn 19)          │
│                                     │
│ LABOR:                              │
│ Final Population: 2,200             │
│ Avg Worker Skill: 87%               │
│ Labor Reassignments: 4              │
│ (fewer = better skill retention)    │
│                                     │
│ FEEDBACK:                           │
│ Final Approval: 75%                 │
│ Final Pollution: 28%                │
│ Lowest Approval: 58% (Turn 16)      │
│ Highest Pollution: 45% (Turn 14)    │
│                                     │
│ STRATEGY ANALYSIS:                  │
│ You executed a "Balanced Strategy"  │
│ - Unlocked terraform mid-game       │
│ - Maintained stable approval        │
│ - Managed pollution effectively     │
│ - Solid worker skill retention      │
│                                     │
│ TO IMPROVE:                         │
│ • Unlock terraform earlier (Turn 9) │
│ • Upgrade terraform more (Level 7+) │
│ • Reduce labor reassignments (3 max)│
│                                     │
│ [BACK] ←                            │
└─────────────────────────────────────┘
```

**Compare to #1 View:**
```
┌─────────────────────────────────────┐
│ STRATEGY COMPARISON                 │
│                                     │
│              YOU      #1 TopPlayer  │
│ Score:       7.86M    8.53M (+8%)   │
│ Networth:    3.50M    2.80M (-20%)  │
│ Efficiency:  1.40×    1.45×         │
│ Sustain:     0.95×    1.08×         │
│ Mastery:     1.70×    1.95× ⬅️ Key  │
│                                     │
│ KEY DIFFERENCES:                    │
│                                     │
│ Terraform Unlock:                   │
│ You: Turn 12 | Them: Turn 9 ⭐      │
│ (3 turns earlier = more usage)      │
│                                     │
│ Terraform Level:                    │
│ You: Level 5 | Them: Level 8 ⭐     │
│ (cheaper terraforms = more uses)    │
│                                     │
│ Terraforms Used:                    │
│ You: 6 times | Them: 8 times ⭐     │
│ (more optimization passes)          │
│                                     │
│ Worker Skill:                       │
│ You: 87% avg | Them: 85% avg       │
│ (you were better here!)             │
│                                     │
│ LESSON:                             │
│ TopPlayer prioritized terraform     │
│ early and heavily, sacrificing      │
│ some economic growth for long-term  │
│ flexibility. This paid off in the   │
│ Mastery multiplier (1.95× vs 1.70×)│
│                                     │
│ Try a "Terraform Rush" strategy     │
│ next run?                           │
│                                     │
│ [TRY THEIR STRATEGY] 🎯             │
│ [BACK] ←                            │
└─────────────────────────────────────┘
```

**Interactions:**
- Auto-displays after Turn 21 completes
- Shows full breakdown of score calculation
- Highlights leaderboard position
- [VIEW FULL STATS] - detailed turn-by-turn analysis
- [COMPARE TO #1] - learn from top player
- [PLAY AGAIN] - start new run (resets to Turn 1)

---

### 9. Persistent UI Elements

**Status Bar (Always Visible):**
```
┌─────────────────────────────────────┐
│ TURN 14/21        [2:45] ⏱️          │ ← Left: Turn counter, Timer
│ 💰 242,700  ⛽ 1,100  🔬 350 RP     │ ← Right: Resources
│ Networth: 2.4M (+180k) ↗️            │ ← Bottom: Networth + delta
└─────────────────────────────────────┘
```

**Always shows:**
- Current turn / max turns
- Time remaining (5:00 countdown, red when <1:00)
- Current resources (Credits, Fuel, RP)
- Networth (updates live as changes made)
- Networth delta (↗️ up, ↘️ down from last turn)

**Color Coding:**
- Green: Positive values, increases
- Red: Deficits, warnings, timer <1:00
- Orange: Cautions, moderate issues
- White/Gray: Neutral info

---

### 10. Loading & Error States

**Initial App Load:**
```
┌─────────────────────────────────────┐
│                                     │
│         BATTLE REALMS               │
│                                     │
│   You have 21 turns to build an    │
│            empire.                  │
│                                     │
│         [Loading...] ⚙️              │
│                                     │
└─────────────────────────────────────┘
```

**Connection Lost (During Turn):**
```
┌─────────────────────────────────────┐
│ ⚠️ CONNECTION LOST                   │
│                                     │
│ Your changes are saved locally.     │
│ They will sync when connection      │
│ is restored.                        │
│                                     │
│ You can continue making changes,    │
│ but cannot end turn until online.   │
│                                     │
│ [RETRY CONNECTION] 🔄               │
│ [CONTINUE OFFLINE] ✓                │
└─────────────────────────────────────┘
```

**Sync Conflict (Someone else modified data):**
```
┌─────────────────────────────────────┐
│ ⚠️ SYNC CONFLICT                     │
│                                     │
│ Your game state is out of sync.     │
│ This shouldn't happen in solo mode. │
│                                     │
│ Please contact support or restart.  │
│                                     │
│ [RESTART RUN] 🔄                    │
│ [CONTACT SUPPORT] 📧                │
└─────────────────────────────────────┘
```

**Timer Expired:**
```
┌─────────────────────────────────────┐
│ ⏱️ TIME EXPIRED                      │
│                                     │
│ Your 5-minute turn timer ran out.   │
│ All changes this turn were          │
│ cancelled.                          │
│                                     │
│ No actions were applied.            │
│ Turn 14 completed with no changes.  │
│                                     │
│ [CONTINUE TO TURN 15] ▶️            │
└─────────────────────────────────────┘
```

---

### 11. Onboarding Flow (First-Time User)

**Screen 1: Welcome**
```
┌─────────────────────────────────────┐
│                                     │
│   ╔═══════════════════════════════╗ │
│   ║ YOU HAVE 21 TURNS TO BUILD AN ║ │
│   ║          EMPIRE.              ║ │
│   ╚═══════════════════════════════╝ │
│                                     │
│   Every turn lasts 5 minutes.       │
│   Every decision has consequences.  │
│   There are no do-overs.            │
│                                     │
│   Your goal: Maximize Networth by   │
│              Turn 21                │
│                                     │
│   Your competition: Everyone else   │
│                    trying the same  │
│                                     │
│   Your advantage: Strategy          │
│                                     │
│   [BEGIN TURN 1] ▶️                  │
│   [HOW TO PLAY] ?                   │
│                                     │
└─────────────────────────────────────┘
```

**No tutorial:** Just throw them in Turn 1.

**[HOW TO PLAY] opens:**
```
┌─────────────────────────────────────┐
│ HOW TO PLAY                         │
│                                     │
│ Each turn has 6 phases:             │
│                                     │
│ 1. Production Report                │
│    See what you produced last turn  │
│                                     │
│ 2. Maintenance Payment              │
│    Pay to keep sectors online       │
│                                     │
│ 3. Sector Market                    │
│    Buy/sell/terraform sectors       │
│                                     │
│ 4. Labor Assignment                 │
│    Assign population to work        │
│                                     │
│ 5. Research                         │
│    Unlock terraform technology      │
│                                     │
│ 6. End Turn Summary                 │
│    Review and confirm changes       │
│                                     │
│ You have 5 minutes per turn.        │
│ If time expires, changes cancel.    │
│                                     │
│ Learn by playing. Good luck!        │
│                                     │
│ [START GAME] ▶️                      │
└─────────────────────────────────────┘
```

**That's it. No hand-holding.**

Players discover:
- What sectors do (by reading descriptions)
- What terraform does (by unlocking it)
- What strategies work (by trying and failing)

**Philosophy:** The game teaches through play, not tutorials.
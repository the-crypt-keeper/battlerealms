# Battle Realms: A Modern Homage to Door Game Strategy

## Vision Statement

**What We're Building:**
A mobile strategy game that recaptures the unique gameplay experience of 1990s BBS door games—specifically Barren Realms Elite and Falcon's Eye—where scarcity, asynchronous coordination, and genuine social dynamics created emergent complexity that modern always-connected games have lost.

**Why We're Building It:**
Modern mobile gaming has devolved into dopamine-extraction systems: energy timers, loot boxes, battle passes, and artificial scarcity designed to create anxiety rather than anticipation. Players are trapped in Skinner boxes that generate revenue through FOMO rather than through actual satisfying gameplay.

The BBS door game era accidentally created something special: games where **constraints were mechanics**, where **scarcity was real**, and where **coordination was genuinely difficult**. These games taught practical life skills—resource optimization, coalition building, reading people, project management under adversarial conditions—because the stakes were real (social reputation, time investment) even if no money changed hands.

**What We Want Players to Learn:**
- Strategic resource management under genuine constraints
- Building and maintaining trust networks across extended time periods
- Coordinating diverse groups toward shared goals despite communication limitations
- Long-term planning over instant gratification
- The satisfaction of earning victories through actual problem-solving

---

## Core Design Philosophy

### The Three Pillars

**1. Scarcity Creates Value**
- Limited turns per day (5, 7, or 10 - randomly chosen per World)
- 5-minute timer per turn (real constraint, not artificial)
- Fixed World slots (26 players max)
- Zero-sum resource competition in later phases

**Contrast with modern mobile:** Energy systems that refill constantly, microtransactions that bypass waits. Our scarcity is the game, not a paywall.

**2. Asynchronous Coordination is the Challenge**
- No real-time chat
- Forum-only communication (global + alliance)
- Must coordinate across time zones
- Trust is mechanically enforced through reputation systems

**Contrast with modern gaming:** Voice chat, instant messaging, synchronized raids. We force players to develop the harder skill of asynchronous organization.

**3. Reputation Persists**
- Player names carry across Worlds
- Public "Betrayal Counter" tracks broken trust
- Actions remembered for weeks
- Social consequences for anti-social behavior

**Contrast with modern gaming:** Anonymous matchmaking, ephemeral lobbies, no consequences for toxicity. We make reputation matter.

---

## Inspirations & What We're Stealing

### From Barren Realms Elite (BRE)

**Core Mechanics:**
- Multi-phase progression (Growth → Consolidation → World War)
- Inter-BBS (now inter-World) warfare
- The "Nuke" requiring massive coordination (80% of players)
- Bank interest exploitation through sequential money-passing
- Turn-based daily play with strict limits

**The Coordination Challenge:**
BRE's genius was the Gooie Kablooie—a weapon requiring days of coordination across 80%+ of your BBS to build and launch, while opponents prepared their own. Failed attempts meant vulnerability. Success required:
- Project management across dozens of people
- Dealing with different schedules and time zones
- Managing egos and disputes
- Maintaining momentum over days
- Handling potential saboteurs

This wasn't just a game mechanic—it was a **lesson in collective action problems**.

**What Made BRE Special:**
The advanced strategy guide showed sophisticated emergent gameplay:
- Team hierarchies (Leader, War Coordinator, Diplomat, 2nd Man, Insider)
- Food market exploitation for capital formation
- Sequential bank interest schemes (geometric money growth)
- Maintenance cost arbitrage through trade deals
- Complex attack coordination with travel times

### From Falcon's Eye (FE)

**Core Mechanics:**
- Race/faction system (9 races with different modifiers)
- Worker skill levels (5.0-10.0) with switching costs
- Building improvement tech tree (40+ buildings)
- Magic system (5 spell books, cost triples per casting)
- Complex relationship/approval systems

**The Specialization System:**
FE created **path dependency** through:
- Worker skills that reset when reassigned (5.0 → 10.0 takes time)
- Race bonuses that encouraged different playstyles
- Building prerequisites that locked you into strategic choices
- Spell books that couldn't be changed

This forced players to **specialize and cooperate** rather than try to be good at everything.

**What Made FE Special:**
- Deep economic simulation with multiple feedback loops
- Approval Rating affecting population growth
- Pollution as production ceiling
- Cultural/diplomatic relations affecting trade viability
- Long-term strategic planning over months

---

## Game Systems Design

### Resource Economy

**Primary Resources:**
- **Credits** - Universal currency for purchases
- **Fuel** - Consumed by population & military (food equivalent)
- **Ore** - Input for production (building/military)
- **Population** - The engine driving all production

**Secondary Resources:**
- **Intel** - Enables espionage & reconnaissance
- **Research Points** - Unlocks efficiency improvements

**Resource Interdependencies:**
- Can't have military without Ore + Credits
- Can't grow population without Fuel + Habitats
- Can't increase efficiency without Research
- Can't gather intelligence without Spies (population allocation)

### The Sector System

Players control **Sectors** (space regions), each with specific production:

| Sector Type | Produces | Consumes | Notes |
|-------------|----------|----------|-------|
| **Asteroid Fields** | Ore | Credits (maintenance) | Stable, medium yield |
| **Gas Giants** | Fuel | Credits | High yield, generates pollution |
| **Habitats** | Population capacity | Credits, Fuel | Required for growth |
| **Trade Stations** | Credits | Credits (smaller) | Scales with population |
| **Research Colonies** | Research Points | Credits, Fuel | Reduces pollution |
| **Industrial Zones** | Builds military/structures | Ore, Credits | Generates pollution |

**Key Constraints:**
- Total sectors limited by Command Rating (starts at 50)
- Sector prices rise as supply decreases (zero-sum in Phase 2+)
- Each sector requires maintenance (Credits + Fuel)
- Failed maintenance = sector goes offline

### Population & Labor

**Six Labor Types:**

| Type | Produces | Starting Skill | Max Skill | Notes |
|------|----------|----------------|-----------|-------|
| **Miners** | Ore from Asteroids | 50% | 100% | +5% skill per turn |
| **Engineers** | Operate Industrial Zones | 50% | 100% | Build military/structures |
| **Researchers** | Research Points | 50% | 100% | Unlock efficiency |
| **Traders** | Credits from Trade Stations | 50% | 100% | Scale with population |
| **Extractors** | Fuel from Gas Giants | 50% | 100% | High pollution |
| **Spies** | Intel | 50% | 100% | Enable recon/sabotage |

**The Skill System:**
- Workers start at 50% efficiency when first assigned
- Gain +5% skill per turn until 100%
- **Reassigning resets skill to 50%** ← Critical switching cost
- Creates path dependency: Can't efficiently pivot strategies mid-game

**Population Dynamics:**
- Growth rate determined by Approval Rating + Habitat capacity
- Density matters: <70% capacity = immigration, >90% = emigration
- Losing battles, deficits, overcrowding reduce Approval
- Winning battles, growth, low density increase Approval

### Military System (Rock/Paper/Scissors)

**Three Force Categories:**

#### **Fighters** (Space Superiority)
- **Strong vs:** Bombers (2x damage)
- **Weak vs:** Defense Platforms (0.5x damage)
- **Stats:** High attack (8), Low defense (3)
- **Cost:** 3 Ore, 100 Credits
- **Maintenance:** 5 Credits/turn, 0.025 Fuel/unit
- **Use Case:** Counter enemy Bombers, raid weakly defended targets

#### **Bombers** (Strike Craft)
- **Strong vs:** Defense Platforms (2x damage), Structures
- **Weak vs:** Fighters (0.5x damage)
- **Stats:** Medium attack (6), Medium defense (5)
- **Cost:** 6 Ore, 200 Credits
- **Maintenance:** 10 Credits/turn, 0.025 Fuel/unit
- **Special:** Can target specific structures in attacks
- **Use Case:** Break fortified positions, destroy economy

#### **Defense Platforms** (Static Defense)
- **Strong vs:** Fighters (2x damage)
- **Weak vs:** Bombers (0.5x damage)
- **Stats:** Low attack (4), High defense (10)
- **Cost:** 10 Ore, 400 Credits
- **Maintenance:** 20 Credits/turn, 0.010 Fuel/unit
- **Limitation:** Cannot attack (defense only)
- **Use Case:** Protect your sectors, force attackers into bad trades

#### **Capital Ships** (Force Projection)
- **Universal:** Good vs everything (6 attack/6 defense), great at nothing
- **Cost:** 60 Ore, 5,000 Credits (10x other units)
- **Maintenance:** 100 Credits/turn, 0.5 Fuel/unit
- **Special:** Required to transport Fighters/Bombers to distant targets
  - 1 Capital Ship carries 100 other units
  - Cannot attack without Capital Ships in fleet
- **Use Case:** Enable long-range operations, generalist force

**Information Asymmetry:**
- Default intel: "Player M has 15,000 total military strength"
- To see composition: Send Spy (costs Intel, 70% success) OR fight them once
- **Critical strategic element:** Guessing wrong means your counter-force gets crushed

**Example Scenario:**
```
You see: "Player Q has 20,000 military strength"
You don't know if that's:
  - 2,000 Fighters (would destroy your Bomber fleet)
  - 2,000 Defense Platforms (vulnerable to your Bombers)
  - 1,000 mixed forces (depends on ratio)
  
Send spy? (Costs 50 Intel, 70% success rate)
Or attack blind and learn the hard way?
```

### Feedback Systems: Approval & Pollution

#### **Approval Rating (0-100%)**

**Affects Population Growth:**
- 100% approval = +5% max growth per turn
- 50% approval = 0% growth (stable)
- 0% approval = -5% decline per turn (emigration)

**Increases From:**
- Winning battles (+2-5 depending on scale)
- Positive Credit balance (+1/turn if growing)
- Low population density (+1/turn if <70% capacity)
- Structures (Entertainment Districts, etc.)
- Positive diplomatic relations

**Decreases From:**
- Losing battles (-3-8 depending on scale)
- Deficit spending (-2/turn)
- Failed maintenance payments (-5/turn)
- Overcrowding (-2/turn if >90% capacity)
- Being attacked (-1 per attack received)
- War with liked nations (-2/turn)

**Critical Event:**
- Below 30% approval: 10% chance/turn of **Revolt**
  - Lose 20% population + 10% military
  - Approval resets to 40%

#### **Pollution (0-100%)**

**Generated By:**
- Industrial Zones: +0.5/turn per sector
- Military production: +0.1 per 100 units built
- Gas Giant Extraction: +0.3/turn per sector

**Reduced By:**
- Research Colonies: -0.2/turn per sector
- Habitats: -0.1/turn per sector (life support systems)
- Natural decay: -1% of current pollution per turn

**Production Penalties:**

| Pollution | Production Multiplier |
|-----------|----------------------|
| 0-20% | 100% (no effect) |
| 21-40% | 95% |
| 41-60% | 85% |
| 61-80% | 70% |
| 81-100% | 50% (choking) |

**Critical Event:**
- Above 80%: **Ecological Crisis**
  - -10 Approval per turn
  - 5% chance/turn of losing random sector to contamination

**Strategic Trade-off:**
You could go all-in on Industrial Zones for massive military... but you'll choke on pollution within 10-15 turns unless you balance with Research Colonies.

---

## The Core Gameplay Loop

### Turn Structure (5-Minute Timer)

When you start a turn, a **5-minute countdown begins**. If time expires, turn auto-completes with no changes applied. This creates genuine time pressure.

**Phase 1: Production Report (30s - view only)**
```
Your 150 Miners extracted 450 Ore (+5% efficiency from Research)
Your 200 Traders generated 600,000 Credits (+12% from population growth)
15 Fighters returned from battle against Player Q (victory, minimal losses)

⚠ LOW FUEL - Will run deficit of 2,500 next turn
✓ All sectors operational
✓ Approval: 67% (Stable)
⚠ Pollution: 58% (Moderate impact: -15% production)
```

**Phase 2: Maintenance Payment (30s)**
- Auto-deduct if resources available
- If short: Choose what to pay/skip
- Unpaid sectors go offline, unpaid military deserts

**Phase 3: Markets & Diplomacy (1-2min)**
- Buy/Sell resources on shared market (prices fluctuate)
- Send/receive trade deals with other players
- Propose/accept alliance treaties
- Read forum messages (global + alliance)

**Phase 4: Development (1-2min)**
- Reassign population (with skill reset warnings)
- Buy/sell sectors (prices rise as supply decreases)
- Queue structures (take multiple turns to build)
- Build military units (instant, requires Ore + Credits)

**Phase 5: Military Orders (1min)**
- Attack another Realm (choose target, forces, objective)
- Join Group Attack (if one is forming)
- Deploy Spies (Intel gathering/sabotage)

**Phase 6: Research & Messages (30s)**
- Allocate Research priorities (affects what improves)
- Post to forums (coordinate with allies)
- End turn (or let timer expire)

### The "Cooperation Exploit" - Resource Chain Amplification

**The Mechanic:**
Certain production types get bonuses based on stockpile size:
- **Trade Stations:** +1% revenue per 1M Credits in treasury (max +50%)
- **Industrial Zones:** +1% efficiency per 10k Ore stockpiled (max +50%)

**The Exploit:**
1. Alliance members coordinate login times
2. Player A finishes turn → sends all Credits to Player B
3. Player B's turn: Earns boosted production, sends accumulated wealth to Player C
4. Continue down chain of 5-10 players
5. Last player splits the pot back to everyone

**Example:**
```
Day 1 with 5-player chain, each starting with 1M Credits:

Player A: 1M → earn 100k production → send 1.1M to B
Player B: 1.1M + 1M own = 2.1M → earn 252k → send 2.352M to C
Player C: 2.352M + 1M = 3.352M → earn 403k → send 3.755M to D
Player D: 3.755M + 1M = 4.755M → earn 571k → send 5.326M to E
Player E: 5.326M + 1M = 6.326M → earn 759k → splits 7.085M back

Each player ends with 1.417M (41.7% daily return)
```

**Requirements:**
- Must be in same Alliance (only auto-accept trades)
- Must coordinate exact login times
- Must trust everyone (or one person runs with the money)
- Last player in chain needs enough storage capacity

**This recreates the exact dynamic from BRE's bank interest scheme:**
- Massive returns through cooperation
- Requires trust + coordination
- One defector ruins everything
- Creates natural alliance hierarchies (who plays when?)

---

## Three-Phase World Progression

Each World has a **finite lifespan** divided into three distinct phases. Phase lengths are randomly generated at World creation.

### **Phase 1: Growth (10-14 days)**

**Characteristics:**
- Sectors plentiful (100 per player available)
- Sector prices low (3,000-5,000 Credits)
- No PvP attacks allowed (except pirates/NPCs)
- No inter-World attacks
- Focus: Optimize economy, grab sectors, experiment with builds

**Winning Strategy:**
- Secure 40-50 sectors quickly
- Balance sector types for self-sufficiency
- Invest in skill development (avoid reassigning workers)
- Build Research Colonies early (pollution will hurt later)
- Form alliances, establish trust

**Common Mistakes:**
- Over-investing in military (no one to fight yet)
- Neglecting Research (falling behind on efficiency)
- Poor sector balance (fuel/ore shortages)
- Not joining an alliance early

### **Phase 2: Consolidation (10-14 days)**

**Characteristics:**
- Sector supply frozen (zero-sum competition)
- PvP attacks enabled (within World only)
- Sector prices spike (10,000-20,000 Credits)
- Alliance warfare begins
- Focus: Military buildup, resource raids, territory control

**Winning Strategy:**
- Establish military dominance within your Alliance
- Coordinate attacks with allies (don't go solo)
- Use Spies to identify targets (avoid bad matchups)
- Raid for resources (Credits/Fuel/Ore) not just sectors
- Protect your economy (Defense Platforms matter now)

**Common Mistakes:**
- Attacking without intel (wrong unit composition)
- Neglecting defense (getting counter-raided)
- Breaking alliance trust (betrayal counter++)
- Overextending military (maintenance costs explode)

**Key Dynamic:**
This is when **alliances solidify or fracture**. Players who cooperated in Phase 1 now face pressure:
- Should we attack that wealthy neutral player?
- Do we trust each other with resource chains?
- Who leads our Alliance into Phase 3?

### **Phase 3: World War (7-10 days)**

**Characteristics:**
- Inter-World attacks enabled
- **Doomsday Device** becomes buildable
- Alliance vs. Alliance mega-battles
- Winner-takes-all endgame
- Focus: Coordinate 80%+ of World for victory

**Winning Strategy:**
- Achieve consensus on Doomsday Device funding
- Coordinate defense rotations (protect the Device)
- Execute coordinated strikes on enemy Worlds
- Prevent internal betrayals (purge unreliable players)

**Common Mistakes:**
- Failing to achieve 80% participation (Device never launches)
- Poor defense coordination (enemy destroys Device)
- Internal power struggles (wastes time)
- Attacking wrong target World (pick vulnerable one)

**The Doomsday Device:**

**Requirements:**
- 80% of World's total population contributes Research
- 75% of World's Ore production for 3 consecutive turns
- 5 turns to build (can be sabotaged by enemy Spies)
- Requires 10,000,000 Credits upfront funding

**Effect When Launched:**
- Targets enemy World (chosen at construction start)
- Destroys 10% of all sectors immediately
- Continues destroying 5% more per turn for 5 turns
- Total: 35% sector destruction if not stopped

**Counter-Play:**
- Enemy World must coordinate defense
- Requires ~60% of World's Fighters to destroy Device
- Each turn Device survives = more damage accumulates
- Defending is harder than building (coordination problem)

**The Coordination Challenge:**
Building the Device requires:
- Convincing 21+ players (80% of 26) to contribute
- Maintaining contribution over 3 turns (some will get attacked, go offline)
- Protecting construction from enemy Spies (requires Intel spending)
- Choosing target wisely (attack strongest? or most vulnerable?)
- Preparing follow-up attacks (Device alone doesn't win)

**This is the culmination of everything:**
- Trust networks built in Phase 1
- Military strength from Phase 2
- Communication/coordination skills throughout

**One defector can ruin weeks of effort** - which is exactly the point.

---

## Alliance System & Communication

### No Direct Messages, Forum-Only

**Available Forums:**
1. **Global Forum** - All players can read/post (public diplomacy)
2. **Alliance Forums** - Only alliance members see (coordination)

**Why This Restriction?**
- Forces **public accountability** for agreements
- Creates **shared information** (everyone sees the same messages)
- Prevents **secret side-deals** that undermine trust
- Makes **betrayals visible** (hard to lie when messages persist)

### Alliance Mechanics

**Creation:**
- Costs 10,000 Credits
- Requires 3 founding members
- Alliance leader designated (most respected/experienced player)
- Leader can: Set policies, kick members, propose votes

**Alliance Actions:**

**1. Group Attacks**
- Any member can propose
- 48-hour window for others to join
- Specify: Target (Player/World), Forces committed
- Returns distributed by contribution percentage
- Example:
  ```
  [GROUP ATTACK PROPOSED]
  Target: Player Q (World 3)
  Objective: Capture Sectors
  Window: 42 hours remaining
  
  Committed:
  [M] 500 Bombers, 50 Capital Ships
  [A] 300 Fighters, 30 Capital Ships
  [C] 200 Fighters
  
  Estimated Success: 85% (based on known intel)
  [JOIN ATTACK] [VIEW INTEL] [DISCUSS]
  ```

**2. Intel Sharing**
- Spy reports visible to all alliance members
- Shared Intel database (who's strong/weak)
- Pooled Intel resources for major ops

**3. Resource Chains**
- Coordinate login times for sequential resource passing
- Trust requirement: Must believe last player will split pot
- One defector ruins entire chain

**4. Diplomatic Votes**
- Declare war on another Alliance
- Accept peace offers
- Kick/recruit members
- Contribute to Doomsday Device

**Alliance Capacity:**
- Maximum 12 players per alliance
- Can have multiple alliances per World
- Alliance reputation persists across Worlds

### Reputation & Betrayal System

**What Counts as Betrayal:**
1. **Leaving alliance during Phase 3** (when coordination matters most)
2. **Keeping resources from a resource chain** (not splitting pot)
3. **Refusing Doomsday contribution after pledging**
4. **Attacking alliance members**
5. **Sharing alliance intel with enemies**

**Public Reputation Stats (visible to all):**
```
PlayerName [ID: M]
Worlds Played: 7
Victories: 2 (World 3, World 5)
Defeats: 5
Betrayals: 0 ← This is the key metric
Alliances Formed: 9
Attacks Launched: 47
Successful Defenses: 23
```

**Betrayal Counter** is **permanent** - there's no way to reduce it. This creates:
- **Social consequences** - high betrayal = hard to find allies
- **Strategic choices** - is this betrayal worth the permanent reputation hit?
- **Trust signals** - zero betrayals = reliable player

**Example Scenario:**
```
You're in a 5-player resource chain.
You're last in chain with 8M Credits.
Contract: Split evenly (1.6M each).

Option A: Honor agreement (-0 betrayals, +trust)
Option B: Keep everything (+1 betrayal, +6.4M Credits, burned bridges)

Next World, players see your Betrayal: 1
They're less likely to include you in chains.
After 3-4 betrayals, you're essentially blacklisted.
```

---

## World Generation & Lifecycle

### World Creation Parameters

When a new World spawns, it randomly generates:

| Parameter | Range | Impact |
|-----------|-------|--------|
| **Turns Per Day** | 5, 7, or 10 | Hardcore vs. Casual |
| **Phase 1 Length** | 10-14 days | Time to build economy |
| **Phase 2 Length** | 10-14 days | PvP intensity |
| **Phase 3 Length** | 7-10 days | Endgame urgency |
| **Sector Multiplier** | 0.8x - 1.2x | Resource abundance |

**These are visible before joining** - players can choose Worlds matching their playstyle.

**Example Worlds:**
```
WORLD 07 [HARDCORE]
Turns/Day: 5
Phase 1: 10 days
Phase 2: 10 days  
Phase 3: 7 days
Sector Multiplier: 0.8x (scarce)
Slots: 12/26 filled
Spawns in: 3 days

WORLD 08 [CASUAL]
Turns/Day: 10
Phase 1: 14 days
Phase 2: 14 days
Phase 3: 10 days
Sector Multiplier: 1.2x (abundant)
Slots: 8/26 filled
Spawns in: 5 days
```

### World Lifecycle

**Pre-Spawn (7 days):**
- World visible in lobby
- Players can "reserve" slots (commitment to join)
- Chat available for pre-alliance formation
- When 26 players reserved OR 7 days pass → World spawns

**Active (4-6 weeks):**
- Progresses through 3 phases
- Turn counters reset daily (24hr after your last turn)
- Players can join until World is full (26 max)
- Late joiners start in Protection (can't be attacked for 3 days)

**Post-Game (Persistent Record):**
- Final rankings published
- Victory/defeat recorded in player stats
- Alliance performance tracked
- World becomes read-only history (can browse end state)

**Victory Conditions:**

**Individual Victory:**
1. Highest Networth at game end
2. Survived all 3 phases
3. Contributed to winning Alliance

**Alliance Victory:**
1. Successfully launched Doomsday Device that destroyed target
2. OR: Highest combined Networth at game end
3. OR: Eliminated all other Alliances

**World Victory:**
1. Successfully defended against all Doomsday Devices
2. OR: Launched most successful Doomsday Device
3. Total Networth ranking across all Worlds (bragging rights)

---

## Player Identification & Interface

### 26-Player ID System

Each player gets **single-letter ID (A-Z)** for their World:

**In all interfaces:**
- "Player M attacked Player F"
- "Alliance [NOVA] (M, A, C, Q) declared war on [VOID] (F, G, K, L)"
- "Group Attack: M+A+C vs. Player Q (World 3)"

**Your own ID** is always **highlighted/colored** for visibility.

**In Forums:**
```
[M] PlayerName: "We should coordinate attack on World 7"
[A] OtherPlayer: "Agreed. I can contribute 500 Bombers."
[C] ThirdPlayer: "I'll spy on their defenses first."
```

Keeps it **scannable** - you can quickly see who's talking without reading full names.

### Mobile UI Design

**Core Principle:** Every action must be achievable in **5 minutes** with minimal taps.

**Main Turn Screen:**
```
┌─────────────────────────────────────┐
│ TURN 47/70       [3:42 remaining] ⏱ │
│ Phase 2: Consolidation (Day 18/12)  │
├─────────────────────────────────────┤
│ ⚠ LOW FUEL - Will deficit 2,500     │
│ ⚠ Pollution: 58% (-15% production)  │
│ ✓ All sectors operational           │
├─────────────────────────────────────┤
│ [1] 📊 Production Report             │
│ [2] 💰 Markets & Trade               │
│ [3] 🏗️ Development                   │
│ [4] ⚔️ Military                      │
│ [5] 💬 Messages (3 new)              │
│                                      │
│ [END TURN] ✅                        │
└─────────────────────────────────────┘
```

**Design Priorities:**
1. **Timer always visible** - creates urgency
2. **Critical warnings at top** - can't miss problems
3. **Clear visual hierarchy** - icons + labels
4. **One-tap access** to all sub-screens
5. **End Turn** button prominent (confirm dialog prevents accidents)

**Production Report:**
```
┌─────────── 📊 PRODUCTION ───────────┐
│                                      │
│ 💵 CREDITS:  +650,000  [+5% ↑]      │
│   └ Trade Stations: +500k            │
│   └ Market Sales: +150k              │
│                                      │
│ ⛽ FUEL:      +1,200    [-300 ⚠]     │
│   └ Gas Giants: +1,500               │
│   └ Consumption: -300                │
│                                      │
│ ⛏️ ORE:       +800      [stable]     │
│   └ Asteroids: +800                  │
│                                      │
│ 🕵️ INTEL:     +50       [↑]          │
│   └ Spies: +50                       │
│                                      │
│ 👥 POPULATION: 1,850 (+25 ↑)        │
│   └ Approval: 67% [Stable]           │
│   └ Density: 74% [Good]              │
│                                      │
│ ☠️ POLLUTION:  58% [+2 this turn]    │
│   └ Impact: -15% production          │
│                                      │
│ [CONTINUE] ▶️                         │
└──────────────────────────────────────┘
```

**Key Features:**
- **Emoji icons** for quick visual scanning
- **Color coding:** Green (good), Yellow (warning), Red (critical)
- **Trend indicators:** ↑↓ arrows show direction
- **Tooltips available** (tap any line for details)

**Attack Screen:**
```
┌──────── ⚔️ LAUNCH ATTACK ─────────┐
│                                    │
│ TARGET: [Player Q] 🎯               │
│   World 3, ID: Q                   │
│   Known Intel: 2 days old          │
│   Estimated Forces: 12,000         │
│     └ Composition: UNKNOWN         │
│                                    │
│ OBJECTIVE:                         │
│   ○ Capture Sectors (10%)          │
│   ● Pillage Resources (15%)        │
│   ○ Destroy Military (20%)         │
│                                    │
│ YOUR FORCES:                       │
│   Fighters:  [500]  ✈️             │
│   Bombers:   [200]  💣             │
│   Platforms: [  0]  🛡️ (can't send)│
│   Capital:   [ 10]  🚀 (required)  │
│                                    │
│ ESTIMATED:                         │
│   Success: 65% ⚠️                   │
│   Losses: ~150 units               │
│   Travel: 2 turns                  │
│                                    │
│ 🕵️ [SEND SPY FIRST] (-50 Intel)    │
│                                    │
│ [LAUNCH] ⚔️  [CANCEL] ❌            │
└────────────────────────────────────┘
```

**Smart Defaults:**
- Auto-calculates required Capital Ships (1 per 100 units)
- Warns if sending wrong unit types vs. known enemy comp
- Shows travel time (inter-World attacks take longer)
- Encourages Intel gathering (Spy button prominent)

---

## Technical Architecture (High-Level)

### Technology Stack Considerations

**Backend:**
- **Turn-based architecture** - not real-time, can use traditional request/response
- **Daily turn reset** - cron job at midnight (per player, 24hr from last turn)
- **Asynchronous combat resolution** - battles resolved server-side between turns
- **Forum/message system** - simple threaded discussion model

**Mobile App:**
- **5-minute timer** - client-side countdown, server validates on "End Turn"
- **Offline-capable** - cache World state, sync on turn start
- **Push notifications** - "Your turn available", "You were attacked", "Group attack forming"
- **Low data usage** - critical for mobile (text-heavy, minimal graphics)

**Database Considerations:**
- **Player records** - persistent across Worlds (reputation, stats)
- **World state** - isolated per World (can archive completed Worlds)
- **Turn history** - log all actions for replay/dispute resolution
- **Market state** - shared across World, updates real-time

### Anti-Cheat Measures

**Multi-Accounting:**
- **Device fingerprinting** - detect same device/IP
- **Behavioral analysis** - coordinated actions between accounts
- **Resource chain validation** - flag suspicious one-way trades
- **Penalties:** Account bans, reputation resets

**Automation/Bots:**
- **Turn timing patterns** - humans don't click exactly every 24hr
- **Action sequences** - bots follow predictable patterns
- **CAPTCHA on suspicious behavior** - not every turn (annoying)
- **Rate limiting** - can't spam actions faster than human-possible

**Exploits:**
- **Server-side validation** - never trust client
- **Transaction atomicity** - can't duplicate resources via race conditions
- **Audit logs** - every action logged with timestamp
- **Rollback capability** - if exploit discovered, can revert World state

---

## Monetization (Non-Exploitative)

**Core Principle:** Never sell power. Never sell time. Never create FOMO.

### What We Will NOT Do:
- ❌ Energy systems that require payment to bypass
- ❌ Loot boxes / gacha mechanics
- ❌ Pay-to-win units/bonuses
- ❌ Time-limited exclusive content
- ❌ Aggressive push notifications
- ❌ "Deals" that expire to create urgency

### What We WILL Do:

**1. Cosmetic Customization ($2-5)**
- Custom Realm names/emblems
- Visual themes (UI skins)
- Victory animations
- Forum badges/titles

**2. Convenience (Not Power) ($3-8)**
- **Historical World Access** - view archived Worlds you weren't in
- **Advanced Statistics** - detailed analytics on your performance
- **Multi-World Slots** - play in 2-3 Worlds simultaneously (if you want the cognitive load)
- **Priority Support** - faster response to bugs/issues

**3. Premium Worlds ($5/world)**
- **Curated competition** - verified accounts only (no multi-accounting)
- **Faster phases** - 5-day instead of 10-14 day phases (for experienced players)
- **Larger Worlds** - 50 player slots instead of 26
- **Prize pools** - winner gets Credits for cosmetics

**4. "Support the Game" Tier ($5/month)**
- Everything above
- Exclusive forum badge showing support
- Early access to new features (1 week before free players)
- Name in credits

**Revenue Model:**
- Free-to-play base game (fully competitive, no limits)
- ~5% of players convert to cosmetic purchases
- ~1% subscribe to support tier
- Target: Sustainable, not extractive

---

## Development Roadmap

### Version 1.0 (MVP - 6 months)

**Core Systems:**
- ✅ Turn-based gameplay (5-min timer)
- ✅ Six resource types (Credits, Fuel, Ore, Population, Intel, Research)
- ✅ Six sector types
- ✅ Six labor types with skill system
- ✅ Three military unit types (Fighters, Bombers, Platforms)
- ✅ Approval & Pollution feedback systems
- ✅ Basic combat resolution (attack/defend)
- ✅ Forum system (global + alliance)
- ✅ Alliance creation & management
- ✅ Three-phase World progression
- ✅ 26-player Worlds with single-letter IDs

**NOT in v1.0:**
- ❌ Structures/buildings (permanent for now)
- ❌ Espionage/sabotage (just recon Spies)
- ❌ Capital Ships (added in v1.1)
- ❌ Inter-World attacks (just local PvP in Phase 2)
- ❌ Doomsday Device (v1.1)
- ❌ Multiple themes (Space only)

**Goal:** Prove the core loop is fun. Does the 5-minute constraint work? Do players form alliances? Does reputation matter?

### Version 1.1 (Endgame Content - 3 months)

**Add:**
- ✅ Capital Ships & force projection
- ✅ Inter-World attacks (Phase 3)
- ✅ Doomsday Device construction
- ✅ Espionage operations (sabotage, steal intel)
- ✅ Structure system (10-15 buildings)
- ✅ Tech tree (5-7 research paths)

**Goal:** Complete the Phase 3 endgame. Make World vs World warfare meaningful.

### Version 1.2 (Depth & Polish - 3 months)

**Add:**
- ✅ Advanced unit types (Stealth Fighters, Siege Bombers)
- ✅ Special events (random bonuses/crises)
- ✅ Tournament mode (curated competition)
- ✅ Replay system (watch historical battles)
- ✅ Enhanced statistics dashboard

**Goal:** Add replayability and depth for experienced players.

### Version 2.0 (New Themes - 6 months)

**Add:**
- ✅ Fantasy theme (Falcon's Eye spiritual successor)
  - Kingdoms, Mages, Dragons, Castles
- ✅ Western theme (Frontier Towns, Outlaws, Gold Rush)
- ✅ Cyberpunk theme (Corporations, Hackers, Megacities)

**Goal:** Prove the mechanics work across themes. Each theme has different flavor but same core systems.

---

## Success Metrics

### Player Engagement
- **DAU/MAU ratio** (daily active / monthly active)
  - Target: >30% (high for mobile strategy)
- **Average session length**
  - Target: 5-7 minutes (one turn + reading forums)
- **Multi-day retention**
  - Day 7: >40%
  - Day 30: >20%
  - Day 90: >10%

### Social Dynamics
- **Alliance participation rate**
  - Target: >70% of players in alliances by Phase 2
- **Forum post frequency**
  - Target: >5 posts per player per World
- **Betrayal rate**
  - Target: <10% (most players honor agreements)
- **Resource chain participation**
  - Target: >30% of alliances attempt chains

### Competitive Health
- **Skill-based outcomes**
  - Correlation between experience and victory: >0.6
  - Random luck factor: <20% of outcome variance
- **Comebacks possible**
  - Players ranked 10-15 can win: >15% of Worlds
- **Snowballing controlled**
  - Early leader wins: <40% of Worlds

### Monetization (Sustainable)
- **Conversion to any payment:** >5%
- **Average revenue per paying user:** $15-25 over 6 months
- **Churn rate of paying users:** <30% per quarter
- **NPS (Net Promoter Score):** >50

### Community Health
- **Toxicity reports:** <1% of players per World
- **Player-organized tournaments:** >5 per quarter
- **User-generated strategy guides:** >10 active
- **Discord/external community size:** >1000 engaged members

---

## Why This Will Work

### The Problem We're Solving

**Modern mobile gaming has trained players to expect:**
- Instant gratification
- Constant availability
- No consequences
- Pay-to-skip mechanics

**But what players actually WANT (even if they don't know it):**
- Meaningful decisions with lasting impact
- Social experiences that matter
- Genuine accomplishment
- Respect for their time

**We're betting that a segment of players is hungry for:**
- Strategy over twitch reflexes
- Cooperation over competition
- Long-term planning over dopamine hits
- Genuine scarcity over artificial timers

### Our Unique Position

**We're not competing with:**
- Clash of Clans (real-time, pay-to-win, always-on)
- Civilization (single-player, not mobile-first)
- Eve Online (too complex, requires desktop)
- Board game apps (no persistent world)

**We're creating a new category:**
- **Asynchronous multiplayer strategy**
- **Genuine scarcity** (not artificial energy)
- **Social complexity** (not just matchmaking)
- **Mobile-first** (not desktop port)

### The Market Opportunity

**Target Audience:**
1. **Former BBS gamers** (now 40-50 years old)
   - Nostalgic for the experience
   - Have disposable income
   - Prefer strategic depth over action
   - 5-10 million potential players worldwide

2. **Strategy game enthusiasts** (25-45 years old)
   - Play Civilization, Paradox games, board games
   - Want mobile strategy that respects intelligence
   - Frustrated with pay-to-win mechanics
   - 20-50 million potential players

3. **"Hardcore casual" gamers** (30-50 years old)
   - Limited time but want depth
   - 5 minutes/day is perfect
   - Value skill over time investment
   - 50-100 million potential players

**Conservative Estimate:**
- Total addressable market: 10M players
- Realistic capture: 0.1% = 10,000 players
- Conversion rate: 5% = 500 paying users
- ARPU: $20/year = $10,000/year revenue

**Optimistic Scenario:**
- TAM: 50M players
- Capture: 0.5% = 250,000 players
- Conversion: 8% = 20,000 paying users
- ARPU: $30/year = $600,000/year revenue

---

## Risks & Mitigations

### Risk 1: "No one wants constrained gameplay anymore"

**Evidence Against:**
- Wordle succeeded with 1 puzzle/day
- Duolingo limits free lessons
- Chess.com limits daily games (free tier)

**Mitigation:**
- MVP with small test audience (500 players)
- A/B test: 5 turns/day vs. 10 turns/day
- Monitor churn: Do constraints help or hurt retention?

### Risk 2: "Coordination is too hard without real-time chat"

**Evidence Against:**
- BBS games succeeded with forum-only communication
- Diplomacy (board game) thrives on async negotiation
- Email chess is still popular

**Mitigation:**
- In-game planning tools (attack scheduler, resource chain calculator)
- Alliance coordination dashboard
- Push notifications for critical events

### Risk 3: "Mobile players won't commit to 4-6 week games"

**Evidence Against:**
- Idle games retain for months
- Clash of Clans clans persist for years
- Fantasy sports leagues run all season

**Mitigation:**
- Clear phase structure (can quit after Phase 1, ~2 weeks)
- Substitute/replacement system (if you quit, AI takes over)
- Shorter "Blitz" Worlds (1-2 weeks total) for casual players

### Risk 4: "One griefer/betrayer ruins everyone's fun"

**Evidence Against:**
- Eve Online thrives on betrayal (but has consequences)
- Among Us is literally about betrayal
- Diplomacy wouldn't exist without it

**Mitigation:**
- **Permanent betrayal counter** (reputation matters)
- **Kick from alliance** (majority vote)
- **Quarantine system** (high betrayal = relegated to "Exile Worlds")

### Risk 5: "Can't monetize without pay-to-win"

**Evidence Against:**
- Fortnite: $5B/year, zero pay-to-win
- Path of Exile: Cosmetics only, successful
- Dota 2: Battle Pass model, no power

**Mitigation:**
- Focus on passionate niche (not mass market)
- Premium Worlds with prize pools
- Cosmetic customization for identity expression
- "Support the devs" culture (Dwarf Fortress model)

---

## Conclusion: What We're Really Building

**This isn't just a game. It's a social experiment.**

We're testing whether modern players—accustomed to instant gratification, unlimited access, and zero consequences—can rediscover the joy of:

- **Delayed gratification:** Waiting 24 hours for your next turn
- **Genuine scarcity:** You can't buy more turns
- **Lasting consequences:** Your reputation follows you
- **Asynchronous cooperation:** Coordinating across time zones via forums
- **Earned victories:** Winning through strategy, not spending

**If we succeed, we prove:**
- Constraints create value
- Scarcity breeds engagement
- Cooperation is more satisfying than pay-to-win
- Mobile gaming can be intellectually satisfying

**If we fail, we learn:**
- Modern audiences truly want different things
- The BBS era was a product of its time
- Nostalgia isn't a viable market

**Either way, it's worth trying.**

Because right now, mobile gaming is a wasteland of Skinner boxes and dark patterns. And a generation of players who experienced something better—games that taught real skills, built real communities, and created real memories—deserves one more chance to play a game that **respects them**.

---

## Appendices

### A. Glossary of Terms

**BBS (Bulletin Board System):** Pre-internet dial-up computer systems (1980s-90s) where users connected via modem to play games, read messages, and download files.

**Door Game:** A game hosted on a BBS that "took control" of the modem connection during play, then returned control to the BBS software.

**Turn-based:** Gameplay structure where players take discrete "turns" rather than acting in real-time.

**Asynchronous:** Players don't need to be online simultaneously; actions taken at different times interact.

**Zero-sum:** One player's gain is another's loss (e.g., fixed number of sectors).

**Networth:** Total value of all assets (sectors, military, resources, population).

**Approval Rating:** Measure of population satisfaction (0-100%) affecting growth.

**Pollution:** Environmental degradation from production, reducing efficiency.

**Realm:** A player's empire/nation/territory in the game.

**World:** A game instance with 26 players progressing through 3 phases.

**Alliance:** Formal group of players cooperating toward shared goals.

**Betrayal Counter:** Permanent stat tracking broken agreements.

**Resource Chain:** Cooperative strategy where players pass resources sequentially for amplified gains.

**Doomsday Device:** Endgame superweapon requiring 80%+ World cooperation.

**Phase 1 (Growth):** Initial peaceful expansion period.

**Phase 2 (Consolidation):** PvP combat within World.

**Phase 3 (World War):** Inter-World attacks and Doomsday Devices.

### B. Frequently Asked Questions

**Q: What if I miss my turns for a day?**
A: Your realm persists, but you'll fall behind. Missing multiple days = likely defeat unless allies cover for you.

**Q: Can I play in multiple Worlds at once?**
A: Yes (premium feature). But cognitive load is high—most players stick to 1-2.

**Q: What happens if my alliance betrays me?**
A: You can leave (costs reputation if in Phase 3) or try to organize a counter-coup. Drama is part of the game.

**Q: Is this pay-to-win?**
A: Absolutely not. Money buys cosmetics only. Skill and coordination determine victory.

**Q: How long does a World last?**
A: 4-6 weeks depending on randomly generated phase lengths.

**Q: What if someone quits mid-game?**
A: Their realm becomes AI-controlled (defensive, won't attack). Sectors/resources remain capturable.

**Q: Can I be in multiple alliances?**
A: No. One alliance per World. Choose wisely.

**Q: Is there a single-player mode?**
A: No. The entire point is human interaction. You can practice against AI in tutorial.

**Q: What prevents griefing?**
A: Permanent reputation system. High betrayal counter = hard to find allies in future Worlds.

**Q: Is voice chat supported?**
A: No. Forum-only communication is intentional design. Use external Discord if desired.

---

### C. Core Design Inspirations Summary

| Element | Inspired By | What We Took | What We Changed |
|---------|-------------|--------------|-----------------|
| Daily turn limit | BRE | Scarcity creates value | 5-minute timer per turn |
| Multi-phase progression | BRE | Growth → PvP → Endgame | Randomized phase lengths |
| Doomsday Device | BRE (Gooie Kablooie) | Requires 80% cooperation | Inter-World targeting |
| Resource amplification | BRE (bank interest) | Sequential passing multiplies gains | Stockpile-based bonuses |
| Labor specialization | Falcon's Eye | Worker skill 50-100% | Simplified to 6 types |
| Approval/Feedback systems | Falcon's Eye | Population growth control | Added Pollution system |
| Race diversity | Falcon's Eye | Specialization niches | Planned for v2.0 (themes) |
| Rock/Paper/Scissors combat | General strategy games | Unit counters | Air/Ground/Static trinity |
| Reputation systems | Eve Online | Betrayal has consequences | Permanent public counter |
| Forum-only communication | BBS era | Async coordination | No direct messages |

---

**Document Version:** 1.0  
**Last Updated:** 2025-01-29  
**Status:** Pre-Development Design Doc  
**Next Steps:** Build MVP prototype, playtest core loop with 26 players

---

*"The best games teach you something about the real world. The worst games teach you to spend money."*
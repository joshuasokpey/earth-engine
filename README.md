# Earth-Engine
A living simulation project for the GreenRes Hackathon. 

# Design-Bible
# EARTH ENGINE — SYSTEMS INTERACTION BIBLE
### The Nervous System of a Civilisation
**Companion volume to the Earth Engine Internal Design Bible.**
Where the Design Bible answers *what Earth Engine is and why*, this volume answers *what is
wired to what, in which direction, with what strength, and after how long*.

---

## 0. HOW TO READ THIS DOCUMENT

### 0.1 What this document is

This is a systems engineering manual, not a design document. It contains no gameplay proposals,
no UI mockups, and no narrative. It contains the complete directed graph of the Earth Engine
simulation: **84 systems, 254 canonical edges, 31 named feedback loops**, each specified precisely enough
that an engineer can implement it without asking a designer what was meant.

The governing claim of the Design Bible — *"no system exists in isolation"* (§1.7) — is not a
slogan here. It is a schema constraint. Every system in Part II has at least one incoming edge
and at least one outgoing edge. A system that could not satisfy that was cut before this document
was written.

### 0.2 The one idea this document exists to encode

**Earth Engine does not simulate objects. It simulates relationships.**

A river is not a blue ribbon with a `width` field. A river is the place where *rainfall*,
*groundwater*, *irrigation draw*, *evaporation*, *fish stock*, *transport*, *disease vector load*
and *twelve household incomes* meet and have to agree with each other. Delete the river and you do
not lose an object — you lose forty-one edges, and the graph tears.

The player will eventually discover this the only way it can honestly be discovered: by changing
one thing and watching thirty things move.

### 0.3 Notation

Every relationship in this document is written in one canonical form:

```
SOURCE.variable  --[ sign | weight | lag | shape ]-->  TARGET.variable
```

| Field | Values | Meaning |
|---|---|---|
| **sign** | `+` / `−` | Direction. `+` = source up ⇒ target up. |
| **weight** | `0.05 … 1.0` | Coupling strength as a coefficient, tunable from JSON. `≥0.7` strong, `0.3–0.7` moderate, `<0.3` weak. |
| **lag** | `0d`, `5d`, `1s`, `1y`, `5y` | Delay before the effect is measurable. `d`=ticks/days, `s`=seasons (90 ticks), `y`=years (360 ticks). |
| **shape** | `lin`, `sat`, `thr`, `exp`, `mult`, `prob` | Response curve — see §9.2. |

Example, read aloud: *"Rainfall drives soil moisture positively, strongly, within a day, on a
saturating curve."*

```
CLI-04.rainfall --[ + | 0.85 | 0d | sat ]--> HYD-01.soilMoisture
```

### 0.4 Domain colour code

Used consistently in this document, in the Systems Atlas, and in the Design Bible's §9.2 UI
language. Domain colour is the single visual key that ties all three together.

| Domain | Prefix | Colour | Hex | Character |
|---|---|---|---|---|
| Climate | `CLI` | Sky blue | `#4A90D9` | Exogenous. Drives everything, is driven by almost nothing. |
| Hydrology | `HYD` | Water blue | `#2F7FD6` | The spine. Highest betweenness in the entire graph. |
| Ecology & Land | `ECO` | Deep green | `#1F7A41` | Slowest. Destroyed in a tick, rebuilt over years. |
| Agriculture | `AGR` | Leaf green | `#3F9E52` | The transducer: turns environment into economy. |
| Economy | `ECN` | Indigo | `#4046B4` | Fastest propagation. Moves shocks across the village in one season. |
| Society & Health | `SOC` | Purple / Red | `#8244C8` / `#D13B3B` | Where consequence becomes human. Highest inertia in recovery. |
| Infrastructure | `INF` | Orange | `#E07A1F` | Buffers. Converts money into resistance-to-shock. |
| Environment Quality | `ENV` | Grey-gold | `#B08A3E` | Accumulators. Slow to load, slow to clear. |
| Governance | `GOV` | Warm gold | `#E8A020` | The player's only hand on the wheel. |
| Fisheries | `FSH` | Teal | `#17A2A8` | Second livelihood; couples ecology directly to income. |

### 0.5 Update tiers

Inherited from Design Bible §5.1 and binding here. Every system chapter declares its tier.

| Tier | Cadence | Why systems land here |
|---|---|---|
| **Fast** | every tick (1 day) | Physically volatile: weather, flow, movement, power draw. |
| **Medium** | every 5 ticks | Biologically buffered: moisture, growth, health, accumulation. |
| **Slow** | every season (90 ticks) | Decisions and demographics: migration, prices, forest, births. |
| **Annual** | every 360 ticks | Trend-level: education attainment, infrastructure depreciation. |
| **Event** | on threshold crossing | Floods, outbreaks, harvests, migrations. |

The tiering is not only a performance decision. It is a **statement about the world**: weather is
fast and forgettable, ecosystems and people are slow and unforgiving. A player who learns nothing
else should learn that the fast systems are the ones you notice and the slow ones are the ones
that decide your fate.

### 0.6 Scope tags

Carried forward from the Design Bible unchanged. This document specifies the full graph; the tag
says when it gets built.

`🟢 MVP` — hackathon build · `🟡 V2` — after the core loop validates · `🔵 FUTURE` — architecture must not foreclose it

**Rule:** a 🟡/🔵 system may appear in the graph, but every 🟢 system must be fully connected using
only other 🟢 systems. The MVP graph must be a closed, working subgraph — never a graph with
holes where the V2 systems will eventually go.

### 0.7 The machine-readable twin

Everything in Parts I, II, III and X is generated from, and must stay in sync with,
`/data/systems-graph.json`:

```jsonc
{
  "systems": [ { "id":"HYD-03", "name":"River Flow", "domain":"HYD",
                 "tier":"fast", "scope":"MVP", "visible":["flowRate"], "hidden":["baseflow"] } ],
  "edges":   [ { "from":"CLI-04.rainfall", "to":"HYD-03.flowRate",
                 "sign":"+", "weight":0.7, "lag":"1d", "shape":"sat",
                 "mechanism":"direct runoff into channel", "loops":["B1"] } ],
  "loops":   [ { "id":"R1", "type":"reinforcing", "path":[...], "doubling":"~2 seasons" } ]
}
```

The Systems Atlas renders from this file. The interaction matrix in Part X is projected from it.
If a coefficient is changed in code but not here, **this document is wrong and must be corrected** —
the Design Bible's §14 "document everything" rule applies to itself, and to this volume.

---

# PART I — COMPLETE SYSTEMS CATALOGUE

84 systems. Each is a chapter in Part II. `In`/`Out` are the edge counts recorded in
`/data/systems-graph.json` for the transcribed core, and are the crude but useful first measure of
how load-bearing a system is. Chapters specify further V2/FUTURE edges that are described in prose
and transcribed as those systems are built.

## I.1 Climate — the exogenous layer

| ID | System | Tier | Scope | In | Out | One-line role |
|---|---|---|---|---|---|---|
| CLI-01 | Climate Drift | Annual | 🟢 | 1 | 6 | The scenario dial. The only place "climate change" enters as a number. |
| CLI-02 | Season Phase | Fast | 🟢 | 0 | 9 | The metronome. Sets planting, rainfall baseline, labour demand. |
| CLI-03 | Temperature | Fast | 🟢 | 3 | 8 | Heat. Drives evaporation, crop stress, health load. |
| CLI-04 | Rainfall | Fast | 🟢 | 3 | 9 | The single most consequential exogenous variable in the game. |
| CLI-05 | Wind | Fast | 🟡 | 2 | 4 | Evaporation term, turbine output, dust and smoke transport. |
| CLI-06 | Solar Irradiance | Fast | 🟢 | 2 | 3 | Photovoltaic output and crop light-limitation. |
| CLI-07 | Potential Evapotranspiration | Fast | 🟢 | 4 | 4 | The demand side of every water balance. |

## I.2 Hydrology — the spine

| ID | System | Tier | Scope | In | Out | One-line role |
|---|---|---|---|---|---|---|
| HYD-01 | Soil Moisture | Medium | 🟢 | 6 | 5 | The variable crops actually respond to. Not rainfall — moisture. |
| HYD-02 | Groundwater | Slow | 🟢 | 5 | 5 | The savings account. Slow to fill, easy to overdraw, punishing to restore. |
| HYD-03 | River Flow | Fast | 🟢 | 6 | 11 | Highest out-degree in the graph. The village's central nervous trunk. |
| HYD-04 | Surface Runoff | Fast | 🟢 | 4 | 4 | The difference between rain that helps and rain that destroys. |
| HYD-05 | Stored Water | Medium | 🟢 | 5 | 4 | The buffer that decides whether a dry week is an inconvenience or a crisis. |
| HYD-06 | Water Quality | Medium | 🟢 | 6 | 4 | Quiet killer. Couples waste and sanitation to health. |
| HYD-07 | Flood | Event | 🟢 | 4 | 9 | Fast, violent, and leaves damage that outlives the water. |
| HYD-08 | Drought Stress | Medium | 🟢 | 4 | 8 | An accumulator, not a flag. Degrees, not states. |
| HYD-09 | Sediment & Turbidity | Medium | 🟡 | 3 | 4 | Erosion's delivery mechanism into the water column. |

## I.3 Ecology & Land — the slow layer

| ID | System | Tier | Scope | In | Out | One-line role |
|---|---|---|---|---|---|---|
| ECO-01 | Forest Density | Slow | 🟢 | 5 | 8 | Asymmetric: cleared in a tick, regrown in a decade. The regret engine. |
| ECO-02 | Soil Quality | Slow | 🟢 | 6 | 4 | Fertility. The stock that quietly decides next decade's yields. |
| ECO-03 | Soil Erosion | Medium | 🟡 | 4 | 3 | The mechanism by which land loss becomes permanent. |
| ECO-04 | Nutrient Cycling | Slow | 🟡 | 4 | 3 | Ties livestock, residue and fallow back to fertility. |
| ECO-05 | Biodiversity | Annual | 🟡 | 5 | 4 | Aggregate ecosystem integrity; gates pollination and fish recruitment. |
| ECO-06 | Fish Stock | Slow | 🟢 | 5 | 3 | A renewable resource with a collapse threshold. The tragedy-of-commons node. |
| ECO-07 | Pollinators | Slow | 🟡 | 3 | 2 | Small, elegant, and invisible until it is gone. |
| ECO-08 | Pest & Crop Disease | Medium | 🟡 | 4 | 2 | Warming's second-order attack on yield. |
| ECO-09 | Land Cover Allocation | Event | 🟢 | 3 | 5 | The zero-sum ledger: forest, farm, settlement, fallow. |
| ECO-10 | Carbon Stock | Annual | 🔵 | 3 | 2 | Present so the National/Global roadmap stages are not foreclosed. |

## I.4 Agriculture & Fisheries — the transducer

| ID | System | Tier | Scope | In | Out | One-line role |
|---|---|---|---|---|---|---|
| AGR-01 | Crop Growth & Yield | Medium/Event | 🟢 | 9 | 6 | Where environment becomes money. The most-watched number in the game. |
| AGR-02 | Irrigation | Medium | 🟢 | 4 | 4 | Resilience that can become the disaster. The double-edge. |
| AGR-03 | Livestock | Slow | 🟡 | 5 | 5 | The household's second buffer, and a nutrient loop participant. |
| AGR-04 | Fisheries Harvest | Medium | 🟢 | 5 | 4 | Effort against a renewable stock. Income independent of rainfall. |
| AGR-05 | Post-Harvest Loss | Event | 🟡 | 4 | 2 | Between a good harvest and a good year sits storage. |
| AGR-06 | Food Supply | Slow | 🟢 | 5 | 3 | Aggregate calories reaching the market and the household store. |
| AGR-07 | Farm Labour Demand | Slow | 🟢 | 4 | 4 | Seasonal. Where child labour becomes economically rational. |
| AGR-08 | Agricultural Knowledge | Annual | 🟡 | 3 | 2 | A lifetime of planting knowledge going out of calibration with the climate. |

## I.5 Economy — the fast propagator

| ID | System | Tier | Scope | In | Out | One-line role |
|---|---|---|---|---|---|---|
| ECN-01 | Food Price | Slow | 🟢 | 5 | 7 | The pivot. Almost every environmental shock reaches people through this node. |
| ECN-02 | Market Activity | Slow | 🟢 | 5 | 5 | The village's visible economic pulse. |
| ECN-03 | Household Income | Slow | 🟢 | 8 | 8 | Per-household, per-archetype. The most heterogeneous variable in the sim. |
| ECN-04 | Savings & Assets | Slow | 🟢 | 4 | 5 | Depth of the household's shock absorber. Determines who survives a bad year. |
| ECN-05 | Employment | Slow | 🟢 | 5 | 4 | Who has work, at what wage, in what season. |
| ECN-06 | Inflation | Annual | 🟡 | 3 | 3 | General price level. Erodes fixed incomes — the Elder's exposure channel. |
| ECN-07 | External Trade | Slow | 🟡 | 4 | 3 | Imports blunt price spikes; the road decides whether they arrive. |
| ECN-08 | Credit & Debt | Slow | 🟡 | 4 | 4 | The mechanism by which one bad season becomes five. |
| ECN-09 | Enterprise Stock | Annual | 🟡 | 4 | 3 | Vendors, workshops, mills. Employment that isn't farming. |
| ECN-10 | Government Budget | Slow | 🟢 | 5 | 6 | The player's action ceiling — and it shrinks exactly when it's needed most. |
| ECN-11 | Taxation | Slow | 🟢 | 3 | 2 | Converts household prosperity into public capacity. |
| ECN-12 | External Aid | Event | 🟡 | 3 | 2 | Arrives late, leaves early, treats symptoms. |
| ECN-13 | Remittances | Slow | 🟡 | 3 | 3 | Migration's one benign return path. |

## I.6 Society & Health — where consequence becomes human

| ID | System | Tier | Scope | In | Out | One-line role |
|---|---|---|---|---|---|---|
| SOC-01 | Food Security | Medium | 🟢 | 5 | 6 | Household calories available vs. required. |
| SOC-02 | Water Security | Medium | 🟢 | 4 | 5 | Litres per person per day, and how far someone walks for them. |
| SOC-03 | Nutrition | Medium | 🟢 | 4 | 4 | Sustained food security, integrated over time. Slow to break, slower to repair. |
| SOC-04 | Health | Medium | 🟢 | 7 | 6 | Per-citizen. The convergence point of water, air, food and care. |
| SOC-05 | Disease Burden | Medium/Event | 🟡 | 5 | 4 | Outbreak risk from water, sanitation, density and nutrition. |
| SOC-06 | Education Attainment | Annual | 🟡 | 3 | 4 | A stock that takes a childhood to build and one season to interrupt. |
| SOC-07 | School Attendance | Slow | 🟢 | 4 | 3 | Not a policy variable. A readout of household desperation. |
| SOC-08 | Child Labour | Slow | 🟢 | 4 | 3 | The household's rational, terrible short-term fix. |
| SOC-09 | Migration Pressure | Slow | 🟢 | 6 | 7 | An accumulator with a threshold and a very slow decay. |
| SOC-10 | Population | Slow | 🟢 | 4 | 7 | Labour, demand, tax base, and density all at once. |
| SOC-11 | Births | Annual | 🟡 | 3 | 2 | Responds to security, not to policy. |
| SOC-12 | Deaths | Annual | 🟡 | 4 | 3 | Modelled honestly, never punitively. |
| SOC-13 | Stress | Medium | 🟡 | 5 | 5 | Psychological load. Degrades decisions, health, and cohesion together. |
| SOC-14 | Social Cohesion | Annual | 🟡 | 5 | 5 | The invisible infrastructure that decides how a shock is absorbed. |
| SOC-15 | Crime & Insecurity | Slow | 🟡 | 4 | 4 | Emerges from deprivation and low cohesion; suppresses enterprise and trade. |
| SOC-16 | Climate Awareness | Annual | 🟡 | 3 | 3 | Changes what households do under identical objective conditions. |
| SOC-17 | Institutional Trust | Annual | 🟡 | 4 | 3 | Determines whether a policy is complied with or ignored. |
| SOC-18 | Community Wellbeing | Slow | 🟢 | 8 | 0 | Terminal aggregate. Reads from everything, writes to nothing, by design. |

## I.7 Infrastructure — the buffers

| ID | System | Tier | Scope | In | Out | One-line role |
|---|---|---|---|---|---|---|
| INF-01 | Power Availability | Fast | 🟢 | 4 | 6 | A hard ceiling on clinic, pump and mill capacity. |
| INF-02 | Energy Mix | Event | 🟢 | 2 | 4 | The clean/dirty split. The clearest trade-off in the game. |
| INF-03 | Water Infrastructure | Event | 🟢 | 3 | 4 | Tower, wells, harvesting, treatment. |
| INF-04 | Transport Network | Event | 🟢 | 3 | 6 | Roads and the bridge. Quiet until it fails, then everything fails with it. |
| INF-05 | Housing Stock | Slow | 🟢 | 4 | 3 | Condition, occupancy, abandonment. |
| INF-06 | Health Facility Capacity | Event | 🟢 | 3 | 3 | Capped by power. An unpowered clinic is a soft clinic. |
| INF-07 | School Capacity | Event | 🟢 | 2 | 3 | Quality multiplier on every attending child. |
| INF-08 | Waste Management | Medium | 🟢 | 3 | 4 | Accumulation, collection, burning. Small system, four sharp edges. |
| INF-09 | Sanitation | Medium | 🟡 | 3 | 3 | Excreta management. The other half of water quality. |
| INF-10 | Flood Defence | Event | 🟢 | 2 | 3 | Front-loaded benefit; can displace risk downstream. |
| INF-11 | Communications | Annual | 🔵 | 2 | 3 | Early warning, market price information, remittance rails. |
| INF-12 | Infrastructure Condition | Annual | 🟡 | 4 | 5 | Everything decays. Maintenance is the least glamorous edge in the graph. |

## I.8 Environment Quality

| ID | System | Tier | Scope | In | Out | One-line role |
|---|---|---|---|---|---|---|
| ENV-01 | Air Quality | Medium | 🟢 | 4 | 4 | Loads slowly, clears slowly, bills the health system. |
| ENV-02 | Emissions | Fast | 🟢 | 3 | 3 | Combustion accounting: generators, waste burning, cooking fuel. |
| ENV-03 | Environmental Health Index | Annual | 🟡 | 6 | 2 | Composite; the "is this place getting better" number. |

## I.9 Governance — the player's hand

| ID | System | Tier | Scope | In | Out | One-line role |
|---|---|---|---|---|---|---|
| GOV-01 | Policy Portfolio | Event | 🟢 | 2 | 12 | Every active player action, as persistent state. |
| GOV-02 | Emergency Response | Event | 🟢 | 4 | 4 | Speed × reach × preparedness, at the moment it matters. |
| GOV-03 | Disaster Preparedness | Slow | 🟢 | 2 | 3 | Insurance. Invisible until it pays. |
| GOV-04 | Governance Capacity | Annual | 🟡 | 4 | 4 | How much of a budget actually becomes an outcome. |

**Catalogue total: 84 systems — 53 🟢 MVP, 29 🟡 V2, 2 🔵 Future** (verified against
`/data/systems-graph.json`, not counted by hand).

**MVP closure.** The MVP systems form a runnable subgraph on their own. Ten edges run from a V2
system into an MVP system with weight ≥ 0.6 — `ECO-03→ECO-02`, `AGR-03→ECN-04`, `ECN-09→ECN-05`,
`SOC-05→SOC-04`, `SOC-05→INF-06`, `SOC-06→ECN-03`, `INF-09→HYD-06`, and `INF-12→{INF-01, INF-04,
INF-06}`. Every one of these is an **additive refinement term that evaluates to zero while its
source system is unbuilt**, never a multiplicative gate. The CI check in §X.4 enumerates them and
fails if any becomes load-bearing.

---

# PART II — SYSTEM CHAPTERS: INPUTS, OUTPUTS, EQUATIONS

Each chapter is self-contained and implementable. `W` = weight, `Lag` in ticks(d)/seasons(s)/years(y).
All coefficients shown are **defaults**, and all live in the JSON file named in the chapter header.

---

## II.1 CLIMATE

### CLI-01 · Climate Drift
`Annual · 🟢 MVP · climate.json`

**Purpose.** The single scenario dial. It is the only place in the entire simulation where
"climate change" exists as a number, and it is deliberately exogenous: the player cannot move it.
This is the honesty constraint — a village cannot fix the global climate, it can only decide how
well it absorbs what the global climate does to it.

| Variable | Kind | Range | Notes |
|---|---|---|---|
| `drift` | Hidden | 0.0 – 1.0 | 0 = the Year-1 stable baseline; 1 = the report's "clouds passed without releasing water" |
| `driftRate` | Hidden | 0.0 – 0.08 /yr | How fast drift accumulates in this scenario |
| `variabilityGain` | Hidden | 1.0 – 2.5 | Multiplier on rainfall variance — *unreliability rises faster than the mean falls* |

**Inputs** — scenario file only. No in-simulation edge writes to `drift`. (🔵 FUTURE: regional
emissions coupling for the multi-settlement stages.)

**Outputs**

| To | Sign | W | Lag | Shape | Mechanism |
|---|---|---|---|---|---|
| `CLI-04.seasonalBaseline` | − | 0.80 | 0d | lin | Less rain per season, on average |
| `CLI-04.variance` | + | 0.60 | 0d | lin | And far less predictably — this is the part that actually breaks farming |
| `CLI-03.temperature` | + | 0.45 | 0d | lin | Baseline warming |
| `CLI-02.seasonOnsetJitter` | + | 0.70 | 0d | lin | The rains arrive at a different time, not just in a smaller amount |
| `AGR-08.knowledgeMismatch` | + | 0.65 | 1y | lin | A lifetime of planting dates stops matching reality |
| `ECO-08.pestPressure` | + | 0.30 | 1y | lin | Warmer, wetter shoulders extend pest seasons |

**Equation**
```
drift(y+1) = clamp(drift(y) + driftRate * scenarioCurve(y), 0, 1)
```

**Failure condition.** `drift > 0.75` sustained: the settlement cannot be made food-secure by
farming alone at any policy setting. This is intentional and must remain reachable — but see
§8.4, every failure state has a legible off-ramp (fisheries, trade, remittance, migration-as-adaptation).

**Recovery condition.** None from inside the simulation. Drift does not fall. The recoverable
quantity is *exposure*, never the hazard.

**Loops.** Origin of R1, R4, R9. Participates in no balancing loop — this is precisely what makes
it dangerous, and precisely why it is the scenario dial rather than a player lever.

---

### CLI-02 · Season Phase
`Fast · 🟢 MVP · climate.json`

**Purpose.** The metronome. Four seasons × 90 ticks. Every biological and economic rhythm in the
simulation is phase-locked to this, which is what makes the year feel like a year rather than a
counter.

| Variable | Kind | Range | Notes |
|---|---|---|---|
| `phase` | Visible | 0.0 – 4.0 | Continuous; integer part is the season index |
| `season` | Visible | wet / harvest / dry / planting | Rendered as the HUD season label |
| `onsetJitter` | Hidden | −20 … +20 ticks | Displacement of the rains' arrival, driven by drift |

**Outputs** — `CLI-04.seasonalBaseline` (+0.90, 0d, lin) · `CLI-03.tempBaseline` (+0.60, 0d, lin) ·
`AGR-01.growthStage` (+1.00, 0d, thr) · `AGR-07.labourDemand` (+0.85, 0d, thr) ·
`ECN-01.supplyPulse` (+0.70, 1s, thr) · `ECO-06.spawningWindow` (+0.50, 0d, thr) ·
`SOC-07.termCalendar` (+1.00, 0d, thr) · `HYD-03.baseflowSeasonality` (+0.55, 0d, sin) ·
`ECN-05.seasonalWage` (+0.50, 0d, thr)

**Equation**
```
phase = (tick / 90.0) mod 4
seasonalBaseline(phase) = table lookup, cubic-interpolated between the four season anchors
```

**Failure.** Onset jitter beyond ±25 ticks decouples planting from rain arrival entirely — the
mechanism behind Chain D3 (§III).
**Recovery.** Automatic each cycle; jitter itself only recovers if drift falls, which it does not.

---

### CLI-03 · Temperature
`Fast · 🟢 MVP · climate.json`

**Purpose.** Heat. Modelled as a daily value around a seasonal baseline. Reaches the player
through three quite different channels — evaporation, crop stress, and human health — which is
exactly why it earns its own system rather than being folded into weather.

| Variable | Kind | Range | Notes |
|---|---|---|---|
| `temperature` | Hidden | 18 – 44 °C | Not surfaced as a number; surfaced as heat shimmer and stress |
| `heatDays` | Hidden | count/season | Days above the crop-stress threshold — the variable that actually matters |

**Inputs** — `CLI-02.phase` (+0.60) · `CLI-01.drift` (+0.45) · `ECO-01.forestDensity` (−0.15, 1y, lin — local canopy cooling)

**Outputs**

| To | Sign | W | Lag | Shape | Mechanism |
|---|---|---|---|---|---|
| `CLI-07.PET` | + | 0.85 | 0d | exp | Evaporative demand rises non-linearly with temperature |
| `AGR-01.tempStress` | + | 0.55 | 5d | thr | Above the threshold, yield loss per heat-day |
| `SOC-04.health` | − | 0.25 | 5d | thr | Heat stress on elders and outdoor workers |
| `SOC-13.stress` | + | 0.20 | 5d | lin | Sleep, comfort, working conditions |
| `ECO-06.fishStock` | − | 0.20 | 1s | thr | Warm, low water reduces dissolved oxygen |
| `ECO-08.pestPressure` | + | 0.35 | 1s | lin | Faster pest generation times |
| `INF-01.powerDemand` | + | 0.15 | 0d | lin | 🟡 cooling load |
| `AGR-03.livestockCondition` | − | 0.25 | 1s | thr | Heat and water stress on animals |

**Equation**
```
temperature(d) = tempBaseline(phase) + drift*driftTempGain + gauss(0, dailyVariance)
heatDays += (temperature > cropHeatThreshold) ? 1 : 0
```

**Failure.** `heatDays > 30/season` → yield ceiling drops below subsistence regardless of water.
**Recovery.** Seasonal, automatic. Canopy restoration (ECO-01) buys back ~0.5–1.5 °C locally over years.

---

### CLI-04 · Rainfall
`Fast · 🟢 MVP · climate.json`

**Purpose.** The most consequential exogenous variable in Earth Engine. Everything downstream of
water — which is nearly everything — begins here. Note carefully: rainfall is *not* what crops
respond to. Crops respond to soil moisture (HYD-01). The gap between those two variables is where
soil quality, forest cover and irrigation earn their place in the graph.

| Variable | Kind | Range | Notes |
|---|---|---|---|
| `rainfall` | **Visible** | 0 – 120 mm/d | Rendered literally: this is the one raw physical number the world shows directly |
| `rain30` | Hidden | mm | Rolling 30-tick total — the memory that drought stress reads |
| `intensity` | Hidden | 0 – 1 | Fraction arriving as a high-intensity burst — decides infiltration vs. runoff |

**Inputs** — `CLI-02.seasonalBaseline` (+0.90) · `CLI-01.drift` (−0.80) · `CLI-01.variabilityGain` (+0.60 on variance) ·
🟡 `ECO-01.forestDensity` (+0.08, 5y, lin — *deliberately weak*; see the scope note below)

> **Scope note, binding.** In MVP, rainfall is **exogenous**. Planting trees does not make it rain.
> The V2 forest→rainfall term is capped at 0.08 and gated behind a config flag, because the causal
> arrow the project is honestly teaching runs the other way: **land use decides how much damage a
> given rainfall does**, not how much rain falls. A player who concludes "plant trees to control
> the weather" has learned something false, and the simulation would be the thing that taught it.

**Outputs**

| To | Sign | W | Lag | Shape | Mechanism |
|---|---|---|---|---|---|
| `HYD-01.soilMoisture` | + | 0.85 | 0d | sat | Infiltration, modulated by soil quality and intensity |
| `HYD-04.runoff` | + | 0.75 | 0d | thr | The fraction that does not infiltrate |
| `HYD-03.flowRate` | + | 0.70 | 1d | sat | Direct channel input |
| `HYD-02.recharge` | + | 0.45 | 1s | sat | Deep percolation — slow, and gated by soil |
| `HYD-05.harvestFill` | + | 0.40 | 0d | lin | Rainwater harvesting catch |
| `HYD-08.droughtStress` | − | 0.90 | 5d | lin | Relieves the accumulator |
| `HYD-06.waterQuality` | − | 0.25 | 1d | thr | First-flush washes waste into the channel |
| `ENV-01.airQuality` | + | 0.30 | 0d | lin | Rain scrubs particulates |
| `AGR-05.dryingLoss` | + | 0.20 | 0d | thr | Rain on drying racks and harvested grain |

**Equation**
```
rainfall(d) = seasonalBaseline(phase + onsetJitter)
            * (1 - drift * driftRainGain)
            * lognormal(1.0, baseVariance * variabilityGain)
intensity  = clamp(0.3 + 0.5*drift + noise, 0, 1)     // drift makes rain burstier, not just rarer
```

**Failure.** `rain30 < 0.35 × seasonalNormal` for two consecutive seasons → drought stress passes
Critical; see HYD-08.
**Recovery.** A single above-normal season restores soil moisture in ~15 ticks, river flow in
~30, groundwater in **3–8 seasons**, and soil quality in **years**. The staggered recovery times
are the entire lesson: *the fast variables lie to you about how recovered you are.*

**Loops.** B1 (rain→flow→draw), R1 (drift→rain→yield→income→...), B6 (rain→runoff→flood).

---

### CLI-05 · Wind
`Fast · 🟡 V2 · climate.json`

**Purpose.** MVP treats wind as pure ambience (Design Bible §5.2.5). V2 promotes it to a variable
because it closes three real edges: evaporative demand, turbine output, and the transport of dust
and smoke.

| Variable | Kind | Range |
|---|---|---|
| `windSpeed` | Hidden | 0 – 18 m/s |
| `harmattanIndex` | Hidden | 0 – 1 (dry-season dust-laden wind) |

**Inputs** — `CLI-02.phase` (+0.55) · `CLI-01.drift` (+0.20)
**Outputs** — `CLI-07.PET` (+0.35, 0d, lin) · `INF-01.windPower` (+0.80, 0d, cubic-capped) ·
`ENV-01.dustLoad` (+0.45, 0d, thr) · `ECO-03.windErosion` (+0.30, 1s, thr)

**Failure.** Harmattan above 0.7 with bare soil → measurable topsoil loss and a respiratory
health signal in the same season.
**Recovery.** Seasonal; erosion damage is not recovered, only slowed by cover.

---

### CLI-06 · Solar Irradiance
`Fast · 🟢 MVP · climate.json`

**Purpose.** Light. Two consumers: photovoltaic output and crop photosynthesis. Modelled simply as
a seasonal curve attenuated by cloud and dust.

**Inputs** — `CLI-02.phase` (+0.70) · `ENV-01.dustLoad` (−0.30, 0d, lin) · cloud cover (−0.55)
**Outputs** — `INF-01.solarOutput` (+0.90, 0d, lin) · `AGR-01.lightFactor` (+0.35, 5d, sat) ·
`CLI-07.PET` (+0.40, 0d, lin)

**Equation** `irradiance = clearSkyMax(phase) * (1 - 0.55*cloud) * (1 - 0.30*dust)`

**Failure.** Sustained dust load cutting PV output >25% during a dry season — the reason
solar-only power planning needs a margin.
**Recovery.** Immediate on rain (dust washout).

---

### CLI-07 · Potential Evapotranspiration (PET)
`Fast · 🟢 MVP · climate.json`

**Purpose.** The demand side of every water balance in the simulation. Rainfall is supply; PET is
what the atmosphere takes back. Every water system in Part II.2 subtracts this term. It is the
quiet reason a drought under a warming trend is worse than the same rainfall deficit was thirty
simulated years earlier.

**Inputs** — `CLI-03.temperature` (+0.85, exp) · `CLI-06.irradiance` (+0.40) · `CLI-05.windSpeed` (+0.35) ·
`ECO-01.forestDensity` (−0.20, 1y — canopy shading of the soil surface)

**Outputs** — `HYD-01.soilMoisture` (−0.70, 0d) · `HYD-03.flowRate` (−0.30, 0d, ∝ surface area) ·
`HYD-05.storedWater` (−0.25, 0d) · `HYD-02.groundwater` (−0.15, 1s)

**Equation** (deliberately Hargreaves-flavoured, not Penman-Monteith — see §9.1)
```
PET = k_pet * (temperature + 17.8) * sqrt(tempRange) * irradianceFactor * (1 + 0.02*windSpeed)
```

**Failure.** `PET > rainfall` sustained across a full wet season → net negative water balance;
groundwater begins its multi-year decline even if the river still looks normal. **This is the
single most important invisible failure in the simulation** — the world looks fine while the
savings account empties.
**Recovery.** Requires PET < rainfall for multiple seasons. Not achievable by policy; only
absorbable by storage and demand reduction.

---

## II.2 HYDROLOGY

### HYD-01 · Soil Moisture
`Medium (5d) · 🟢 MVP · climate.json + agriculture.json`

**Purpose.** The variable crops actually experience. Placing this system between rainfall and
yield — rather than wiring rain straight to harvest — is what makes soil quality, forest cover,
mulching and irrigation *mean* something instead of being decorations on a rainfall multiplier.

| Variable | Kind | Range | Notes |
|---|---|---|---|
| `moisture[tile]` | Hidden | 0.0 – 1.0 | Per farmland tile; surfaced only as crop colour and vigour |
| `fieldCapacity[tile]` | Hidden | 0.2 – 0.6 | Set by soil quality — better soil holds more water for longer |

**Inputs**

| From | Sign | W | Lag | Shape | Mechanism |
|---|---|---|---|---|---|
| `CLI-04.rainfall` | + | 0.85 | 0d | sat | Infiltration = rain × (1 − runoffFraction) |
| `AGR-02.irrigationDelivery` | + | 0.75 | 0d | sat | Applied water, independent of rain |
| `CLI-07.PET` | − | 0.70 | 0d | lin | Evaporative loss |
| `ECO-02.soilQuality` | + | 0.50 | 1s | lin | Organic matter raises both infiltration and holding capacity |
| `ECO-01.forestDensity` | + | 0.20 | 1y | lin | Windbreak and shade on adjacent tiles |
| `HYD-04.runoffFraction` | − | 0.60 | 0d | lin | Water that leaves before it soaks in |

**Outputs** — `AGR-01.moistureFactor` (+0.95, 5d, sat) · `HYD-02.recharge` (+0.40, 1s, thr — only
above field capacity) · `ECO-02.biologicalActivity` (+0.30, 1s) · `HYD-08.droughtStress` (−0.60, 5d) ·
`ECO-07.pollinatorForage` (+0.20, 1s)

**Equation**
```
moisture += (rain*(1-runoffFrac) + irrigation - PET*cropCoefficient) / rootZoneDepth
moisture  = clamp(moisture, 0, fieldCapacity(soilQuality))
recharge  = max(0, moisture - fieldCapacity) * percolationRate
```

**Failure.** `moisture < wiltingPoint` for >10 consecutive ticks during the growth stage →
irreversible yield loss for that cycle, *even if rain returns the following week*. This is the
single most important non-linearity in the agricultural chain, and the reason timing matters more
than totals.
**Recovery.** 10–20 ticks with normal rain. Fast — deceptively so.

---

### HYD-02 · Groundwater
`Slow (1s) · 🟢 MVP · climate.json`

**Purpose.** The settlement's savings account, and the most instructive system in the entire
simulation. It fills slowly, drains invisibly, and its depletion is only legible through
second-order symptoms — wells running low, river baseflow thinning in the dry season — long after
the withdrawal that caused it.

| Variable | Kind | Range | Notes |
|---|---|---|---|
| `groundwater` | Hidden | 0.0 – 1.0 (of capacity) | Never shown as a number. Surfaced through well fill and dry-season baseflow |
| `rechargeRate` | Hidden | — | Modulated by soil quality and land cover |

**Inputs** — `HYD-01.recharge` (+0.60, 1s, thr) · `CLI-04.rainfall` (+0.45, 1s, sat) ·
`AGR-02.irrigationDraw` (−0.80, 0d, lin) · `HYD-05.wellDraw` (−0.50, 0d, lin) · `CLI-07.PET` (−0.15, 1s)

**Outputs** — `HYD-03.baseflow` (+0.65, 1s, lin) · `HYD-05.wellYield` (+0.70, 0d, sat) ·
`HYD-01.capillaryRise` (+0.15, 1s) · `ECO-01.deepRootAccess` (+0.25, 1y) · `AGR-02.irrigationCeiling` (+0.85, 1s, thr)

**Equation**
```
groundwater += (recharge + baseflowInfiltration) - (irrigationDraw + wellDraw + phreaticET)
groundwater  = clamp(groundwater, 0, aquiferCapacity)
irrigationCeiling = smoothstep(0.15, 0.40, groundwater)   // wells fail progressively, not suddenly
```

**Failure.** `groundwater < 0.15` → irrigation capacity collapses and dry-season river baseflow
approaches zero. **The trap:** this is most likely to happen in the season *after* a successful
irrigation-buffered harvest, when the player believes the drought problem is solved. Chain R1 in
§VIII.
**Recovery.** 3–8 seasons of above-normal rainfall with irrigation held below recharge. The
slowest recoverable physical stock in the MVP. Nothing the player can buy accelerates it —
only reduced draw and improved infiltration (soil, forest) do.

**Loops.** R6 (irrigation → yield → confidence → more irrigation → depletion), B2 (depletion →
ceiling → forced reduction).

---

### HYD-03 · River Flow
`Fast · 🟢 MVP · climate.json`

**Purpose.** Highest out-degree node in the graph: eleven outgoing edges spanning water, food,
income, transport, health and recreation. When the Design Bible calls the river "the central
visual and simulation spine", this is the mechanical justification — no other single system
touches as many domains.

| Variable | Kind | Range | Notes |
|---|---|---|---|
| `flowRate` | **Visible** | 0 – 100 m³/s | Bound directly to rendered channel width, speed and colour — never a separate art value |
| `stage` | **Visible** | m | Water level; drives bank exposure and flood threshold |
| `baseflow` | Hidden | — | The groundwater-fed component that keeps the river alive between rains |

**Inputs** — `CLI-04.rainfall` (+0.70, 1d, sat) · `HYD-04.runoff` (+0.65, 0d, lin) ·
`HYD-02.baseflow` (+0.65, 1s, lin) · `AGR-02.irrigationDraw` (−0.55, 0d) ·
`HYD-05.householdDraw` (−0.20, 0d) · `CLI-07.PET` (−0.30, 0d, ∝ surface area)

**Outputs**

| To | Sign | W | Lag | Shape | Mechanism |
|---|---|---|---|---|---|
| `HYD-07.floodRisk` | + | 0.85 | 0d | thr | Above bankfull stage |
| `AGR-02.irrigationAvailability` | + | 0.70 | 0d | sat | Abstraction ceiling |
| `HYD-05.towerFill` | + | 0.60 | 0d | sat | Pumped supply |
| `ECO-06.fishHabitat` | + | 0.75 | 1s | sat | Wetted area and connectivity |
| `HYD-06.dilution` | + | 0.55 | 0d | sat | Low flow concentrates contaminants |
| `SOC-02.waterSecurity` | + | 0.50 | 0d | sat | Direct household abstraction |
| `INF-04.fordPassability` | + | 0.30 | 0d | thr | 🟡 low crossings |
| `HYD-09.sedimentTransport` | + | 0.45 | 0d | lin | Capacity to carry load |
| `SOC-14.cohesion` | + | 0.10 | 1y | lin | The river as shared place, not only resource |
| `ECN-02.marketActivity` | + | 0.15 | 1s | lin | Fish and water-borne trade |
| `AGR-03.livestockWater` | + | 0.40 | 0d | sat | Stock watering |

**Equation**
```
flow = baseflow(groundwater) + runoffRouted(rain, forest, soil) - draws - evapLoss
stage = ratingCurve(flow)          // simple power law, channel-geometry calibrated
```

**Failure.** Two independent axes, never one scale (Design Bible §4.3.1):
`flow < 0.15×normal` → Critical drought state; `stage > bankfull` → Flood state.
**A river can be in both within one season** — hardened drought soil sheds an intense storm
instead of absorbing it. Any implementation that collapses these into a single "river health"
slider is wrong and should be rejected in review.
**Recovery.** Flow: 20–40 ticks after rain returns. Ecological function (ECO-06): 2–6 seasons.
Reputation with the households that stopped relying on it: years.

---

### HYD-04 · Surface Runoff
`Fast · 🟢 MVP · climate.json`

**Purpose.** The difference between rain that helps and rain that destroys. Runoff is where forest
density, soil quality and land cover convert an identical rainfall event into either a good season
or a flood. It is a small system with enormous leverage.

**Inputs** — `CLI-04.rainfall` (+0.75, thr on intensity) · `ECO-01.forestDensity` (−0.60, 1y, lin) ·
`ECO-02.soilQuality` (−0.45, 1s, lin) · `ECO-09.impervious` (+0.35, 0d, lin)

**Outputs** — `HYD-03.flowRate` (+0.65, 0d) · `HYD-07.floodRisk` (+0.70, 0d, thr) ·
`ECO-03.erosion` (+0.60, 1s, lin) · `HYD-01.soilMoisture` (−0.60, 0d — water that leaves does not soak in)

**Equation**
```
runoffFraction = clamp(baseRunoff
                      + intensity*intensityGain
                      - forestDensity*k_forest
                      - soilQuality*k_soil, 0.05, 0.95)
runoff = rainfall * runoffFraction
```

**Failure.** `runoffFraction > 0.7` — the state in which normal rain produces flood peaks and
still leaves the fields dry. The signature of a deforested, degraded catchment, and the clearest
single demonstration that drought and flood are the same problem.
**Recovery.** Tracks forest and soil recovery: 2–10 years.

---

### HYD-05 · Stored Water
`Medium · 🟢 MVP · climate.json + actions.json`

**Purpose.** Tower, wells and harvesting tanks. The buffer that decides whether a dry fortnight is
an inconvenience or a household crisis. Its existence is why the settlement does not track the
river's daily volatility one-for-one — real infrastructure absorbs short shocks, and this system
is that absorption made explicit.

| Variable | Kind | Range | Notes |
|---|---|---|---|
| `fill` | **Visible** | 0.0 – 1.0 | Rendered as a physical gauge on the tower itself, per Design Bible §4.3.6 |
| `capacity` | Hidden | m³ | Raised by INF-03 investment |
| `quality` | Hidden | 0.0 – 1.0 | Separate from quantity — treatment affects this, not fill |

**Inputs** — `HYD-03.flowRate` (+0.60, sat) · `HYD-02.wellYield` (+0.70, sat) ·
`CLI-04.harvestFill` (+0.40) · `SOC-10.population` (−0.55, household draw) · `AGR-02.draw` (−0.35)

**Outputs** — `SOC-02.waterSecurity` (+0.90, 0d, sat) · `AGR-02.irrigationCeiling` (+0.30) ·
`INF-06.clinicFunction` (+0.35, thr — a clinic without water is a soft clinic, exactly as a clinic
without power is) · `HYD-06.deliveredQuality` (+0.60)

**Equation** `fill += inflow(river, wells, harvest) − draw(population, irrigation) − leakage(condition)`

**Failure.** `fill < 0.15` → water rationing: SOC-02 falls, collection time rises, and the burden
lands disproportionately on children and women — which in this simulation means SOC-07 school
attendance falls *through the water channel*, entirely independently of the food channel. Two
separate causal routes to the same empty classroom.
**Recovery.** Days to weeks if the river is healthy; otherwise gated by HYD-02 and therefore years.

---

### HYD-06 · Water Quality
`Medium · 🟢 MVP · climate.json`

**Purpose.** The quiet killer. Couples waste, sanitation, livestock and flooding to human health
along a path the player is not watching, which is exactly the §8.8 pattern the Design Bible names:
*the originating action and the visible cost never live on the same dashboard.*

**Inputs** — `INF-08.wasteOverflow` (−0.65, 1s, thr) · `INF-09.sanitationCoverage` (+0.70, 1s) ·
`HYD-03.dilution` (+0.55, 0d, sat) · `HYD-07.flood` (−0.75, 0d, event) ·
`AGR-03.livestockDensity` (−0.25, 1s) · `INF-03.treatment` (+0.80, 0d)

**Outputs** — `SOC-05.diseaseRisk` (−0.80, 5d, exp) · `SOC-04.health` (−0.45, 5d) ·
`ECO-06.fishStock` (−0.35, 1s) · `SOC-02.usableWater` (+0.50, 0d)

**Equation**
```
quality += recoveryRate*(1-quality)
         - wasteLoad*k1 - floodEvent*k2 - livestockLoad*k3
         + treatment*k4
quality  = clamp(quality, 0, 1);  effectiveConcentration = load / max(flow, floor)
```
Note the divisor: **the same waste load in a low-flow river is far more dangerous than in a full
one.** Drought and sanitation are not independent problems.

**Failure.** `quality < 0.35` + population density above threshold → outbreak probability crosses
into the event tier (SOC-05).
**Recovery.** 1–3 seasons after the load is removed. Faster than groundwater, slower than flow.

---

### HYD-07 · Flood
`Event · 🟢 MVP · climate.json`

**Purpose.** Fast, violent, and — critically — it leaves damage that outlives the water. A flood
that lasts four ticks costs two planting cycles. That asymmetry is the whole design intent.

**Inputs** — `HYD-03.stage` (+0.85, thr) · `HYD-04.runoff` (+0.70) ·
`ECO-01.forestDensity` (−0.55, 1y) · `INF-10.floodDefence` (−0.80, 0d, per-tile)

**Outputs**

| To | Sign | W | Lag | Shape | Mechanism |
|---|---|---|---|---|---|
| `AGR-01.floodDamage` | + | 0.90 | 0d | event | Current crop destroyed |
| `AGR-01.residualDamage` | + | 0.55 | **1s** | decay | *The next cycle is compromised too* |
| `ECO-02.soilQuality` | − | 0.45 | 1s | lin | Topsoil and nutrient stripping |
| `HYD-06.waterQuality` | − | 0.75 | 0d | event | Latrines, waste and livestock into the channel |
| `INF-04.bridgePassable` | − | 0.95 | 0d | thr | Binary; see Chain F2 |
| `INF-05.housingCondition` | − | 0.50 | 0d | event | |
| `SOC-13.stress` | + | 0.55 | 0d | event | |
| `GOV-02.responseLoad` | + | 0.80 | 0d | event | Demand spike exactly when access is worst |
| `ECN-10.emergencySpend` | + | 0.60 | 1s | event | Unbudgeted, crowds out planned investment |

**Equation**
```
floodRisk = P(stage > bankfull)
          * (1 - forestDensity*k_f)
          * (1 - defenceCoverage[tile])
if roll(floodRisk): tile.flooded = true for duration ∝ (stage - bankfull)
residualDamage[tile] = 0.55, decaying 0.25/season
```

**Failure.** Flood coinciding with bridge disruption and low preparedness — Chain F2. Response
capacity is lowest precisely when demand is highest.
**Recovery.** Water: days. Farmland: **two seasons**. Soil: years. Trust in the flood defence that
displaced the water onto someone else: longer.

---

### HYD-08 · Drought Stress
`Medium · 🟢 MVP · climate.json`

**Purpose.** An accumulator with decay — never a boolean. This is what allows the world to show
*degrees* of drought rather than snapping between two art states, and it is what makes the visual
language of §9.2 continuous rather than stepped.

**Inputs** — `CLI-04.rain30` (−0.90, 5d) · `HYD-01.soilMoisture` (−0.60) ·
`HYD-02.groundwater` (−0.40, 1s) · `HYD-03.flowRate` (−0.50)

**Outputs** — `AGR-01.yield` (−0.75, mult) · `ECO-02.soilQuality` (−0.35, 1s) ·
`ECO-01.forestStress` (+0.40, 1s) · `AGR-03.livestockCondition` (−0.55, 1s) ·
`SOC-13.stress` (+0.45, 1s) · **world render desaturation** (+1.00, 0d — bound directly, no
intermediate mood value) · `ECO-06.fishStock` (−0.30, 1s) · `SOC-09.migrationPressure` (+0.30, 1s)

**Equation**
```
droughtStress += (deficitRatio > 0) ? accumRate*deficitRatio : -decayRate
droughtStress  = clamp(droughtStress, 0, 1)
deficitRatio   = 1 - rain30 / seasonalNormal(phase)
```
`accumRate ≈ 3 × decayRate` — **stress builds three times faster than it clears.** This asymmetry
is deliberate and is the mechanical basis of the "regret" emotional beat.

**Failure.** `> 0.75` sustained → Critical: crop failure, livestock sale, migration pressure
crossing threshold across multiple households simultaneously.
**Recovery.** 1–2 seasons of normal rain for the stress value; far longer for what it did.

---

### HYD-09 · Sediment & Turbidity
`Medium · 🟡 V2 · climate.json`

**Purpose.** The delivery mechanism that turns erosion on the hillside into a problem in the water
— and eventually into a problem in the reservoir, the fishery and the treatment plant.

**Inputs** — `ECO-03.erosion` (+0.75, 1s) · `HYD-04.runoff` (+0.60) · `ECO-01.forestDensity` (−0.50, 1y)
**Outputs** — `HYD-06.waterQuality` (−0.40) · `ECO-06.spawningHabitat` (−0.45, 1s — silted gravel) ·
`HYD-05.treatmentCost` (+0.35) · `HYD-03.channelCapacity` (−0.25, 5y — aggradation raising flood risk)

**Failure.** Sustained high turbidity → fishery recruitment failure and a slow rise in flood
frequency at unchanged rainfall.
**Recovery.** 3–10 years. One of the slowest edges in the graph.

---

## II.3 ECOLOGY & LAND

### ECO-01 · Forest Density
`Slow · 🟢 MVP · ecology.json`

**Purpose.** The most asymmetric system in Earth Engine: cleared in a single tick, restored over
a decade. That asymmetry is ecologically honest and is the mechanical origin of the "regret"
emotional beat. Forest is also the graph's best example of an invisible service — it does almost
nothing the player can see, and four things they will feel.

| Variable | Kind | Range | Notes |
|---|---|---|---|
| `density[tile]` | **Visible** | 0.0 – 1.0 | Continuous, never stepped — rendered as canopy count and layering |
| `age[tile]` | Hidden | seasons | Young growth delivers partial services; mature delivers full |
| `protected[tile]` | Hidden | bool | Tree Protection action state |

**Inputs** — `GOV-01.plantForest` (+0.90, 1s, ramp) · `GOV-01.treeProtection` (+0.55, 0d, suppresses clearing) ·
`ECO-09.ambientClearing` (−0.60, 1s, lin) · `HYD-08.droughtStress` (−0.40, 1s) · `SOC-16.awareness` (+0.20, 1y)

**Outputs**

| To | Sign | W | Lag | Shape | Mechanism |
|---|---|---|---|---|---|
| `HYD-04.runoffFraction` | − | 0.60 | 1y | lin | Canopy interception + root macropores |
| `HYD-07.floodRisk` | − | 0.55 | 1y | lin | Compound of the above |
| `ECO-03.erosion` | − | 0.70 | 1s | lin | Root binding |
| `ECO-05.biodiversity` | + | 0.65 | 1y | sat | Habitat area and structure |
| `ECO-02.soilQuality` | + | 0.35 | 5y | lin | Litter and organic matter |
| `CLI-03.localTemperature` | − | 0.15 | 1y | lin | Shading and transpiration |
| `ECN-03.fuelwoodIncome` | + | 0.25 | 1s | lin | The reason clearing happens at all |
| `ECO-10.carbonStock` | + | 0.80 | 1y | lin | 🔵 |

**Equation**
```
density += regrowthRate*plantingEffort*(1-density)                 // logistic, slow
         + ambientRegrowth*(1-density)*moistureFactor
         - clearingRate*max(0, ambientPressure - protection)
         - droughtStress*dieback
```
`regrowthRate ≈ 0.04/season`, `clearingRate ≈ 0.35/season` — **roughly nine times faster to destroy
than to rebuild.** Do not tune this to parity; the asymmetry is the lesson.

**Failure.** `density < 0.25` catchment-wide → runoff fraction crosses 0.7, and the settlement now
floods on rainfall it used to absorb. The player will usually not connect the two events, which is
why the Atlas exists.
**Recovery.** 8–15 simulated years to mature canopy. Partial services (erosion, runoff) return at
~40% density, i.e. within 3–5 years — the fastest-paying resilience investment available and the
only MVP chain whose payoff fits inside a demo session (Chain R2).

**Loops.** B4 (forest→flood→damage), R7 (clearing→income→more clearing), R8 (loss→erosion→lower
regrowth→further loss).

---

### ECO-02 · Soil Quality
`Slow · 🟢 MVP · agriculture.json`

**Purpose.** Fertility as a stock. Quietly decides what next decade's yields can be, while every
visible signal in the game is about this season.

**Inputs** — `HYD-08.droughtStress` (−0.35, 1s) · `HYD-07.floodDamage` (−0.45, 1s) ·
`ECO-03.erosion` (−0.65, 1s) · `ECO-04.nutrientReturn` (+0.55, 1y) ·
`ECO-01.litterfall` (+0.35, 5y) · `AGR-01.croppingIntensity` (−0.40, 1y, 🟡)

**Outputs** — `AGR-01.soilFactor` (+0.70, 5d, mult) · `HYD-01.fieldCapacity` (+0.50, 1s) ·
`HYD-04.runoffFraction` (−0.45, 1s) · `HYD-02.rechargeEfficiency` (+0.35, 1s)

**Equation** `soilQuality += (nutrientReturn + litter)*k_in − (erosion + drought + flood + intensity)*k_out`,
with `recoveryRate ≈ 0.02/season` under fallow.

**Failure.** `< 0.30` → yields cannot exceed ~55% of potential at any rainfall or irrigation level.
The land itself has become the constraint, and no water action fixes it.
**Recovery.** 5–15 years natural; 3–6 with active fallow and residue return (🟡).

---

### ECO-03 · Soil Erosion
`Medium · 🟡 V2 · ecology.json`

**Purpose.** The mechanism that makes land degradation permanent rather than cyclical.

**Inputs** — `HYD-04.runoff` (+0.60) · `ECO-01.forestDensity` (−0.70) · `ECO-09.bareGroundFraction` (+0.55) · `CLI-05.windErosion` (+0.30, 🟡)
**Outputs** — `ECO-02.soilQuality` (−0.65, 1s) · `HYD-09.sedimentLoad` (+0.75, 0d) · `AGR-01.rootingDepth` (−0.30, 5y)

**Equation** `erosion = rainErosivity × slope × (1 − cover) × (1 − soilStructure)` — a deliberately
simplified USLE skeleton (see §9.1).

**Failure.** Sustained loss > regeneration → an irreversible downward yield ceiling.
**Recovery.** Effectively none on a playthrough timescale. **This is the only genuinely permanent
loss in the simulation**, and it should be treated as such in tuning.

---

### ECO-04 · Nutrient Cycling
`Slow · 🟡 V2 · ecology.json`

**Purpose.** Closes the loop between livestock, crop residue, fallow and fertility — the reason a
mixed farming system out-performs a monocrop one over decades.

**Inputs** — `AGR-03.manure` (+0.60, 1s) · `AGR-01.residueReturn` (+0.45, 1s) ·
`ECO-09.fallowFraction` (+0.50, 1y) · `HYD-04.leaching` (−0.35, 1s)
**Outputs** — `ECO-02.soilQuality` (+0.55, 1y) · `AGR-01.nutrientFactor` (+0.40, 1s) · `HYD-06.eutrophication` (−0.20, 1s)

**Failure.** Continuous cropping with residue removal and no livestock → measurable fertility decline within 4 years.
**Recovery.** 2–5 years once returns resume.

---

### ECO-05 · Biodiversity
`Annual · 🟡 V2 · ecology.json`

**Purpose.** Aggregate ecosystem integrity. Not decoration: it gates pollination and fish
recruitment, and it is the index the Climate Dashboard uses for "is this landscape getting better".

**Inputs** — `ECO-01.forestDensity` (+0.65, sat) · `HYD-06.waterQuality` (+0.40) ·
`ECO-09.habitatFragmentation` (−0.50) · `ECO-08.pesticideUse` (−0.35, 🔵) · `HYD-03.flowRegime` (+0.35)
**Outputs** — `ECO-07.pollinators` (+0.70, 1y) · `ECO-06.recruitment` (+0.45, 1y) ·
`ECO-08.naturalPestControl` (+0.40, 1y) · `ENV-03.index` (+0.60, 1y)

**Failure.** `< 0.30` → pollination and natural pest control both fail; yields drop 10–20% at
unchanged water and soil, which reads to the player as an unexplained bad year. The Atlas is how
they explain it.
**Recovery.** 5–20 years, and only if habitat returns first.

---

### ECO-06 · Fish Stock
`Slow · 🟢 MVP · ecology.json`

**Purpose.** A renewable resource with a collapse threshold, and the simulation's cleanest
tragedy-of-the-commons: every household's individually rational response to a bad harvest is to
fish harder, and the aggregate of those rational choices removes the buffer they were reaching for.

| Variable | Kind | Range | Notes |
|---|---|---|---|
| `stock` | Hidden | 0.0 – 1.0 of carrying capacity | Surfaced as catch-per-trip, visible boat activity, and drying-rack fullness |
| `recruitment` | Hidden | — | New year-class strength; set a season before it is felt |

**Inputs** — `HYD-03.flowRate` (+0.75, 1s, sat) · `HYD-06.waterQuality` (+0.45, 1s) ·
`AGR-04.fishingEffort` (−0.80, 0d, lin) · `ECO-05.biodiversity` (+0.45, 1y) ·
`HYD-09.siltation` (−0.45, 1s) · `CLI-03.temperature` (−0.20, 1s, thr)

**Outputs** — `AGR-04.catchPerEffort` (+0.90, 0d, sat) · `SOC-01.foodSecurity` (+0.35, 1s — protein
independent of rainfall) · `ECN-03.fisherIncome` (+0.70, 1s)

**Equation** (logistic with harvest — the classic surplus-production form, and the right level of
fidelity per §9.1)
```
stock += r * stock * (1 - stock/K_effective) - catch
K_effective = K0 * habitatFactor(flow, quality, silt)
catch       = q * effort * stock                        // catch-per-effort falls as stock falls
```

**Failure.** `stock < 0.20` → recruitment collapse; catch-per-effort falls faster than effort can
compensate, and the fishery stops being a drought buffer at exactly the moment it is most needed.
**This is the single most important interaction between the fishing reach and the drought chain**,
and it is why the fishery is not simply "a second income".
**Recovery.** 4–10 seasons *if effort falls*, which requires an alternative income to exist. If it
does not, the stock does not recover — the households cannot afford to let it.

**Loops.** R11 (harvest failure → effort ↑ → stock ↓ → income ↓ → effort ↑), B9 (CPUE decline
eventually forces effort down).

---

### ECO-07 · Pollinators
`Slow · 🟡 V2 · ecology.json`
**Inputs** — `ECO-05.biodiversity` (+0.70) · `ECO-01.forestDensity` (+0.40, edge habitat) · `HYD-01.forageMoisture` (+0.20)
**Outputs** — `AGR-01.pollinationFactor` (+0.35, 1s, sat on pollinator-dependent crops) · `ECO-05.feedback` (+0.20)
**Failure.** `< 0.35` → a 10–25% yield penalty confined to specific crops; invisible without the Atlas.
**Recovery.** 2–5 years, tracking habitat.

---

### ECO-08 · Pest & Crop Disease Pressure
`Medium · 🟡 V2 · agriculture.json`
**Inputs** — `CLI-03.temperature` (+0.35, 1s) · `CLI-01.drift` (+0.30, 1y) ·
`ECO-05.naturalControl` (−0.40, 1y) · `AGR-01.monocropFraction` (+0.35, 1y)
**Outputs** — `AGR-01.pestLoss` (+0.45, 5d, mult) · `AGR-05.storageLoss` (+0.30, 1s)
**Failure.** Outbreak year: 30–50% yield loss decoupled from water entirely — the player's water
investments appear to have failed, and they have not.
**Recovery.** 1–2 seasons; faster with diversity, slower with monocrop.

---

### ECO-09 · Land Cover Allocation
`Event · 🟢 MVP · ecology.json`

**Purpose.** The zero-sum ledger. Every tile is exactly one of forest / cropland / fallow /
settlement / water / bare. Making this explicit is what forces the opportunity cost of Plant
Forest to be real rather than rhetorical.

**Inputs** — `GOV-01.landActions` (event) · `SOC-10.population` (+0.45, 1y → settlement expansion) ·
`ECN-03.income` (−0.30, 1y → prosperity reduces clearing pressure)
**Outputs** — `ECO-01.availableArea` (±1.00) · `AGR-01.croppedArea` (±1.00) ·
`HYD-04.imperviousFraction` (+0.35) · `ECO-05.fragmentation` (+0.40, 1y) · `ECO-03.bareGround` (+0.50)

**Equation** `Σ coverFraction = 1.0` enforced every allocation event; conversions are logged as
transactions so the Cause-and-Effect map can attribute them.

**Failure.** Cropland expansion into the riparian buffer — short-term yield gain, permanent flood
and water-quality penalty. The most common self-inflicted wound in playtesting.
**Recovery.** Reversible in principle; ECO-01's regrowth clock applies.

---

### ECO-10 · Carbon Stock
`Annual · 🔵 FUTURE · ecology.json`
Present so the National/Global roadmap stages are not architecturally foreclosed.
**Inputs** — `ECO-01.forestDensity` (+0.80) · `ECO-02.soilCarbon` (+0.40) · `ENV-02.emissions` (−0.30)
**Outputs** — 🔵 regional climate coupling; `ENV-03.index` (+0.30). No MVP consumer.

---

## II.4 AGRICULTURE & FISHERIES

### AGR-01 · Crop Growth & Yield
`Medium, resolving at harvest · 🟢 MVP · agriculture.json`

**Purpose.** The transducer. This is where environment becomes money, and it has the highest
in-degree of any system in the graph (nine inputs) because *every* environmental variable
eventually expresses itself here.

| Variable | Kind | Range | Notes |
|---|---|---|---|
| `growthStage[tile]` | **Visible** | fallow→planted→growing→mature→harvested | Rendered as actual crop geometry and colour |
| `stressState[tile]` | **Visible** | healthy / stressed / failed | Parallel overlay, applies at any stage |
| `yield[tile]` | **Visible** | 0 – potential | Resolves at the harvest event |

**Inputs** — `HYD-01.moistureFactor` (+0.95, mult) · `ECO-02.soilFactor` (+0.70, mult) ·
`CLI-03.tempStress` (−0.55, mult) · `HYD-07.floodDamage` (−0.90, mult) ·
`HYD-08.droughtStress` (−0.75, mult) · `ECO-08.pestLoss` (−0.45, mult, 🟡) ·
`ECO-07.pollination` (+0.35, mult, 🟡) · `AGR-08.plantingTiming` (+0.50, mult, 🟡) ·
`AGR-07.labourAvailable` (+0.60, sat)

**Outputs** — `AGR-06.foodSupply` (+1.00, 1s) · `ECN-03.farmIncome` (+0.90, 1s) ·
`SOC-01.householdFoodStock` (+0.85, 1s) · `AGR-07.harvestLabourDemand` (+0.60, 0d) ·
`ECO-04.residue` (+0.40, 1s) · `AGR-05.storedVolume` (+0.80, 0d)

**Equation** — **multiplicative, never additive.** This is a design decision with teeth:
```
yield = potentialYield
      * moistureFactor * soilFactor * (1 - tempStress) * (1 - floodDamage)
      * (1 - droughtStress) * (1 - pestLoss) * pollinationFactor * timingFactor
      * labourFactor
```
Each factor is clamped to [0,1]. Because they multiply, **any single severe stressor craters the
harvest regardless of how good everything else is** — which is the honest agronomic result and
prevents the "average of my inputs was fine" failure mode that additive models produce.

**Failure.** `yield < 0.35 × potential` → household food stock insufficient for the season;
triggers the SOC-08 child-labour branch and begins SOC-09 migration-pressure accumulation.
**Recovery.** One good season restores income; **two to three restore savings**; soil-mediated
losses take years. The gap between "income recovered" and "household actually recovered" is
where the player learns what resilience means.

---

### AGR-02 · Irrigation
`Medium · 🟢 MVP · agriculture.json + actions.json`

**Purpose.** The double-edge, and the clearest "surprise" beat available: a genuinely effective
resilience action that, applied without moderation, manufactures the water crisis it was bought to
prevent.

**Inputs** — `GOV-01.irrigationBuilt` (+1.00, event) · `HYD-03.availability` (+0.70, sat) ·
`HYD-02.groundwaterCeiling` (+0.85, thr) · `INF-01.pumpPower` (+0.60, thr)
**Outputs** — `HYD-01.soilMoisture` (+0.75, 0d, sat) · `HYD-02.groundwater` (−0.80, 0d) ·
`HYD-03.flowRate` (−0.55, 0d) · `AGR-01.moistureFactor` (+0.70, 5d)

**Equation**
```
delivery = min(demand(cropStage, moistureDeficit),
               availability(river, groundwater, storage),
               pumpCapacity(power))
draw     = delivery / efficiency          // efficiency 0.45 flood … 0.85 drip (🟡)
```

**Failure.** `Σ draw > recharge` sustained across two seasons → HYD-02 collapse. The failure is
**deferred and displaced**: it lands on the well and the dry-season river, one to two years after
the decision, on a different dashboard.
**Recovery.** Immediate on cessation for flow; years for the aquifer.
**Loops.** R6 (success → confidence → expansion → depletion), B2 (depletion → ceiling → forced cut).

---

### AGR-03 · Livestock
`Slow · 🟡 V2 · agriculture.json`

**Purpose.** The household's second buffer, moving on a different clock from crops — which is
precisely why it matters. A household with animals experiences the same drought differently.

**Inputs** — `AGR-06.feedAvailability` (+0.60, 1s) · `HYD-03.stockWater` (+0.40) ·
`HYD-08.droughtStress` (−0.55, 1s) · `ECN-04.assetSale` (−0.70, event) · `SOC-04.veterinaryAccess` (+0.25, 🔵)
**Outputs** — `ECN-04.assetValue` (+0.65, 1s) · `SOC-03.proteinNutrition` (+0.35, 1s) ·
`ECO-04.manure` (+0.60, 1s) · `ECO-09.grazingPressure` (+0.45, 1y) · `HYD-06.waterQuality` (−0.25, 1s)

**Failure.** Distress sale: the household converts its multi-year buffer into one season of food,
and its recovery capacity for the *next* shock is gone. The most under-appreciated mechanism in
the poverty-trap loop (R3).
**Recovery.** 2–4 years to rebuild a herd — assuming no further shock, which is exactly what a
drifting climate does not offer.

---

### AGR-04 · Fisheries Harvest
`Medium · 🟢 MVP · agriculture.json`

**Purpose.** Effort applied to a renewable stock. Provides a livelihood whose exposure channel is
*flow and water quality* rather than *rainfall and soil* — genuine portfolio diversification for
the settlement, and the reason the fishing reach is not decoration.

| Variable | Kind | Range | Notes |
|---|---|---|---|
| `effort` | **Visible** | boats × trips | Rendered as canoes on the water and activity at the landing |
| `catch` | **Visible** | kg/season | Rendered as drying-rack fullness and market fish stalls |
| `CPUE` | Hidden | kg/trip | The number fishers actually experience — and the early-warning signal |

**Inputs** — `ECO-06.stock` (+0.90, sat) · `HYD-03.flowRate` (+0.45, thr — passable water) ·
`AGR-01.yieldFailure` (+0.55, 1s — **crop failure pushes effort into the river**) ·
`ECN-03.alternativeIncome` (−0.40, 1s) · `INF-04.marketAccess` (+0.30)

**Outputs** — `ECO-06.stock` (−0.80, 0d) · `ECN-03.fisherIncome` (+0.75, 1s) ·
`SOC-01.proteinSupply` (+0.45, 1s) · `ECN-02.marketSupply` (+0.35, 1s)

**Equation** `catch = q · effort · stock`, with `effort = baseEffort × (1 + distressGain × cropFailure)`

**Failure.** Effort rising while CPUE falls — the classic pre-collapse signature. Visible in-world
as *more canoes returning with less fish*, which is a legible signal the player can read without
any UI at all.
**Recovery.** Requires effort reduction, which requires an alternative income. Fisheries recovery
is therefore **an economic problem wearing an ecological costume** — one of the sharpest lessons
the graph can deliver.

---

### AGR-05 · Post-Harvest Loss & Storage
`Event · 🟡 V2 · agriculture.json`

**Purpose.** Between a good harvest and a good year sits storage. A 30% post-harvest loss and a
30% yield loss are the same number to a household, and only one of them is cheap to fix.

**Inputs** — `INF-12.storageCondition` (−0.60) · `CLI-04.rainAtHarvest` (+0.35, thr) ·
`ECO-08.pests` (+0.30) · `INF-01.coldChainPower` (−0.40, 🔵)
**Outputs** — `AGR-06.netFoodSupply` (−0.55, 0d) · `ECN-03.marketableSurplus` (−0.50, 1s)

**Failure.** Loss > 35% converts a technically adequate harvest into a hungry season.
**Recovery.** Immediate on storage investment — the highest benefit-per-cedi action in the
catalogue, and deliberately unglamorous.

---

### AGR-06 · Food Supply
`Slow · 🟢 MVP · agriculture.json + economy.json`

**Purpose.** Aggregate calories reaching household stores and the market. The junction where
farming, fishing, livestock and trade become one number that the price system reads.

**Inputs** — `AGR-01.yield` (+1.00) · `AGR-04.catch` (+0.35) · `AGR-03.livestockProducts` (+0.25) ·
`ECN-07.imports` (+0.45, 1s) · `AGR-05.losses` (−0.55)
**Outputs** — `ECN-01.supply` (+1.00, 0s, inverse-price) · `SOC-01.foodSecurity` (+0.80) · `ECN-02.marketStock` (+0.70)

**Failure.** Supply < 60% of population requirement → price spike (ECN-01), rationing behaviour,
and the beginning of Chain D1's human phase.
**Recovery.** One good season for supply; **longer for prices**, which are sticky downward — a
detail worth implementing, because it is the reason a recovered harvest does not immediately feel
like a recovered household.

---

### AGR-07 · Farm Labour Demand
`Slow · 🟢 MVP · economy.json`

**Purpose.** Seasonality of work. Its peak coincides with the school term, which is where child
labour stops being a moral failing and becomes an arithmetic problem — exactly the framing the
Design Bible requires.

**Inputs** — `CLI-02.phase` (+0.85, thr at planting and harvest) · `AGR-01.croppedArea` (+0.70) ·
`SOC-10.workingAgePopulation` (−0.40, supply side) · `AGR-02.mechanisation` (−0.35, 🔵)
**Outputs** — `ECN-05.employment` (+0.70, 0s) · `ECN-05.wageRate` (+0.45, thr — scarcity raises wages) ·
`SOC-08.childLabourDemand` (+0.55, 1s) · `AGR-01.labourFactor` (+0.60)

**Failure.** Demand exceeding available adult labour at harvest → either yield loss in the field
or children pulled from school. The household chooses. **The player never does** — this is the
§1.4 boundary, enforced in code.
**Recovery.** Seasonal.

---

### AGR-08 · Agricultural Knowledge
`Annual · 🟡 V2 · agriculture.json`

**Purpose.** Encodes the report's own line — *"a lifetime of knowledge stopped matching reality."*
A Farmer's accumulated planting calendar is an asset under a stable climate and a liability under
a drifting one.

**Inputs** — `CLI-01.drift` (+0.65, 1y → mismatch) · `SOC-06.education` (−0.30 on mismatch, faster
adaptation) · `SOC-16.climateAwareness` (−0.40 on mismatch) · `GOV-01.extensionService` (−0.50, 🟡)
**Outputs** — `AGR-01.timingFactor` (−0.50, 1s) · `AGR-01.cropChoiceFit` (−0.30, 1y)

**Equation** `mismatch = |plantingDate_learned − optimalDate(drift)| / tolerance`, decaying at
`adaptRate` per season of exposure — a mechanical representation of learning without a full memory
system.

**Failure.** Mismatch > 0.6 → yield penalty arrives **before** absolute rainfall totals fall
enough to explain it. The farmer is doing everything right and it is not working, which is the
most emotionally accurate thing this simulation can reproduce.
**Recovery.** 2–4 seasons of exposure, or one season with extension support.

---

## II.5 ECONOMY

### ECN-01 · Food Price
`Slow · 🟢 MVP · economy.json`

**Purpose.** The pivot of the entire simulation. Almost every environmental shock reaches human
beings through this one node. A drought does not make a child hungry — a drought makes food
expensive, and *that* makes a child hungry. Keeping the mechanism explicit is what makes the
lesson transferable to real policy.

| Variable | Kind | Range | Notes |
|---|---|---|---|
| `price` | **Visible** | ₵/kg | Economy Dashboard + market stall stock density |
| `priceIndex` | Visible | rel. Year 1 | The number households actually feel |
| `stickiness` | Hidden | 0.0 – 1.0 | Prices rise fast and fall slowly — implement this, it matters |

**Inputs** — `AGR-06.foodSupply` (−0.90, inverse) · `SOC-10.population` (+0.55, demand) ·
`ECN-07.imports` (−0.50, 1s) · `ECN-06.inflation` (+0.40, 1y) · `ECN-08.speculativeHoarding` (+0.30, 🟡)

**Outputs**

| To | Sign | W | Lag | Shape | Mechanism |
|---|---|---|---|---|---|
| `SOC-01.foodSecurity` | − | 0.85 | 1s | lin | Purchasing power |
| `ECN-03.farmerRevenue` | + | 0.45 | 1s | lin | **Sellers gain what buyers lose** — the same event, opposite signs |
| `ECN-03.elderRealIncome` | − | 0.80 | 1s | lin | Fixed income buys less: Nana's exposure channel |
| `SOC-08.childLabour` | + | 0.50 | 1s | thr | Households substitute labour for purchases |
| `SOC-09.migrationPressure` | + | 0.45 | 1s | lin | |
| `ECN-02.marketActivity` | − | 0.35 | 1s | sat | Volume falls even as value rises |
| `SOC-15.crime` | + | 0.25 | 1s | thr | 🟡 Deprivation-linked |

**Equation**
```
targetPrice = basePrice * (demand / max(supply, supplyFloor))^elasticity
price      += (targetPrice > price) ? fastAdjust*(target-price)
                                    : slowAdjust*(target-price)     // fastAdjust ≈ 3×slowAdjust
```
The `supplyFloor` term prevents divide-by-zero during total harvest failure and yields a capped,
large-but-finite spike — numerically safe and narratively correct.

**Failure.** `priceIndex > 2.5` sustained one season → food insecurity across all non-farming
households simultaneously. Note the distributional detail: **the Farmer's revenue may rise in the
same event that ruins the Elder and the Vendor.** A single village-wide "prosperity" number would
hide this, and hiding it would be a pedagogical failure.
**Recovery.** Prices lag supply by 1–2 seasons on the way down. Deliberate.

---

### ECN-02 · Market Activity
`Slow · 🟢 MVP · economy.json`
**Purpose.** The visible economic pulse — stall occupancy, crowd density, goods on tables. A pure
read-out of upstream systems, rendered rather than narrated.
**Inputs** — `AGR-06.supply` (+0.70) · `ECN-03.aggregateIncome` (+0.75) · `ECN-01.price` (−0.35, sat) ·
`INF-04.roadAccess` (+0.45, thr) · `SOC-10.population` (+0.50)
**Outputs** — `ECN-03.vendorIncome` (+0.85, 1s) · `ECN-05.employment` (+0.40) ·
`ECN-11.taxBase` (+0.45, 1s) · `SOC-14.cohesion` (+0.25, 1y — the market as social institution) ·
`ECN-07.tradeVolume` (+0.35)
**Failure.** Activity < 0.4 of baseline → vendor households fall below income threshold and begin
accumulating migration pressure, which thins the market further (loop R5).
**Recovery.** Tracks income, 1–3 seasons.

---

### ECN-03 · Household Income
`Slow · 🟢 MVP · economy.json + citizens.json`

**Purpose.** The most heterogeneous variable in the simulation, and deliberately so. It is
computed **per household, by archetype**, because the entire point of the citizen model is that
the same shock produces different outcomes for different exposure channels.

| Archetype | Primary source | Exposure | Lag to shock |
|---|---|---|---|
| Farmer | `AGR-01.yield × ECN-01.price` | Direct, violent | Same season |
| Fisher | `AGR-04.catch × price` | Flow & water quality | 1 season |
| Vendor | `ECN-02.marketActivity` | Aggregate, smoothed | 1–2 seasons |
| Teacher | `ECN-10.publicWage` | Budget-linked, insulated | 2–4 seasons |
| Health worker | `ECN-10.publicWage` | Budget-linked | 2–4 seasons |
| Construction | `ECN-10.capitalSpend + ECN-09` | Investment-cycle | 1–2 seasons |
| Elder | fixed pension | **Price, not income** | Immediate, invisible |
| Unemployed | casual + transfers | Everything, worst | Immediate |
| Child | none (dependent) | Household, indirect | Via household |

**Inputs** — `AGR-01.yield` (+0.90) · `AGR-04.catch` (+0.75) · `ECN-01.price` (± by role) ·
`ECN-02.market` (+0.85) · `ECN-05.employment` (+0.70) · `ECN-10.publicWage` (+0.80) ·
`GOV-01.subsidy` (+0.60, direct) · `ECN-13.remittances` (+0.50, 1y)

**Outputs** — `SOC-01.foodSecurity` (+0.85) · `ECN-04.savings` (+0.60) · `ECN-11.tax` (+0.55) ·
`SOC-09.migrationPressure` (−0.75) · `SOC-07.schoolAffordability` (+0.55) ·
`SOC-04.healthAccess` (+0.45) · `ECN-02.demand` (+0.70) · `SOC-13.stress` (−0.60)

**Equation** `income = Σ(sourceYield × price × participation) + transfers − debtService`,
evaluated at the household level each season boundary.

**Failure.** Below the subsistence threshold for two consecutive seasons → the household enters
the SOC-08/SOC-09 branch: child labour first, asset sale second, migration third. The order is
fixed and observable, and it is the same order real households use.
**Recovery.** Income recovers in one good season. **Savings take three. Assets take five.** A
household is not recovered when its income is.

---

### ECN-04 · Savings & Assets
`Slow · 🟢 MVP · economy.json`
**Purpose.** The depth of the shock absorber. Determines who survives a bad year intact and who
is permanently reset by it — the single best predictor of divergent outcomes under an identical
shock.
**Inputs** — `ECN-03.income` (+0.60, surplus only) · `SOC-01.foodDeficit` (−0.70, drawdown) ·
`AGR-03.livestockValue` (+0.65) · `ECN-08.debt` (−0.55) · `ECN-06.inflation` (−0.35, erosion)
**Outputs** — `SOC-09.migrationPressure` (−0.70, buffer) · `SOC-04.healthAccess` (+0.45) ·
`AGR-02.investmentCapacity` (+0.50) · `SOC-13.stress` (−0.55) · `ECN-08.creditworthiness` (+0.60)
**Failure.** Savings = 0 with a food deficit → distress asset sale, then debt. Once assets are
gone, the *next* shock of the same size produces a far worse outcome. **This is the poverty trap,
and it needs no special-case code — it falls out of the graph.**
**Recovery.** 3–8 seasons of surplus. The slowest household-level recovery in the model.

---

### ECN-05 · Employment
`Slow · 🟢 MVP · economy.json`
**Inputs** — `AGR-07.farmLabourDemand` (+0.70) · `ECN-09.enterpriseStock` (+0.55) ·
`ECN-10.publicJobs` (+0.45) · `SOC-10.workingAge` (supply) · `INF-12.constructionPipeline` (+0.40)
**Outputs** — `ECN-03.income` (+0.70) · `SOC-13.stress` (−0.45) · `SOC-09.migration` (−0.55) ·
`SOC-15.crime` (−0.35, 🟡)
**Failure.** Employment < 0.55 of working-age population sustained → migration threshold crossings
cluster, and the labour that remains is insufficient for the next planting (loop R2).
**Recovery.** 1–3 seasons; faster if enterprise stock survived.

---

### ECN-06 · Inflation
`Annual · 🟡 V2 · economy.json`
**Inputs** — `ECN-01.foodPrice` (+0.50) · `ECN-07.importCost` (+0.40) · `ECN-12.aidInflow` (+0.20)
**Outputs** — `ECN-03.realIncome` (−0.60) · `ECN-04.savingsErosion` (−0.35) · `ECN-10.realBudget` (−0.45)
**Failure.** Fixed-income households (Elder, public wage) lose purchasing power without any
observable event occurring — the most invisible harm in the model.
**Recovery.** Multi-year; asymmetric, like all price effects.

---

### ECN-07 · External Trade
`Slow · 🟡 V2 · economy.json`
**Purpose.** The valve that blunts price spikes — **but only if the road is open.** This makes
INF-04 an economic system as much as an infrastructure one.
**Inputs** — `INF-04.roadCondition` (+0.70, thr) · `ECN-01.priceDifferential` (+0.60) ·
`ECN-10.tradePolicy` (±0.40) · `AGR-06.surplus` (+0.50, export side)
**Outputs** — `ECN-01.price` (−0.50, 1s) · `AGR-06.supply` (+0.45) · `ECN-03.traderIncome` (+0.40)
**Failure.** Road impassable during a price spike → imports cannot arrive in the exact window they
are needed. Chain F2 is not only a humanitarian failure, it is an economic one.
**Recovery.** Immediate on road restoration.

---

### ECN-08 · Credit & Debt
`Slow · 🟡 V2 · economy.json`
**Purpose.** The mechanism by which one bad season becomes five.
**Inputs** — `SOC-01.foodDeficit` (+0.65, borrowing need) · `ECN-04.collateral` (+0.60, access) ·
`ECN-03.income` (−0.50, repayment) · `ECN-01.price` (+0.35)
**Outputs** — `ECN-03.disposableIncome` (−0.55, debt service) · `ECN-04.savings` (−0.50) ·
`SOC-13.stress` (+0.50) · `AGR-02.investmentCapacity` (−0.40)
**Failure.** Debt service > 30% of income → the household cannot invest in the resilience that
would prevent the next shock. **Debt converts a one-season problem into a structural one.**
**Recovery.** 2–5 years. Debt relief (🟡 action) is fast but treats the symptom, exactly as
Subsidise Farmers does.

---

### ECN-09 · Enterprise Stock
`Annual · 🟡 V2 · economy.json`
Mills, workshops, transport, trading businesses — non-farm employment capacity.
**Inputs** — `ECN-02.marketActivity` (+0.60) · `INF-01.power` (+0.55, thr) ·
`ECN-08.credit` (+0.45) · `SOC-15.crime` (−0.35) · `SOC-10.population` (+0.30)
**Outputs** — `ECN-05.employment` (+0.65) · `ECN-11.taxBase` (+0.40) · `ECN-03.income` (+0.50)
**Failure.** Enterprise closure is fast and re-opening is slow — an asymmetry that makes economic
diversification much harder to rebuild than to lose.
**Recovery.** 2–5 years.

---

### ECN-10 · Government Budget
`Slow · 🟢 MVP · economy.json`

**Purpose.** The player's action ceiling — and the cruellest edge in the graph, because it
**shrinks exactly when it is most needed.** Every crisis simultaneously raises demand for
spending and destroys the tax base that funds it.

**Inputs** — `ECN-11.taxRevenue` (+0.80) · `ECN-12.externalAid` (+0.40, event) ·
`GOV-02.emergencySpend` (−0.70, event) · `INF-12.maintenanceObligation` (−0.45, annual) ·
`GOV-01.ongoingCommitments` (−0.60)
**Outputs** — `GOV-01.actionCeiling` (+1.00, hard gate) · `ECN-03.publicWages` (+0.80) ·
`INF-06/07.capacityInvestment` (+0.70) · `GOV-04.governanceCapacity` (+0.40) ·
`INF-12.maintenanceFunding` (+0.65) · `ECN-05.publicJobs` (+0.45)

**Equation** `budget += taxRevenue + aid − wages − maintenance − commitments − emergency`

**Failure.** Budget < maintenance obligation → infrastructure begins depreciating faster than it
is repaired, which raises future emergency costs, which further reduces the budget. **Loop R4, the
fiscal death spiral**, and the reason the Design Bible's Chain D1 ends where it does.
**Recovery.** Requires the tax base to recover, which requires household income to recover, which
takes years. Aid (🟡) is a bridge, not a cure.

---

### ECN-11 · Taxation
`Slow · 🟢 MVP · economy.json`
**Inputs** — `ECN-03.aggregateIncome` (+0.80) · `ECN-02.marketActivity` (+0.45) ·
`SOC-17.institutionalTrust` (+0.35, compliance, 🟡) · `SOC-10.population` (+0.40)
**Outputs** — `ECN-10.budget` (+0.85) · `ECN-03.disposableIncome` (−0.30)
**Failure.** Tax base contraction is *lagged and compounding* — the revenue hit arrives a season
after the income hit, which means the budget is still spending at pre-crisis levels when it
collapses.
**Recovery.** Tracks income + 1 season.

---

### ECN-12 · External Aid · `Event · 🟡 V2`
**Inputs** — `HYD-07/HYD-08.disasterSeverity` (+0.70, threshold-triggered, lagged 1s) · `SOC-12.mortality` (+0.40) · 🔵 external visibility
**Outputs** — `ECN-10.budget` (+0.40) · `SOC-01.emergencyFood` (+0.60, 1s)
**Failure.** Aid arrives late (1 season lag is deliberate), addresses symptoms, and **stops**. When
it lapses without a resilience investment made during the window, pressure re-accumulates — Chain M2.
**Recovery.** n/a — aid is a pulse, never a stock.

---

### ECN-13 · Remittances · `Slow · 🟡 V2`
**Purpose.** Migration's one benign return path, and a genuinely ambiguous one.
**Inputs** — `SOC-09.migrantsDeparted` (+0.60, 1y lag) · `ECN-07.externalEconomy` (+0.40) · `INF-11.transferRails` (+0.30, 🔵)
**Outputs** — `ECN-03.income` (+0.50) · `ECN-04.savings` (+0.40) · `SOC-09.migrationPressure` (−0.30) ·
`AGR-07.labourSupply` (−0.55, **the same people are gone from the fields**)
**Failure.** A village can become remittance-dependent: household cash improves while its
productive capacity hollows out. Not a failure state the game punishes — a real pattern it depicts.

---

## II.6 SOCIETY & HEALTH

### SOC-01 · Food Security · `Medium · 🟢 MVP · citizens.json`
**Purpose.** Household calories available vs. required. The variable the citizen decision model
reads first, every time.
**Inputs** — `AGR-01.ownProduction` (+0.70) · `ECN-03.income` (+0.60) · `ECN-01.price` (−0.85) ·
`AGR-04.fishProtein` (+0.30) · `ECN-04.savingsDrawdown` (+0.40, temporary)
**Outputs** — `SOC-03.nutrition` (+0.85, 1s) · `SOC-08.childLabour` (−0.65, thr) ·
`SOC-09.migrationPressure` (+0.60) · `SOC-13.stress` (+0.70) · `AGR-03.distressSale` (+0.55) ·
`SOC-04.health` (+0.50, 1s)
**Failure.** `< 0.6` for one season → child labour branch. `< 0.4` → asset sale. `< 0.3` sustained →
migration decision at the next season boundary.
**Recovery.** One good harvest, or one season of sustained income.

---

### SOC-02 · Water Security · `Medium · 🟢 MVP · citizens.json`
**Purpose.** Litres per person per day — **and how far someone walks for them.** The collection-time
term is what makes this a second, independent route to an empty classroom.
**Inputs** — `HYD-05.storedWater` (+0.90) · `HYD-03.flowRate` (+0.50) · `HYD-06.quality` (+0.60, usable
fraction) · `INF-03.distributionCoverage` (+0.70) · `SOC-10.population` (−0.45)
**Outputs** — `SOC-04.health` (+0.70, 5d) · `SOC-05.diseaseRisk` (−0.65) · `SOC-13.stress` (+0.45) ·
`SOC-07.attendance` (−0.40 via collection time, 1s) · `AGR-03.livestockWater` (+0.35)
**Failure.** `< 15 L/person/day` → health decline within one season; collection time > 2h/day →
measurable school attendance loss among girls specifically (🟡 when gender is modelled).
**Recovery.** Days if storage refills; otherwise gated by HYD-02 and therefore years.

---

### SOC-03 · Nutrition · `Medium · 🟢 MVP · citizens.json`
Sustained food security integrated over time — the stock behind the flow.
**Inputs** — `SOC-01.foodSecurity` (+0.85, integrated over 2 seasons) · `AGR-04.protein` (+0.35) ·
`AGR-03.dairyMeat` (+0.30, 🟡) · `SOC-05.diseaseBurden` (−0.40, absorption)
**Outputs** — `SOC-04.health` (+0.80, 1s) · `SOC-06.learningCapacity` (+0.50, 1y — **malnourished
children learn measurably less, and the effect outlives the hunger**) · `ECN-03.labourProductivity` (+0.45) ·
`SOC-12.mortalityRisk` (−0.55)
**Failure.** Sustained deficit in children under 5 → permanent reduction in adult income potential.
The longest-delayed edge in the entire graph: **a bad season in Year 3 shows up in the labour
market in Year 18.** 🔵 for full expression, but the accumulator must exist from MVP so the data is
there when the decade loop arrives.
**Recovery.** Adults: 1–2 seasons. Childhood stunting: never fully.

---

### SOC-04 · Health · `Medium · 🟢 MVP · citizens.json`
**Purpose.** Per-citizen. The convergence point of water, air, food and care — four independent
domains meeting in one number, which is why it is the best single diagnostic in the game.
**Inputs** — `SOC-03.nutrition` (+0.80) · `SOC-02.waterSecurity` (+0.70) · `HYD-06.waterQuality` (+0.45) ·
`ENV-01.airQuality` (+0.45) · `INF-06.clinicCapacity` (+0.60) · `SOC-05.disease` (−0.75) ·
`CLI-03.heatStress` (−0.25) · `SOC-13.stress` (−0.35)
**Outputs** — `ECN-03.labourProductivity` (+0.65) · `SOC-12.mortality` (−0.70) ·
`INF-06.clinicDemand` (−0.60, inverse) · `SOC-07.attendance` (+0.40) · `SOC-13.stress` (−0.30) ·
`SOC-18.wellbeing` (+0.55)
**Equation** — target-seeking, never instantaneous:
`health += (targetHealth(nutrition, water, air, care) − health) × recoveryRate(clinicCapacity)`.
The smoothing is what makes decline *watchable* rather than punitive.
**Failure.** `< 0.4` → productivity loss compounds the income shortfall that caused it (loop R3).
**Recovery.** 1–3 seasons with care available; indefinite without.

---

### SOC-05 · Disease Burden · `Medium/Event · 🟡 V2 · citizens.json`
**Inputs** — `HYD-06.waterQuality` (−0.80, exp) · `INF-09.sanitation` (−0.70) ·
`SOC-10.density` (+0.45) · `SOC-03.nutrition` (−0.55, susceptibility) · `HYD-07.flood` (+0.65, event)
**Outputs** — `SOC-04.health` (−0.85) · `INF-06.clinicLoad` (+0.80) · `ECN-05.labourAbsence` (−0.50) ·
`SOC-12.mortality` (+0.50)
**Equation** — compartmental-lite: `outbreakP = logistic(contamination × density × (1 − nutrition))`;
once triggered, an SIR-style curve over 20–40 ticks rather than a per-tick penalty.
**Failure.** Outbreak during a flood with the bridge down: demand peaks, supply and access
collapse together. The worst compound event the graph can produce.
**Recovery.** 1–2 seasons post-outbreak; sanitation fixes prevent recurrence, care only shortens it.

---

### SOC-06 · Education Attainment · `Annual · 🟡 V2 · citizens.json`
A stock that takes a childhood to build and one season to interrupt.
**Inputs** — `SOC-07.attendance` (+0.85, integrated) · `INF-07.schoolQuality` (+0.55) ·
`SOC-03.childNutrition` (+0.50) · `SOC-04.health` (+0.30)
**Outputs** — `ECN-03.adultIncomePotential` (+0.70, **10–15y**) · `SOC-16.climateAwareness` (+0.50, 5y) ·
`AGR-08.adaptationSpeed` (+0.45, 5y) · `SOC-11.fertilityTiming` (−0.30, 10y, 🔵)
**Failure.** Interrupted schooling is only partially recoverable — a year out is not a year delayed.
**Recovery.** Re-enrolment restores the flow; the missing stock is permanent.

---

### SOC-07 · School Attendance · `Slow · 🟢 MVP · citizens.json`
**Purpose. Not a policy variable.** Attendance is a *readout* of household desperation, computed as
the count of children whose household has not reassigned them. Nobody scripts the school emptying.
**Inputs** — `SOC-08.childLabour` (−0.90) · `SOC-01.foodSecurity` (+0.55) ·
`SOC-02.collectionTime` (−0.40) · `INF-04.roadAccess` (+0.30, thr) · `SOC-04.childHealth` (+0.35)
**Outputs** — `SOC-06.attainment` (+0.85) · `ECN-03.teacherJustification` (+0.30) · `SOC-18.wellbeing` (+0.40)
**Failure.** `< 0.5` → the visible school-emptying beat. Rendered as **fewer children present at
the building during school hours** — no caption, no notification explaining why.
**Recovery.** Attendance returns within a season of household stabilisation. Attainment does not.

---

### SOC-08 · Child Labour · `Slow · 🟢 MVP · citizens.json`
The household's rational, terrible short-term fix. Modelled without judgement, because the
judgement is the player's to reach.
**Inputs** — `SOC-01.foodSecurity` (−0.75, thr) · `AGR-07.labourDemand` (+0.55) ·
`ECN-03.income` (−0.65) · `SOC-16.awareness` (−0.30, 🟡)
**Outputs** — `SOC-07.attendance` (−0.90) · `AGR-01.labourFactor` (+0.35, **it does work**) ·
`SOC-06.attainment` (−0.70, 1y)
**Failure.** Persistence beyond ~2 seasons converts a temporary household coping strategy into a
permanent reduction in that child's lifetime income — the report's Ama, mechanised.
**Recovery.** Immediate on household stabilisation; the attainment loss is not recovered.

---

### SOC-09 · Migration Pressure · `Slow · 🟢 MVP · citizens.json`
**Purpose.** An accumulator with a threshold and a deliberately slow decay. Migration is a
*decision*, not a reflex — it accumulates over seasons and, once made, does not reverse.
**Inputs** — `ECN-03.income` (−0.75, sustained) · `SOC-01.foodSecurity` (−0.70) ·
`SOC-02.waterSecurity` (−0.45) · `ECN-04.savings` (−0.60, buffer) · `SOC-14.cohesion` (−0.35, ties) ·
`SOC-13.stress` (+0.40)
**Outputs** — `SOC-10.population` (−0.90, event at threshold) · `AGR-07.labourSupply` (−0.70) ·
`ECN-02.marketDemand` (−0.55) · `ECN-11.taxBase` (−0.60) · `INF-05.abandonment` (+0.85) ·
`ECN-13.remittances` (+0.45, 1y) · `SOC-14.cohesion` (−0.40)
**Equation**
```
pressure += stressGain*(unmetNeeds) − decayRate*(stability)
if pressure > threshold at season boundary: household migrates    // never mid-season
```
`decayRate ≈ 0.3 × stressGain` — pressure clears far more slowly than it builds.
**Failure.** Cluster migration: several households crossing in one season, which removes enough
labour and demand to push the remainder toward the threshold — **loop R2, the outflow spiral**.
**Recovery.** Pressure decays over 2–4 stable seasons. **Departed households do not return** in
MVP; the population loss is permanent, and so is its effect on the tax base.

---

### SOC-10 · Population · `Slow · 🟢 MVP · citizens.json`
**Inputs** — `SOC-09.migration` (−0.90) · `SOC-11.births` (+0.60, 🟡) · `SOC-12.deaths` (−0.50, 🟡) · 🔵 in-migration
**Outputs** — `AGR-07.labourSupply` (+0.75) · `ECN-02.demand` (+0.70) · `ECN-11.taxBase` (+0.65) ·
`HYD-05.waterDraw` (+0.60) · `INF-08.wasteGeneration` (+0.65) · `SOC-05.densityRisk` (+0.40) ·
`ECO-09.settlementArea` (+0.35, 1y)
**Failure.** Population loss > 25% → the settlement cannot staff its own harvest or fund its own
services. Below ~40% loss the market ceases to function as a market.
**Recovery.** MVP: none (fixed population, no births). 🟡: 5–15 years.

---

### SOC-11 · Births · `Annual · 🟡 V2` — **Inputs** `SOC-01/03.security` (+0.45), `SOC-04.health` (+0.40), `SOC-06.education` (−0.30, 10y) · **Outputs** `SOC-10.population` (+0.60), `ECN-03.dependencyRatio` (+0.35). Responds to security, never to policy.

### SOC-12 · Deaths · `Annual · 🟡 V2` — **Inputs** `SOC-04.health` (−0.70), `SOC-03.nutrition` (−0.55), `SOC-05.disease` (+0.50), `INF-06.careAccess` (−0.45) · **Outputs** `SOC-10.population` (−0.50), `SOC-14.cohesion` (−0.35), `ECN-03.householdCapacity` (−0.40). Modelled honestly, never as a gotcha: elevated rates only follow sustained, *visible* crisis conditions.

### SOC-13 · Stress · `Medium · 🟡 V2`
**Inputs** — `SOC-01.foodInsecurity` (+0.70) · `ECN-03.incomeShortfall` (+0.65) · `ECN-08.debt` (+0.50) ·
`HYD-07.disaster` (+0.55, event) · `SOC-14.cohesion` (−0.45, support)
**Outputs** — `SOC-04.health` (−0.35) · `ECN-03.productivity` (−0.40) · `SOC-09.migration` (+0.40) ·
`SOC-15.crime` (+0.30) · **decision quality** (−0.35 — 🟡: stressed households discount the future
more steeply, which makes them choose short-term actions. The player is watching a population make
worse choices *because of conditions the player created*, which is the sharpest form the thesis takes.)

### SOC-14 · Social Cohesion · `Annual · 🟡 V2`
The invisible infrastructure that decides how a shock is absorbed.
**Inputs** — `SOC-09.outMigration` (−0.50) · `ECN-03.inequality` (−0.45) · `SOC-15.crime` (−0.40) ·
`ECN-02.marketAsInstitution` (+0.30) · `GOV-04.fairness` (+0.35)
**Outputs** — `SOC-09.migrationPressure` (−0.35, ties hold people) · `GOV-02.mutualAid` (+0.55) ·
`SOC-13.stress` (−0.45) · `SOC-17.trust` (+0.50) · `SOC-18.wellbeing` (+0.50)
**Failure.** Below 0.35 the settlement stops absorbing shocks collectively and starts absorbing
them household-by-household — the same shock now produces far more migration.
**Recovery.** 3–8 years. One of the slowest social stocks.

### SOC-15 · Crime & Insecurity · `Slow · 🟡 V2` — **Inputs** `ECN-05.unemployment` (+0.50), `SOC-01.deprivation` (+0.45), `SOC-14.cohesion` (−0.55), `SOC-10.youthBulge` (+0.25) · **Outputs** `ECN-09.enterprise` (−0.40), `ECN-07.trade` (−0.30), `SOC-14.cohesion` (−0.40), `SOC-13.stress` (+0.35). A consequence system, never a threat system — no antagonist exists in Earth Engine.

### SOC-16 · Climate Awareness · `Annual · 🟡 V2` — **Inputs** `SOC-06.education` (+0.55), `GOV-01.climateEducation` (+0.70), lived shock experience (+0.40, 1y) · **Outputs** `AGR-08.adaptationSpeed` (+0.50), `ECO-01.protectionCompliance` (+0.40), household risk-response (+0.35). Changes what households do under *identical objective conditions* — the V2 decision model's key differentiator.

### SOC-17 · Institutional Trust · `Annual · 🟡 V2` — **Inputs** `GOV-02.responsePerformance` (+0.60), `ECN-11.taxFairness` (+0.35), `GOV-01.promiseKeeping` (+0.45), `SOC-14.cohesion` (+0.40) · **Outputs** `ECN-11.taxCompliance` (+0.50), `GOV-01.policyUptake` (+0.55), `SOC-09.migration` (−0.25). Determines whether a policy is complied with or merely announced.

### SOC-18 · Community Wellbeing · `Slow · 🟢 MVP · citizens.json`
**Terminal aggregate. Reads from eight systems, writes to none.** This is a hard architectural
constraint: nothing may take Wellbeing as an input, or the dependency graph becomes circular in a
way that is impossible to reason about or debug.
**Inputs** — `SOC-04.health` (+0.25) · `SOC-01.foodSecurity` (+0.20) · `SOC-07.attendance` (+0.15) ·
`ECN-03.income` (+0.15) · `SOC-14.cohesion` (+0.10) · `ENV-03.environmentalHealth` (+0.05) ·
`SOC-02.waterSecurity` (+0.05) · `ECN-05.employment` (+0.05)
**Outputs** — none. Dashboard and 🟡 music-layer only.
**Equation** `wellbeing = Σ wᵢ · normalise(inputᵢ)`, computed **last** in each slow tick.

---

## II.7 INFRASTRUCTURE

### INF-01 · Power Availability · `Fast · 🟢 MVP · energy.json`
**Purpose.** A hard ceiling, not a soft bonus. Three systems physically cannot exceed what power
allows: the clinic, the irrigation pumps, and the mill. This is the cross-domain dependency the
Design Bible insists on — healthcare is not an isolated meter.
**Inputs** — `INF-02.installedCapacity` (+1.00) · `CLI-06.irradiance` (+0.90 × solarShare) ·
`CLI-05.wind` (+0.80 × windShare, 🟡) · `SOC-10.demand` (−0.55) · `INF-12.gridCondition` (+0.40)
**Outputs** — `INF-06.clinicCapacity` (+0.70, **thr — a clinic without power is a soft clinic**) ·
`AGR-02.pumpCapacity` (+0.60, thr) · `ECN-09.enterpriseCapacity` (+0.55) ·
`INF-03.waterPumping` (+0.65) · `SOC-06.eveningStudy` (+0.25, 🟡) · `AGR-05.coldChain` (+0.40, 🔵)
**Equation** `available = Σ(capacityᵢ × availabilityFactorᵢ) − losses; deficit = max(0, demand − available)`
**Failure.** Deficit > 20% → clinic and pumps both degrade *simultaneously*, in the same season, from a
single cause. Compound failures like this are why the graph is worth drawing.
**Recovery.** Immediate on capacity addition (fossil: 1 season; solar: 1–2).

### INF-02 · Energy Mix · `Event · 🟢 MVP · energy.json`
**Purpose.** The clean/dirty split — the clearest, most legible trade-off Earth Engine can present,
and the one Design Bible §8.4 names as the single best teaching moment in the build.
**Inputs** — `GOV-01.buildSolar/buildFossil/buildWind` (event) · `ECN-10.budget` (gate)
**Outputs** — `INF-01.capacity` (+1.00) · `ENV-02.emissions` (+0.90 × fossilShare, **0 for solar**) ·
`ECN-10.operatingCost` (−0.45 fossil fuel, −0.10 solar) · `INF-12.maintenanceLoad` (+0.35)
**The paired chains, side by side:**
`Fossil` → power ↑ *now*, cheap → emissions ↑ → air quality ↓ → **health ↓, on a different dashboard, one season later**
`Solar` → power ↑ *slower*, dearer → **no emissions edge exists at all** → health untouched
**Failure.** Fossil share > 70% sustained → a measurable settlement-wide health penalty that the
player will not attribute to the energy decision unless the Atlas has taught them to.
**Recovery.** Air quality clears 2–4 seasons after fossil share falls.

### INF-03 · Water Infrastructure · `Event · 🟢 MVP · actions.json`
Tower, wells, rainwater harvesting, treatment.
**Inputs** — `GOV-01.waterActions` · `ECN-10.budget` · `INF-12.condition` (−0.45)
**Outputs** — `HYD-05.capacity` (+0.80) · `HYD-05.fillRate` (+0.60) · `HYD-06.treatedQuality` (+0.80) · `SOC-02.coverage` (+0.70)
**Failure.** Capacity without maintenance: a full tower that leaks is a slow, invisible loss.
**Recovery.** Immediate on repair. The cheapest resilience in the catalogue and the easiest to forget.

### INF-04 · Transport Network · `Event · 🟢 MVP · actions.json`
**Purpose.** Quiet until it fails, then everything fails with it. The bridge is a **binary** state
carrying six edges — the highest consequence-per-bit in the simulation.
**Inputs** — `HYD-07.floodState` (−0.95, thr) · `INF-12.roadCondition` (+0.60) · `GOV-01.improveRoads` (+0.70, 🟡)
**Outputs** — `ECN-07.tradeAccess` (+0.70, thr) · `ECN-02.marketAccess` (+0.55) ·
`GOV-02.responseSpeed` (+0.75) · `SOC-07.schoolAccess` (+0.35) · `INF-06.clinicAccess` (+0.60) · `ECN-01.priceVolatility` (−0.40)
**Failure.** Bridge impassable during a flood: **response capacity collapses in the exact window
demand peaks.** Chain F2, and the reason the bridge deserved to be modelled rather than decorated.
**Recovery.** Days for water to recede; a season for structural repair if damaged.

### INF-05 · Housing Stock · `Slow · 🟢 MVP`
**Inputs** — `ECN-03.income` (+0.50, upkeep) · `SOC-09.migration` (−0.85, abandonment) ·
`HYD-07.floodDamage` (−0.50) · `SOC-10.population` (+0.40, demand)
**Outputs** — `SOC-04.shelterHealth` (+0.35) · `ECO-09.settlementFootprint` (+0.40) · **visible settlement condition** (+1.00)
**Failure.** Abandonment is the most legible visual signal of migration in the entire world render —
and it must be a *consequence* of SOC-09, never an independently triggered art state.

### INF-06 · Health Facility Capacity · `Event · 🟢 MVP`
**Inputs** — `GOV-01.healthcareInvestment` (+0.80) · `INF-01.power` (+0.70, **hard cap**) ·
`ECN-10.staffingBudget` (+0.55) · `HYD-05.water` (+0.35)
**Outputs** — `SOC-04.recoveryRate` (+0.70) · `SOC-12.mortality` (−0.55) · `SOC-05.outbreakDuration` (−0.45)
**Failure.** Capacity < demand during an outbreak → recovery rate falls exactly when it is needed.
Investing here without securing power under-delivers — a deliberate, honest cross-system dependency.

### INF-07 · School Capacity · `Event · 🟢 MVP`
**Inputs** — `GOV-01.improveSchool` (+0.85) · `ECN-10.teacherWages` (+0.60)
**Outputs** — `SOC-06.attainmentRate` (+0.60) · `SOC-07.enrolmentCeiling` (+0.50) · `ECN-05.teacherJobs` (+0.35)
**Failure.** Quality investment is worthless while attendance is collapsing for household reasons —
**you cannot fix the school by improving the school.** One of the most valuable non-obvious lessons
the graph produces.
**Recovery.** Instant capacity; 10–15 years for the attainment payoff (Chain R3).

### INF-08 · Waste Management · `Medium · 🟢 MVP`
**Inputs** — `SOC-10.population` (+0.65, generation) · `GOV-01.wasteCollection` (−0.80) · `GOV-01.burnWaste` (−0.90 stock, event)
**Outputs** — `HYD-06.waterQuality` (−0.65, thr at overflow) · `ENV-02.emissions` (+0.70, burning only) ·
`SOC-05.diseaseRisk` (+0.45) · `ENV-01.airQuality` (−0.60, burning)
**The paired choice, mirroring energy exactly:** *Collect* = ongoing cost, no hidden edge.
*Burn* = free, instant, and writes to air quality on the same tick. Same trade-off shape as
Fossil vs. Solar, deliberately — the report frames them as the same category and the mechanics match.
**Failure.** Overflow near the river or wells → the water-quality channel opens and health declines
from a source the player has never looked at.

### INF-09 · Sanitation · `Medium · 🟡 V2`
**Inputs** — `GOV-01.sanitationInvestment` (+0.80) · `SOC-10.density` (−0.40) · `HYD-07.flood` (−0.70, event)
**Outputs** — `HYD-06.waterQuality` (+0.70) · `SOC-05.diseaseRisk` (−0.75) · `SOC-04.health` (+0.40)
**Failure.** Flood + poor sanitation is the highest-probability outbreak pathway in the model.

### INF-10 · Flood Defence · `Event · 🟢 MVP`
**Inputs** — `GOV-01.floodBarrier` (+0.90) · `INF-12.condition` (+0.50)
**Outputs** — `HYD-07.floodProbability[protected]` (−0.80) · `HYD-07.floodProbability[downstream]` (+0.25, 🟡) ·
`AGR-01.protectedYield` (+0.45)
**The teachable nuance (🟡).** A barrier does not remove floodwater; it relocates where the water
goes. Protected tiles gain, unprotected downstream tiles lose. **Flood defence can displace risk
rather than eliminate it** — a real, non-obvious infrastructure lesson, and one worth the extra edge.
**Recovery.** Front-loaded benefit; degrades with condition, not time-since-build.

### INF-11 · Communications · `Annual · 🔵 FUTURE`
**Outputs** — `GOV-02.earlyWarning` (+0.60) · `ECN-07.priceInformation` (+0.45) · `ECN-13.transferRails` (+0.40)
Present so the architecture does not foreclose it. No MVP consumer.

### INF-12 · Infrastructure Condition · `Annual · 🟡 V2`
**Purpose.** Everything decays. Maintenance is the least glamorous edge in the graph and one of
the most consequential, because deferred maintenance is invisible until it is a failure.
**Inputs** — `ECN-10.maintenanceFunding` (+0.75) · age (−0.40/yr) · `HYD-07.floodDamage` (−0.55) ·
`GOV-04.governanceCapacity` (+0.35)
**Outputs** — `INF-01/03/04/06/07/10.effectiveCapacity` (+0.60 each) · `ECN-10.emergencyRepairCost` (−0.50)
**Failure.** Funding below the depreciation rate → capacity silently erodes while the asset still
appears on the map. The player believes they own a clinic; they own a building.
**Recovery.** Immediate on funding, but arrears compound: a decade of deferral costs more than a
decade of maintenance.

---

## II.8 ENVIRONMENT QUALITY

### ENV-01 · Air Quality · `Medium · 🟢 MVP · energy.json`
**Purpose.** Loads slowly, clears slowly, and bills a different system than the one that caused it.
**Inputs** — `ENV-02.emissions` (−0.85) · `CLI-04.rainfall` (+0.30, washout) · `CLI-05.dispersion` (+0.35, 🟡) ·
`ECO-01.forestDensity` (+0.15, 5y)
**Outputs** — `SOC-04.health` (−0.55, 5d, cumulative) · `SOC-05.respiratoryBurden` (+0.50) ·
`CLI-06.irradiance` (−0.25, dust/haze) · `ENV-03.index` (+0.45)
**Equation** `airQuality += recovery×(1−airQuality) − (fossilRate×k1 + burnRate×k2 + dust×k3)`
**Failure.** Sustained poor air adds a slow, compounding health cost that appears on the Health
dashboard, never the Energy one. This displacement **is** the teaching mechanism (§8.8), not a bug.
**Recovery.** 2–4 seasons after the source stops.

### ENV-02 · Emissions · `Fast · 🟢 MVP` — **Inputs** `INF-02.fossilGeneration` (+0.90), `INF-08.wasteBurning` (+0.70), cooking fuel (+0.35, 🟡) · **Outputs** `ENV-01.airQuality` (−0.85), `ECO-10.carbon` (+0.60, 🔵), `ENV-03.index` (−0.40). Combustion accounting, nothing more — but it is the only place the settlement contributes to the problem it is suffering from, and that irony should be visible.

### ENV-03 · Environmental Health Index · `Annual · 🟡 V2` — composite of `ECO-01`, `ECO-05`, `HYD-06`, `ENV-01`, `ECO-02`, `ECO-06`. **Outputs** `SOC-18.wellbeing` (+0.30), dashboard trend only. The "is this landscape getting better" number, and the one a player watching a five-year recovery arc will care most about.

---

## II.9 GOVERNANCE

### GOV-01 · Policy Portfolio · `Event · 🟢 MVP · actions.json`
**Purpose.** Every active player action, held as persistent state. Twelve outgoing edges — the
highest of any system — because this *is* the player's hand on the wheel, and the only one.
**Inputs** — player action via `applyAction()` · `ECN-10.budget` (hard gate, checked **before** commit)
**Outputs** — one edge per action, into the system that action writes to. Full list in §X.
**The binding constraint (Design Bible §1.4, §7.6).** No action in this system may target an
individual citizen. Every action is a *condition applied to a place, a category, or the
settlement*. "Subsidise Farmers" conditions a household class; it does not command Kwame. Any
proposed action that can only be phrased as an instruction to one named person is rejected at
design time, not patched at implementation time.
**Failure.** Budget exhaustion → no actions available at the moment the settlement most needs them.
Loop R4.

### GOV-02 · Emergency Response · `Event · 🟢 MVP`
**Inputs** — `GOV-03.preparedness` (+0.80) · `INF-04.accessibility` (+0.75) · `ECN-10.emergencyFunds` (+0.60) ·
`SOC-14.mutualAid` (+0.45)
**Outputs** — `HYD-07.damageSeverity` (−0.65) · `SOC-12.mortality` (−0.55) ·
`ECN-10.budget` (−0.70, unbudgeted) · `SOC-17.trust` (+0.50 if effective, −0.60 if not)
**Failure.** The compound case again: flood → bridge down → response cannot reach → damage
un-buffered → budget drained by emergency spend → less prevention next year.
**Recovery.** Capacity restores with budget; **trust does not** — it is earned back over years.

### GOV-03 · Disaster Preparedness · `Slow · 🟢 MVP`
**Purpose.** Insurance. It does nothing observable until a shock arrives, which is an honest
representation of why preparedness spending is politically hard in good years.
**Inputs** — `GOV-01.disasterPrep` (+0.90) · `INF-12.condition` (+0.30)
**Outputs** — `HYD-07.damageMultiplier` (−0.60) · `HYD-08.droughtImpact` (−0.40) · `GOV-02.responseSpeed` (+0.55)
**Failure.** None *as such* — the failure mode is social: a run of good seasons makes this look
like waste. The game must never editorialise about that with a "told you so"; it lets the payoff,
when it comes, speak entirely for itself.

### GOV-04 · Governance Capacity · `Annual · 🟡 V2` — **Inputs** `ECN-10.budget` (+0.50), `SOC-06.education` (+0.40), `SOC-17.trust` (+0.45), `ECN-09.economicComplexity` (+0.30) · **Outputs** action effectiveness multiplier (+0.55), `INF-12.maintenanceEfficiency` (+0.45), `ECN-11.collection` (+0.40), `SOC-17.trust` (+0.35). Determines how much of a budget actually becomes an outcome — the difference between money spent and work done.

---

# PART III — THE CAUSE AND EFFECT WEB

## III.0 How to read this Part

Part II already contains **every edge** in the simulation, with sign, weight and lag. Part III is
not a second copy of that data — it is the **traversal**: what actually happens, in order, over
time, when a variable moves.

Each entry follows one format:

- **Immediately (0–5 ticks)** — same-tick and fast-tier consequences
- **Within one season (≤90 ticks)** — medium-tier and first-order decisions
- **Within one year** — slow-tier stocks and household decisions resolving
- **Within five years** — the slow stocks: soil, forest, education, cohesion, aquifer
- **Amplifiers** — variables that make the effect larger
- **Suppressors** — variables that damp it
- **Loops entered** — cross-reference to §VIII

---

## III.1 RAINFALL ↓ — the master chain

The chain the whole project exists to teach, expanded to every branch the graph actually contains.

```
RAINFALL ↓
├── IMMEDIATELY (0–5 ticks)
│   ├── Soil moisture ↓ ────────────────┐
│   ├── Surface runoff ↓                │
│   ├── River flow ↓ ──────┐            │
│   ├── Drought stress ↑   │            │
│   └── Air quality ↓ (no washout)      │
│                          │            │
├── WITHIN ONE SEASON      │            │
│   ├── Crop moisture factor ↓ ─────────┤
│   ├── Crop yield ↓ ◄─────────────────-┘
│   ├── Irrigation draw ↑  (households compensate)
│   │     └── Groundwater ↓ ── River baseflow ↓ ── **river falls faster than rain did**
│   ├── Water tower fill ↓ ── Water security ↓ ── Collection time ↑
│   │     └── School attendance ↓  (the water route, independent of food)
│   ├── Fish habitat ↓ ── Fish stock ↓
│   │     └── AND fishing effort ↑ (crop failure pushes effort to the river)
│   │           └── Stock ↓↓  **both blades of the scissors close at once**
│   ├── Livestock condition ↓ ── Distress sale ↑ ── Household assets ↓
│   └── Water quality ↓ (less dilution, same waste load)
│         └── Disease risk ↑
│
├── WITHIN ONE YEAR
│   ├── Food supply ↓ ── Food price ↑↑
│   │     ├── Farmer revenue: ↑ or ↓ (price up, volume down — depends on elasticity)
│   │     ├── Elder real income ↓↓   (fixed income, rising prices — pure loss)
│   │     ├── Vendor income ↓        (volume falls faster than margin rises)
│   │     └── Household food security ↓
│   │           ├── Child labour ↑ ── School attendance ↓ (the food route)
│   │           ├── Nutrition ↓ ── Health ↓ ── Productivity ↓ ── Income ↓  ⟲ R3
│   │           └── Savings drawdown ── Assets ↓ ── Debt ↑
│   ├── Migration pressure ↑ (accumulating, not yet crossing)
│   ├── Tax base ↓ ── Government budget ↓
│   │     └── Maintenance funding ↓ ── Infrastructure condition ↓  ⟲ R4
│   └── Clinic load ↑ against a budget that is falling
│
└── WITHIN FIVE YEARS
    ├── Migration threshold crossings cluster ── Population ↓
    │     ├── Farm labour ↓ ── Cropped area ↓ ── Food supply ↓  ⟲ R2
    │     ├── Market activity ↓ ── Vendor income ↓ ── more migration  ⟲ R5
    │     └── Tax base ↓↓ ── Budget ↓↓ ── recovery becomes unaffordable  ⟲ R4
    ├── Soil quality ↓ (drought degradation + erosion on bare ground)
    │     └── **Yield ceiling permanently lower, at any future rainfall**
    ├── Education attainment ↓ for the cohort that left school
    │     └── Adult income potential ↓ 10 years out  ⟲ R10, the generational loop
    ├── Social cohesion ↓ (out-migration severs ties)
    │     └── Shock absorption capacity ↓ ── next drought hurts more
    └── Forest density ↓ (drought dieback + fuelwood pressure from poorer households)
          └── Runoff fraction ↑ ── **flood risk ↑ during the recovery rains**  ⟲ B4 broken
```

**The sting in the tail, and the single most important thing in this document:** the drought's
final act is to make the *flood* worse. Cover was lost, soil hardened, channels silted. The rains
return — and instead of relief, the first big storm takes the farmland. A player who has watched
this once understands catchment management better than any tutorial could teach them.

**Amplifiers.** `CLI-01.drift` (deeper and more frequent) · low `ECO-02.soilQuality` (less buffer) ·
low `ECO-01.forestDensity` (worse infiltration) · low `ECN-04.savings` (no absorber) ·
low `HYD-05.storage` (no buffer) · high `ECN-08.debt` (no slack) · low `SOC-14.cohesion` (no mutual aid).
**Suppressors.** Irrigation *in moderation* · storage capacity · savings · fisheries as an
alternative income · trade access · crop diversity (🟡) · disaster preparedness · forest cover.
**Loops entered.** R1, R2, R3, R4, R5, R6, R10, R11; B1, B2, B6.

---

## III.2 FOREST DENSITY ↓

```
FOREST ↓
├── IMMEDIATELY   fuelwood/timber income ↑ (this is why it happens)
├── ONE SEASON    runoff fraction ↑ · erosion ↑ · soil moisture ↓ · sediment load ↑
├── ONE YEAR      flood probability ↑ at unchanged rainfall  ← *the key non-obvious result*
│                 soil quality ↓ · biodiversity ↓ · local temperature ↑
├── FIVE YEARS    pollinators ↓ ── yield ↓ on dependent crops
│                 fish spawning habitat ↓ (siltation) ── fish stock ↓
│                 soil erosion has removed the topsoil that regrowth would need  ⟲ R8
└── Amplifiers: steep slope, bare ground, drought dieback, poverty (fuelwood demand)
    Suppressors: Tree Protection, alternative fuel (INF-01 power), prosperity, awareness
    Loops: R7 (clearing→income→clearing), R8 (loss→erosion→harder regrowth), B4 broken
```
**Recovery asymmetry, stated plainly:** clearing rate ≈ 0.35/season; regrowth ≈ 0.04/season.
**Nine seasons of growth undone in one.** Do not tune toward parity.

---

## III.3 FOOD PRICE ↑

```
FOOD PRICE ↑
├── IMMEDIATELY   household purchasing power ↓ · market volume ↓ (value may rise)
├── ONE SEASON    Elder real income ↓↓ (fixed) · Vendor income ↓ · Farmer revenue ↑ or ↓
│                 food security ↓ ── child labour ↑ ── attendance ↓
│                 savings drawdown ↑ · borrowing ↑
├── ONE YEAR      nutrition ↓ ── health ↓ ── clinic load ↑ ── budget ↑ spending
│                 migration pressure ↑ across non-farming households first
├── FIVE YEARS    inequality ↑ (farmers with surplus gain; everyone else loses)
│                 cohesion ↓ · crime ↑ (🟡) · enterprise ↓
└── Amplifiers: import blockage (road down), hoarding, inflation, low savings
    Suppressors: trade access, storage, subsidy, fisheries protein, own-production share
    Loops: R3, R5; B3 (price → import → supply → price)
```
**The distributional point, which a single village-prosperity number would erase:** the same
price spike *helps* a Farmer with surplus and *ruins* the Elder next door. Earth Engine must never
average these into one happiness bar.

---

## III.4 GROUNDWATER ↓

```
GROUNDWATER ↓
├── IMMEDIATELY   (nothing visible — this is the entire problem)
├── ONE SEASON    well yield ↓ · irrigation ceiling ↓ · dry-season river baseflow ↓
├── ONE YEAR      water security ↓ · irrigation fails in the season it is most needed
│                 river runs low earlier each year at unchanged rainfall
├── FIVE YEARS    aquifer may not recover within the playthrough
│                 deep-rooted trees stressed ── forest dieback ── runoff ↑ ── recharge ↓  ⟲ R9
└── Amplifiers: irrigation expansion after a *successful* irrigated season (R6), population growth,
    hardened soil (low infiltration), high PET
    Suppressors: rainwater harvesting, efficient irrigation, forest cover, soil organic matter
    Loops: R6, R9; B2
```
**Why this is the most instructive system in the game:** every symptom appears somewhere else,
one to two years late. The player who learns to watch the *dry-season* river rather than the wet-season
one has genuinely understood hydrology.

---

## III.5 POWER AVAILABILITY ↑ (via Fossil)

```
FOSSIL POWER ↑
├── IMMEDIATELY   power ↑ · clinic capacity ↑ · pump capacity ↑ · enterprise capacity ↑
│                 emissions ↑ (same tick)
├── ONE SEASON    air quality ↓ · irradiance ↓ slightly (haze) → solar yield ↓
├── ONE YEAR      respiratory burden ↑ ── health ↓ ── clinic load ↑
│                 **the health cost lands on a dashboard the player was not looking at**
├── FIVE YEARS    cumulative health deficit · productivity ↓ · fuel operating cost compounding
└── The counter-chain (Solar): identical power benefit, **the emissions edge does not exist**.
    Running both side by side for one season is the clearest teaching moment the simulation has.
    Loops: R12 (fossil → haze → solar underperforms → more fossil) — a genuine lock-in trap.
```

---

## III.6 SCHOOL ATTENDANCE ↓

```
ATTENDANCE ↓
├── IMMEDIATELY   fewer children rendered at the school building. No caption. No notification.
├── ONE SEASON    child labour ↑ ── farm labour factor ↑ (**it does help the harvest** — that is why it happens)
├── ONE YEAR      education attainment accumulation stalls for that cohort
├── FIVE YEARS    ── nothing visible ──
└── TEN–FIFTEEN YEARS
      adult income potential ↓ ── household income ↓ ── their children's attendance ↓  ⟲ R10
      climate awareness ↓ ── adaptation speed ↓ ── yield under drift ↓
      governance capacity ↓ (educated population is a governance input)
```
**R10 is the longest loop in the graph and the most important one for the project's thesis.** It
closes across a *generation*. The MVP will not run long enough to close it — the accumulator must
exist anyway, so that when the decade loop arrives the data is already there.

---

## III.7 MIGRATION ↑

```
MIGRATION ↑
├── IMMEDIATELY   home → abandoned (visible) · population ↓
├── ONE SEASON    farm labour ↓ · market demand ↓ · vendor income ↓ · tax base ↓
├── ONE YEAR      cropped area ↓ (labour-limited) ── food supply ↓ ── price ↑
│                 remaining households' migration pressure ↑  ⟲ R2 accelerating
│                 remittances begin ↑ (1y lag) — partial, ambiguous relief
├── FIVE YEARS    cohesion ↓ · school under-enrolled · clinic under-used but under-funded
│                 the settlement can no longer staff its own harvest
└── Suppressors: subsidy, savings, cohesion, alternative income (fisheries, enterprise), remittances
    Loops: R2 (the outflow spiral), R5 (market thinning), R4 (fiscal)
```
**Migration is not modelled as failure.** It is an adaptation with costs, and for some households
it is the correct decision. The simulation must never score it as a loss condition.

---

## III.8 Compact traversals for the remaining high-leverage variables

| Variable ↑ | Immediate | 1 season | 1 year | 5 years | Loops |
|---|---|---|---|---|---|
| **Temperature** | PET ↑ | soil moisture ↓, heat-days ↑ | yield ↓, health ↓, pests ↑ | fish stock ↓, livestock ↓ | R1 |
| **Irrigation** | soil moisture ↑ | yield ↑, groundwater ↓ | income ↑, aquifer ↓ | **water crisis it was bought to prevent** | R6, B2 |
| **Fishing effort** | catch ↑ | stock ↓, CPUE ↓ | income ↓ despite more effort | stock collapse, buffer gone | R11, B9 |
| **Waste (uncollected)** | — | water quality ↓ | disease ↑, clinic load ↑ | trust ↓, cohesion ↓ | R13 |
| **Savings** | — | shock absorption ↑ | migration pressure ↓ | asset base ↑, resilience ↑ | B5 |
| **Government budget** | action ceiling ↑ | investment ↑ | capacity ↑, maintenance funded | condition ↑, emergency cost ↓ | B7 |
| **Cohesion** | — | mutual aid ↑ | migration ↓, stress ↓ | shock absorption ↑↑ | B8 |
| **Debt** | consumption ↑ | disposable income ↓ | investment capacity ↓ | poverty trap | R3 |
| **Road/bridge open** | access ↑ | trade ↑, price volatility ↓ | response speed ↑ | market integration ↑ | B3 |
| **Soil quality** | — | moisture holding ↑ | yield ↑, runoff ↓ | recharge ↑, drought buffer ↑ | B10 |
| **Air quality** | — | respiratory load ↓ | health ↑, productivity ↑ | mortality ↓ | B11 |
| **Preparedness** | — | (nothing) | (nothing) | **damage halved when the shock lands** | B12 |

---

# PART IV — FOOD-WEB DEPENDENCY GRAPH

## IV.1 Trophic reading of the simulation

Earth Engine's dependency graph has the same shape as an ecological food web, and reading it that
way is genuinely useful:

| Ecological analogue | Earth Engine layer | Systems |
|---|---|---|
| **Solar input** | Exogenous climate | CLI-01…07 |
| **Primary producers** | Water & land converting climate into usable stock | HYD-01…09, ECO-01…10 |
| **Primary consumers** | Livelihoods converting stock into product | AGR-01…08, FSH |
| **Secondary consumers** | Economy converting product into claims | ECN-01…13 |
| **Apex / integrators** | Society consuming everything | SOC-01…18 |
| **Decomposers / recyclers** | Infrastructure & governance returning capacity | INF, GOV, ENV |
| **Terminal sink** | SOC-18 Community Wellbeing | reads all, writes none |

## IV.2 Central hubs — ranked by (in-degree × out-degree)

| Rank | System | In | Out | Score | Why it is a hub |
|---|---|---|---|---|---|
| 1 | **HYD-03 River Flow** | 6 | 11 | 66 | Touches water, food, income, transport, health, ecology |
| 2 | **ECN-03 Household Income** | 8 | 8 | 64 | Every environmental shock converts to a human outcome here |
| 3 | **AGR-01 Crop Yield** | 9 | 6 | 54 | The transducer: environment in, economy out |
| 4 | **SOC-04 Health** | 7 | 6 | 42 | Convergence of water, air, food, care |
| 5 | **SOC-09 Migration Pressure** | 6 | 7 | 42 | The settlement's release valve, and its unravelling |
| 6 | **ECN-01 Food Price** | 5 | 7 | 35 | The pivot from environment to household |
| 7 | **HYD-01 Soil Moisture** | 6 | 5 | 30 | The variable crops actually feel |
| 8 | **ECN-10 Government Budget** | 5 | 6 | 30 | The player's ceiling, and it moves |
| 9 | **SOC-10 Population** | 4 | 7 | 28 | Labour, demand, tax, density, waste |
| 10 | **ECO-01 Forest Density** | 5 | 8 | 40 | Slowest, quietest, widest reach |

**Design consequence:** these ten systems must be implemented first and tested hardest. A bug in
HYD-03 is a bug in eleven other systems. A bug in ECO-10 is a bug in nothing yet.

## IV.3 Bottlenecks — single points of failure

| Bottleneck | What it gates | Failure signature | Mitigation in-graph |
|---|---|---|---|
| **INF-01 Power** | clinic + pumps + enterprise | three unrelated systems degrade in the same season | diversified mix, storage 🔵 |
| **INF-04 Bridge** | trade, response, market, school access | binary — everything downstream stops at once | 🟡 second crossing, road investment |
| **HYD-02 Groundwater** | irrigation ceiling, dry-season baseflow | invisible until both fail together | harvesting, efficiency, forest |
| **ECN-10 Budget** | *every* player action | player becomes a spectator | broad tax base, maintenance discipline |
| **HYD-05 Storage** | household water security | rationing, collection time, attendance | capacity + harvesting |
| **ECO-06 Fish Stock** | the drought buffer | collapses exactly when relied upon | effort control, alternative income |

## IV.4 Cascading-failure ranking

Measured as *number of systems reaching a degraded state within one year of the trigger*.

| Trigger | Systems degraded ≤1y | Depth |
|---|---|---|
| Bridge down during flood + outbreak | 14 | catastrophic, compound |
| Groundwater collapse after irrigation expansion | 11 | deep, delayed, invisible |
| Sustained drought (drift > 0.6) | 19 | the widest cascade in the graph |
| Fossil-only power for 3 years | 6 | narrow but permanent-ish |
| Forest clearing > 50% | 9 | slow onset, very slow recovery |
| Fish stock collapse during crop failure | 7 | removes the buffer, not just the income |
| Budget collapse | 12 | disables *response*, so everything else worsens |

## IV.5 Recovery-time ranking — the most important table in this document

| Speed | Systems | Typical recovery | Implication for the player |
|---|---|---|---|
| **Hours–days** | River flow, runoff, power, road access, air scrubbing | <10 ticks | These lie to you. They recover first and look like the crisis is over. |
| **Weeks** | Soil moisture, stored water, market activity | 10–40 ticks | Visible relief. Still nothing structural. |
| **1–2 seasons** | Drought stress, water quality, crop cycle, income, price | 90–180 ticks | The first *honest* recovery signal. |
| **1–3 years** | Savings, livestock, fish stock, enterprise, air quality | 1–3 y | Household capacity returns here, not before. |
| **3–8 years** | Groundwater, soil quality, cohesion, debt clearance | 3–8 y | The stocks that decide the *next* shock's severity. |
| **8–15 years** | Forest maturity, education attainment, health cohort effects | 8–15 y | Beyond most playthroughs. Plant anyway. |
| **Never** | Erosion-lost topsoil, childhood stunting, departed households | — | The genuinely permanent losses. There must be some. |

**The teaching claim this table encodes:** *the speed at which a system recovers is inversely
proportional to how much it determines your resilience to the next shock.* Everything that heals
fast is cosmetic; everything that matters heals slowly. A player who internalises only this has
learned the single most transferable lesson climate adaptation has to offer.

---

# PART V — HUMAN LIFE SIMULATION

Ten archetypes, each traced from an environmental variable to the point where the chain naturally
terminates — which, in a closed system, means the point where it re-enters itself. Every arrow
below is an edge specified in Part II. Nothing here is narrative; it is the graph, walked.

## V.1 FARMER — direct exposure, violent, same-season

```
Rainfall ↓ → Soil moisture ↓ → Crop yield ↓ → Farm income ↓
  → Food purchased ↓ → Household nutrition ↓ → Health ↓ → Labour productivity ↓
  → Next season's yield ↓ (weaker labour on the same land)          ⟲ R3 poverty trap
  → Savings drawdown → Livestock sold → Asset base ↓
  → Manure input ↓ → Soil nutrient return ↓ → Soil quality ↓
  → Yield ceiling ↓ permanently                                     ⟲ R14 land degradation
  → Borrowing ↑ → Debt service ↑ → Disposable income ↓
  → Cannot fund irrigation → Exposure to next drought unchanged     ⟲ R3 again, tighter
  → Migration pressure ↑ → (threshold) → Household departs
  → Farm labour supply ↓ → Neighbours' cropped area ↓ → Food supply ↓
  → Food price ↑ → Every remaining household worse off              ⟲ R2 outflow spiral
  → Tax base ↓ → Budget ↓ → Irrigation subsidy unaffordable
  → No household can now afford the adaptation that would have prevented this  ⟲ R4
```
**Terminates** at R4, the fiscal loop, which has no exit inside the household layer — only a
policy intervention made *before* the spiral, which is precisely the lesson.

## V.2 CHILD — indirect exposure, delayed by a decade

```
Rainfall ↓ → Household food security ↓ → needsChildLabour = true
  → School attendance ↓ → Education attainment accumulation stalls
  → (nothing visible for 10 years)
  → Adult income potential ↓ → Household income ↓ (as an adult)
  → Their children's food security ↓ → Their children's attendance ↓   ⟲ R10 generational
  → Climate awareness ↓ → Adaptation speed ↓ → Yield under drift ↓
  → Governance capacity ↓ (educated population is a governance input)
  → Policy effectiveness ↓ → Every action the player takes delivers less  ⟲ R15
Parallel branch, same trigger:
  → Nutrition ↓ during growth → Cognitive development ↓ (permanent)
  → Learning capacity ↓ even when attendance resumes
  → **Returning to school does not return the child to where they would have been**
```
**Terminates** in the workforce, fifteen simulated years later. This is the longest edge in Earth
Engine and the strongest argument for the tick architecture supporting a decade loop from day one.

## V.3 FISHER — different exposure channel, same destination

```
Rainfall ↓ → River flow ↓ → Wetted habitat ↓ → Fish stock ↓
  → Catch per trip ↓ → Fisher income ↓
Simultaneously, from the OTHER side:
Crop failure (neighbours) → Distress diversification → Fishing effort ↑↑
  → Catch ↑ briefly → Stock ↓↓ → Catch per trip ↓↓
  → **More boats, less fish, lower income for everyone fishing**     ⟲ R11 commons collapse
  → Protein supply to village ↓ → Nutrition ↓ (all households, not just fishers)
  → The drought buffer the village was counting on is gone
  → Migration pressure ↑ among fishers AND farmers simultaneously
  → Landing activity ↓ → Drying racks empty → Market fish stalls empty
  → Vendor income ↓ → Market activity ↓                              ⟲ R5
Recovery requires effort ↓, which requires alternative income,
  which requires the harvest to recover, which requires the rain.
  → **The fishery cannot recover until the thing it was buffering has already recovered.**
```

## V.4 MARKET TRADER (VENDOR) — aggregate exposure, lagged, smoothed

```
Crop yield ↓ → Marketable surplus ↓ → Market supply ↓
  → Price ↑ but volume ↓↓ → Vendor margin × volume → Vendor income ↓
  → Vendor household purchasing power ↓ → Their food security ↓
  → Vendor reduces stall days → Market activity ↓ (visible: fewer stalls)
  → Market as a social institution weakens → Cohesion ↓
  → Information flow between households ↓ (markets are how a village knows itself)
  → Mutual aid coordination ↓ → Shock absorption ↓
  → Next shock produces more migration than the same shock would have  ⟲ B8 weakened
  → Enterprise stock ↓ → Non-farm employment ↓
  → The only livelihood not exposed to rainfall has become exposed to rainfall
```

## V.5 TEACHER — insulated income, exposed vocation

```
Drought → Tax base ↓ → Government budget ↓ → Public wage bill under pressure
  → Teacher income ↓ (lagged 2–4 seasons — insulated, not immune)
Meanwhile, independent of their income:
  → Household food stress across the village → Child labour ↑ → Attendance ↓
  → Classroom empties while the teacher is still paid to teach
  → School quality investment underperforms (you cannot fix the school by improving the school)
  → Education attainment ↓ → 10y: adult income ↓ → tax base ↓ → teacher wage ↓  ⟲ R10 closing
  → Teacher household migrates → School capacity ↓ structurally
  → **Even if every child returned tomorrow, there is now nobody to teach them**
```

## V.6 HEALTHCARE WORKER — demand rises as capacity falls

```
Drought → Water security ↓ + Water quality ↓ + Nutrition ↓
  → Disease burden ↑ + Health ↓ → Clinic demand ↑↑
Simultaneously:
  → Tax base ↓ → Budget ↓ → Staffing and supply budget ↓ → Clinic capacity ↓
  → AND power availability constrained → Clinic capacity ↓ again (cold chain, lighting, pumps)
  → Effective care per patient ↓ → Recovery rate ↓ → Health ↓  ⟲ R16 care spiral
  → Mortality ↑ → Cohesion ↓ → Trust in institutions ↓
  → Tax compliance ↓ → Budget ↓ → Clinic capacity ↓             ⟲ R17 legitimacy spiral
  → Health worker household stress ↑ → migration → capacity ↓ structurally
```
**The compound case:** flood + outbreak + bridge down. Demand peaks, access fails, and capacity is
already degraded by a budget that a drought emptied two years earlier.

## V.7 GOVERNMENT WORKER — the budget is the biography

```
Household incomes ↓ → Tax revenue ↓ (lagged one season — **the budget is still spending
  at pre-crisis levels when the revenue arrives short**)
  → Emergency response spend ↑ (unbudgeted) → Capital budget raided
  → Maintenance funding ↓ → Infrastructure condition ↓
  → Effective capacity of clinic / school / water / roads ↓ (all at once)
  → Service quality ↓ → Institutional trust ↓ → Tax compliance ↓
  → Revenue ↓ again                                              ⟲ R4 + R17 compounding
  → Public wages delayed → Government worker household income ↓
  → Governance capacity ↓ → Each cedi of budget delivers less outcome
  → **The player's actions become less effective precisely as they become more necessary**
```

## V.8 CONSTRUCTION WORKER — the investment cycle made human

```
Budget ↑ → Infrastructure investment ↑ → Construction employment ↑
  → Household income ↑ → Local demand ↑ → Market activity ↑ → Vendor income ↑
  → Tax base ↑ → Budget ↑                                        ⟲ B7 virtuous, the ONLY
                                                                    strong balancing loop
                                                                    the player controls directly
Inverted:
Budget ↓ → Investment ↓ → Construction employment ↓ → Income ↓
  → Skilled workers migrate → Build capacity ↓
  → Future infrastructure costs ↑ (must import labour)
  → **The capacity to recover has itself migrated**
```

## V.9 UNEMPLOYED CITIZEN — every exposure channel at once

```
Employment ↓ → Income ≈ casual + transfers only
  → Food security ↓ immediately (no buffer, no own production)
  → Health ↓ → Employability ↓ → Employment ↓                    ⟲ R3, tightest form
  → Child labour ↑ earliest of any archetype (least slack)
  → Savings 0 → any shock becomes debt → debt service → permanent income reduction
  → Stress ↑↑ → decision horizon shortens → short-term choices dominate
  → (🟡 V2) Household chooses Burn Waste over Collection, fuelwood over purchased energy
  → Air quality ↓, forest ↓ → **the poorest households are pushed into the choices that
    damage the commons, by conditions the player set**                ⟲ R18
  → Crime risk ↑ → Cohesion ↓ → Enterprise ↓ → Employment ↓      ⟲ R19
```
**This chain must never be scored as a moral outcome.** It is a structural one. The Design Bible's
§1.4 argument exists precisely so this chain reads as *conditions producing behaviour* rather than
*people making bad choices*.

## V.10 ELDERLY CITIZEN — the price channel, invisible and immediate

```
Food price ↑ → Fixed income buys less → Real income ↓ (no yield channel, no wage channel)
  → Food intake ↓ → Nutrition ↓ → Health ↓ (from a lower baseline, faster)
  → Clinic dependence ↑ → against capacity constrained by the same budget crisis
  → Mobility low → cannot migrate, cannot switch livelihood, cannot fish
  → **Absorbs the entire shock in place**
  → Household support demand ↑ → Working-age household member's disposable income ↓
  → That member's own savings ↓ → their migration pressure ↑
  → If they migrate: elder remains, support ↓, health ↓↓
  → Mortality ↑ → Cohesion ↓ (elders are cohesion's largest single contributor)
  → Community memory of previous droughts ↓ (🔵 Memory system)
  → **The village loses the people who remember how the last one was survived**
```

## V.11 What the ten chains have in common

Every chain, without exception, passes through **household income or household food security**
before it reaches a person. Not one climate variable touches a citizen attribute directly.

That is not stylistic. It is the mechanical guarantee, enforced in code by the layer boundary in
§XII of the Design Bible, that the lesson the player learns is the true one: *climate shocks reach
people through economic structure, and economic structure is the thing policy can actually change.*

---

# PART VI — ECONOMY SIMULATION: EVERY FLOW OF MONEY

## VI.1 Where money originates

Four sources only. Everything else is circulation.

| Origin | Mechanism | Rate | Notes |
|---|---|---|---|
| **Agricultural sale** | `yield × price × marketedFraction` | seasonal | The dominant source. Climate-exposed by construction. |
| **Fisheries sale** | `catch × fishPrice` | seasonal | Second source, different exposure channel |
| **External aid** | event-triggered, lagged 1 season | 🟡 | Enters as budget, not household income |
| **Remittances** | `departedHouseholds × transferRate` | annual, 1y lag | 🟡 The only inflow migration creates |

🔵 FUTURE: export earnings once regional trade exists. In MVP the settlement is close to a closed
economy, which is honest for its scale and makes the flows traceable.

## VI.2 The circulation map

```
                     ┌──────────── EXTERNAL AID (🟡) ──────┐
                     │                                     ▼
  RAIN ──► YIELD ──► HOUSEHOLD ──tax──► GOVERNMENT ──wages──► HOUSEHOLD
            │        INCOME                BUDGET      ▲       (public)
            │          │  ▲                  │         │
            │          │  │                  ├─► INFRASTRUCTURE INVESTMENT
            │          │  └── subsidy ◄──────┤         │
            │          │                     ├─► MAINTENANCE
            │          ▼                     └─► EMERGENCY RESPONSE
            │      MARKET PURCHASE                    (unbudgeted, crowds out the above)
            │          │
            │          ▼
            └────► VENDOR INCOME ──tax──► GOVERNMENT
                       │
                       ▼
                   SAVINGS ──► ASSETS (livestock, tools, roof)
                       │            │
                       │            └──distress sale──► CONSUMPTION (one-way)
                       ▼
                   DEBT SERVICE ──► (leaves the settlement) ✗
```

## VI.3 Where money accumulates

| Stock | Fills from | Drains to | Time constant |
|---|---|---|---|
| Household savings | income surplus | food deficit, health shock, debt | 3–8 seasons |
| Household assets | savings, livestock reproduction | distress sale, flood, disease | 2–5 years |
| Government budget | tax, aid | wages, maintenance, investment, emergency | 1–2 seasons |
| Enterprise capital | retained margin, credit | closure, theft, obsolescence | 2–5 years |
| Infrastructure value | investment | depreciation, disaster | 10–30 years |

**Infrastructure is the only stock that converts money into *resistance to future shocks*.**
Every other stock converts money into the ability to survive the current one. That distinction is
the entire resilience-vs-relief trade-off, expressed as accounting.

## VI.4 Where money disappears (leakage)

| Leak | Mechanism | Magnitude | Player-controllable? |
|---|---|---|---|
| **Debt service to outside lenders** | interest | 5–30% of income when indebted | Indirectly (prevent the debt) |
| **Import purchases** | trade deficit | rises exactly during scarcity | Partly (storage, production) |
| **Post-harvest loss** | spoilage before sale | 10–35% of harvest | **Yes — cheapest fix available** |
| **Depreciation** | infrastructure decay | 4–10%/yr unmaintained | Yes (maintenance) |
| **Emergency spend** | unbudgeted response | spikes 20–60% of budget | Yes (preparedness ↓ this) |
| **Migration** | departing household takes its savings | permanent | Indirectly |
| **Governance loss (🟡)** | budget that does not become outcome | 10–40% at low capacity | Yes (education, trust) |

**The design point:** four of the seven leaks are largest during a crisis, and three of them are
cheapest to fix during calm. This asymmetry is the mechanical form of "resilience is bought in
good years", and it needs no dialogue to make the argument.

## VI.5 Price formation, in full

```
supply(t)  = Σ yields + Σ catch + imports − losses − storedCarryover
demand(t)  = population × perCapitaRequirement × (1 + incomeElasticity × Δincome)
target     = basePrice × (demand / max(supply, floor))^ε          ε ≈ 1.3 for staples
price(t)   = price(t−1) + adjust × (target − price(t−1))
             adjust = 0.55 rising, 0.18 falling                    // sticky downward
```

## VI.6 Government fiscal cycle

```
revenue      = Σ households(income × effectiveTaxRate × complianceRate(trust))
             + marketLevy(marketActivity) + aid
committed    = publicWages + activeSubsidies + debtService
discretionary= revenue − committed − maintenanceObligation
actionCeiling= max(0, discretionary + reserves)
```
**The cruelty is structural and intentional:** `revenue` falls one season *after* the shock,
`committed` does not fall at all, and `emergency` spikes immediately. `actionCeiling` therefore
reaches its minimum one to two seasons into a crisis — the exact moment the player most needs it.
Every player who experiences this once understands counter-cyclical fiscal policy without the term
ever being used.

---

# PART VII — ENVIRONMENTAL SYSTEMS WEB

## VII.1 The water cycle, closed

```
        ┌──────────────── PET (temp, wind, sun) ───────────────┐
        │                                                      ▼
   RAINFALL ──┬──► INTERCEPTION (forest) ──► evaporated back ──┘
              │
              ├──► INFILTRATION ──► SOIL MOISTURE ──┬──► CROP TRANSPIRATION ──► yield
              │      ▲                              ├──► DEEP PERCOLATION ──► GROUNDWATER
              │      │ (soil quality, cover)        └──► capillary rise ◄──────┘
              │                                                       │
              └──► SURFACE RUNOFF ──► RIVER FLOW ◄──── BASEFLOW ◄─────┘
                       │                  │
                       │                  ├──► ABSTRACTION ──► storage ──► household / irrigation
                       │                  ├──► fish habitat
                       │                  └──► FLOOD (when stage > bankfull)
                       └──► EROSION ──► SEDIMENT ──► turbidity, siltation, channel aggradation
```
Every arrow is an edge in Part II. **The cycle closes**: abstraction returns as return-flow, forest
interception returns as evaporation, percolation returns as baseflow. A water-balance assertion
should be added to the test suite — total in minus total out minus Δstorage ≈ 0 each season, within
tolerance. If it drifts, a system is leaking water and the simulation is lying.

## VII.2 The carbon and vegetation web

```
FOREST DENSITY ──┬──► canopy interception ──► runoff ↓ ──► flood risk ↓
                 ├──► root binding ──► erosion ↓ ──► soil quality ↑ ──► infiltration ↑ ─┐
                 ├──► litterfall ──► organic matter ──► water holding ↑ ────────────────┤
                 ├──► shade ──► local temp ↓ ──► PET ↓ ──► soil moisture ↑ ─────────────┤
                 ├──► habitat ──► biodiversity ↑ ──┬──► pollinators ──► yield ↑         │
                 │                                 └──► natural pest control ──► yield ↑│
                 ├──► carbon stock ↑ (🔵)                                               │
                 └──► fuelwood supply ──► household energy ──► (clearing pressure) ◄────┘
                                                                    ⟲ R7
```
Note the last edge: **forest is also a resource households consume.** Protection is not free —
it removes an income and an energy source from the poorest households. Any implementation that
makes Tree Protection costless has removed the dilemma that makes it interesting.

## VII.3 The pollution and health web

```
FOSSIL GENERATION ──┐
WASTE BURNING ──────┼──► EMISSIONS ──► AIR QUALITY ↓ ──┬──► respiratory burden ↑ ──► health ↓
COOKING FUEL (🟡) ──┘                    │             └──► irradiance ↓ ──► solar output ↓ ⟲ R12
                                         └──► (rain washout) ──► deposition

UNCOLLECTED WASTE ──► leachate ──┐
POOR SANITATION ─────────────────┼──► WATER QUALITY ↓ ──┬──► disease risk ↑ ──► health ↓
LIVESTOCK DENSITY ───────────────┤                      ├──► fish stock ↓
FLOOD EVENT ─────────────────────┘                      └──► usable water ↓ ──► water security ↓
                                    ▲
                          LOW RIVER FLOW (less dilution)
```
**The concentration term is the important part.** The same waste load in a drought-shrunk river is
several times more dangerous than in a full one. Drought and sanitation are not independent
problems, and the graph says so.

## VII.4 The nutrient and soil web

```
CROP RESIDUE ──┐
LIVESTOCK MANURE ──┼──► NUTRIENT RETURN ──► SOIL QUALITY ──┬──► yield ↑
FALLOW PERIOD ─────┘                              ▲        ├──► water holding ↑ ──► drought buffer
                                                  │        ├──► infiltration ↑ ──► recharge ↑
DROUGHT STRESS ──┐                                │        └──► runoff ↓ ──► erosion ↓ ──┐
FLOOD SCOUR ─────┼──► DEGRADATION ────────────────┘                                       │
EROSION ─────────┴──────────────────────────────────────────────────────────────────────◄─┘
CONTINUOUS CROPPING (🟡)
```
**Soil is the slowest and most consequential stock in the environmental layer.** It is the only
one where loss can be genuinely permanent, and it is invisible on every dashboard.

## VII.5 Cross-domain couplings that surprise people

| Coupling | Path | Why it is non-obvious |
|---|---|---|
| Forest → school attendance | forest ↓ → flood ↑ → farmland damage → income ↓ → child labour ↑ | Four hops, two domains, two seasons |
| Solar → fish stock | solar → no emissions → clearer air → *and* no fuel cost → budget → water treatment → fish habitat | Energy choice reaching ecology through fiscal space |
| Sanitation → yield | sanitation ↑ → disease ↓ → labour availability ↑ → timely planting → yield ↑ | Health as an agricultural input |
| Education → flood risk | education ↑ → awareness ↑ → protection compliance ↑ → forest ↑ → runoff ↓ | Fifteen-year lag, entirely real |
| Bridge → nutrition | bridge down → imports blocked → price ↑ → intake ↓ | Infrastructure as a nutrition variable |
| Livestock → water quality | herd density → manure load → river contamination → disease | The buffer that becomes a hazard |

These six are the Systems Atlas's best demonstrations, and they are exactly the connections the
player is meant to *discover* rather than be told.

---

# PART VIII — NETWORK ANALYSIS

## VIII.1 Highest-leverage variables

Leverage = (systems reached within 2 hops) × (player controllability) × (persistence of effect).
This is the ranked answer to *"if I could change one number, which one?"*

| Rank | Variable | Reach | Control | Persistence | Why |
|---|---|---|---|---|---|
| 1 | `ECO-01.forestDensity` | 14 | direct | 8–15 y | Touches flood, erosion, soil, biodiversity, temperature, income. The only MVP action whose payoff is visible inside a demo session. |
| 2 | `HYD-02.groundwater` | 11 | indirect | 3–8 y | Gates irrigation and dry-season flow. Cannot be bought back. |
| 3 | `ECO-02.soilQuality` | 9 | slow | 5–15 y | Sets the yield ceiling at every future rainfall. |
| 4 | `ECN-10.budget` | 12 | direct | 1–2 y | Gates every other action. Falls when needed most. |
| 5 | `ECN-04.householdSavings` | 8 | via subsidy | 3–8 y | Decides who survives a shock intact. |
| 6 | `INF-12.infrastructureCondition` | 10 | direct, dull | 10–30 y | Silently multiplies or divides every capacity in the settlement. |
| 7 | `SOC-14.cohesion` | 9 | indirect | 3–8 y | Decides whether shocks are absorbed collectively or individually. |
| 8 | `ECO-06.fishStock` | 6 | via effort | 4–10 s | The buffer that vanishes exactly when relied on. |
| 9 | `HYD-05.storageCapacity` | 7 | direct | 10+ y | Converts a volatile river into a reliable supply. |
| 10 | `SOC-06.education` | 8 | direct | 15+ y | Longest payoff in the game; compounds through every later decision. |

**The pattern worth naming:** the top ten are dominated by *stocks*, not flows. Flows are what the
player watches; stocks are what determines outcomes. A UI that surfaces only flows will produce
players who optimise the wrong thing.

## VIII.2 Most influential systems (out-degree, weighted)

`HYD-03 River` (11) → `GOV-01 Policy` (12) → `ECO-01 Forest` (8) → `ECN-03 Income` (8) →
`SOC-09 Migration` (7) → `ECN-01 Price` (7) → `SOC-10 Population` (7) → `ECN-10 Budget` (6)

## VIII.3 Most vulnerable systems

Vulnerability = in-degree × (1 / recovery speed) × absence of internal buffer.

| System | Why vulnerable | Buffer available? |
|---|---|---|
| `ECO-06 Fish Stock` | Non-linear collapse threshold; effort rises as stock falls | Only effort control, which requires income |
| `HYD-02 Groundwater` | Invisible drawdown, decade-scale recharge | Harvesting, efficiency, forest |
| `ECN-04 Savings` | Drains fast, refills slowly, no floor | Subsidy, livestock, remittances |
| `SOC-06 Education` | One interrupted year is not recoverable | None in-year |
| `ECO-02 Soil` | Loss can be permanent (erosion) | Residue, fallow, cover |
| `ECN-09 Enterprise` | Closes fast, reopens slowly | Credit, power reliability |
| `SOC-14 Cohesion` | Migration severs it faster than time rebuilds it | Market, communal space, fair governance |

## VIII.4 Critical infrastructure & single points of failure

| SPOF | Systems that stop | Redundancy available |
|---|---|---|
| **The bridge** | trade, emergency response, market access, school access, clinic access, price stability | 🟡 second crossing; 🟡 improved roads |
| **Power** | clinic capacity, irrigation pumps, water pumping, enterprise, cold chain | Mixed generation; 🔵 storage |
| **The water tower** | household water security, clinic function, drought buffer | Wells + harvesting (partial) |
| **The market** | vendor livelihoods, price discovery, food distribution, social cohesion | None modelled — deliberately |
| **The budget** | *every* player action | Broad tax base, reserves, aid |

**Design note.** The market has no redundancy on purpose. Some institutions genuinely have no
substitute, and a simulation that provides a fallback for everything teaches that nothing is
irreplaceable — which is false and is the opposite of the intended lesson.

## VIII.5 The complete loop register

### Reinforcing loops (R) — these amplify; they are the danger and the recovery

| ID | Name | Path | Doubling / halving time | Break it by |
|---|---|---|---|---|
| **R1** | Climate–yield spiral | drift → rainfall ↓ → yield ↓ → income ↓ → less adaptation investment → exposure ↑ | ~3 seasons | Irrigation, storage, diversification |
| **R2** | Outflow spiral | migration → labour ↓ → yield ↓ → income ↓ → migration ↑ | ~2 y | Subsidy, employment, cohesion |
| **R3** | Poverty trap | income ↓ → nutrition ↓ → health ↓ → productivity ↓ → income ↓ | ~2 seasons | Health access, food transfer, savings |
| **R4** | Fiscal death spiral | budget ↓ → maintenance ↓ → capacity ↓ → outcomes ↓ → tax base ↓ → budget ↓ | ~3 y | Maintenance discipline, reserves, aid |
| **R5** | Market thinning | vendors leave → market ↓ → vendor income ↓ → vendors leave | ~2 y | Market investment, demand support |
| **R6** | Irrigation confidence trap | irrigation → yield ↑ → confidence ↑ → expansion → groundwater ↓ → yield ↓↓ | ~4 seasons, **delayed** | Abstraction cap tied to recharge |
| **R7** | Clearing–income loop | poverty → fuelwood clearing → income ↑ → but flood ↑ → poverty ↑ | ~2 y | Alternative energy, protection + compensation |
| **R8** | Erosion–regrowth lock | forest ↓ → erosion ↑ → soil ↓ → regrowth harder → forest ↓ | ~5 y | Terracing 🔵, cover crops, protection |
| **R9** | Aquifer–vegetation loop | groundwater ↓ → deep roots stressed → canopy ↓ → infiltration ↓ → recharge ↓ | ~5 y | Draw reduction, forest protection |
| **R10** | Generational education loop | attendance ↓ → attainment ↓ → adult income ↓ → their children's attendance ↓ | **15 y** | School quality, food transfer during shock |
| **R11** | Commons collapse (fisheries) | crop failure → effort ↑ → stock ↓ → income ↓ → effort ↑ | ~4 seasons | Alternative income, effort limits |
| **R12** | Fossil lock-in | fossil → haze → solar output ↓ → more fossil needed | ~2 y | Solar overbuild, air-quality action |
| **R13** | Sanitation–health loop | waste ↑ → disease ↑ → productivity ↓ → income ↓ → collection unaffordable → waste ↑ | ~2 y | Collection as a fixed budget line |
| **R14** | Land degradation | distress sale of livestock → manure ↓ → soil ↓ → yield ↓ → distress sale | ~4 y | Herd rebuilding support, residue return |
| **R15** | Governance decay | education ↓ → governance capacity ↓ → policy effectiveness ↓ → outcomes ↓ → trust ↓ → compliance ↓ | ~8 y | Education, visible delivery |
| **R16** | Care spiral | health ↓ → clinic load ↑ → per-patient care ↓ → recovery ↓ → health ↓ | ~2 seasons | Capacity + power |
| **R17** | Legitimacy spiral | poor response → trust ↓ → tax compliance ↓ → budget ↓ → worse response | ~3 y | Preparedness (visible competence) |
| **R18** | Deprivation–commons loop | poverty → short-horizon choices (burn, clear) → environment ↓ → poverty ↑ | ~3 y | Income floor, cheap clean alternatives |
| **R19** | Insecurity loop | unemployment → crime ↑ → enterprise ↓ → employment ↓ | ~2 y | Employment, cohesion |

### Balancing loops (B) — these stabilise; they are what recovery is made of

| ID | Name | Path | Strength |
|---|---|---|---|
| **B1** | Hydrological negative feedback | flow ↓ → abstraction physically limited → flow preserved | Strong, automatic |
| **B2** | Aquifer ceiling | groundwater ↓ → irrigation ceiling ↓ → draw ↓ | Strong but **late** |
| **B3** | Trade arbitrage | price ↑ → imports ↑ → price ↓ | Strong *if the road is open* |
| **B4** | Forest–flood regulation | forest ↑ → runoff ↓ → flood ↓ → farmland preserved | Strong, slow, the best investment available |
| **B5** | Savings buffer | shock → savings drawn → consumption maintained | Moderate; depletes |
| **B6** | Runoff–flood damping | soil quality ↑ → infiltration ↑ → peak flow ↓ | Moderate |
| **B7** | Fiscal virtuous cycle | investment → employment → income → tax → budget | **The only strong balancing loop the player drives directly** |
| **B8** | Cohesion absorption | shock → mutual aid → migration ↓ → cohesion preserved | Moderate; erodes under repeated shock |
| **B9** | CPUE self-limitation | stock ↓ → catch per trip ↓ → effort eventually ↓ | Weak and **too late** — the reason fisheries need policy |
| **B10** | Soil–moisture buffer | soil quality ↑ → water holding ↑ → drought impact ↓ | Slow, powerful |
| **B11** | Air quality recovery | source removed → deposition and washout → quality ↑ | Moderate, 2–4 seasons |
| **B12** | Preparedness damping | preparedness ↑ → damage ↓ → emergency spend ↓ → budget preserved | Strong, invisible until it fires |

### Delayed, hidden and runaway classification

- **Delayed loops** (effect ≥ 1 year after cause): R6, R8, R9, R10, R15, B4, B10 — these are the
  loops players cannot learn from experience alone within one playthrough. **The Atlas exists for
  these.**
- **Hidden loops** (no visible intermediate state): R9 (aquifer–vegetation), R12 (fossil lock-in),
  R15 (governance decay), R18 (deprivation–commons). Each needs at least one deliberate
  observable — a dashboard trend or a world-render cue — or it will read as unfairness.
- **Runaway risk** (no balancing partner at high gain): R2, R4, R11. These three need explicit
  tuning attention: each must have at least one policy lever that measurably bends it, or the
  Design Bible's "hope" requirement (§1.6) fails mechanically rather than narratively.
- **Fastest-paying resilience loop:** B4 via forest planting — the only MVP loop whose full arc
  fits inside a 30–45 minute demo session. §XIII of the Design Bible makes this the milestone-3
  acceptance test, and this document concurs.

## VIII.6 Domino and butterfly effects

**Domino (deterministic cascade).** Bridge down → response delayed → flood damage un-buffered →
farmland lost for two seasons → income ↓ → child labour ↑ → attendance ↓ → attainment ↓.
Eight systems, one binary state change, no randomness required.

**Butterfly (small input, disproportionate outcome).** A **five-tick shift in rainfall onset**,
with unchanged seasonal total, can cost 30–40% of yield if it lands across the germination window.
Same water, different week, half a harvest. This is the most honest single demonstration of why
"climate change" is not adequately described as "less rain", and it should be a scripted demo
scenario.

**Threshold effects (small change, state transition).**
`groundwater 0.16 → 0.14` — irrigation ceiling collapses ·
`stock 0.21 → 0.19` — fishery recruitment fails ·
`stage bankfull ± 2 cm` — flood or no flood ·
`forest 0.26 → 0.24` — runoff crosses into flood-generating regime.
Each is a cliff. The UI should never mark them explicitly (that would do the player's inference
for them), but the world must show the approach — a visibly dropping well, thinning catch, rising
water — so the fall is legible in hindsight. **Fairness is achieved by making the approach
observable, not by warning about the cliff.**

---

# PART IX — SIMULATION MATHEMATICS

## IX.1 The fidelity rule

Every relationship in this document is implemented at the *simplest* level that preserves
direction, relative magnitude and timing. This is not a compromise — it is the correct engineering
choice for a system whose purpose is teaching structure, not predicting numbers.

| Do use | Do not use |
|---|---|
| Weighted coefficients from JSON | Peer-reviewed process models |
| Logistic growth for stocks | Age-structured population matrices |
| Surplus-production fisheries | Multi-cohort virtual population analysis |
| Hargreaves-style PET | Penman–Monteith with full met inputs |
| USLE-skeleton erosion | Physically-based sediment transport |
| SIR-lite outbreak curve | Full compartmental epidemiology |
| Supply/demand power law | General equilibrium models |
| State machines for discrete states | Continuous PDEs |

**Test:** would a domain expert watching for five minutes say anything moves *backwards*? If no,
the fidelity is sufficient. If yes, fix the direction — never the decimal places.

## IX.2 The six response shapes

Every edge uses exactly one. This keeps the codebase to six functions rather than four hundred
bespoke formulas.

```js
lin (x, k)          => k * x                                  // proportional
sat (x, k, half)    => k * x / (x + half)                     // diminishing returns  (Michaelis–Menten)
thr (x, lo, hi)     => smoothstep(lo, hi, x)                  // soft threshold, never a hard if()
exp (x, k)          => Math.exp(k * x) - 1                    // accelerating (temperature, disease)
mult(factors[])     => factors.reduce((a,b) => a*b, 1)        // multiplicative limitation (yield)
prob(p)             => seededRandom() < p                     // discrete events (flood, outbreak)
```

**Rule: no bare `if (x > threshold)` in simulation code.** Every threshold is a `thr()` with a
transition band. Hard thresholds produce visual popping, un-debuggable oscillation at the boundary,
and a world that feels arbitrary. Soft thresholds produce a world that feels like it is *becoming*
something, which is the entire pacing thesis of §1.6.

## IX.3 Stock-and-flow integration

Every stock uses the same explicit Euler form, which is stable at these time steps and readable by
anyone:
```js
stock += (inflow - outflow) * dt;
stock  = clamp(stock, min, max);
```
For stocks with natural regeneration, logistic form:
```js
stock += r * stock * (1 - stock / capacity) * dt - harvest;
```
Applies to: forest density, fish stock, livestock, soil quality, groundwater (with capacity),
enterprise stock, cohesion. **Seven systems, one function.**

## IX.4 Delay implementation

Three mechanisms, chosen by what the delay physically *is*:

| Delay type | Implementation | Used for |
|---|---|---|
| **Transport** | ring buffer, fixed length | rainfall → river, price → purchasing |
| **Accumulator** | integrate then threshold | migration pressure, drought stress, attainment |
| **Exponential smoothing** | `y += (target − y) × rate` | health, prices, condition, trust |

The third is the workhorse and doubles as the renderer's smoothing (Design Bible §12.8): medium
and slow-tier values are interpolated toward their new value rather than snapping, which is what
makes a 90-tick update look continuous.

## IX.5 Stochasticity

```js
rng = mulberry32(scenarioSeed);        // ONE seeded generator for the whole simulation
```
Everything random draws from it. Consequences: a scenario is exactly reproducible for testing and
for a judge demo, while feeling non-deterministic to a player. Weather uses **lognormal** variance
(right-skewed: occasional very wet days, never negative), event triggers use `prob()`, and
per-household variation uses a fixed per-household offset drawn once at world generation so the
same household is consistently slightly luckier or unluckier — which reads as character rather
than noise.

## IX.6 Tuning surface

```
/data/simulation/
  climate.json      rainfall baselines, drift curve, PET coefficients, variance
  hydrology.json    infiltration, recharge, routing, flood thresholds, quality decay
  ecology.json      regrowth/clearing rates, logistic r and K, erosion coefficients
  agriculture.json  yield factors, crop coefficients, storage loss, labour demand
  economy.json      elasticities, price stickiness, tax rates, wage levels
  citizens.json     archetype baselines, need thresholds, migration parameters
  infrastructure.json capacities, depreciation, power curves
  actions.json      cost, magnitude, duration, delay for every player action
  loops.json        gain multipliers per named loop, for balance testing
```
`loops.json` is unusual and worth keeping: a per-loop gain multiplier lets a designer damp a
runaway loop **as a loop** during a balance pass, rather than hunting the six coefficients that
compose it. Default all to 1.0.

## IX.7 Numerical safety

| Hazard | Guard |
|---|---|
| Divide-by-zero on total crop failure | `max(supply, supplyFloor)` in every price expression |
| Negative stocks | `clamp(0, …)` after every integration |
| Runaway positive feedback | per-loop gain cap in `loops.json`; assertion on Δ/tick |
| Water non-conservation | seasonal balance assertion: in − out − Δstorage ≈ 0 |
| Population non-conservation | births + immigration − deaths − migration = Δpopulation, asserted |
| Compounding float drift over 10k+ ticks | integer tick counter; recompute derived aggregates rather than accumulating them |
| Threshold oscillation | hysteresis band on every state machine (enter at x, exit at x ± δ) |

## IX.8 Performance budget

| Tier | Systems | Cost/tick | Note |
|---|---|---|---|
| Fast | 11 | O(citizens + tiles) | Keep to arithmetic; no allocation in the loop |
| Medium | 18 | O(tiles) / 5 | Amortise across ticks: process 1/5 of tiles per tick |
| Slow | 26 | O(households) / 90 | Season boundary; a visible frame hitch here is acceptable and can be masked |
| Annual | 7 | O(n) / 360 | Free |

Household-level aggregation (Design Bible §6.5) is the main scaling lever: compute per-citizen
only where the individual reaction *is* the content.

---

# PART X — MASTER INTERACTION MATRIX

## X.1 Reading the matrix

Cell = effect of **row system** on **column system**.

| Code | Meaning |
|---|---|
| `··` | No interaction |
| `+` / `-` | Weak (weight < 0.3) |
| `++` / `--` | Strong (weight ≥ 0.6) |
| `↔` | Bidirectional |
| `»` | Delayed ≥ 1 season |
| `?` | Conditional (gated by a third system) |
| `⟲` | Participates in a named feedback loop |

Combinations are read together: `++»⟲` = strong positive, delayed, inside a loop.

## X.2 Core matrix — the 24 systems that carry the MVP

Abbreviated headers: RN rainfall · TM temperature · SM soil moisture · GW groundwater · RV river ·
FL flood · DR drought · FO forest · SQ soil quality · FS fish stock · CY crop yield · IR irrigation ·
FP food price · MK market · HI household income · SV savings · FSc food security · HL health ·
SCH school · MIG migration · POP population · PW power · BDG budget · INF infrastructure

| ↓ affects → | RN | TM | SM | GW | RV | FL | DR | FO | SQ | FS | CY | IR | FP | MK | HI | SV | FSc | HL | SCH | MIG | POP | PW | BDG | INF |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| **RN** | · | · | ++ | +» | ++ | ++⟲ | -- | + | · | + | ++» | · | -» | +» | +» | · | +» | · | · | -» | · | · | · | · |
| **TM** | · | · | -- | -» | - | · | ++ | -» | · | -» | --⟲ | + | +» | · | -» | · | -» | -- | · | +» | · | + | · | · |
| **SM** | · | · | · | +» | · | · | -- | + | +» | · | ++ | ↔ | -» | · | +» | · | +» | · | · | · | · | · | · | · |
| **GW** | · | · | + | · | ++» | · | -- | +» | · | · | +» | ++? | -» | · | +» | · | +» | + | · | -» | · | · | · | · |
| **RV** | · | · | + | ↔ | · | ++? | -- | + | · | ++ | +? | ++ | -» | + | +» | · | ++ | ++ | · | -» | · | · | · | + |
| **FL** | · | · | + | + | ↔ | · | · | - | --» | - | --⟲ | - | ++» | -- | --» | --» | --» | --» | -» | ++» | -» | - | --» | --⟲ |
| **DR** | · | · | -- | -- | -- | · | · | -» | --» | -- | --⟲ | ++? | ++» | -- | --» | --» | --» | -» | -» | ++⟲ | -» | · | --» | · |
| **FO** | +?» | -» | +» | +» | +» | --⟲ | -» | · | ++» | +» | +» | · | · | · | +» | · | +» | + | · | · | · | · | · | · |
| **SQ** | · | · | ++ | +» | · | -» | -» | +» | · | · | ++ | -» | -» | · | +» | · | +» | · | · | · | · | · | · | · |
| **FS** | · | · | · | · | · | · | · | · | · | · | · | · | -» | ++ | ++⟲ | + | ++ | + | · | -» | · | · | + | · |
| **CY** | · | · | · | · | · | · | · | · | -» | -⟲ | · | ↔ | --⟲ | ++ | ++ | ++» | ++ | +» | +» | --⟲ | -» | · | ++» | · |
| **IR** | · | · | ++ | --⟲ | -- | · | -- | · | · | -» | ++ | · | · | · | +» | · | +» | · | · | -» | · | -? | -» | · |
| **FP** | · | · | · | · | · | · | · | +» | · | +» | +↔ | · | · | -- | ↔ | --» | --⟲ | --» | --» | ++⟲ | -» | · | +» | · |
| **MK** | · | · | · | · | · | · | · | · | · | +» | ++ | · | ↔ | · | ++ | +» | + | · | · | -- | · | · | ++» | · |
| **HI** | · | · | · | · | · | · | · | -?» | · | +? | +» | ++ | +» | ++ | · | ++ | ++ | ++» | ++ | --⟲ | +» | · | ++ | +» |
| **SV** | · | · | · | · | · | · | · | · | · | · | +» | +» | · | + | +» | · | ++ | + | + | --⟲ | · | · | +» | · |
| **FSc** | · | · | · | · | · | · | · | -?» | · | +? | · | · | · | + | +» | ↔ | · | ++» | ++ | --⟲ | +» | · | · | · |
| **HL** | · | · | · | · | · | · | · | · | · | · | +» | · | · | + | ++⟲ | +» | +» | · | + | -» | ++» | · | -» | · |
| **SCH** | · | · | · | · | · | · | · | +» | · | · | +» | · | · | · | ++»» | · | · | + | · | -» | · | · | +»» | · |
| **MIG** | · | · | · | -» | · | · | · | -» | · | -» | --⟲ | · | +» | --⟲ | --⟲ | -- | -» | -» | -» | ⟲ | -- | · | --⟲ | -» |
| **POP** | · | · | · | -» | -» | · | · | -» | -» | -» | +» | · | ++ | ++ | +» | · | -» | -» | ++ | · | · | -- | ++ | -» |
| **PW** | · | · | · | -? | -? | · | · | · | · | · | +? | ++? | · | + | +» | · | + | ++? | + | · | · | · | -» | +» |
| **BDG** | · | · | · | · | · | -» | -» | +» | · | +» | +» | +» | -» | + | ++ | +» | ++ | ++» | ++ | --» | · | ++ | · | ++⟲ |
| **INF** | · | · | + | +» | + | --» | -» | · | +» | +» | ++ | ++ | -» | ++ | ++ | +» | ++ | ++ | ++ | --» | +» | ++ | -» | · |

**Reading examples.**
`FO → FL` = `--⟲`: forest strongly reduces flood, inside loop B4.
`IR → GW` = `--⟲`: irrigation strongly depletes groundwater, inside loop R6 — *the cell that
contains the whole irrigation lesson*.
`SCH → HI` = `++»»`: school attendance strongly raises income, delayed **twice** — a decade out.
`FP → CY` = `+↔`: price raises yield (planting incentive) while yield lowers price. Bidirectional,
and the reason agricultural markets oscillate.

## X.3 Density statistics

| Metric | Value | Reading |
|---|---|---|
| Systems | 84 | 53 MVP · 29 V2 · 2 Future |
| Directed edges (transcribed) | 254 | `/data/systems-graph.json`; more specified in Part II prose |
| Mean degree | 6.0 | Every system averages nearly six connections |
| Graph density | 0.036 | Sparse enough to reason about, dense enough to surprise |
| **Systems inside a mutual cycle** | **65 of 84** | 77% of the simulation can reach itself back — *the graph really is one organism* |
| Named loops | 31 (19 R, 12 B) | |
| Systems with zero out-edges | 8 | SOC-18 by design; the rest are V2/FUTURE stubs awaiting their consumers |
| Systems with zero in-edges | 8 | CLI-01/CLI-02 exogenous by design; the rest are V2 stubs awaiting their drivers |
| Top hub by in×out | ECN-03 Household Income (15 in × 8 out) | Every environmental shock becomes human here |
| Runner-up | AGR-01 Crop Yield (12 × 6) | The transducer |

**Empty-degree systems are a build signal, not a modelling error.** A V2 system with zero
transcribed in-edges means its drivers have not been transcribed yet; a V2 system with zero
out-edges means nothing consumes it yet. Both must be non-zero before that system is considered
implemented — this is the Design Bible's §12.12 merge question, made checkable by script.

**The headline number is the mutual-cycle count.** Sixty-five of eighty-four systems sit on at
least one cycle that returns to them. That is not an accident of enumeration — it is the
mathematical statement of the Design Bible's thesis. Change one thing and, in principle, the
change comes back round to it.

## X.4 The complete edge list

The edge list is maintained as data, not prose, in **`/data/systems-graph.json`** — `from`, `to`,
`sign`, `weight`, `lag`, `shape`, `mechanism`, `loops`. That file is the single source of truth:
this matrix is projected from it, the Systems Atlas renders from it, and the simulation loads its
coefficients from it. It currently transcribes **254 edges** covering every MVP system and every
named loop; the remaining V2/FUTURE edges are specified in Part II prose and are transcribed as
those systems are built.

**Four consistency checks must pass in CI** (`node tools/check-graph.js`):
1. Every edge's `from`/`to` resolves to a declared system — *currently 0 dangling*.
2. Every system declares a tier, scope, and source JSON file.
3. Every V2→MVP edge with weight ≥ 0.6 is registered as an additive refinement — *currently 10,
   all enumerated in Part I*.
4. Every system on a named loop's `path` exists, and every loop closes.
5. Every 🟢 MVP system has at least one driver and one consumer — *this check found three
   genuinely unwired MVP systems on first run (INF-05, ECO-09, SOC-18) and they were fixed.
   The check earns its place.*

---

# APPENDIX A — IMPLEMENTATION ORDER

Derived from the graph, not from enthusiasm. Build in dependency order; each stage is testable
before the next begins.

| Stage | Systems | Acceptance test |
|---|---|---|
| 1 | CLI-02, CLI-03, CLI-04, CLI-07 | Rainfall varies by season; drift dial changes it plausibly |
| 2 | HYD-01, HYD-03, HYD-04, HYD-02 | Water balance closes to tolerance over a simulated year |
| 3 | ECO-01, ECO-02, ECO-09 | Forest clearing measurably raises runoff within a year |
| 4 | AGR-01, AGR-06, HYD-08 | **Chain D1 to the yield node** reproduces end to end |
| 5 | ECN-01, ECN-02, ECN-03 | Yield failure produces a price spike and differentiated income effects |
| 6 | SOC-01, SOC-04, SOC-07, SOC-08, SOC-09 | Ama/Kwame/Nana reproduce: three archetypes, one shock, three outcomes |
| 7 | ECN-10, ECN-11, GOV-01 | Budget contracts under crisis; action ceiling falls |
| 8 | INF-01, INF-02, ENV-01, ENV-02, INF-08 | **Chains E1 vs E2 legible side by side in one session** |
| 9 | ECO-06, AGR-04 | Fishery collapse reproduces under combined effort and drought |
| 10 | INF-04, HYD-07, GOV-02, GOV-03 | Chain F2 reproduces: flood, bridge, delayed response |
| 11 | Remaining 🟡 | Each addition changes an outcome, or it is cut |

# APPENDIX B — WHAT THIS DOCUMENT DELIBERATELY DOES NOT CONTAIN

- **No gameplay design.** Not one screen, control or feature. Those live in the Design Bible.
- **No narrative.** Ama, Kwame and Nana appear only as archetype references.
- **No difficulty tuning.** Coefficients here are *plausible defaults*, not balance decisions.
- **No prediction claims.** This is a plausible, internally consistent, directionally correct
  approximation. It is not a forecast and must never be presented as one.
- **No real place or person.** The settlement and its citizens are fictional composites, per
  Design Bible §1.9. Real climate data may calibrate starting conditions; the resulting village is
  never presented as a documentary record of anywhere.

---

*Companion volume to the Earth Engine Internal Design Bible. Living document — when the code and
this document disagree, one of them is a bug, and §14 of the Design Bible says to fix both.*

# Garden Irrigation Plan

**Property:** 9 Heather Garden, Newbury  
**Plot:** ~21m × 21m (441m²)  
**Water source:** Linked water butts (SE of house) — **4×200 L = ~800 L** now; optional expand to **8×200 L = ~1600 L** (§5f)  
**Pump (owned):** [Hozelock Jet 3000 K7](https://www.hozelock.com/product/hozelock-jet-3000-k7/)  
**Water-butt level (owned):** AJ-SR04M waterproof ultrasonic → ESP8266 + ESPHome → Home Assistant  
**Drawing reference:** Proposed Garden Design, Mark Newell, Feb 2001 (`HG Garden.png`)

---

## 1. What this system is for

Mature ornamental garden with **spaced** shrubs/trees around a lawn. Water each plant with **pressure-compensating (PC) drippers**, not a soaker along the gaps.

**Goal:** drought insurance — deep soak at rootballs.

**Out of scope:** lawn · lavender hedge (hand-water) · soaker as main method

---

## 2. Zones: **one zone is enough**

**Decision: 1 zone — whole garden.** You asked for the simplest setup; hydraulics support it.

| Option | Flow | Verdict |
|--------|------|---------|
| **1 zone — whole garden** | **~120 L/h** (ordered) | **Recommended for you** — pump, pipe and butts all cope |
| 2 zones — East / West | ~100 L/h each | Only if you later want to water one side alone |
| 5 zones | ~30–60 L/h each | Unnecessary with drippers |

### Why one zone works

| Check | Number | Limit | OK? |
|-------|--------|-------|-----|
| Emitter demand | **~120 L/h** (~2 L/min) | Jet 3000 = **3000 L/h** | Yes — ~4% of pump capacity |
| Pressure after reducer | **~1.5 bar** at hub | CETA PC window **~1–3.5 bar** | Yes |
| Friction (see layout below) | Each fence branch ~**100 L/h** | 16 mm LDPE over ~35–70 m | Yes — same as old “half garden” case |
| Water per ~60 min run (80 drippers) | **~240 L** | Butts **~800 L** (→~1600 L if expanded) | Yes |
| Presscontrol / dry-run | **~2 L/min** continuous | Needs real flow, not dead-head | OK — keep network open while watering |

### Trade-off (accept this)

- Every timed run waters **all** drippers — you cannot soak only the west border without adding a shut-off later.  
- For drought insurance on a mature ornamental garden, that is usually fine: run **60–90 min** when plants are stressing; skip days when they aren’t.  
- Optional later: fit a cheap valve mid-spine if you ever want to shut off one side (not required for day one).

### Fence-only layout (no pipe on the house)

West is reached by going **around the fences**: east → north → west (~55–60 m U-loop). That is still **one zone**.

If the far west end is weak on test, use **Option B** in §7b (second 16 mm on the east fence so west is a separate ~100 L/h branch). Start with the simple single U-spine (§7b Option A).

---

## 3. Plant count → water budget (single zone)

Counted **drawn plant icons** on the plan (not name labels). Excluded lavender band and cobbles.

| Build (ordered) | Emitters | Flow |
|-----------------|----------|------|
| **CETA red 2 L/h** | **20** | **40 L/h** |
| **CETA black 4 L/h** | **20** | **80 L/h** |
| **One zone total** | **40** | **~120 L/h** |

| Run time | Water used | vs ~800 L store |
|----------|------------|-----------------|
| **60 min** | ~**120 L** | Fine |
| **75 min** | ~**150 L** | Fine (design soak) |
| **90 min** | ~**180 L** | Fine |

**Emitter rule of thumb:** **2 L/h** on smaller shrubs/pots · **4 L/h** on larger shrubs/trees.  
Plants are **≤1 m from the fence** → **50 m of 4 mm** is enough (~1.0–1.2 m per drop).  
Original drawing count was ~52 plants / ~100 emitters — Stage 1 uses **40 drippers**; add more later if needed.

### Assigning 2 L/h vs 4 L/h (walk-round)

| Use **2 L/h** (red) | Use **4 L/h** (black) |
|---------------------|------------------------|
| Small / medium shrubs | Large shrubs / trees |
| Pots & containers | Plants that wilt first in drought |
| Newly planted, modest size | Wide canopy / deep roots |
| Dense planting | Sparse big specimens |

In a **75 min** run: ~**2.5 L** vs ~**5 L** per plant. In a **60 min** morning run: ~**2 L** vs ~**4 L**.  
Doubt → start **2 L/h**; upgrade later if that plant still stresses. CETA PC drippers have a **flow direction** — fit the right way round.

---

## 4. Pipe length & far-end drippers (up to ~70 m)

**Yes, length matters — but with this design, the last drippers still work.**

Pressure falls along the hose (friction). What the last dripper “sees” is start pressure minus that loss.

| Setup | At ~70 m to the last plant |
|-------|----------------------------|
| **PC drippers + 16 mm (or 20 mm) spine + ~1.5 bar at hub** | **OK** — far emitters still ~2 L/h |
| Non-PC drippers | Far end gets less water |
| Long run of thin **4 mm** tube as the main line | Bad — large pressure loss |

**Why it works here (1-zone / fence U-loop)**
- Total flow **~120 L/h** on one 16 mm perimeter spine (~55–60 m)
- Set ~**1.5 bar** at the hub; **PC** drippers hold ~**2 L/h** if far-end pressure stays in range
- **Stage 1 must cup-test the last west-fence dripper** — if weak, fit Option B (parallel east-fence feed) so each path ~**100 L/h**

**Rules**
- Use **16 mm LDPE** on fences only — not 4 mm as the main line  
- Use **4 mm only** for short drops to each plant  
- **Must** use **PC** drippers — especially far west / NW corners  
- No spine on the house wall

---

## 5. Pump much bigger than dripper demand — will it complain or break things?

The Jet 3000 can do **3000 L/h / 3.5 bar**. The ordered build only needs **~120 L/h**. That mismatch is normal and manageable.

### What does *not* happen
The pump does **not** force 3000 L/h through the drippers. Flow is set by the emitters + pressure reducer. Demand is low → the pump simply runs toward the high-pressure / low-flow end of its curve.

### Two separate risks

**A) Pipework & drippers (over-pressure)**  
3.5 bar can stress or blow cheap fittings and microtube.  
**Fix:** pressure reducer to **~1.5 bar** after the pump. With that fitted, low dripper demand does **not** smash the irrigation side.

**B) Pump itself (heat / cycling)**  
Low flow → less water cooling the pump → it can run warmer. A Presscontrol may short-cycle if flow is tiny or if you dead-head against closed valves.  
**Fix:**
- Fit a **Presscontrol** (auto on/off + dry-run protection)  
- Keep the network **open** while watering so there is real flow (**~120 L/h / ~2 L/min** — enough for Presscontrol)  
- Don’t leave the pump powered against a **closed** master valve for long  
- Optional small accumulator helps reduce cycling (nice-to-have, not required to start)

```
Jet 3000 max ──────────── 3000 L/h / 3.5 bar
After reducer ──────────── ~1.5 bar into the drip network
Whole garden (1 zone) ─── ~120 L/h  ← ordered design point
```

---

## 5b. Jet 3000 K7 — what goes where (fixed positions)

Verified against the official [Hozelock Jet 3000 K7 booklet](https://www.hozelock.com/download/jet-3000-k7/) (SKU **7606**).

### What’s in the box (use these — don’t invent extras)

| Letter | Part | Role |
|--------|------|------|
| **A** | Pump body | Sits on the ground by the butts |
| **B** | Power cable (~1.5 m+) | To RCD socket / Presscontrol |
| **C** | Carry handle | Lift by this, never by the cable |
| **L** | Inlet (suction) | Where the butt hose connects |
| **D** | Outlet (pressure) | Where Presscontrol / irrigation starts |
| **I / K** | Filler cap / fill hole | Prime here once with clean water |
| **M** | Drain cap | Winter empty only — leave shut in season |
| **O** | On/off switch | Leave **ON** if a timer/Presscontrol switches power |
| **P** | Spanner | For filler cap |
| **S** | 1″ male/male coupler | Inlet adaptor |
| **T** | **7 m suction hose** | Butts → pump (coil spare; keep run short) |
| **Q** | **Non-return valve** | On suction end — keeps hose full when pump stops |
| **R** | **Strainer / filter** | On end in the butt — stops leaves/grit |
| **G / F / H** | Curved outlet connector + nut + high-flow coupling | **G + F** on outlet; **H not used** (see below) |

### Pump threads (confirmed on site)

| Port | Measurement | Thread |
|------|-------------|--------|
| **Inlet (L)** | ~29.4 mm ID | **1″ BSP female** — supplied **S** coupler screws straight in |
| **Outlet (D)** | via curved **G** | **1″ BSP** body port; **G** is stepped for 25/32 mm hose **or** 1″ female + standard hose fittings |

The supplied **H** (high-flow coupling) is for **large-bore** hose (15–32 mm), **not** standard 12.5 mm garden hose. **Do not use H** for the irrigation hub.

### Hub connection — **decision: standard small Hozelock connector**

Use the **normal Hozelock garden-hose snap fitting** (12.5 mm bore) on curved connector **G**, not the high-flow **H** coupling.

Per the Jet 3000 booklet, **G** can take a **1″ female tap nose** and standard quick hose fittings. Cut off the unused stepped end of **G** (Fig. 4 in manual) if needed to expose the 1″ thread section.

```
Pump outlet (G + F)
  → standard Hozelock hose-end connector (small / garden size)
  → short reinforced garden hose (~30–50 cm)
  → 1″ BSP female hose tail  →  KATSU Presscontrol inlet
  → 1″ BSP female hose tail  →  short hose  →  Gardena Master Unit inlet (Hozelock-style snap)
  → 13 mm LDPE out to east fence
```

| Link | Fitting |
|------|---------|
| Pump → hose | **Standard Hozelock hose connector** on **G** (owned / buy one if missing) |
| Hose → KATSU | **1″ BSP female → hose barb** (×2 — inlet and outlet sides of KATSU) |
| KATSU → Gardena | Short garden hose with **Hozelock snap** ends, or snap on Gardena inlet adaptor |

KATSU **151024** ports are **1″ BSP female** — screw **male** hose tails in. Gardena Master Unit inlet accepts the same Hozelock-style snap as standard garden hose.

Order on the **suction** side (Hozelock assembly):

```
[inside butt]  R strainer → Q non-return → T hose → S coupler → L pump inlet
```

### Specific recommendation — put the pump HERE

**Sit the pump on the ground (patio / hardstanding) right next to the water butts.**  
Not on a shelf, not on the fence, not up the wall.

```
        ┌──────── butt(s) ────────┐
        │                         │
        │    R strainer  ●        │  ← fixed: ~30 cm ABOVE butt floor
        │         │               │
        └─────────┼───────────────┘
                  │ short suction hose (no kinks)
                  ▼
            ┌───────────┐
   ground → │ Jet 3000  │  ← pump ON THE GROUND beside butts
            │   (A)     │
            └─────┬─────┘
                  │ outlet
                  ▼
         Presscontrol → 1.5 bar reducer → filter → master valve → 16 mm fence spine
```

| Item | Fixed place | Why |
|------|-------------|-----|
| **Pump** | **Ground** beside butts | Short suction path; easy to keep primed for timer use |
| **Strainer (R)** | Fixed height **~30 cm above the butt floor** (brick / weight so it can’t float up) | Hozelock: keep off the sludge layer at the bottom |
| **NRV (Q)** | On the suction assembly as supplied | Holds water in the hose when the pump stops |
| **Spare suction hose** | Coiled on the ground | The 7 m is for reach — don’t create extra height |

### Ignore “match the water level” — do this instead

The water surface moves. You do **not** chase it.

1. **Fix the strainer ~30 cm above the butt floor** and leave it there.  
2. When the butts are reasonably full, the strainer is underwater → pump works.  
3. When the water drops to near the strainer, **stop watering** (or top up). Running with the strainer in air = dry-run / loss of prime.  
4. The Presscontrol’s dry-run protection is a backstop, not a plan to empty the butts dry.

So: pump position is **fixed on the ground**. Usable water is “everything above the strainer”. Below that = empty for irrigation purposes.

Hozelock’s “150 mm below the surface” just means: **don’t run when the butt is nearly empty**. It is not a instruction to move the pump up and down.

### Legal limit (background only)

Max suction height is **7 m** (water surface → pump inlet). On the ground beside the butts you are nowhere near that. You only get into trouble if you mount the pump high on a wall.

### Priming — once per season (not every timer run)

1. Assemble suction: strainer + NRV + hose → pump inlet (all seals present, airtight).  
2. Open filler cap (**I**), fill pump body with clean water (**K**), refit cap. You do **not** fill the whole 7 m hose by hand.  
3. Outlet path open (master valve open / drippers free).  
4. Power on — first prime may take a few noisy minutes; stop and re-check seals if not primed in **5 minutes**.  
5. After that: leave drain cap shut, don’t break suction joints, leave switch **O** ON → timer/Presscontrol can power-cycle without re-priming **if the NRV holds**.

### Hub order (pressure side)

```
Pump outlet (G) → standard Hozelock hose snap → short garden hose
  → KATSU 151024 Presscontrol          ← auto on/off + dry-run; 1″ BSP ports
  → short garden hose (Hozelock snap)
  → Gardena Master Unit 1000           ← ~1.5 bar reducer + filter
  → NC 230 V brass solenoid            ← siphon stop; powered with pump (§5d)
  → 16 mm LDPE to east fence → U-spine
```

### Frost

Below **5°C**: unplug, open drain (**M**), store pump frost-free. Spring: reassemble, re-prime once.

---

## 5c. Water-butt level — AJ-SR04M + ESPHome

**Sensor:** **AJ-SR04M** waterproof ultrasonic (puck probe on cable). Same family as JSN-SR04T — standard `TRIG` + `ECHO` interface, **not** the pump’s high-flow **H** coupling.

| Spec | Value |
|------|-------|
| Interface | `TRIG` + `ECHO` (HC-SR04 compatible) |
| Range | ~**0.25–4.5 m** (blind zone ~25 cm below probe) |
| Mount | **Lid of water butt**, probe pointing **straight down** at water surface |
| Controller | ESP8266 D1 Mini Pro (existing `esp-home-waterbutt`) |
| Integration | ESPHome → Home Assistant |

### ESPHome (working config)

```yaml
sensor:
  - platform: ultrasonic
    trigger_pin:
      number: D1
      inverted: true
    echo_pin: D2
    name: "Waterbutt distance"
    id: waterbutt_distance
    update_interval: 300s
    unit_of_measurement: "m"
    accuracy_decimals: 3
    filters:
      - median:
          window_size: 5
          send_every: 5
```

Add a **template sensor** for fill % — calibrate `empty_distance` and `full_distance` once with the butt empty-ish and full.

### Use in the watering schedule

| Reading | Action |
|---------|--------|
| Level **OK** (> ~25% above strainer) | Normal timer runs |
| Level **low** (approaching strainer) | **Skip** timed runs; alert in HA |
| Level **invalid** / blind zone | Probe too close to water — treat as “full enough” or reposition |

The ultrasonic sensor complements (does not replace) the **strainer height rule** in §5b: stop irrigation when water falls to ~30 cm above the butt floor.

**Install tips:** keep probe vertical; shield from inlet splash; small hood if condensation drips on the face; power sensor from **5 V** if the board allows.

---

## 5d. Siphon when pump is off — NC solenoid (automated)

**Symptom (seen on install):** water keeps running through the spine/drippers after the pump stops. Started once the Gardena completed a continuous path to the emitters.

**Cause:** gravity feed from the butts, not the pump still running:

```
Butts (water high)
  → suction NRV (allows forward flow)
  → pump body → Gardena → spine → drippers
```

The suction NRV keeps the hose primed; it does **not** stop water leaving the butts downhill. A manual 13 mm valve would fix this but the system is **HA / Zigbee automated** — use a solenoid instead.

### Decision — brass NC 230 V solenoid

| Spec | Choice |
|------|--------|
| Type | **2-way normally closed (NC)** — closed with no power |
| Voltage | **230 V AC** (wire in parallel with the pump on the Zigbee plug) |
| Ports | **½″ BSP** |
| Product | [Amazon B09Y45K21L](https://www.amazon.co.uk/dp/B09Y45K21L) (~£14) — brass body |
| Rejected | GREDIA 12 V plastic ([B08KS6WHM2](https://www.amazon.co.uk/dp/B08KS6WHM2)) — works, but needs a separate 12 V PSU |

```
Zigbee plug (UpXNBor)
  ├─ Jet 3000 pump
  └─ NC solenoid coil (same live/neutral; keep connections dry / IP box)

… → Gardena → [NC solenoid, arrow → spine] → 16 mm
```

| Plug | Pump | Solenoid | Spine |
|------|------|----------|-------|
| **ON** | Runs | **Opens** | Waters |
| **OFF** | Stops | **Closes** | Siphon stopped |

**12 V NC valves** are also fine (cheaper) if you add a 12 V PSU on the same plug; prefer **0 bar / semi-direct** types. For this install, **230 V brass is preferred** (fewer parts, tougher body).

Optional: keep the owned **13 mm flow-control valve** as a manual isolate for maintenance — not required for daily automation.

---

## 5e. Commissioning — prove the hub before finishing drippers

**Install status (Jul 2026):** spine + Gardena fitted; drippers work once oriented correctly (CETA PC — flow direction matters). Cup-test: open 4 mm pours; fitted dripper is a **slow** drip (~2 L/h red ≈ 30–40 ml/min).

### Test order

1. **Flush spine** — open far end of 16 mm; pump on → strong stream within ~1 min.  
2. **Hub OK?** No/weak stream → fix Gardena direction, filter, valves, pump before more drippers.  
3. **Take-off OK?** Remove dripper — 4 mm should pour. No pour → re-punch take-off / unkink 4 mm.  
4. **Dripper OK?** Refit; wait 30–60 s — slow drip is correct. Nothing → rinse/swap dripper; check **orientation**.  
5. Fit remaining drippers in batches; cup-test **near + far west**.  
6. Fit **NC solenoid**; plug off → flow must stop within seconds.

**Presscontrol note:** one dripper (~2 L/h) is far below the ~2 L/min the Presscontrol likes — flush with the far end open, or test with several drippers fitted.

---

## 5f. Water butts, roof catchment & expanding storage

**Site:** 9 Heather Garden (satellite annotated — house in green; scale bar **5 m**).  
**Feed:** **two downpipes** (red arrows on plan photo) on the SE / street-side eaves → linked butt bank.  
**Current store:** **4 × 200 L ≈ 800 L** (linked at bottom).  
**Irrigation demand (Stage 2):** **~80 drippers** → ~**240 L/h** → **~240 L** per 60 min morning run.

### Will the roof fill the butts?

Rough catchment on the two marked downpipes (plan area of the roof slopes they serve): **~60–90 m²** (estimate from satellite + 5 m scale — not a survey).

| Rainfall on that catchment | Water into butts (× ~0.85 runoff) |
|----------------------------|-----------------------------------|
| **5 mm** shower | ~**250–380 L** |
| **10 mm** | ~**500–760 L** |
| **20 mm** | ~**1000–1500 L** |
| Newbury-ish **year** (~650–750 mm) | Tens of thousands of litres — **far more than 800–1600 L** |

**Yes — they will fill**, repeatedly, whenever useful rain returns. Filling **800 L** from empty needs roughly **one solid 10–15 mm** event on this catchment (or several lighter ones). Filling **1600 L** needs roughly **15–25 mm** cumulative useful rain.

**Summer drought caveat:** in a heatwave with little rain, butts **do not refill from the sky**. Extra tanks buy **buffer between rain events**, not a continuous summer supply from an empty start.

### Is buying 4 more butts (~£ + space) worth it?

| | **Stay at 800 L** | **Add 4 → 1600 L** |
|--|-------------------|---------------------|
| 60 min @ 240 L/h | ~**3** runs | ~**6–7** runs |
| Heatwave (every 2–3 days) | ~**1 week** of soaks | ~**2–3 weeks** |
| Fill from rain | Easy | Still easy when rain returns |
| Cost / footprint | — | 4 butts + link hose + space by SE wall |

**Verdict:** **Worth it if** you want a longer dry-spell buffer and have space to link another 4×200 L at the **bottom**. Not essential if you can top up from mains float occasionally — **800 L already covers ~3 full irrigation days**.

**Middle path:** add **2** butts first (→ **1200 L**, ~5 runs) and only go to 8 if a heatwave still empties you.

### Rules if expanding

1. Link **all** butts at the **bottom** (same water level).  
2. Both downpipes into the bank (or one bank fed by both).  
3. Overflow to drain when full — don’t flood the patio.  
4. AJ-SR04M in one butt still represents the whole linked bank.

---

## 6. System architecture

```
[linked water butts]
        │
[Jet 3000 K7 + 7m suction + NRV]   ← low beside butts; prime body once
        │
[Hozelock snap → KATSU Presscontrol]  ← auto on/off + dry-run
        │
[Gardena Master Unit ~1.5 bar]     ← reducer + filter
        │
[NC 230 V brass solenoid]          ← siphon stop; with Zigbee plug (§5d)
        │
   16 mm fence spine (U-loop)      ← one zone
   east → north → west fence
        │
   4 mm drop to every plant
```

---

## 6b. How to fit drippers — **4 mm from fence spine (default)**

Your spine sits on the **fence**, not next to the plants. So every plant gets a **4 mm drop** from the spine to a dripper at the root zone. Do **not** punch drippers into the spine as the normal method.

```
Fence timber
    ║
    ║  16 mm LDPE spine (cable-tied / U-nailed)
    ║════╤══════════════╤══════════════
    ║    │              │
    ║  4 mm tube      4 mm tube
    ║    │              │
    ║  dripper        dripper
    ║  on stake       on stake
    ║  (at plant)     (at plant)
```

### How each drop is built

1. **Punch** a hole in the 16 mm spine above / near the plant’s fence position.  
2. Push in a **4 mm take-off / barbed connector** (or cut in a 13→4 mm reducing tee).  
3. Run **4 mm micro tube** down/across to the plant (typically **1–3 m**; keep under **~4 m**).  
4. Fit a **CETA PC dripper** on the end (**2 L/h red** or **4 L/h black** as ordered):
   - Leave the **barb on**, push onto 4 mm, stake the outlet.  
5. Aim at the **root zone** (outer canopy edge on big shrubs), outlet just visible above mulch.

### Tube budget (all plants on 4 mm)

| Estimate | Length |
|----------|--------|
| ~40 drippers × ≤1 m from fence | ~**40–48 m** of 4 mm |
| **Ordered** | **50 m** (4 mm/6 mm) |

One spine can feed many 4 mm laterals; flow is set by the drippers (~2 L/h each), not by the tube.

### Optional: punch dripper into spine

Only if a plant sits **directly under** the fence line and you truly don’t want a drop. Remove the CETA barb and push the spout into a punched hole. **Not the plan for this garden.**

---

## 7. Pipe map (one zone, fence-only)

Butts sit on the **east side of the house** (SE). Hub stays there. Spines run **only on fences** — **not** along the house wall.

```
                    NORTH fence
    ┌──────────◄──────────◄────────────────────┐
    │ NW                                    NE │
    │  │                                    │  │
    │  ▼ west fence                    east ▲  │
    │  │    (lawn — no pipe)            fence│  │
    │  │                                    │  │
    │ Conserv.                           butts● hub
    └──────────────────────────────────────────┘
                    HOUSE (no pipe on house)
```

**One watering zone** — whole garden waters together.

---

## 7b. How to run East and West from the butts (fence only)

You cannot clip to the house, so the **west border is reached by going around the outside**: east fence → north fence → west fence.

### Hub → east fence

Short run from the butts across the **east gravel / cobble bed** to the **east fence** (a few metres in the border — not along the house). Hub gear stays by the butts.

### Option A — one continuous fence spine (simplest) ✅ start here

One 16 mm pipe, open as **one zone**:

```
Butts → east fence NORTH → NE corner → north fence WEST → NW corner → west fence SOUTH
                                                                      (toward conservatory)
```

| Leg | Approx. | Serves |
|-----|---------|--------|
| East fence, butts → NE | ~15–18 m | East border, cobbles, feature, NE |
| North fence, NE → NW | ~21 m | North border (4 mm drops into bed) |
| West fence, NW → SW | ~15–21 m | West border, NW patio (stop at / before conservatory) |

**Total spine ~55–60 m** — fine on a 100 m coil.

- Cap or flush valve at the **SW end** (far end of the west fence).  
- Punch 4 mm drops all along to plants.  
- Lavender + lawn stay off.

**Hydraulics:** ~120 L/h travels the whole U. Far west plants are last. With **PC drippers**, **16 mm** spine and **~1.5 bar** at the hub this is workable — **prove it in Stage 1** (cup test on the last west dripper). If that end is weak, use Option B.

### Option B — two fence spines (better balance)

If Stage 1 shows far west drippers under-delivering, split at the hub so each branch only carries ~100 L/h:

```
[hub] tee
   ├── East spine: north on east fence → NE (cap or short north-fence stub)
   └── West spine: north on east fence (second 16 mm beside the first)
                   → NE → west on north fence → south on west fence
```

For ~15–18 m you run **two 16 mm pipes on the east fence** (one above the other, same fixings). At NE they split. Still **fence only**, still **one zone** (both open). Extra ~15–18 m — still inside the 100 m coil.

### What not to do

| Don’t | Why |
|-------|-----|
| Run spine along the **house wall** | Ruled out |
| Cut across the **lawn** | Ugly, vulnerable |
| Use **4 mm** as the long perimeter main | Too much friction |

### Approximate lengths (buy 100 m LDPE once)

| Piece | m |
|-------|---|
| Butts → east fence | ~2–4 |
| East fence | ~18 |
| North fence | ~21 |
| West fence | ~18 |
| Contingency / Option B double-run | ~15 |
| **Total** | **~70–75** |

---

## 8. Install once + annual mulching

You mulch every year and only want to fit this once. Design for **surface-under-mulch**, not deep burial — and make the system **findable and fork-safe** when you remulch.

### Where the pipe should live

| Do | Don’t |
|----|--------|
| Lay **16 mm LDPE on the soil surface** (or in a shallow scrape), then mulch over it | Dig it deep into the border (hard to repair; easy to spear when forking mulch) |
| Keep the **hub** (pump, Presscontrol, reducer, valves) on hardstanding by the butts — never under mulch | Bury valves or the filter where you can’t reach them |
| Run the spine along **bed edges / path margins** where you mulch less aggressively | Zig-zag loosely through the middle of beds if you can avoid it |
| Peg the pipe every **1–1.5 m** so it doesn’t drift when you rake | Leave loose loops that catch on a fork |

Mulch over the line is ideal: it hides the pipe, cuts UV damage, and reduces evaporation. Annual mulching then means **adding a layer on top**, not digging the bed out.

### Drippers vs mulch

- Put each dripper on a **short stake**, with the outlet just above or barely under the mulch surface so you can still see it working.  
- Aim water at the **root zone** (outer canopy edge on big shrubs), not hard against the stem.  
- After each mulch top-up, **walk the line once**: uncover any buried stakes, clear mulch clogging outlets, push stakes back upright.  
- If a dripper vanishes under thick bark, it still waters underground — but you can’t spot blockages. Visible stakes are worth the slight look for a permanent system.

### Remulching without wrecking the system

1. **Know the routes** — keep a simple sketch of the fence U-spine (E→N→W) and hub / master valve.  
2. **Top up, don’t dig** — tip new mulch on and spread with hands/soft rake near known pipe lines; avoid digging forks along bed edges where the spine runs.  
3. **Mark permanently** — small garden markers, brick pavers, or coloured cable ties at valve take-offs and bed entry points so next year’s you isn’t guessing.  
4. **Flush once a year** after mulching season — open end caps, run the system a few minutes, clear grit that mulch/soil pushed in.  
5. **Spare drippers & goof plugs** in the shed — when you accidentally spear a line, cut out the damage, join, or plug a hole in minutes.

### Winter (UK) — still “install once”

- Before hard frost: open end caps / run briefly to **drain** low spots, or leave end caps loose so water isn’t trapped.  
- LDPE and PC drippers can stay in place year-round under mulch; the vulnerable bit is the **pump hub** — store the Jet 3000 frost-free if it isn’t already under cover.  
- Spring: reassemble hub, flush, check a far dripper on **each** spine branch.

### What this means for shopping / layout

- Buy **pipe pegs / hold-downs** (plenty — every metre on both spines).  
- Prefer **staked drippers at each plant** (4 mm from fence) so outlets stay visible after mulching.  
- Leave a little **spare pipe coiled** at the hub and at far ends for future repairs without rebuying a whole coil.  
- Keep filter and valves **hand-high by the butts** so annual maintenance never means digging.

**Bottom line:** surface pipe + mulch cover + visible dripper stakes + marked routes = one install that survives yearly mulching if you top up carefully and don’t fork blindly along the bed edges.

**Chosen layout for annual mulching:** run the **spine on the fence** (see §8b) and drop **4 mm** to every plant — remulching never touches the main pipe; only short tubes and stakes sit in the beds.

---

## 8b. Fence-mounted spine (fixing options)

If the spine runs along the **fence**, the main pipe stays out of the mulched beds. **Every plant** gets a **4 mm drop** from the spine to a staked dripper (see §6b).

### Preferred: two ordinary cable ties + U-nail

**Yes — this works.** You don’t need double-headed ties.

Your method:
1. Hammer a galvanised **U-nail** into a fence rail.  
2. **Tie A:** loop through the U-nail and cinch (hanging loop / “eye”).  
3. **Tie B:** pass loosely around the **16 mm pipe**, then through Tie A’s loop, and cinch **snug not crushing**.

```
Fence timber
    ║
    ║  ∩  ← galvanised U-nail
    ║  │
    ║  ○══ Tie A (through U-nail) ── hanging loop
    ║       │
    ║       ○══ Tie B around 16 mm pipe (through Tie A, loose on pipe)
```

| Part | Spec | Notes |
|------|------|-------|
| **Cable ties** | **Black**, **≥200 mm long**, **≥3.6 mm wide** (4.8 mm better) | e.g. [300×4.8 mm ×100 ~cheap](https://www.amazon.co.uk/dp/B08YNY6GY9) · [Gocableties 300×4.8 mm](https://www.amazon.co.uk/dp/B01GK5J8GK) · prefer **UV / weather-resistant** wording |
| **U-nails** | Galvanised, ~**25–40 mm** | Screwfix / Wickes / [Amazon](https://www.amazon.co.uk/dp/B007OC4A0S) |

**Do not buy** tiny indoor ties like [Reulin 100×2.5 mm](https://www.amazon.co.uk/dp/B079JS1BK7):
- **100 mm** is too short for U-nail + pipe + hanging loop  
- **2.5 mm** is thin and fails faster in sun  

**How to fit**
1. U-nail into a **fence rail** (not a thin board edge) — leave a gap for Tie A.  
2. Tie A through the U → cinch.  
3. Tie B around the spine → through Tie A → cinch **loose enough that the pipe can move slightly** (don’t oval/crush LDPE).  
4. Space every **60–80 cm**.

**Why it’s good**
- Cheap, Amazon-easy  
- Pipe hangs slightly off the fence (less rub)  
- Cut Tie B to drop the pipe for repairs without pulling the U-nail  

**Caveats**
- **Black only** (natural/white dies in UV)  
- Expect to **replace ties every few years** on a sunny fence  
- Galvanised U-nails only  

**Verdict:** Use this method. Skip double-headed ties unless you already have them.

---

### Alternative: nail-in round cable clip (one-piece)

## 8c. Long spine (~70 m) — joins and flow

You may need **more than one length** of 16 mm LDPE to get from the butts to the far side. That is normal and **will not choke the system** if you join it correctly.

### Use barbed joiners — not narrow internal sleeves

Standard irrigation join is a **16 mm double-barbed straight connector**:

```
  [====pipe====][|barb|barb|][====pipe====]
                 ↑ short stub, full bore through the middle
```

- The barb goes **inside** each pipe end — but the fitting is a **short full-bore stub**, not a long narrow sleeve.  
- At branch flow (~**100 L/h** each side), one join adds **negligible** loss compared with tens of metres of pipe friction.  
- A few joins along the fence is fine; **do not** use tight homemade internal sleeves or push-fit plumbing inserts meant for smaller bore.

| Fitting | Use | Example |
|---------|-----|---------|
| **16 mm barbed straight joiner** | Join two lengths | [HydroSure double barb 16 mm](https://www.waterirrigation.co.uk/pack-of-25-x-hydrosure-double-barbed-joiner-16mm-x-16mm-black.html) (~£0.48 each) |
| **16 mm barbed elbow / tee** | Corners and zone splits | Hozelock / HydroSure / Irrigation Online |
| **Ratchet clip** (optional) | Lock pipe onto barb at ~1.5 bar | Recommended at joins on the fence for peace of mind |

### How to join (install once)

1. Cut pipe square with a sharp knife or pipe cutters.  
2. Warm each end in **hot water 10–30 s** (makes LDPE flexible).  
3. Push both ends fully onto the barb — **straight push**, don’t twist.  
4. Optional: **ratchet clip** over the join.  
5. Place joins **on the fence** next to a clip — never in a mulched bed.

### How many lengths?

| Approach | Notes |
|----------|-------|
| **One continuous coil** | A 100 m coil can do the whole ~70 m run in one piece if you can handle unwinding it — **fewest joins**, but awkward to install alone. |
| **2–3 lengths** (typical) | e.g. butt → mid-garden → far corner. One join every **20–30 m** at a fence post. **Recommended** for a one-person install. |
| **Many short bits** | Avoid — every unnecessary join is another leak risk and nothing to gain at ~100 L/h per branch. |

### Flow reality check

Your limit is **friction along 70 m**, not the joins:

- Each branch ~**100 L/h** (total ~**206 L/h**) through 16 mm pipe at ~1.5 bar — well within capacity.  
- **PC drippers** at the far end still get even flow.  
- If worried, put a **join at each fence post** (accessible) rather than one long uncut run that’s impossible to replace later.

**Leave 1–2 m spare coiled at the hub** and at the far end for future repairs without buying a new coil.

---

## 9. Watering schedule

This system is **drought insurance**, not a daily automatic garden. Mature shrubs/trees want **deep, infrequent** soaks. Skip runs when rain has done the job.

### Design soak (memorise this)

| Setting | Value | Why |
|---------|-------|-----|
| **Chosen run (HA)** | **60 minutes** morning | ~**120 L** — soft/normal soak; OK for established shrubs |
| Design default | **75 minutes** | ~**150 L** — deep soak if heatwave |
| Soft stress | **60 min** (~120 L) | Mild dry spell |
| Hard drought | **90 min** (~180 L) | Comfortable on butts |
| Flow | ~**120 L/h** / ~**2 L/min** | Whole garden, one zone |
| Control | Zigbee plug + NC solenoid | Same schedule opens pump and valve (§5d) |

**Per outlet in a 75 min run** (ordered mix):

| Emitter | Water delivered |
|---------|-----------------|
| **2 L/h** (red) | ~**2.5 L** |
| **4 L/h** (black) | ~**5 L** |

Use one outlet per plant to start; add a second later only if needed.

That is a proper root-zone soak, not a surface sprinkle.

### How often (Newbury summer)

Water when plants need it — not on a fixed calendar. Use this as a starting guide:

| Conditions | How often | Run length | Notes |
|------------|-----------|------------|-------|
| Cool / showery / soil still moist | **Off** | — | Check soil 10 cm down; don’t run on habit |
| Normal dry summer (no useful rain) | **Every 4–7 days** | **75 min** | Established borders |
| Hot dry spell (heatwave, weeks with little rain) | **Every 2–3 days** | **75 min** | Watch butts — refill between runs if needed |
| Extreme drought / plants wilting by morning | **Every other day** | **75–90 min** | One run per day max; never two same day |
| New plantings (first 1–2 summers) | **Every 2–3 days** in dry weather | **60–75 min** | Or hand-water newcomers more often |
| Pots on the system | Same schedule as borders | — | Pots dry faster — if they crisp while borders are fine, hand-top or add a second dripper |
| Lavender / lawn | **Not on this system** | — | Hand-water lavender if needed; lawn stays off |

**Rule of thumb:** deep soak → wait until the top ~5–10 cm of border soil is dry again → soak. Prefer **fewer long runs** over daily short ones.

### Time of day

| Prefer | Avoid |
|--------|-------|
| **Early morning** (e.g. 5:00–8:00) | Midday heat (waste to evaporation) |
| Evening OK if mornings aren’t possible | Overnight wet foliage on disease-prone plants (less of an issue with drippers at the soil) |

Set the timer for **one morning start**. Do not run twice a day — butts and roots both prefer one deep pass.

### Timer recipe (copy this)

1. **Duration:** **60 minutes** (stretch to 75–90 in heatwave)  
2. **Days:** start with **Tue + Fri** in a dry July/August; drop to **once a week** in mild dry weather; pause entirely after decent rain  
3. **Start:** ~06:00  
4. **Rain / butts:** HA skip if AJ-SR04M low or after ≥10 mm rain  
5. Pump switch **O** left **ON**; Zigbee plug switches pump **+ NC solenoid**

After any useful rain, dig a finger into the bed: if moist at knuckle depth, **skip the next timed run**.

### Butts / how many runs you get

| Starting store | Safe runs before refill |
|----------------|-------------------------|
| ~800 L full | **~3** × 60 min runs @ ~240 L/h (80 drippers) before empty |
| ~1600 L (8 butts) | **~6–7** × 60 min runs — ~2 weeks of every-2–3-day heatwave watering |
| After rain / float top-up | Refill between soaks as needed |
| Near strainer (~30 cm above floor) | **Stop** — do not run |
| **AJ-SR04M in Home Assistant** | Automate “skip run if level low” (§5c) |

**Hard rule:** never schedule a second full run the same day hoping the butts refill. One soak → check butts → refill / wait for rain → next soak another day.

With a **mains float valve**, you can run on the 2–3 day drought cadence more confidently. Without it, plan around rainfall and manual hose top-up.

### What “plants need water” looks like

- Leaves soft / dull by **morning** (afternoon flop that recovers overnight is often heat, not drought)  
- Soil dry below the mulch at ~10 cm  
- New growth wilting; pots light when lifted  

If only one or two plants look bad, hand-water those — don’t run the whole garden for them.

### Seasonal outline

| Season | Schedule |
|--------|----------|
| **Apr–May** | Usually off for established plants; water new planting only in dry weeks |
| **Jun** | Start watching; first 75 min runs if dry |
| **Jul–Aug** | Main season — use the table above |
| **Sep** | Taper as nights cool / rain returns |
| **Oct–Mar** | Off. Drain / store pump below 5°C (§5b) |

### Quick card (stick by the timer)

```
DEFAULT: 60 min @ ~06:00 (Zigbee plug = pump + NC solenoid)
DRY SUMMER: every 4–7 days
HEATWAVE: every 2–3 days / 75–90 min
AFTER RAIN: skip next run
BUTTS LOW (near strainer / HA sensor): STOP
NEVER: twice in one day / empty butts dry / pump off with solenoid stuck open
```

---

## 10. Build order

1. Sit Jet 3000 **on the ground beside butts**; fit kit suction (strainer → NRV → hose); fix strainer **~30 cm above butt floor**  
2. Fill pump body once → prove prime; leave switch ON for Zigbee/Presscontrol use  
3. Hub: **G** + Hozelock snap → KATSU → Gardena → **NC solenoid** → 16 mm to east fence (§5d, §5e)  
4. Mount **AJ-SR04M** in butt lid; confirm distance readings in Home Assistant (§5c)  
5. Optional mains float-valve top-up  
6. Fence-mount 16 mm **U-spine**: east fence → north fence → west fence (~0.6–1.0 m height) — **done**  
7. Punch 4 mm take-offs; run 4 mm to each plant; fit PC drippers **correct way round** on stakes  
8. Cup-test near + far west; prove plug **OFF** stops all flow (solenoid)  

---

## 11. Shopping — **ordered 18 Jul 2026**

Amazon order **placed** (~**£161** items + ~£2 postage). Delivery windows **19–23 Jul 2026**. Scaled build: **40 PC drippers** (20×2 L/h + 20×4 L/h ≈ **120 L/h**).

### Owned (not in this Amazon order)

| Item | Notes |
|------|-------|
| [Hozelock Jet 3000 K7](https://www.hozelock.com/product/hozelock-jet-3000-k7/) | Pump + suction kit |
| Galvanised **U-nails** | Fence clips with cable ties |
| **13 mm flow-control valve** | Optional manual isolate (automation uses solenoid) |
| **Water-butt link kit** | Link butts at **bottom** |
| **AJ-SR04M** + ESP8266 | Water-butt level → HA (§5c) |
| **UpXNBor Zigbee plug** | Times pump + solenoid (10 A continuous OK for 600 W pump) |

### Ordered (Amazon checkout ~£161)

| Qty | Item | ~£ |
|-----|------|-----|
| 1 | KATSU / AimTools **Presscontrol 151024** (adjustable) | 18.99 |
| 1 | **Gardena Master Unit 1000** (13333-20) — ~1.5 bar + filter | 12.99 |
| 1 | **16 mm (13 mm ID) LDPE** ×**100 m** | 60.49 |
| 1 | Grow Insane **13 mm** fittings pack (30-piece) | 9.99 |
| 1 | **CETA 2 L/h red** ×20 (barb) | 8.29 |
| 1 | **CETA 4 L/h black** ×20 (barb) | 8.29 |
| **2** | **4 mm hypodermic off-takes** ×25 (=**50**) | 9.98 |
| 1 | **Antelco key hole punch** | 2.38 |
| 1 | **4 mm/6 mm micro tube** ×**50 m** | 16.99 |
| 1 | UCLEVER drip **stakes** ×100 | 7.99 |
| 1 | Black cable ties **300×4.8 mm** ×100 | 4.67 |

### Still to buy / sort at install

| Item | Notes |
|------|-------|
| **NC 230 V brass solenoid ½″** | Siphon stop after Gardena — [B09Y45K21L](https://www.amazon.co.uk/dp/B09Y45K21L) (~£14). Wire with pump on Zigbee plug (§5d) |
| **½″ BSP female adapters / hose tails** | Onto solenoid male ports |
| **1″ BSP female → hose barb** ×2 | KATSU inlet + outlet if still needed — e.g. [B0DMW6M9PG](https://www.amazon.co.uk/dp/B0DMW6M9PG) |
| **Short reinforced garden hose** | Bridges as required; jubilee clips |
| **32 mm corrugated pond hose 10 m** | Gutter diverter (~27 mm) → butt — [eBay ~£21](https://www.ebay.co.uk/itm/164344849802) |
| **Butt blanking (20 mm holes)** | Prefer **M12 bolt + rubber washers + penny washers + nut** sandwich (keeps hose adapters free). ¾″ flanged plugs only for ~25 mm holes |
| Optional: mains float valve | Stage later |
| Extra 4 mm / drippers | Only if walk-round needs more than 40 |

Historical planning tables (A–E carts, Water Irrigation bulk quotes) kept below for reference — **do not re-order** items already listed above.

---

### A. Pump control & pressure (critical)

| Qty | What | Approx. £ | Sample product |
|-----|------|-----------|----------------|
| 1 | Automatic pump pressure controller (**adjustable 1–3.5 bar**) | **£13–£19** | **Best:** [AimTools KATSU 151024 £12.99](https://www.aimtools.co.uk/products/151024) · Amazon [B086WWD8XF](https://www.amazon.co.uk/dp/B086WWD8XF) (£18.99) |
| *or* 1 | Genuine Presscontrol (premium) | **£111** | [Henry Pumps Presscontrol](https://www.henrypumps.co.uk/presscontrol-pump-pressure-controller-1116-p.asp) (£111 inc. VAT) |
| 1 | Pressure reducer ~1.5 bar + filter | **£13–£17** | **Best in stock:** [Gardena Master Unit 1000 £12.99](https://www.amazon.co.uk/dp/B0BNLNFV2T) · [Hozelock 7022 £16.59](https://www.easygardenirrigation.co.uk/products/hozelock-universal-pressure-reducer-7022) when available |
| 1 | **NC solenoid** after Gardena (siphon stop) | **~£14** | **Chosen:** [½″ brass 230 V NC B09Y45K21L](https://www.amazon.co.uk/dp/B09Y45K21L) · alt: 12 V GREDIA [B08KS6WHM2](https://www.amazon.co.uk/dp/B08KS6WHM2) + PSU |
| 2 | 1″ BSP female → hose barb | **~£6–10** | Amazon [B0DMW6M9PG](https://www.amazon.co.uk/dp/B0DMW6M9PG) · standard **Hozelock hose snap** on pump **G** (owned) |
| 0–1 | Optional small accumulator | **£15–£25** | Amazon: `0.75L pressure tank pump` |

**Why 151024, not 151015:** the cheaper 151015 is factory-**fixed** at 1.5 bar cut-in and had very low stock. The **151024 is adjustable (1–3.5 bar)** for a similar price, so you can dial pressure to suit the drip network.

**Subtotal A (budget path):** ~**£35–£50** · **Premium path:** ~**£135–£150**

---

### B. Water butts

| Qty | What | Approx. £ | Sample product |
|-----|------|-----------|----------------|
| 1 kit | Water-butt link kit | **£7–£8** | **Best:** [ANGIX Amazon ~£7.49](https://www.amazon.co.uk/dp/B09XXJG3L4) · [Strata HydroSure ~£7.35](https://www.amazon.co.uk/dp/B001DYWUNW) · *or* 3× Hozelock 19 mm connectors if you prefer branded |
| 1 | Mains top-up float valve (optional / Stage 4) | **£11–£15** | [Torbeck Screwfix £11.29](https://www.screwfix.com/p/torbeck-bottom-entry-fill-valve-1-2-/8120t) · [Hydroflow kit £14.99](https://hydroprowashington.co.uk/product/hydroflow-auto-top-up-float-kit/) |
| 0–1 | Extra mesh strainer (if kit strainer weak) | **£4–£8** | Match suction hose OD — Jet 3000 kit already includes strainer + NRV |

A simple float valve is fine for drought backup (less selective than RainAid-style valves). Jet 3000 suction kit already has filter + non-return valve — only buy another strainer if needed.

**Subtotal B (link kit only):** ~**£8–£12** · **With float-valve top-up:** ~**£20–£30**

---

### C. Main pipe & zone valves

| Qty | What | Approx. £ | Sample product |
|-----|------|-----------|----------------|
| 1× **100 m** | 16 mm OD / ~13 mm ID black LDPE spine | **£45–£61** | **Buyable now:** [Amazon 16 mm (13 mm ID) ×100 m ~£60.49](https://www.amazon.co.uk/s?k=Irrigation+16mm+13mm+ID+LDPE+100m) (B2C — correct type) · *alts:* Evergreen ~£52 + P&P · eBay 13/16 mm 100 m ~£45–£50 · Irrigation Online lists £17.14 but is **Sold Out** — ignore until restocked · use **13 mm** fittings |
| 1 | 13 mm flow-control valve(s) — **master shut-off** (optional East/West later) | **£7–£13** | [Hozelock 2765 2-pack £13.29](https://www.easygardenirrigation.co.uk/products/hozelock-flow-control-valve-13mm-2765) — use one as master, keep second as spare / future branch isolation · 1-zone does **not** need two live zone valves |
| 1 pack | 13 mm tees / elbows / joiners / end plugs | **£9** | **Best value:** [Amazon 30-piece pack](https://www.amazon.co.uk/dp/B07BPCNRN9) — enough for spine joins + manifold · *or* buy Hozelock packs separately if you prefer the brand |
| 1 | Hole punch | **£0–£4** | Included in Water Irrigation Large fittings pack · *or* [Hozelock 2799 £4.19](https://www.easygardenirrigation.co.uk/products/hozelock-hole-punch-tool-2799) |

**Pipe sizing:** Irrigation Online coil = **16 mm OD / ~13.6 mm ID** → all fittings above are **13 mm**. Do **not** buy HydroSure “16 mm ID” soaker joiners for this spine.

**Subtotal C:** ~**£75–£90** (100 m coil ~£60 + master valve + fittings pack £9 + punch if needed)

---

### D. Drippers & micro tube

| Qty | What | Approx. £ | Sample product |
|-----|------|-----------|----------------|
| **13×** 10-packs (= **130**) | 2 L/h **PC** CETA on 4 mm | **£41** | [Hydrosure CETA ×10 @ £3.12](https://www.waterirrigation.co.uk/pack-of-10-hydrosure-pressure-compensated-cetatm-micro-irrigation-dripper-2-l-h.html) → **£40.56** · leave barb on · *avoid* Amazon ASTA path (~£156 for same count) |
| **1× 100 m** | 4 mm micro tube | **£27** | [HydroSure 4 mm ×100 m £26.58](https://www.waterirrigation.co.uk/hydrosure-micro-pipe-4mm-x-100m-black.html) · *not* 4×25 m Hozelock (~£112) · *not* a separate 10 m test coil |
| **1× 100** | 4 mm barbed joiners (spine take-offs) | **£7.50** | [×100 @ £7.44](https://www.waterirrigation.co.uk/pack-of-100-hydrosure-barbed-joiner-4mm-x-4mm-black.html) beats ten ×10-packs · punch into spine wall |
| 1 pack | Stakes / hold-downs for 4 mm at plants | **£5–£10** | Cable-tie dripper to a short cane, or buy micro-tube stakes |
| **100** | Black cable ties (loop through U-nail) | **£3–£9** | [300×4.8 mm ×100](https://www.amazon.co.uk/dp/B08YNY6GY9) · black only · two ties per clip (U-nail + pipe) |
| **1 pack** | Galvanised fencing U-staples ~20–40 mm | **£4** | [Amazon ~£3.99](https://www.amazon.co.uk/dp/B007OC4A0S) · or Screwfix/Wickes |
| — | Spine joiners | *(in §C pack)* | Use **13 mm** straight joiners from the fittings pack — not 16 mm-ID soaker joiners |

**Subtotal D (CETA + 100 m 4 mm + joiners + fence):** ~**£90–£100**

Do **not** buy fixed-spacing dripline as the main emitter — plants aren’t on a 30 cm grid.

---

### E. Optional automation

| Qty | What | Approx. £ | Sample product |
|-----|------|-----------|----------------|
| 1 | Digital water timer | **£30–£45** | Amazon: `Hozelock Select Controller` / `Gardena water timer` |

---

### F. Consumables

| Qty | What | Approx. £ |
|-----|------|-----------|
| 1 | PTFE tape | **£2** |
| 1 pack | Hose clips / ratchet clamps | **£5** |
| — | Cable ties + marker | **£3** |

**Subtotal F:** ~**£10**

---

### Worked example budgets (inc. VAT, rounded)

**Budget core (specialist bulk)** — AimTools KATSU + Gardena reducer + Irrigation Online LDPE + Water Irrigation 4 mm/CETA/joiners + 13 mm valves/fittings + fence + butt kit  

| Line | £ |
|------|---|
| A Pump control (AimTools + Gardena) | 35 |
| B Butts (link kit) | 10 |
| C Spine + valves + 13 mm fittings | 85 |
| D 4 mm + CETA + take-offs + fence | 95 |
| F Consumables / adaptors | 15 |
| Multi-shop P&P (estimate) | 15 |
| **Total** | **~£255** |

**With drought float-valve top-up**

| | £ |
|--|---|
| Core as above | 255 |
| Float valve | 15 |
| **Total** | **~£270** |

**Fully optioned** — above + timer + premium Presscontrol  

| | £ |
|--|---|
| With float valve | 270 |
| Timer | 35 |
| Upgrade Presscontrol (+£100 vs AimTools KATSU) | 100 |
| **Total** | **~£405** |

---

### Best-value buying plan (bulk first — not Amazon-only)

**Rule:** if you need ~100 of something, buy the 100-pack / 100 m coil. Do **not** buy a 10 m “test” coil of pipe you already know you need 100 m of. Cut the Stage-1 test length off the full coil.

Prices checked **17 Jul 2026**, inc. VAT where stated. Add P&P at checkout (Easy Garden Irrigation: **£5.95**, free over **£200**; AimTools free over **£200**; Water Irrigation / Irrigation Online — check basket).

#### Unit-cost comparison (why bulk wins)

| Item | Need | Wasteful buy | Efficient buy | Saving |
|------|------|--------------|---------------|--------|
| Spine LDPE | ~70–100 m | Short coils (10–25 m) then rebuy | **100 m** Amazon ~**£60** (in stock) · Evergreen ~£52 · eBay ~£45–£50 · Irrigation Online £17 listing is **Sold Out** | Buy **100 m once** |
| 4 mm micro tube | ~100 m | 4× Hozelock 25 m Amazon ~**£112** · or 10 m test **£10** then buy again | [HydroSure 4 mm ×100 m **£26.58**](https://www.waterirrigation.co.uk/hydrosure-micro-pipe-4mm-x-100m-black.html) | **~£85** vs Amazon Hozelock |
| PC drippers 2 L/h | ~130 | ASTA Amazon 5-packs ×26 ~**£156** (£1.20/ea) | [CETA 10-pack ×13 **£3.12** = **£40.56**](https://www.waterirrigation.co.uk/pack-of-10-hydrosure-pressure-compensated-cetatm-micro-irrigation-dripper-2-l-h.html) (£0.31/ea) | **~£115** |
| 4 mm take-off joiners | ~80–100 | 10-packs ×10 @ £1.43 = **£14.30** | [4 mm joiner ×100 **£7.44**](https://www.waterirrigation.co.uk/pack-of-100-hydrosure-barbed-joiner-4mm-x-4mm-black.html) (£0.07/ea) · if OOS: [Large fittings pack **£22.19**](https://www.waterirrigation.co.uk/hydrosure-essential-4mm-barbed-micro-mixed-fittings-pack-large.html) (30 joiners + punch tool) ×2 if needed | **~£7+** |
| Presscontrol | 1 | Amazon KATSU 151024 **£18.99** | [AimTools 151024 **£12.99**](https://www.aimtools.co.uk/products/151024) | **£6** |
| 16 mm spine joiners | 5–10 | Many tiny packs | Need **13 mm** barbed joiners for 16 mm OD / ~13 mm ID spine — **not** HydroSure “16 mm ID” soaker joiners. Use [13 mm connector pack ~£9](https://www.amazon.co.uk/dp/B07BPCNRN9) or Hozelock 13 mm fittings | — |

**Pipe sizing note:** Amazon/Evergreen “16 mm (13 mm ID)” LDPE = **16 mm outside / ~13 mm inside** → use **13 mm** Hozelock-style valves & fittings. Do **not** buy 16 mm-ID soaker joiners for this coil.

---

#### Cart A — Water Irrigation (main irrigation kit) ~**£95–£110** + P&P

Buy this as **one order** (covers test + full garden — cut a few metres of pipe for Stage 1).

| Qty | Item | £ | Link |
|-----|------|---|------|
| 1 | 4 mm micro pipe **100 m** | **26.58** | [link](https://www.waterirrigation.co.uk/hydrosure-micro-pipe-4mm-x-100m-black.html) |
| **13** | CETA 2 L/h PC drippers (10-pack) | **40.56** | [link](https://www.waterirrigation.co.uk/pack-of-10-hydrosure-pressure-compensated-cetatm-micro-irrigation-dripper-2-l-h.html) |
| 1 | 4 mm barbed joiners **×100** (spine → 4 mm take-offs) | **7.44** | [link](https://www.waterirrigation.co.uk/pack-of-100-hydrosure-barbed-joiner-4mm-x-4mm-black.html) — if OOS use Large fittings pack instead |
| 1 | Large 4 mm fittings pack (joiners + tees + punch tool) *optional top-up* | **22.19** | [link](https://www.waterirrigation.co.uk/hydrosure-essential-4mm-barbed-micro-mixed-fittings-pack-large.html) |
| 1 | Hole punch (if not in fittings pack) | ~**3–5** | search site `hole punch` |

Leave CETA **barb on** → push onto 4 mm. Punch spine → push 4 mm joiner into wall → 4 mm tube to plant.

---

#### Cart B — 100 m LDPE spine ~**£45–£61** (inc. delivery if Prime / free P&P)

| Qty | Item | £ | Link |
|-----|------|---|------|
| 1 | 16 mm OD / 13 mm ID LDPE **100 m** | **~60.49** | Amazon B2C listing you found — **correct product**; Prime delivery often beats specialist+P&P |
| *or* 1 | Same type 100 m | **~52** | [Evergreen Irrigation](https://www.evergreen-irrigation.co.uk/products/ldpe-pipe) (+ P&P; shop closed mid–late Jul 2026) |
| *or* 1 | 13/16 mm 100 m | **~45–50** | eBay search `13/16mm 100m LDPE irrigation` — check seller ratings + free postage |

**Note (Jul 2026):** [Irrigation Online 16 mm LDPE](https://irrigationonline.co.uk/products/16mm-LDPE-Pipe.html) still *lists* ~£17 but checkout shows **Sold Out** — do not plan on that price until it is actually in stock.

---

#### Cart C — pump control ~**£26–£35** + P&P

| Qty | Item | £ | Link |
|-----|------|---|------|
| 1 | KATSU **151024** | **12.99** | [AimTools](https://www.aimtools.co.uk/products/151024) (cheaper than Amazon £18.99) |
| 1 | Pressure reducer ~1.5 bar + filter | **12.99** | [Gardena Master Unit 1000 Amazon](https://www.amazon.co.uk/dp/B0BNLNFV2T) · Hozelock 7022 ~£16.59 at Easy Garden when in stock |

Hub hose tails: 2× 1″ BSP female barb + standard Hozelock snap on **G** (~£6–10).

---

#### Cart D — master valve + 13 mm fittings ~**£20–£30**

| Qty | Item | £ | Link |
|-----|------|---|------|
| 1 | Hozelock 2765 flow valves **2-pack** (master + spare) | **13.29** | [Easy Garden Irrigation](https://www.easygardenirrigation.co.uk/products/hozelock-flow-control-valve-13mm-2765) — **one zone:** only the master is required in-line; second valve optional for a future East/West split |
| 1 | 13 mm tee/elbow/joiner/end-plug pack | **~9** | [Amazon 30-piece](https://www.amazon.co.uk/dp/B07BPCNRN9) — includes tees for hub split to East + West |

Combine Cart D with anything else at Easy Garden if you can push that order toward **£200** free delivery — otherwise accept £5.95 once.

---

#### Cart E — fence + butts (cheap consumables) ~**£20–£30**

| Qty | Item | £ | Link |
|-----|------|---|------|
| 1–2 | Black cable ties 300×4.8 mm ×100 | **~£3–£9** | [B08YNY6GY9](https://www.amazon.co.uk/dp/B08YNY6GY9) |
| 1 | Galvanised fencing U-staples | **~4** | [Amazon](https://www.amazon.co.uk/dp/B007OC4A0S) |
| 1 | Water-butt link kit | **~7.50** | [Amazon ANGIX](https://www.amazon.co.uk/dp/B09XXJG3L4) |

---

### Revised stages (still staged — but bulk-efficient)

**Stage 1 — prove chain (~£55–£80 goods + share of pipe)**  
Order **Cart C** (controller + reducer) **and** at least Cart B (100 m LDPE) **or** Cart A (includes 4 mm + drippers).  
Cut ~10–15 m LDPE + ~5 m of 4 mm + 10–15 drippers from the bulk packs for the test. **Do not** buy separate 10 m coils.

Pass: far dripper ≈ 2 L/h.

**Stage 2 — finish hub + zones (~£25–£40)**  
Cart D valves/fittings + butt links (Cart E partial) + adaptors.

**Stage 3 — install full garden (£0 extra pipe/drippers if Cart A+B already bought)**  
Fence-mount spine; punch take-offs; run 4 mm; stake drippers. Buy Cart E fixings if not done.

**Stage 4 — optional**  
Float valve / timer later.

---

### Efficient core total (rough)

| Basket | ~£ |
|--------|-----|
| A Water Irrigation (4 mm + CETA + joiners) | 75–100 |
| B LDPE spine 100 m (Amazon/eBay) | 55 |
| C Controller + reducer | 26–32 |
| D Valves + 13 mm fittings | 22–30 |
| E Fence + butts | 20–30 |
| P&P across shops (estimate) | 10–25 |
| **Core total** | **~£210–£280** |

Vs all-Amazon path with Hozelock 4 mm rolls + ASTA spikes: often **£350–£450+**.

---

### Amazon order reference (placed 18 Jul 2026)

See **§11 ordered table** above. Key ASINs for re-orders/spares:

- CETA 2 L/h ×20 → [B07KRFDQ5H](https://www.amazon.co.uk/dp/B07KRFDQ5H)  
- CETA 4 L/h ×20 → [B07KRF2C7N](https://www.amazon.co.uk/dp/B07KRF2C7N)  
- Hypodermic off-take ×25 → [B07KR3JSZQ](https://www.amazon.co.uk/dp/B07KR3JSZQ)  
- 13 mm fittings → [B07BPCNRN9](https://www.amazon.co.uk/dp/B07BPCNRN9)  
- Gardena Master Unit → [B0BNLNFV2T](https://www.amazon.co.uk/dp/B0BNLNFV2T)  
- Presscontrol → [B086WWD8XF](https://www.amazon.co.uk/dp/B086WWD8XF) / AimTools 151024  
- NC solenoid (siphon) → [B09Y45K21L](https://www.amazon.co.uk/dp/B09Y45K21L)  
- Cable ties → [B08YNY6GY9](https://www.amazon.co.uk/dp/B08YNY6GY9)  

---

## 12. Decisions summary

| Topic | Decision |
|-------|----------|
| Zones | **1** (whole garden) — fence U-spine east→north→west |
| Spine routing | **Fence only** — butts → east fence → north fence → west fence (§7b). No pipe on the house |
| Emitters | **40 PC drippers:** 20× **2 L/h** + 20× **4 L/h** on **4 mm** drops (≤1 m from fence); fit correct orientation |
| Long runs (~70 m) | OK with 16 mm spine + PC + 1.5 bar at hub |
| Pump ≫ demand | OK with Presscontrol + reducer; don’t dead-head |
| Pump position | **On the ground beside the butts** (fixed). Strainer fixed **~30 cm above butt floor**; stop when water falls to strainer |
| Priming | Fill pump body **once**; keep NRV + airtight joints so timer can power on/off without re-priming |
| Schedule | **60 min** morning via Zigbee; 75–90 min in heatwave (§9) |
| Siphon when pump off | **NC 230 V brass solenoid** after Gardena, powered with pump (§5d) — not a manual daily valve |
| Full garden pass | **~80 drippers** → ~**240 L/h**; **60 min** ≈ **240 L** — fits **~800 L** store (~3 runs); expand butts §5f |
| Lawn / lavender | Off the system |
| Install style | **Fence-mounted spine** (~0.6–1.0 m) with **cable ties + U-nails**; **4 mm drop to every plant** |
| Long spine (~70 m) | **2–3 lengths** joined with **16 mm barbed straight connectors** at fence posts |
| Hub hose connection | **Standard small Hozelock snap** on **G** — not high-flow **H**; KATSU bridged with 1″ BSP hose tails |
| Water-butt level | **AJ-SR04M** + ESPHome → HA; skip runs when low (§5c) |
| Gutter fill line | **32 mm** flex pond hose (10 m) diverter → butt — not 16 mm LDPE |
| Blanking old butt holes | **M12 bolt + rubber washers** sandwich for ~20 mm holes |
| Butt capacity | **4×200 L ≈ 800 L** now; optional **+4 → ~1600 L** for drought buffer (§5f) |
| Emitters (install) | **~80 PC drippers** target (~240 L/h); Stage 1 was 40 |
| Roof catchment | **Two SE downpipes** → butts; ~60–90 m² plan area — fills store after useful rain (§5f) |

---

## 13. Confirm on site

- Living plants vs 2001 drawing  
- Butt bank: **4×200 L linked at bottom**; decide on +4 (§5f); blank unused holes  
- Pump on ground beside butts; fix strainer ~30 cm above butt floor  
- Hub: G → KATSU → Gardena → **NC solenoid** — prove no leaks; plug OFF stops all flow  
- Calibrate AJ-SR04M empty/full distances in ESPHome  
- Thames Water ban / float-valve rules  
- Walk-round: assign 2 L/h vs 4 L/h; drippers correct way round; cup-test far west  

---

*Amazon core order placed **18 Jul 2026** (~£161). Solenoid [B09Y45K21L](https://www.amazon.co.uk/dp/B09Y45K21L) to buy. Historical cart prices below are planning archive only.*
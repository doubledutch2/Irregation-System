# Home Assistant Configuration

**Property:** 9 Heather Garden, Newbury  
**Purpose:** Document all Home Assistant (HA) changes for the garden irrigation system.  
**YAML source of truth:** files under [`yaml/`](yaml/) - this document describes behaviour only; do not paste full YAML here.

**Related hardware (see [Irrigation Plan.md](Irrigation%20Plan.md)):**
- Zigbee plug **UpXNBor** - switches pump + NC 230 V solenoid together (`switch.study_water_pump`)
- ESP8266 (`esp-home-waterbutt`) + **AJ-SR04M** ultrasonic to ESPHome to HA
- Hozelock Jet 3000 K7 + KATSU Presscontrol + Gardena Master Unit + NC solenoid

**Style note:** Use plain readable characters only in this file and under `yaml/` (no section-symbol shorthand; spell out cross-references).

**HA deploy rules (always):**

1. Never update Dashboards - tell the user what to change in the UI.
2. Backup every remote file before editing; also store copies under `ha-backups/YYYY-MM-DD/` on this computer (gitignored).
3. Put Water Garden HA config in `/config/packages/` only.
4. Run `ha core check` as the test deploy; **never restart HA** - the user restarts manually.
5. Never put `initial:` on package helpers the user tunes (Duration, Pause, dripper counts, Phase) - it can overwrite live values on reload/restart.

See Cursor rule `.cursor/rules/home-assistant-deploy.mdc` and [`HA Update Guide.md`](HA%20Update%20Guide.md).

---

## How to use this file

1. When you add or change HA config, put the YAML in [`yaml/`](yaml/) under a clear filename.
2. Add or update a short section below describing **what** it does and **why** - not the YAML itself.
3. Link to the corresponding file(s) under `yaml/`.

---

## Garden dashboard UI (tiles and components)

Lovelace view: title **Garden**, path **`water-butts`**, type **sections**.  
Reference layout (for you to paste/edit in the UI - agents do not write Lovelace): [`yaml/dashboard_garden_water_butts.yaml`](yaml/dashboard_garden_water_butts.yaml).

Entities below come from package `/config/packages/water_garden.yaml` (repo: [`yaml/package_water_garden.yaml`](yaml/package_water_garden.yaml)), plus existing water-butt helpers/sensors.

### Main screen (Newbury) - Start Watering for everyone

Add a **toggle** on the **Newbury** home view (`default_view`) so watering is one tap to start and one tap to stop.

1. Open **Newbury** -> Edit dashboard.
2. Replace the old Start Watering **button** (or add a new **Tile** card).
3. Paste from [`yaml/dashboard_card_main_start_watering.yaml`](yaml/dashboard_card_main_start_watering.yaml):

```yaml
type: tile
entity: switch.water_garden_watering
name: Start Watering
features:
  - type: toggle
```

4. Put it near the top of the screen. Save.

**What the toggle does:**

| Action | Result |
|--------|--------|
| Turn **ON** | Starts the split-run; Telegram start message |
| Turn **OFF** (while running or paused) | Stops the sequence; pump off; Telegram stop |
| Stays **ON** during pause | Sequence still active (butts rebalancing) |
| Goes **OFF** by itself | When the run finishes normally |
| Butts empty | Start blocked; toggle stays/returns **OFF**; Telegram |

Entity: `switch.water_garden_watering` (follows phase: on whenever not `idle`).

No Home Assistant persistent/bell notifications for Water Garden (Telegram only).

**Important:** use `switch.water_garden_watering`, not the pump switch and not a script-only button. The old `input_button.water_garden_start` still toggles start/stop if you keep that tile.

### Section - run controls

| Card | Entity | Name on dashboard | Role |
|------|--------|-------------------|------|
| Tile + toggle | `switch.water_garden_watering` | Start Watering | ON starts sequence; OFF stops. Shows on for first_half / pause / second_half. |
| Tile + toggle | `switch.study_water_pump` | Pump | Manual pump power only - does **not** start the watering sequence. OFF during a sequence half still aborts the sequence. |
| Tile | `input_select.water_garden_phase` | Phase | Shows `idle` / `first_half` / `pause` / `second_half` |
| Tile | `timer.water_garden_segment` | Segment timer | Countdown for the current half or pause |
| Markdown alert | `switch.study_water_pump` (+ phase) | Pump | Green = RUNNING; red = OFF (notes pause if rebalancing). [`dashboard_card_pump_traffic_light.yaml`](yaml/dashboard_card_pump_traffic_light.yaml) |

### Section - Water Butt Gauge

| Card | Entity / content | Name on dashboard | Role |
|------|------------------|-------------------|------|
| Heading | - | Water Butt Gauge | Section title |
| Gauge | `sensor.water_butt_level` | Water Butt Level | Fill % of usable range (0-100) |
| Markdown alert | compares `sensor.water_butt_level` to `sensor.water_butt_minimum_refill` | (under gauge) | Red if not enough to water; green if enough |
| Entities | `sensor.water_butt_litres_left` | Litres left | Usable litres remaining |
| Entities | `sensor.water_butt_litres_required` | Litres required for garden | Calculated: `(red x 2 + black x 4) x Duration/60` |
| Entities (optional) | `sensor.water_garden_flow` | Flow | Calculated L/h from dripper counts |
| Entities | `sensor.water_garden_litres_used_session` | Last litres used (actual) | Live: `(current_cm - start_cm) x L/cm` from last Start; check after butts equalise |
| Entities | `input_number.water_garden_session_expected_litres` | Last litres required (expected) | Snapshot of Litres required at sequence start |
| Entities (optional) | `sensor.water_garden_last_usage_delta` | Last watering vs required | actual - expected (L); positive = used more than expected |

### Section - Automation Settings

| Card | Entity | Name on dashboard | Role |
|------|--------|-------------------|------|
| Heading | - | Automation Settings | Section title |
| Tile | `input_number.water_garden_duration_minutes` | Duration (min) | Total pump-on time; also drives Litres required |
| Tile | `input_number.water_garden_pause_minutes` | Pause (min) | Off time between halves (min **1**; default 15) |
| Tile | `input_number.water_garden_dripper_red_count` | Red drippers (2 L/h) | Fitted count - update when you add/remove reds |
| Tile | `input_number.water_garden_dripper_black_count` | Black drippers (4 L/h) | Fitted count - update when you add/remove blacks |
| Tile | `sensor.rain_minutes_today` | Rain today | Display only (no auto skip yet) |
| Tile | `sensor.ultrasonic_waterbutt` | Level | Raw distance cm from lid sensor to water |
| Tile | `input_number.water_butt_max_high_level` | Max High | Calibration: cm when butts full |
| Tile | `input_number.water_butt_max_low_level` | Max Low | Calibration: cm when pump will run dry |
| Tile | `input_number.water_butt_litres_per_cm` | litre/cm | Calibration: litres per 1 cm level change |

**Litres required formula:** `(Red x 2 + Black x 4) x (Duration minutes / 60)`. Example: 57 red + 31 black, Duration 60 -> `(114+124) x 1 = 238 L`.

**Actual litres used (last sequence):** at Start, HA stores the ultrasonic distance (cm) and expected litres. **Last litres used (actual)** is a live sensor: `(current_cm - start_cm) x litres_per_cm` (distance rises as the butts empty). It is not sent on Telegram - check the dashboard after the butts equalise (e.g. an hour later). It keeps updating until the next Start.

**Dashboard change:** remove the old **Requirement (L)** / cm requirement tiles if still present. Add **Red drippers** and **Black drippers** tiles. Keep **Litres required for garden** as a read-only sensor. Add **Last litres used (actual)** and **Last litres required (expected)** for calibration (`yaml/dashboard_card_watering_usage.yaml`).

### Section - Water butt top-up (mains)

Package: [`yaml/package_water_butt_topup.yaml`](yaml/package_water_butt_topup.yaml) -> `/config/packages/water_butt_topup.yaml`. Valve: `switch.outside_tap_valve`.

| Card | Entity | Name on dashboard | Role |
|------|--------|-------------------|------|
| Tile + toggle | `input_boolean.water_butt_topup_enabled` | Enable top-up | Master enable/disable for fill logic. OFF aborts any campaign. [`dashboard_card_topup_enabled.yaml`](yaml/dashboard_card_topup_enabled.yaml) |
| Tile | `input_button.water_butt_topup_now` | Top-up now | Force-start a staged campaign while idle (Enable top-up must be ON). Not a valve switch — use Outside Tap Valve for that. |
| Tile | `input_select.water_butt_topup_phase` | Top-up phase | `idle` / `burst` / `settling` / `paused` |
| Tile | `timer.water_butt_topup_segment` | Top-up segment | Burst or gap countdown |
| Tile | `input_number.water_butt_topup_burst_minutes` | Top-up burst (min) | Valve open time (default **30** if unset) |
| Tile | `input_number.water_butt_topup_gap_minutes` | Top-up gap (min) | Off time between bursts (default **210** = 3.5 h if unset) |
| Tile | `input_number.water_butt_topup_fill_margin_cm` | Fill margin (cm) | Early close vs Max High (default **2** if unset) |
| Tile | `sensor.water_butt_litres_to_full` | Litres to full | `(distance - Max High) x L/cm` |
| Tile | `sensor.water_butt_minutes_to_full` | Minutes to full | ETA from remaining litres + SWV flow and burst/gap duty cycle. Attribute `eta` is human text (e.g. `2h 15m`). Unavailable until a flow reading exists. |
| Tile | `input_number.water_butt_total_capacity_litres` | Butt capacity (L) | Nameplate bank capacity (default **630**). Campaign abort if valve puts more than this through. |
| Tile | `input_number.water_butt_topup_max_bursts` | Top-up max bursts | Backup limit if flow meter stuck (default **15**). |
| Tile | `sensor.water_butt_topup_campaign_litres` | Campaign litres added | Litres through valve since this campaign started |
| Tile + toggle | `switch.outside_tap_valve` | Outside Tap Valve | Manual override / visibility |
| Tile | `sensor.outside_tap_valve_litres_total` | Tap litres total | Lifetime litres through the valve (never resets). Add after restart. |
| Tile | `sensor.outside_tap_valve_battery` | Tap battery | Zigbee battery % |
| Tile | `input_number.water_butt_topup_battery_low_percent` | Battery low at (%) | Telegram alert threshold (default **20**). Reminds every 6h while below. |
| Tile | `binary_sensor.outside_tap_valve_water_leak` | Tap leak | Safety trip + disables top-up |
| Tile | `binary_sensor.outside_tap_valve_water_supply` | Tap supply | Skip burst if off |

Paste section: [`yaml/dashboard_section_water_butt_topup.yaml`](yaml/dashboard_section_water_butt_topup.yaml).

**Behaviour:** when Enable top-up is ON and `litres_left` stays below `litres_required` for 2 min, HA opens the tap in bursts until the butts are **truly full** after a gap (distance still at/under Max High + 1 cm). Watering always wins: top-up pauses (valve closed) while a watering sequence runs, then resumes.

**Safety budget:** each campaign may add at most **Butt capacity** litres through the valve (and at most **max bursts**). Exceeding either closes the valve, ends the campaign, and turns **Enable top-up OFF** (Telegram). Turn Enable back ON only after you have checked for leaks / sensor faults.

### Adding these in the UI

1. Restart HA if the package was just deployed (so new entities exist).
2. Confirm entities under **Developer Tools -> States** (search `water_garden`).
3. Edit the Garden view: add **Tile** cards for the run-control and settings entities above; keep the existing gauge section.
4. Or replace the whole Garden view block using [`yaml/dashboard_garden_water_butts.yaml`](yaml/dashboard_garden_water_butts.yaml) (must stay indented as a `views:` list item: `  - type: sections`).

---

## Manual watering design (current intent)

No calendar schedule and no rain-skip automation for now.

**What a Start does:**

1. Run the pump for **50%** of `input_number.water_garden_duration_minutes` (default **60** min -> **30** min).
2. Stop for `input_number.water_garden_pause_minutes` (default **15** min) so linked butts can rebalance.
3. Run the pump for the remaining **50%**.
4. After **each** 50% watering segment, Telegram with **level %** and **litres left** (no distance cm).

**Restart safety:** phase is stored in `input_select.water_garden_phase`; segment timing uses `timer.water_garden_segment` with **restore: true**. A recover automation re-syncs the pump after HA start if needed.

**How to change total run length (e.g. 60 -> 90 min):** set **Duration (min)** on the Garden dashboard (or the helper) to `90`. Each half becomes 45 min; pause stays 15 unless you change Pause too.

---

## Apply checklist (do this in Home Assistant)

**Full step-by-step:** [`HA Update Guide.md`](HA%20Update%20Guide.md).

Deploy path for agents: sync repo package YAML into `/config/packages/` over SSH, enable `homeassistant.packages` if needed, backup first, then `ha core check`. Dashboard cards are instructions only - never write Lovelace from here.

Short summary of intended end state:

### A. Create helpers (UI or YAML package)

| Helper | Entity ID | Initial | YAML |
|--------|-----------|---------|------|
| Duration | `input_number.water_garden_duration_minutes` | 60 | [`helper_water_garden_duration_minutes.yaml`](yaml/helper_water_garden_duration_minutes.yaml) |
| Pause | `input_number.water_garden_pause_minutes` | 15 | [`helper_water_garden_pause_minutes.yaml`](yaml/helper_water_garden_pause_minutes.yaml) |
| Phase | `input_select.water_garden_phase` | idle (options: idle, first_half, pause, second_half) | [`helper_water_garden_phase.yaml`](yaml/helper_water_garden_phase.yaml) |
| Segment timer | `timer.water_garden_segment` | restore enabled | [`helper_timer_water_garden_segment.yaml`](yaml/helper_timer_water_garden_segment.yaml) |
| Start toggle | `switch.water_garden_watering` | ON starts / OFF stops (preferred) | [`dashboard_card_main_start_watering.yaml`](yaml/dashboard_card_main_start_watering.yaml) |
| Start button (legacy) | `input_button.water_garden_start` | Also toggles start/stop | [`helper_water_garden_start_button.yaml`](yaml/helper_water_garden_start_button.yaml) |

UI path: **Settings -> Devices & services -> Helpers -> Create helper**. Match entity IDs exactly (or rename after create). For the timer, enable restore if the UI offers it.

### B. Replace scripts

| Script | Action |
|--------|--------|
| `script.water_garden_start` | Paste / replace from [`script_water_garden_start.yaml`](yaml/script_water_garden_start.yaml) |
| `script.water_garden_stop` | Paste / replace from [`script_water_garden_stop.yaml`](yaml/script_water_garden_stop.yaml) |

### C. Automations - replace, add, delete

| Automation | Action | YAML |
|------------|--------|------|
| Water Garden Start (`1784746590314`) | **Replace** | [`automation_water_garden_start.yaml`](yaml/automation_water_garden_start.yaml) |
| Water Garden - Stop (`1784825343018`) | **Replace** | [`automation_water_garden_stop.yaml`](yaml/automation_water_garden_stop.yaml) |
| Water Garden - Pump is on but Water is too low (`1784745405904`) | **Replace** | [`automation_water_garden_pump_on_water_too_low.yaml`](yaml/automation_water_garden_pump_on_water_too_low.yaml) |
| Water Garden - Water butt is full (`1784803698288`) | **Replace** (2 min debounce) | [`automation_water_garden_water_butt_full.yaml`](yaml/automation_water_garden_water_butt_full.yaml) |
| Water Garden - Stop after 1 hour (`1784744462305`) | **Disable or delete** | [`automation_water_garden_stop_after_1_hour.yaml`](yaml/automation_water_garden_stop_after_1_hour.yaml) |
| Water Garden - Segment finished | **Create new** | [`automation_water_garden_segment_finished.yaml`](yaml/automation_water_garden_segment_finished.yaml) |
| Water Garden - Recover after HA restart | **Create new** | [`automation_water_garden_ha_start_recover.yaml`](yaml/automation_water_garden_ha_start_recover.yaml) |

### D. Dashboard

Update the Garden / `water-butts` view from [`dashboard_garden_water_butts.yaml`](yaml/dashboard_garden_water_butts.yaml) (adds Start button, phase, timer, duration, pause tiles).

### E. How to run a watering

1. Set **Duration** (e.g. 60 or 90) and **Pause** (usually 15).
2. Press **Start watering** (preferred), or turn the pump switch ON while phase is `idle`.
3. Watch Telegram + **Phase** / **Segment timer** on the dashboard.
4. To abort mid-run: turn the pump switch OFF (only aborts during `first_half` / `second_half`), or run `script.water_garden_stop`.

---

## Overview

| Concern | HA role |
|---------|---------|
| Start watering | **Start Watering** button/script only -> split-run sequence |
| Manual pump | `switch.study_water_pump` on/off alone - no sequence |
| Run length | `input_number.water_garden_duration_minutes` split into two halves |
| Rebalance pause | `input_number.water_garden_pause_minutes` (default 15) |
| Stop / abort | Pump OFF during a half, low-water, or stop script |
| Skip when butts low | Block start if distance above Max Low; abort mid-run on same |
| Skip after rain | Shown on dashboard only - not automated yet |
| Alerts | Telegram on start, each half done, abort; full/enough with debounce |

---

## Entities and devices

| Role | Entity / notes | YAML |
|------|----------------|------|
| Zigbee plug (pump + solenoid) | `switch.study_water_pump` | *device config pending* |
| Water-butt distance | `sensor.ultrasonic_waterbutt` - **cm** (see ESPHome below) | [`esphome_esp_home_waterbutt.yaml`](yaml/esphome_esp_home_waterbutt.yaml) |
| Start / stop | `switch.water_garden_watering`, `script.water_garden_start`, `script.water_garden_stop` | Toggle preferred; scripts underneath | [`dashboard_card_main_start_watering.yaml`](yaml/dashboard_card_main_start_watering.yaml) |
| Telegram | `notify.notifier_telegram_leon` | *pending* |
| Rain today | `sensor.rain_minutes_today` - display only | *pending* |

---

## ESPHome (water butt)

Stored: [`yaml/esphome_esp_home_waterbutt.yaml`](yaml/esphome_esp_home_waterbutt.yaml)

| Setting | Value | Meaning |
|---------|-------|---------|
| Name | `esp-home-waterbutt` | Host / device name |
| Board | D1 Mini Pro | ESP8266 |
| Static IP | 192.168.1.31 | LAN |
| Sensor name | Ultrasonic Waterbutt | HA entity `sensor.ultrasonic_waterbutt` |
| Unit | **cm** | All HA templates and Telegram assume cm |
| `multiply: 100` | raw metres x 100 | Converts the ultrasonic platform's metre reading into cm |
| `update_interval` | **30s** | Plenty fast vs a multi-minute watering half |
| `timeout` | 4m | Max wait for echo |
| Pins | D1 TRIG (inverted), D2 ECHO | AJ-SR04M style |

**Units clarification (previous concern):** there is no m vs cm mismatch in the live firmware. The entity is explicitly `unit_of_measurement: "cm"` with `multiply: 100`. Telegram and helpers using cm are correct. Prefer moving the OTA password into `!secret` later (it is currently in plain YAML as provided).

---

## Helpers - water butt (calibration)

Distance = **cm from top sensor to water** (larger cm = lower water).

**Usable depth (cm)** = `max_low_level - max_high_level`  
**Usable capacity (L)** = usable depth x `litres_per_cm`

### Input numbers

| Helper | Entity | Purpose |
|--------|--------|---------|
| Litres per cm | `input_number.water_butt_litres_per_cm` | Litres for a 1 cm level change across the linked bank |
| Max High Level | `input_number.water_butt_max_high_level` | cm when full |
| Max Low Level | `input_number.water_butt_max_low_level` | cm when pump will run dry |
| Requirement (litres) | `sensor.water_butt_litres_required` | Calculated from red/black counts x Duration |
| Red drippers (2 L/h) | `input_number.water_garden_dripper_red_count` | Fitted red count (default 57) |
| Black drippers (4 L/h) | `input_number.water_garden_dripper_black_count` | Fitted black count (default 31) |
| Minimum Refill Level (legacy cm) | `input_number.water_butt_minimum_refill_level` | Old cm setpoint - unused |

### Template sensors

| Helper | unique_id | Computes | YAML |
|--------|-----------|----------|------|
| Water Butt Level | `water_butt_level_percent` | Fill % of usable range | [`water_butt_level.yaml`](yaml/water_butt_level.yaml) |
| Water Butt Litres Left | `water_butt_litres_left` | `(low - distance) x L/cm` | [`water_butt_litres_left.yaml`](yaml/water_butt_litres_left.yaml) |
| Water Butt Minimum Refill % | `water_butt_minimum_refill_percent` | cm setpoint as % (entity `sensor.water_butt_minimum_refill`) | [`water_butt_minimum_refill_percent.yaml`](yaml/water_butt_minimum_refill_percent.yaml) |
| Water Butt Litres Required | `water_butt_litres_required` | Litres from dry up to minimum refill cm | [`water_butt_litres_required.yaml`](yaml/water_butt_litres_required.yaml) |

---

## Helpers - watering run

| Helper | Entity | Purpose |
|--------|--------|---------|
| Duration | `input_number.water_garden_duration_minutes` | Total pump-on time (both halves). Change 60 to 90 here. |
| Pause | `input_number.water_garden_pause_minutes` | Off time between halves (default 15) |
| Phase | `input_select.water_garden_phase` | idle / first_half / pause / second_half |
| Segment timer | `timer.water_garden_segment` | Times the current half or pause; **must restore** |
| Start toggle | `switch.water_garden_watering` | ON while sequence active (not idle); turn on/off starts/stops | package template switch |
| Start button (legacy) | `input_button.water_garden_start` | Still toggles start/stop if kept on a dashboard | [`helper_water_garden_start_button.yaml`](yaml/helper_water_garden_start_button.yaml) |

---

## Scripts

| Script | Behaviour | YAML |
|--------|-----------|------|
| `script.water_garden_start` | If idle and not dry: set `first_half`, pump ON, start half-duration timer, Telegram start (level %, litres). Blocks start if already running or water too low (turns pump off if needed). | [`script_water_garden_start.yaml`](yaml/script_water_garden_start.yaml) |
| `script.water_garden_stop` | Cancel timer, phase `idle`, pump OFF, one Telegram (optional `reason` field). | [`script_water_garden_stop.yaml`](yaml/script_water_garden_stop.yaml) |

---

## Automations

### Split-run state machine

| Alias | Behaviour | YAML |
|-------|-----------|------|
| **Water Garden - Segment finished** | On `timer.finished`: first_half -> pump off, Telegram first 50% DONE (level % + litres), start pause timer; pause -> second_half pump on + timer + Telegram second 50% START; second_half -> idle, pump off, Telegram second 50% DONE. Sets phase **before** switching the pump so the stop automation does not abort. | [`automation_water_garden_segment_finished.yaml`](yaml/automation_water_garden_segment_finished.yaml) |
| **Water Garden - Recover after HA restart** | If phase is a watering half and pump is off -> turn on; if pause and pump on -> turn off. | [`automation_water_garden_ha_start_recover.yaml`](yaml/automation_water_garden_ha_start_recover.yaml) |

### Start / stop / safety

| Alias | id | Behaviour | YAML |
|-------|-----|-----------|------|
| **Water Garden Start** | `1784746590314` | Legacy button: idle -> start, else stop | [`automation_water_garden_start.yaml`](yaml/automation_water_garden_start.yaml) |
| **Water Garden - Stop** | `1784825343018` | Pump OFF only while `first_half` or `second_half` -> stop script (avoids double notify on planned half ends) | [`automation_water_garden_stop.yaml`](yaml/automation_water_garden_stop.yaml) |
| **Water Garden - Pump is on but Water is too low** | `1784745405904` | Distance above Max Low during a watering half -> stop with reason (sensor trigger, not /5 poll) | [`automation_water_garden_pump_on_water_too_low.yaml`](yaml/automation_water_garden_pump_on_water_too_low.yaml) |
| **Water Garden - Water butt is full** | `1784803698288` | Full / enough Telegram with **2 minute** `for` debounce | [`automation_water_garden_water_butt_full.yaml`](yaml/automation_water_garden_water_butt_full.yaml) |
| **Water Garden - Stop after 1 hour** | `1784744462305` | **Delete/disable** - conflicts with duration helper | [`automation_water_garden_stop_after_1_hour.yaml`](yaml/automation_water_garden_stop_after_1_hour.yaml) |

### Why the old stop loop is fixed

Previously: any pump OFF called the stop script (turn_off + notify), so timeout/low-water caused duplicate Telegrams. Now planned half-ends set phase to `pause` or `idle` **before** turning the pump off, so the stop automation ignores those transitions.

---

## Dashboard - Garden (`water-butts`)

Full view: [`dashboard_garden_water_butts.yaml`](yaml/dashboard_garden_water_butts.yaml)

| Section | Contents |
|---------|----------|
| Pump / run | Start button, pump toggle, phase, segment timer |
| Water Butt Gauge | Level %, enough banner, litres left / required |
| Automation Settings | Duration, Pause, rain today, ultrasonic cm, calibration input_numbers |

---

## Implementation notes (remaining recommendations)

1. **Stop notify loop** - fixed by phase-gated stop automation + phase-before-switch in segment finished.
2. **Low-water trigger** - fixed: `numeric_state` on ultrasonic, only while watering halves (no 5-minute poll). With 30 s ESPHome updates this reacts within about one reading.
3. **Full/enough debounce** - fixed: `for: 00:02:00`.
4. **Pre-start dry guard** - in start script (blocks + Telegram).
5. **Schedule / rain skip** - deferred (not wanted yet).
6. **Pump entity name** - left as `switch.study_water_pump`.
7. **OTA password** - consider `!secret` in ESPHome; functionality unchanged.

**Calibration tip:** after each half Telegram, compare litres left drop to expected half-run use (about half of ~240 L once Stage 2 is ~80 drippers - or ~60 L per half at Stage 1 ~120 L/h for 30 min). Adjust `litres_per_cm` and level helpers until the numbers make sense.

---

## Change log

| Date | Change | YAML file(s) |
|------|--------|--------------|
| 2026-07-24 | Scaffold + helpers/templates/automations/dashboard/scripts (initial capture) | see `yaml/` |
| 2026-07-24 | Plain-text pass on project markdown | - |
| 2026-07-24 | ESPHome stored; units clarified (cm) | `esphome_esp_home_waterbutt.yaml` |
| 2026-07-24 | Split-run design (50/pause/50), duration helper, restart-safe timer, automation fixes | `helper_water_garden_*.yaml`, `script_water_garden_*.yaml`, `automation_water_garden_*.yaml`, dashboard updates |
| 2026-07-24 | HA host: removed `packages/battery_alert.yaml`; enabled `homeassistant.packages`; `ha core check` OK | host `/config` |
| 2026-07-24 | Deployed split-run package; removed old WG from automations/scripts; `ha core check` OK | `yaml/package_water_garden.yaml` -> `/config/packages/water_garden.yaml` |
| 2026-07-25 | Pump toggle no longer starts sequence; Duration/Pause min 1; Requirement in litres | `package_water_garden.yaml`, `configuration.yaml` templates moved into package |
| 2026-07-25 | Litres required from red/black dripper counts x Duration | `package_water_garden.yaml` |
| 2026-07-26 | Staged mains top-up via Outside Tap Valve; auto when not enough; pause for watering | `package_water_butt_topup.yaml` |

---

*Descriptions only in this file. Store all YAML under [`yaml/`](yaml/).*

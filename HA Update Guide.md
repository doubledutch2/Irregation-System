# Home Assistant - full update guide

## Project deploy rules

1. **Never update Dashboards** from SSH/repo automation - give UI instructions only (see Part 6).
2. **Backup** every file on the HA host before editing (`*.bak-YYYYMMDD-HHMMSS`), and copy those backups into this repo under `ha-backups/YYYY-MM-DD/` (gitignored).
3. **Packages only:** new Water Garden config goes in `/config/packages/` (repo builds a package; sync that file). Do not add new Water Garden logic into top-level `automations.yaml` / `scripts.yaml` when the package can own it.
4. **Test deploy:** after config writes, run `ha core check` over SSH. Do **not** restart Home Assistant - the user restarts manually after a successful check.SSH: `root@192.168.1.49` port `15005`. Config root: `/config`.

### Packages note (important)

On this HA host, packages are **enabled**:

```yaml
homeassistant:
  packages: !include_dir_named packages
```

`battery_alert.yaml` was removed from `/config/packages/` (backup kept as `battery_alert.yaml.bak-*`). Put Water Garden package YAML here.

Enabled and verified with `ha core check` on 2026-07-24.
### Test deploy command

```bash
ssh -p 15005 root@192.168.1.49 'ha core check'
```

Exit success = safe to restart later. Failure = fix or restore backups; do not restart.

---

## UI path (if you prefer clicking) vs packages path

- **Preferred for agents:** one package file under `/config/packages/water_garden.yaml` (helpers + scripts + automations), enable packages, `ha core check`, then you update the dashboard yourself.
- **UI path below** remains useful for understanding entity IDs and for dashboard-only work.

Do this with the pump **off** and phase not mid-run. Keep Telegram nearby so you can see test messages.

All source YAML fragments live under [`yaml/`](yaml/). Entity IDs must match exactly.

---

## Part 0 - Before you start

1. Confirm these already exist and work:
   - `switch.study_water_pump`
   - `sensor.ultrasonic_waterbutt` (cm)
   - `sensor.water_butt_level`, `sensor.water_butt_litres_left`, `sensor.water_butt_minimum_refill`, `sensor.water_butt_litres_required`
   - water-butt `input_number.*` helpers
   - `notify.notifier_telegram_leon`
   - scripts `script.water_garden_start` and `script.water_garden_stop` (package will replace behaviour; disable or remove old YAML copies to avoid duplicates)
2. Optional: screenshot Garden dashboard and Water Garden automations for rollback reference.
3. ESPHome: no change required ([`yaml/esphome_esp_home_waterbutt.yaml`](yaml/esphome_esp_home_waterbutt.yaml)).

---

## Part 1 - Create 5 new helpers

Go to **Settings -> Devices & services -> Helpers -> Create helper**.

### 1.1 Duration (Number)

1. Choose **Number**.
2. Name: `Water Garden Duration`
3. Minimum `10`, Maximum `180`, Step `5`
4. Unit of measurement: `min`
5. Display mode: **Box** (or Slider - either works)
6. Initial / value: `60`
7. Save.
8. Open the helper -> gear / entity settings -> set **Entity ID** to:

   `input_number.water_garden_duration_minutes`

Reference: [`yaml/helper_water_garden_duration_minutes.yaml`](yaml/helper_water_garden_duration_minutes.yaml)

### 1.2 Pause (Number)

1. Create helper -> **Number**
2. Name: `Water Garden Pause`
3. Min `5`, Max `60`, Step `5`, Unit `min`, Initial `15`
4. Entity ID:

   `input_number.water_garden_pause_minutes`

Reference: [`yaml/helper_water_garden_pause_minutes.yaml`](yaml/helper_water_garden_pause_minutes.yaml)

### 1.3 Phase (Dropdown)

1. Create helper -> **Dropdown**
2. Name: `Water Garden Phase`
3. Options - add exactly these four (order matters for readability):
   - `idle`
   - `first_half`
   - `pause`
   - `second_half`
4. Initial option: `idle`
5. Entity ID:

   `input_select.water_garden_phase`

Reference: [`yaml/helper_water_garden_phase.yaml`](yaml/helper_water_garden_phase.yaml)

### 1.4 Segment timer (Timer)

1. Create helper -> **Timer**
2. Name: `Water Garden Segment`
3. If you see **Restore** / restore state: **enable it** (required so HA restart mid-run keeps remaining time).
4. Entity ID:

   `timer.water_garden_segment`

Reference: [`yaml/helper_timer_water_garden_segment.yaml`](yaml/helper_timer_water_garden_segment.yaml)

### 1.5 Start button (Button)

1. Create helper -> **Button**
2. Name: `Water Garden Start`
3. Entity ID:

   `input_button.water_garden_start`

Reference: [`yaml/helper_water_garden_start_button.yaml`](yaml/helper_water_garden_start_button.yaml)

### 1.6 Check

Developer Tools -> States: confirm all five entity IDs exist. Set Phase to `idle` if it is not.

---

## Part 2 - Replace both scripts

Go to **Settings -> Automations & scenes -> Scripts**.

### 2.1 Water Garden - Start

1. Open the existing script **Water Garden - Start** (`script.water_garden_start`).
2. Click the **three dots** -> **Edit in YAML** (wording may be "Edit as YAML").
3. Select all and replace with the full contents of:

   [`yaml/script_water_garden_start.yaml`](yaml/script_water_garden_start.yaml)

4. Save.
5. If HA complains about `stop: ""`, use the UI equivalent: in each `then` branch add action **Stop** / **Stop script** instead of the YAML `stop` key (same meaning).

### 2.2 Water Garden - Stop

1. Open **Water Garden - Stop** (`script.water_garden_stop`).
2. Edit in YAML.
3. Replace with full contents of:

   [`yaml/script_water_garden_stop.yaml`](yaml/script_water_garden_stop.yaml)

4. Save.
5. Confirm the script shows a field **Reason** (optional text). Automations can pass `reason: STOP - water too low`.

---

## Part 3 - Disable the old 1-hour stop

1. **Settings -> Automations & scenes -> Automations**
2. Find **Water Garden - Stop after 1 hour**
3. Toggle it **Off**, or delete it.
4. Do **not** leave it enabled - it fights the new duration helper / split run.

---

## Part 4 - Replace 4 existing automations

For each: open automation -> three dots -> **Edit in YAML** -> replace entire document with the file below -> Save.

Keep the same automation (so history/id stays). You are replacing the body.

### 4.1 Water Garden Start

File: [`yaml/automation_water_garden_start.yaml`](yaml/automation_water_garden_start.yaml)

What it should do after save:

- Triggers: Start button pressed **or** `switch.study_water_pump` turns on
- Condition: Phase is `idle`
- Action: run `script.water_garden_start`

### 4.2 Water Garden - Stop

File: [`yaml/automation_water_garden_stop.yaml`](yaml/automation_water_garden_stop.yaml)

What it should do:

- Trigger: pump turns off
- Condition: Phase is `first_half` **or** `second_half` only
- Action: run `script.water_garden_stop`

### 4.3 Water Garden - Pump is on but Water is too low

File: [`yaml/automation_water_garden_pump_on_water_too_low.yaml`](yaml/automation_water_garden_pump_on_water_too_low.yaml)

What it should do:

- Trigger: `sensor.ultrasonic_waterbutt` goes **above** `input_number.water_butt_max_low_level` (delete any time-pattern `/5` trigger)
- Condition: Phase is `first_half` or `second_half`
- Action: `script.water_garden_stop` with data `reason: STOP - water too low`

### 4.4 Water Garden - Water butt is full

File: [`yaml/automation_water_garden_water_butt_full.yaml`](yaml/automation_water_garden_water_butt_full.yaml)

What it should do:

- Two numeric triggers (full / enough), each with **For: 0:02:00**
- Same Telegram message template as before (trigger id `full` vs `enough`)

---

## Part 5 - Create 2 new automations

**Settings -> Automations & scenes -> Create automation -> Create new automation** -> three dots -> **Edit in YAML** -> paste file -> Save.

### 5.1 Water Garden - Segment finished

Paste: [`yaml/automation_water_garden_segment_finished.yaml`](yaml/automation_water_garden_segment_finished.yaml)

This is the state machine. When the segment timer finishes it:

| Current phase | What happens |
|---------------|--------------|
| `first_half` | Set phase `pause`, pump OFF, Telegram "first 50% DONE" (level %, litres, cm), start pause timer |
| `pause` | Set phase `second_half`, pump ON, start second half timer, Telegram "second 50% START" |
| `second_half` | Set phase `idle`, pump OFF, Telegram "second 50% DONE" |

If YAML paste fails in UI: build with **Trigger type: Event**, event type `timer.finished`, event data entity_id `timer.water_garden_segment`, then a **Choose** action matching the file.

### 5.2 Water Garden - Recover after HA restart

Paste: [`yaml/automation_water_garden_ha_start_recover.yaml`](yaml/automation_water_garden_ha_start_recover.yaml)

- Trigger: Home Assistant start
- Condition: Phase is not `idle`
- Actions: if watering half and pump off -> turn on; if pause and pump on -> turn off; Telegram note

---

## Part 6 - Dashboard (you do this - agents will not edit Lovelace)

Agents must **not** edit `/config/.storage/lovelace*` or any dashboard YAML on the host.

Update the **Garden** / `water-butts` view yourself:

1. Edit dashboard -> Garden view.
2. Add/adjust cards as in [`yaml/dashboard_garden_water_butts.yaml`](yaml/dashboard_garden_water_butts.yaml), or rebuild by hand:

**Section - run controls**

| Card | Entity | Name |
|------|--------|------|
| Tile | `input_button.water_garden_start` | Start watering |
| Tile + toggle | `switch.study_water_pump` | Pump |
| Tile | `input_select.water_garden_phase` | Phase |
| Tile | `timer.water_garden_segment` | Segment timer |

**Section - Water Butt Gauge** - keep existing gauge / markdown / litres.

**Section - Automation Settings** - keep existing tiles, and add:

| Card | Entity | Name |
|------|--------|------|
| Tile | `input_number.water_garden_duration_minutes` | Duration (min) |
| Tile | `input_number.water_garden_pause_minutes` | Pause (min) |

3. Save.
---

## Part 7 - How to change run length later

On the Garden dashboard, change **Duration (min)**:

| Duration | Each half | Pause (default) | Total wall clock |
|----------|-----------|-----------------|------------------|
| 60 | 30 min | 15 min | ~75 min |
| 90 | 45 min | 15 min | ~105 min |

Change **Pause (min)** only if you want a longer/shorter rebalance (default 15).

You do **not** need to edit automations to change 60 vs 90.

---

## Part 8 - Test plan (do in order)

Use a short duration for the first test so you are not waiting an hour.

### 8.1 Dry test of helpers

1. Set Duration to `2` (min), Pause to `1` (min).
2. Confirm Phase is `idle`.
3. Confirm pump is off.

### 8.2 Happy path (2 min total watering)

1. Press **Start watering**.
2. Expect within seconds:
   - Phase -> `first_half`
   - Pump ON
   - Segment timer counting down ~1:00
   - Telegram: START (first 50%) with level %, litres, cm
3. When first half ends (~1 min):
   - Phase -> `pause`
   - Pump OFF
   - Telegram: first 50% DONE with level %, litres, cm
   - Timer counting pause (~1:00)
4. When pause ends:
   - Phase -> `second_half`
   - Pump ON
   - Telegram: second 50% START
5. When second half ends:
   - Phase -> `idle`
   - Pump OFF
   - Telegram: second 50% DONE
6. Set Duration back to `60` and Pause to `15` when finished testing.

### 8.3 Abort test

1. Duration `10`, Start watering.
2. During `first_half`, turn **Pump** OFF.
3. Expect: Phase `idle`, timer cancelled, one STOP Telegram.
4. Should **not** get a second duplicate STOP from an old automation.

### 8.4 Blocked start (optional)

1. Temporarily set Max Low **below** current distance (so "above Max Low" is true - water "too low"), **or** wait until butts are actually low.
2. Press Start.
3. Expect: pump off, Telegram start blocked.
4. Restore Max Low to the real dry setpoint.

### 8.5 Restart safety (optional but recommended)

1. Start a run with Duration `10`.
2. While `first_half` and timer running, **Restart Home Assistant**.
3. After HA is up: Phase should still be `first_half`, timer should still be active (restore), pump should be ON (recover automation if needed).
4. Let it finish or abort with Pump OFF.

---

## Part 9 - Normal daily use

1. Check gauge / enough banner / litres.
2. Set Duration (60 or 90) if needed.
3. Press **Start watering**.
4. Use Telegram + Phase/timer tiles to watch progress.
5. Abort: Pump OFF during a watering half, or run script **Water Garden - Stop**.

Do **not** rely on turning the pump on during `pause` or mid-run to "help" - use Start only from `idle`.

---

## Part 10 - Troubleshooting

| Symptom | Check |
|---------|-------|
| Start does nothing | Phase must be `idle`. Entity IDs for button/phase/timer must match. Start automation enabled. |
| Pump on but no timer | Start script did not complete - check traces for script.water_garden_start. |
| Two Telegrams on every stop | Old Stop automation still calling stop on every pump-off without phase condition - re-apply Part 4.2. |
| Run stops at 60 min early | Old "Stop after 1 hour" still enabled - Part 3. |
| After HA restart pump stays off mid-half | Recover automation missing/disabled; timer restore disabled on helper. |
| Timer finished but stuck in phase | Segment finished automation missing or Choose conditions wrong - check automation trace. |
| Telegram missing level % / litres | Template sensors unavailable - fix water-butt helpers/templates first. |
| YAML paste errors on `stop: ""` | Use UI action "Stop script" in those branches instead. |

**Automation / script traces:** open the item -> Traces (or "Run" history) to see which Choose branch ran.

---

## File map (what each YAML is for)

| File | Use in HA |
|------|-----------|
| `helper_water_garden_duration_minutes.yaml` | Create Number helper |
| `helper_water_garden_pause_minutes.yaml` | Create Number helper |
| `helper_water_garden_phase.yaml` | Create Dropdown helper |
| `helper_timer_water_garden_segment.yaml` | Create Timer helper (restore on) |
| `helper_water_garden_start_button.yaml` | Create Button helper |
| `script_water_garden_start.yaml` | Replace start script |
| `script_water_garden_stop.yaml` | Replace stop script |
| `automation_water_garden_start.yaml` | Replace Start automation |
| `automation_water_garden_stop.yaml` | Replace Stop automation |
| `automation_water_garden_pump_on_water_too_low.yaml` | Replace low-water automation |
| `automation_water_garden_water_butt_full.yaml` | Replace full/enough automation |
| `automation_water_garden_stop_after_1_hour.yaml` | Disable/delete only |
| `automation_water_garden_segment_finished.yaml` | Create new |
| `automation_water_garden_ha_start_recover.yaml` | Create new |
| `dashboard_garden_water_butts.yaml` | Replace Garden view |
| `esphome_esp_home_waterbutt.yaml` | Reference only (no change required) |

---

*After this guide works, set Duration back to 60 (or 90 for heatwave) and Pause to 15.*

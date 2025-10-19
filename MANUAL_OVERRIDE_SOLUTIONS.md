# 🔧 Manual Override Solutions for Climate Scheduler

## Problem Summary

**Issue:** Climate Scheduler overwrites manual temperature changes every 5 minutes  
**Impact:** Can't manually adjust temperature without it reverting  
**Cause:** Scheduler `update_interval: "00:05:00"` enforces schedule regardless of manual changes

---

## Solution 1: Toggle Scheduler ON/OFF (EASIEST) ⭐

### Concept:
When you want manual control, temporarily turn OFF the scheduler switch.

### How:

**In Home Assistant Dashboard:**

1. **Add scheduler switches to dashboard:**
   ```yaml
   # In your dashboard YAML:
   type: entities
   title: Heating Schedulers
   entities:
     - switch.living_room_heating_scheduler
     - switch.bedroom_dormitor_heating_scheduler
     - switch.office_birou_heating_scheduler
     - switch.child_room_cam_luca_heating_scheduler
   ```

2. **When you need manual control:**
   - Turn OFF specific scheduler (e.g., Living Room)
   - Adjust temperature manually
   - Temperature STAYS at your setting
   - Turn scheduler back ON when done

### Pros:
- ✅ Simple (just toggle switch)
- ✅ No configuration changes
- ✅ Full manual control when needed
- ✅ Easy to see which rooms have scheduler disabled

### Cons:
- ⚠️ Must remember to turn scheduler back ON
- ⚠️ If you forget, room won't follow schedule

---

## Solution 2: Increase Update Interval (SIMPLE)

### Current Configuration:
```yaml
# configuration.yaml
climate_scheduler:
  update_interval: "00:05:00"  # Updates every 5 minutes
```

### Recommended Change:
```yaml
# configuration.yaml
climate_scheduler:
  update_interval: "00:15:00"  # Updates every 15 minutes
```

### Impact:

**Before (5-minute updates):**
```
18:00 → Manual: Set to 24°C
18:05 → Scheduler: Overwrites to 22°C ❌ (only 5 min of manual control)
```

**After (15-minute updates):**
```
18:00 → Manual: Set to 24°C
18:15 → Scheduler: Overwrites to 22°C (15 min of manual control)
```

### Your Concern is Valid:
> "If we set in the last 19 min, Climate Scheduler will change back to 22°C in 1 min right?"

**Yes, exactly!** If you change temperature at 18:14, it reverts at 18:15.

### Pros:
- ✅ Easy to implement (one line change)
- ✅ Reduces scheduler "aggressiveness"
- ✅ Longer manual control window

### Cons:
- ⚠️ Still overwrites eventually
- ⚠️ Bad timing = only 1 minute of manual control
- ⚠️ Temperature might drift for 15 min if thermostat needs correction

---

## Solution 3: Manual Override Detection (ADVANCED) ⭐ **BEST**

### Concept:
Create automation that detects manual changes and temporarily disables scheduler.

### How it Works:
```
You manually change temperature
  ↓
Automation detects change
  ↓
Turns OFF scheduler for X hours
  ↓
After X hours, turns scheduler back ON
  ↓
Scheduler resumes normal operation
```

### Implementation:

**Create this automation:**

```yaml
alias: Climate Scheduler - Detect Manual Override
description: Temporarily disable scheduler when manual temperature change detected
triggers:
  # Detect when any thermostat temperature is changed
  - trigger: state
    entity_id:
      - climate.living_calorifer_thermostat
      - climate.bucatarie_calorifer_thermostat
      - climate.hol_intrare_calorifer_thermostat
      - climate.hol_parter_calorifer_thermostat
      - climate.dormitor_calorifer_1_thermostat
      - climate.dormitor_calorifer_2_thermostat
      - climate.birou_calorifer_thermostat
      - climate.sonoff_trvzb_thermostat
    attribute: temperature  # Monitor temperature setpoint changes
    
conditions:
  # Only trigger if change was NOT from scheduler
  # (Scheduler changes have context_user_id = scheduler)
  - condition: template
    value_template: >-
      {{ trigger.to_state.context.user_id != none and
         'climate_scheduler' not in (trigger.to_state.context.parent_id | string) }}

actions:
  # Determine which scheduler to disable based on which thermostat changed
  - choose:
      # Living Room thermostats
      - conditions:
          - condition: template
            value_template: >-
              {{ trigger.entity_id in [
                'climate.living_calorifer_thermostat',
                'climate.bucatarie_calorifer_thermostat',
                'climate.hol_intrare_calorifer_thermostat',
                'climate.hol_parter_calorifer_thermostat'] }}
        sequence:
          - action: switch.turn_off
            target:
              entity_id: switch.living_room_heating_scheduler
          
          # Wait 2 hours, then turn scheduler back ON
          - delay:
              hours: 2
          
          - action: switch.turn_on
            target:
              entity_id: switch.living_room_heating_scheduler
      
      # Bedroom thermostats
      - conditions:
          - condition: template
            value_template: >-
              {{ trigger.entity_id in [
                'climate.dormitor_calorifer_1_thermostat',
                'climate.dormitor_calorifer_2_thermostat'] }}
        sequence:
          - action: switch.turn_off
            target:
              entity_id: switch.bedroom_dormitor_heating_scheduler
          
          - delay:
              hours: 2
          
          - action: switch.turn_on
            target:
              entity_id: switch.bedroom_dormitor_heating_scheduler
      
      # Office thermostat
      - conditions:
          - condition: template
            value_template: >-
              {{ trigger.entity_id == 'climate.birou_calorifer_thermostat' }}
        sequence:
          - action: switch.turn_off
            target:
              entity_id: switch.office_birou_heating_scheduler
          
          - delay:
              hours: 2
          
          - action: switch.turn_on
            target:
              entity_id: switch.office_birou_heating_scheduler
      
      # Child Room thermostat
      - conditions:
          - condition: template
            value_template: >-
              {{ trigger.entity_id == 'climate.sonoff_trvzb_thermostat' }}
        sequence:
          - action: switch.turn_off
            target:
              entity_id: switch.child_room_cam_luca_heating_scheduler
          
          - delay:
              hours: 2
          
          - action: switch.turn_on
            target:
              entity_id: switch.child_room_cam_luca_heating_scheduler

mode: restart  # If another manual change happens, restart timer
```

### How It Works in Practice:

```
18:00 → You manually set Living Room to 24°C
18:00 → Automation detects manual change
18:00 → Turns OFF Living Room scheduler
18:05 → Scheduler tries to update → SKIPPED (scheduler OFF)
19:00 → Still at 24°C (your manual setting)
20:00 → 2 hours passed → Scheduler turns back ON
20:00 → Scheduler resumes normal operation
```

### Customization:

**Change override duration:**
```yaml
- delay:
    hours: 2  # ← Change to 1, 3, 4, etc.
```

**Or use until next schedule change:**
```yaml
- delay:
    # Wait until next schedule entry (e.g., until 22:00 bedtime)
    "{{ (state_attr('switch.living_room_heating_scheduler', 'next_schedule_time') | as_timestamp - now() | as_timestamp) | int }}"
```

### Pros:
- ✅ Automatic detection of manual changes
- ✅ No need to manually toggle scheduler
- ✅ Scheduler auto-resumes after delay
- ✅ Separate control per room

### Cons:
- ⚠️ More complex automation
- ⚠️ Requires testing to ensure detection works
- ⚠️ Fixed override duration (2 hours)

---

## Solution 4: Input Boolean Toggle (HYBRID)

### Concept:
Add toggle button to dashboard that disables ALL schedulers with one tap.

### Implementation:

**Step 1: Create Input Boolean**

```yaml
# configuration.yaml or input_boolean.yaml
input_boolean:
  heating_manual_mode:
    name: Manual Heating Control
    icon: mdi:hand-pointing-up
```

**Step 2: Create Automation to Disable Schedulers**

```yaml
alias: Heating - Enable Manual Mode
triggers:
  - trigger: state
    entity_id: input_boolean.heating_manual_mode
    to: "on"
actions:
  - action: switch.turn_off
    target:
      entity_id:
        - switch.living_room_heating_scheduler
        - switch.bedroom_dormitor_heating_scheduler
        - switch.office_birou_heating_scheduler
        - switch.child_room_cam_luca_heating_scheduler

mode: single
```

**Step 3: Create Automation to Re-enable Schedulers**

```yaml
alias: Heating - Disable Manual Mode
triggers:
  - trigger: state
    entity_id: input_boolean.heating_manual_mode
    to: "off"
actions:
  - action: switch.turn_on
    target:
      entity_id:
        - switch.living_room_heating_scheduler
        - switch.bedroom_dormitor_heating_scheduler
        - switch.office_birou_heating_scheduler
        - switch.child_room_cam_luca_heating_scheduler

mode: single
```

**Step 4: Add to Dashboard**

```yaml
type: button
entity: input_boolean.heating_manual_mode
name: Manual Mode
icon: mdi:hand-pointing-up
tap_action:
  action: toggle
```

### How It Works:

```
Dashboard → Toggle "Manual Mode" ON
  ↓
All 4 schedulers turn OFF
  ↓
Full manual control of all thermostats
  ↓
Adjust temperatures as needed
  ↓
Dashboard → Toggle "Manual Mode" OFF
  ↓
All 4 schedulers turn back ON
  ↓
Schedulers resume normal operation
```

### Pros:
- ✅ One-tap control of all schedulers
- ✅ Visual indicator (button shows ON/OFF)
- ✅ Simple to understand
- ✅ Easy to add to dashboard

### Cons:
- ⚠️ Affects ALL rooms (not per-room)
- ⚠️ Must remember to turn OFF manual mode

---

## Solution 5: Custom Climate Scheduler Modification (EXPERT)

### Concept:
Modify Climate Scheduler integration to detect manual changes and skip next update.

### Implementation:
Would require modifying `custom_components/climate_scheduler/switch.py`:

```python
# In async_update() method:
if self._entity.last_changed_by == "user":
    # Skip this update cycle
    _LOGGER.info(f"Skipping update for {self._entity.name} - manual override detected")
    return
```

### Pros:
- ✅ Built-in detection
- ✅ No extra automations
- ✅ Seamless integration

### Cons:
- ❌ Requires coding knowledge
- ❌ Must maintain fork of integration
- ❌ Breaks on integration updates
- ❌ Complex to implement correctly

**NOT RECOMMENDED** unless you're comfortable with Python development.

---

## 📊 Comparison Table

| Solution | Complexity | Effectiveness | Maintenance | Per-Room Control |
|----------|-----------|---------------|-------------|------------------|
| **Toggle Scheduler** | ⭐ Easy | ✅ Perfect | ✅ None | ✅ Yes |
| **Increase Interval** | ⭐ Easy | ⚠️ Partial | ✅ None | ❌ No |
| **Auto-Detection** | ⭐⭐⭐ Advanced | ✅ Excellent | ⚠️ Monitor | ✅ Yes |
| **Input Boolean** | ⭐⭐ Medium | ✅ Good | ✅ Minimal | ❌ No (all rooms) |
| **Code Modification** | ⭐⭐⭐⭐⭐ Expert | ✅ Perfect | ❌ High | ✅ Yes |

---

## 🎯 My Recommendation

### For You: **Combination Approach**

**Implement BOTH:**

1. **Increase update interval to 15 minutes** (simple, immediate benefit)
2. **Add scheduler toggles to dashboard** (manual override when needed)

### Why This Works Best:

```
Normal operation:
  ↓
Schedulers ON, 15-minute updates
  ↓
Need manual control? Turn OFF specific scheduler
  ↓
Adjust temperature freely
  ↓
Done? Turn scheduler back ON
  ↓
Scheduler resumes at next 15-minute interval
```

### Configuration Changes:

**Step 1: Increase update interval**
```yaml
# configuration.yaml
climate_scheduler:
  update_interval: "00:15:00"  # Changed from 00:05:00
```

**Step 2: Add dashboard card**
```yaml
# In your dashboard:
type: entities
title: Heating Control
entities:
  - entity: switch.living_room_heating_scheduler
    name: Living Room Schedule
  - entity: switch.bedroom_dormitor_heating_scheduler
    name: Bedroom Schedule
  - entity: switch.office_birou_heating_scheduler
    name: Office Schedule
  - entity: switch.child_room_cam_luca_heating_scheduler
    name: Child Room Schedule
```

### Result:

- ✅ Longer manual control window (15 min instead of 5 min)
- ✅ Easy override (toggle scheduler OFF)
- ✅ Visual control on dashboard
- ✅ No complex automations
- ✅ Per-room control

---

## 🔍 Testing Your Choice

### Test Scenario:

1. **Set Living Room to 24°C manually** (at 18:00)
2. **Check what happens** at 18:15 (next update)
3. **Expected:**
   - Scheduler ON → Reverts to 22°C (schedule temperature)
   - Scheduler OFF → Stays at 24°C (your setting)

### Monitor:

```
Developer Tools → States → Search:
climate.living_calorifer_thermostat

Check:
- temperature attribute (setpoint)
- last_changed timestamp
```

---

## 📝 Summary

**Your Question:**
> "What can we do when we have manual adjustment to keep the settings more?"

**Answer:** Three approaches:

1. **Quick Fix:** Increase `update_interval` to `00:15:00` or `00:20:00`
2. **Manual Control:** Toggle scheduler OFF when manual adjustment needed
3. **Advanced:** Auto-detection automation (optional, for power users)

**Recommended:** #1 + #2 (increase interval + add dashboard toggles)

---

**Your Concern:**
> "If we set in the last 19 min, Climate Scheduler will change back in 1 min right?"

**Correct!** This is why toggle control is better than just increasing interval.

**With toggle approach:**
- You turn OFF scheduler when making manual change
- Temperature stays at your setting indefinitely
- Turn scheduler ON when done
- No timing issues!

---

**Status:** 🔧 Solutions provided  
**Recommendation:** Increase interval to 15 min + add dashboard toggles  
**Complexity:** Low (configuration change + dashboard edit)  
**Time to implement:** 10 minutes

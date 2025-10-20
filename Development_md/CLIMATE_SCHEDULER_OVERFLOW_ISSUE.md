# 🔴 NEW ISSUE FOUND: Climate Scheduler Causing Zigbee Overflow

## 🔍 Problem Analysis

### What's Happening

**These errors are NOT from your hybrid automations!** They're from the **Climate Scheduler integration** itself.

### Error Breakdown

#### Error #1: Unsupported Service
```
Error: Entity climate.living_calorifer_thermostat does not support action climate.set_fan_mode
```

**Problem:** Climate Scheduler is trying to set `fan_mode=auto` on thermostats that don't support this feature.

**Affected thermostats:** All 8 thermostats
- Living, Kitchen, Hall Entrance, Hall Ground Floor
- Office, Bedroom 1, Bedroom 2, Child Room

**Why:** Your Sonoff TRV thermostats don't have a fan (they're radiator valves), so `set_fan_mode` doesn't exist.

#### Error #2: Zigbee Queue Overflow (Again!)
```
Error: ZIGBEE_MAX_MESSAGE_LIMIT_REACHED: 3075
Time: 19:54:43 (16 occurrences in 2 seconds!)
```

**Problem:** Climate Scheduler is sending commands to **ALL 8 thermostats simultaneously** when changing schedules.

**Timeline:**
```
19:54:41 → Scheduler tries to set fan_mode on 8 thermostats → 8 messages instantly
19:54:43 → Scheduler tries to set temperature on 8 thermostats → 16+ messages instantly
19:54:45 → Queue overflow! 💥
```

---

## 🎯 Root Cause

### The Climate Scheduler Code Issue

Looking at your scheduler switch (from earlier in conversation), it calls:

```python
# When schedule changes, it executes actions like:
climate.set_fan_mode → ALL thermostats at once
climate.set_hvac_mode → ALL thermostats at once  
climate.set_temperature → ALL thermostats at once
```

**This creates the same problem we just fixed!** All messages sent simultaneously → queue overflow.

---

## ✅ Solutions

### Solution 1: Remove Unsupported Fan Mode (IMMEDIATE)

Your thermostats don't support fan mode, so we need to stop the scheduler from trying to set it.

#### Check Your Climate Scheduler Configuration

1. **Find your scheduler YAML files:**
   ```
   Z:\TradingView\Kiwwi\HomeAsistentGreen\hass-climate-scheduler-main\
   examples\three-profile-system\complete_heating_schedulers.yaml
   ```

2. **Look for `fan_mode` in the actions:**
   ```yaml
   - service: climate.set_fan_mode
     data:
       fan_mode: auto  # ← REMOVE THIS!
   ```

3. **Remove or comment out ALL `set_fan_mode` calls:**
   ```yaml
   # - service: climate.set_fan_mode  # ← Commented out
   #   data:
   #     fan_mode: auto
   ```

### Solution 2: Stagger Climate Scheduler Actions (RECOMMENDED)

The scheduler needs to update thermostats **one at a time** or in **small batches**, not all at once!

#### Option A: Sequential Updates (Safest)

Modify your scheduler switch to update thermostats with delays:

```yaml
action:
  # Group 1: Living Room (4 thermostats)
  - service: climate.set_temperature
    target:
      entity_id:
        - climate.living_calorifer_thermostat
        - climate.bucatarie_calorifer_thermostat
        - climate.hol_intrare_calorifer_thermostat
        - climate.hol_parter_calorifer_thermostat
    data:
      temperature: "{{ temp_value }}"
      hvac_mode: heat
  
  # Wait 2 seconds for Zigbee queue to clear
  - delay:
      seconds: 2
  
  # Group 2: Bedrooms (2 thermostats)
  - service: climate.set_temperature
    target:
      entity_id:
        - climate.dormitor_calorifer_1_thermostat
        - climate.dormitor_calorifer_2_thermostat
    data:
      temperature: "{{ temp_value }}"
      hvac_mode: heat
  
  # Wait 2 seconds
  - delay:
      seconds: 2
  
  # Group 3: Office (1 thermostat)
  - service: climate.set_temperature
    target:
      entity_id:
        - climate.birou_calorifer_thermostat
    data:
      temperature: "{{ temp_value }}"
      hvac_mode: heat
  
  # Wait 2 seconds
  - delay:
      seconds: 2
  
  # Group 4: Child Room (1 thermostat)
  - service: climate.set_temperature
    target:
      entity_id:
        - climate.sonoff_trvzb_thermostat
    data:
      temperature: "{{ temp_value }}"
      hvac_mode: heat
```

#### Option B: Use Parallel Mode with Max (Faster)

```yaml
action:
  - parallel:
      - sequence:
          - service: climate.set_temperature
            target:
              entity_id: climate.living_calorifer_thermostat
            data:
              temperature: "{{ temp_value }}"
              hvac_mode: heat
          - delay:
              milliseconds: 500
      
      - sequence:
          - delay:
              milliseconds: 500
          - service: climate.set_temperature
            target:
              entity_id: climate.bucatarie_calorifer_thermostat
            data:
              temperature: "{{ temp_value }}"
              hvac_mode: heat
      
      # ... repeat for each thermostat with increasing delays
```

---

## 🔧 Immediate Actions

### Action 1: Find Where set_fan_mode Is Called

Search your scheduler configuration:

```powershell
cd Z:\TradingView\Kiwwi\HomeAsistentGreen\hass-climate-scheduler-main
grep -r "set_fan_mode" .
```

Or search in VS Code:
- Press `Ctrl+Shift+F`
- Search for: `set_fan_mode`
- Check results in scheduler YAML files

### Action 2: Remove or Comment Out set_fan_mode

In any file where you find it:

```yaml
# BEFORE (WRONG):
- service: climate.set_fan_mode
  data:
    fan_mode: auto
  target:
    entity_id:
      - climate.living_calorifer_thermostat

# AFTER (FIXED):
# - service: climate.set_fan_mode  # ← Commented out (thermostats don't support this)
#   data:
#     fan_mode: auto
#   target:
#     entity_id:
#       - climate.living_calorifer_thermostat
```

### Action 3: Check When Scheduler Runs

The errors occurred at **19:54:43** - this is likely when:
1. A scheduled time block changed (e.g., 20:00 schedule started)
2. The scheduler tried to update ALL thermostats
3. Queue overflow happened

**Check your schedule times:**
- If you have a 20:00 schedule, that matches the error time (19:54 + 6 min processing)
- The scheduler is triggering mass updates

---

## 📊 Understanding the Timeline

### What Happened at 19:54:

```
19:54:41 → Climate Scheduler activates new schedule
           ↓
19:54:41 → Tries: set_fan_mode on 8 thermostats (FAILS - unsupported)
           ↓
19:54:43 → Tries: set_hvac_mode on 8 thermostats (16 messages)
           ↓
19:54:43 → Tries: set_temperature on 8 thermostats (16+ messages)
           ↓
19:54:45 → ZIGBEE_MAX_MESSAGE_LIMIT_REACHED 💥
```

**Total messages in 4 seconds: 40+ messages!**

---

## 🎯 Recommended Fix Strategy

### Phase 1: Immediate (Fix Errors)
1. ✅ Remove all `set_fan_mode` calls from scheduler
2. ✅ This eliminates Error #1 completely

### Phase 2: Short-term (Reduce Overflow)
1. ✅ Add 2-second delays between thermostat groups
2. ✅ Group thermostats: [4], [2], [1], [1]
3. ✅ This spreads messages across 8 seconds

### Phase 3: Long-term (Optimize)
1. ✅ Modify Climate Scheduler integration to support batching
2. ✅ Or use a custom script to manage updates
3. ✅ Consider rate-limiting in scheduler itself

---

## 📝 Quick Fix Script

Create a custom script to replace scheduler's bulk updates:

```yaml
# scripts.yaml
script:
  heating_set_temperature_staggered:
    alias: "Set Heating Temperature (Staggered)"
    description: "Updates all thermostats with stagger to prevent Zigbee overflow"
    fields:
      temperature:
        description: "Target temperature"
        example: 22
    sequence:
      # Living Room group (4 thermostats)
      - service: climate.set_temperature
        target:
          entity_id:
            - climate.living_calorifer_thermostat
            - climate.bucatarie_calorifer_thermostat
            - climate.hol_intrare_calorifer_thermostat
            - climate.hol_parter_calorifer_thermostat
        data:
          temperature: "{{ temperature }}"
          hvac_mode: heat
      - delay:
          seconds: 2
      
      # Bedroom group (2 thermostats)
      - service: climate.set_temperature
        target:
          entity_id:
            - climate.dormitor_calorifer_1_thermostat
            - climate.dormitor_calorifer_2_thermostat
        data:
          temperature: "{{ temperature }}"
          hvac_mode: heat
      - delay:
          seconds: 2
      
      # Office (1 thermostat)
      - service: climate.set_temperature
        target:
          entity_id:
            - climate.birou_calorifer_thermostat
        data:
          temperature: "{{ temperature }}"
          hvac_mode: heat
      - delay:
          seconds: 2
      
      # Child Room (1 thermostat)
      - service: climate.set_temperature
        target:
          entity_id:
            - climate.sonoff_trvzb_thermostat
        data:
          temperature: "{{ temperature }}"
          hvac_mode: heat
    mode: queued
    max: 5
```

Then use this script in your scheduler instead of direct `climate.set_temperature` calls.

---

## 🔍 How to Find the Problem Code

### Method 1: Search in Home Assistant UI

1. **Settings → Automations & Scenes**
2. **Click "Switches"** tab
3. Find your climate scheduler switches
4. Check each one for `set_fan_mode` calls

### Method 2: Check Configuration Files

Look in these locations:
```
configuration.yaml
automations.yaml
scripts.yaml
packages/*.yaml
custom_components/climate_scheduler/*.yaml
```

Search for:
- `climate.set_fan_mode`
- `fan_mode: auto`
- `service: climate.set_`

---

## ✅ Verification Steps

After removing `set_fan_mode`:

1. **Restart Home Assistant**
2. **Wait for next schedule change**
3. **Check logs** → Should see no more "does not support action climate.set_fan_mode"
4. **Monitor for Zigbee overflow** → May still occur if no delays added

After adding delays:

1. **Trigger a schedule change manually**
2. **Watch logs** → Updates should spread over 8 seconds
3. **No Zigbee overflow errors!** ✅

---

## 🎯 Summary

### Two Separate Issues:

| Issue | Cause | Fix | Priority |
|-------|-------|-----|----------|
| **set_fan_mode errors** | Scheduler calling unsupported service | Remove `set_fan_mode` calls | 🔴 URGENT |
| **Zigbee overflow** | Scheduler updating all 8 at once | Add 2-second delays between groups | 🔴 URGENT |

### Your Hybrid Automations:
- ✅ **NOT the problem** - These are working correctly!
- ✅ **Already staggered** - :00, :02, :05, :08 timing is perfect
- ✅ **Keep as-is** - No changes needed

### The Real Culprit:
- ❌ **Climate Scheduler switch** - Sends bulk updates
- ❌ **No delays** - All thermostats updated simultaneously
- ❌ **Unsupported service** - Trying to set fan_mode

---

## 🚀 Next Steps

1. **Search for `set_fan_mode` in your configuration**
2. **Remove or comment out all instances**
3. **Add delays between thermostat groups in scheduler actions**
4. **Restart Home Assistant**
5. **Test and monitor**

Share the Climate Scheduler configuration files or tell me where the switches are defined, and I'll help you fix them! 🔧

---

**Status:** 🔴 **New issue identified!**  
**Cause:** Climate Scheduler bulk updates  
**Solution:** Remove fan_mode + add delays  
**Hybrid Automations:** ✅ **Still working correctly!**

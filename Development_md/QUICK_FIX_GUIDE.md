# 🚀 Quick Fix Guide - URGENT ACTIONS

## ⚡ Issue #1: HACS Update Failing (404 Error)

### Problem
HACS can't download your integration because there's no GitHub Release with a ZIP file.

### Fix (2 minutes)

**Go to:** https://github.com/Mr4peX/hass-climate-scheduler-weekday/releases/new

**Fill in:**
- **Tag:** `v1.0.0` (select existing)
- **Title:** `Climate Scheduler Weekday v1.0.0`
- **Description:** (copy/paste below)

```markdown
## 🎉 Initial HACS Release - v1.0.0

### What's New
- ✅ Weekday validation fix for same-time schedules
- ✅ Three-profile system (Weekly/Away/Holiday)
- ✅ 252 schedule entries support
- ✅ HACS installation ready

### Installation
Install via HACS by adding custom repository:
`https://github.com/Mr4peX/hass-climate-scheduler-weekday`

### Documentation
📚 Examples and guides on GitHub:
- [CSV Template](https://github.com/Mr4peX/hass-climate-scheduler-weekday/tree/main/examples/schedules)
- [Complete System](https://github.com/Mr4peX/hass-climate-scheduler-weekday/tree/main/examples/three-profile-system)
- [Hybrid Automations](https://github.com/Mr4peX/hass-climate-scheduler-weekday/tree/main/examples/hybrid-external-sensors)
```

**Click:** "Publish release" ✅

**Result:** GitHub auto-generates source code ZIP → HACS can download! ✅

---

## ⚡ Issue #2: Automations Failing (Zigbee Queue Full)

### Problem
All 4 hybrid automations fire at the same time (every 10 minutes at :00, :10, :20, etc.), flooding the Zigbee network with 20+ messages → queue overflow → automations fail.

### Fix (15 minutes)

**Stagger the time patterns** so automations don't fire together.

#### Living Room Automation (4 thermostats)
**Keep as-is:** Fires at :00, :10, :20, :30, :40, :50

```yaml
trigger:
  - platform: state
    entity_id: sensor.living_room_temperature
    for:
      seconds: 30  # ← ADD THIS
  - platform: time_pattern
    minutes: "/10"  # Keep this
```

#### Bedroom Automation (2 thermostats)
**Change to:** Fires at :02, :12, :22, :32, :42, :52

```yaml
trigger:
  - platform: state
    entity_id: sensor.dormitor_temperature
    for:
      seconds: 30  # ← ADD THIS
  - platform: time_pattern
    minutes: "2,12,22,32,42,52"  # ← CHANGE THIS
```

#### Office Automation (1 thermostat)
**Change to:** Fires at :05, :15, :25, :35, :45, :55

```yaml
trigger:
  - platform: state
    entity_id: sensor.birou_temperature
    for:
      seconds: 30  # ← ADD THIS
  - platform: time_pattern
    minutes: "5,15,25,35,45,55"  # ← CHANGE THIS
```

#### Child Room Automation (1 thermostat)
**Change to:** Fires at :08, :18, :28, :38, :48, :58

```yaml
trigger:
  - platform: state
    entity_id: sensor.luca_room_temperature  # or whatever your sensor name is
    for:
      seconds: 30  # ← ADD THIS
  - platform: time_pattern
    minutes: "8,18,28,38,48,58"  # ← CHANGE THIS
```

### How to Apply Changes

1. Open **Home Assistant**
2. Go to **Settings → Automations & Scenes**
3. Find each hybrid automation
4. Click **Edit** (pencil icon)
5. Switch to **YAML mode** (3-dot menu → Edit in YAML)
6. Update the `trigger:` section
7. Add `mode: single` and `max_exceeded: silent` at the bottom
8. **Save** each automation
9. **Reload automations** or restart Home Assistant

### Full Example: Living Room (Complete YAML)

```yaml
alias: Living Room - External Sensor (Hybrid)
description: Sync external temperature with rate limiting
trigger:
  - platform: state
    entity_id: sensor.living_room_temperature
    for:
      seconds: 30
  - platform: time_pattern
    minutes: "/10"
condition: []
action:
  - service: sonoff.send_command
    data:
      device: "{{ device_id(device_entity) }}"
      payload:
        currentTemperature: "{{ (states('sensor.living_room_temperature') | float * 100) | int }}"
    target:
      entity_id:
        - climate.living_calorifer_thermostat
        - climate.bucatarie_calorifer_thermostat
        - climate.hol_intrare_calorifer_thermostat
        - climate.hol_parter_calorifer_thermostat
mode: single  # ← ADD THIS
max_exceeded: silent  # ← ADD THIS
```

---

## 📊 Timeline Comparison

### ❌ BEFORE (All fire together → Queue overflow)
```
19:00:00 → Living (4) + Bedroom (2) + Office (1) + Child (1) = 8 messages 💥
19:10:00 → Living (4) + Bedroom (2) + Office (1) + Child (1) = 8 messages 💥
19:20:00 → Living (4) + Bedroom (2) + Office (1) + Child (1) = 8 messages 💥
```

### ✅ AFTER (Staggered → No overflow)
```
19:00:00 → Living (4 thermostats)
19:02:00 → Bedroom (2 thermostats)
19:05:00 → Office (1 thermostat)
19:08:00 → Child (1 thermostat)

19:10:00 → Living (4 thermostats)
19:12:00 → Bedroom (2 thermostats)
19:15:00 → Office (1 thermostat)
19:18:00 → Child (1 thermostat)
```

**Result:** Messages spread out, Zigbee queue never fills! ✅

---

## ✅ Testing After Fixes

### Test HACS Update
1. Open Home Assistant → HACS
2. Find "Climate Scheduler Weekday"
3. Click 3-dot menu → "Update information"
4. Should show update available
5. Click "Update"
6. Should download successfully! ✅

### Test Automations
1. Wait for next 10-minute mark
2. Check logs: Settings → System → Logs
3. Filter: "automation"
4. Verify:
   - ✅ No "ZIGBEE_MAX_MESSAGE_LIMIT_REACHED" errors
   - ✅ No "device did not respond" errors
   - ✅ Automations execute successfully

### Monitor Execution Times
1. Developer Tools → States
2. Search: `automation.living`
3. Check `last_triggered` attribute
4. Verify staggered timing:
   - Living: ends in :00, :10, :20, :30, :40, :50
   - Bedroom: ends in :02, :12, :22, :32, :42, :52
   - Office: ends in :05, :15, :25, :35, :45, :55
   - Child: ends in :08, :18, :28, :38, :48, :58

---

## 📋 Checklist

**HACS Fix:**
- [ ] Go to GitHub releases page
- [ ] Create new release from tag v1.0.0
- [ ] Add title and description
- [ ] Publish release
- [ ] Verify ZIP download link exists
- [ ] Test HACS update in Home Assistant

**Automation Fix:**
- [ ] Edit Living Room automation (add `for: seconds: 30`)
- [ ] Edit Bedroom automation (change minutes, add `for: seconds: 30`)
- [ ] Edit Office automation (change minutes, add `for: seconds: 30`)
- [ ] Edit Child Room automation (change minutes, add `for: seconds: 30`)
- [ ] Add `mode: single` to all automations
- [ ] Add `max_exceeded: silent` to all automations
- [ ] Save all changes
- [ ] Reload automations
- [ ] Monitor logs for 1 hour
- [ ] Verify no Zigbee errors

---

## 🎯 Expected Results

### After HACS Fix:
- ✅ HACS shows update available
- ✅ Update downloads successfully
- ✅ Integration updated to v1.0.0
- ✅ No more 404 errors

### After Automation Fix:
- ✅ Automations fire at different times
- ✅ No Zigbee queue overflow
- ✅ Thermostats receive temperature updates
- ✅ No "device did not respond" errors
- ✅ System runs smoothly 24/7

---

## 📞 Need Help?

If issues persist:

1. **HACS still shows 404:**
   - Wait 5 minutes for GitHub CDN cache
   - Clear HACS cache: Settings → HACS → 3-dot menu → Clear cache
   - Restart Home Assistant

2. **Zigbee errors continue:**
   - Increase time intervals to 15 minutes instead of 10
   - Check Zigbee network health (Settings → Devices → Zigbee)
   - Add more Zigbee routers (powered devices)

3. **Automations not firing:**
   - Check Developer Tools → Events
   - Listen to: `state_changed` and `time_pattern`
   - Verify sensors are updating

---

**Created:** October 19, 2025  
**Fixes:** 2 critical issues  
**Time Required:** ~20 minutes total  
**Difficulty:** Easy (follow step-by-step)

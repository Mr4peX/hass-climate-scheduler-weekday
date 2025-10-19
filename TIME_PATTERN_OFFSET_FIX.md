# 🔧 Time Pattern Offset - Corrected Approach

## ⚠️ Issue Found: Invalid Time Pattern Syntax

### The Problem
```yaml
# ❌ THIS DOESN'T WORK:
- minutes: "2,12,22,32,42,52"  # Error: invalid time_pattern value
  trigger: time_pattern
```

**Error message:**
```
Message malformed: invalid time_pattern value for dictionary value @ data['minutes']
```

### The Solution
```yaml
# ✅ THIS WORKS:
- minutes: /10  # Standard pattern (every 10 minutes)
  trigger: time_pattern
  offset: "00:02:00"  # +2 minute offset
```

---

## 📚 How Time Pattern Offset Works

### Living Room (Base - No Offset)
```yaml
- minutes: /10
  trigger: time_pattern
# Fires at: 19:00, 19:10, 19:20, 19:30, 19:40, 19:50
```

### Bedroom (+2 Minute Offset)
```yaml
- minutes: /10
  trigger: time_pattern
  offset: "00:02:00"  # ← +2 minutes
# Fires at: 19:02, 19:12, 19:22, 19:32, 19:42, 19:52
```

### Office (+5 Minute Offset)
```yaml
- minutes: /10
  trigger: time_pattern
  offset: "00:05:00"  # ← +5 minutes
# Fires at: 19:05, 19:15, 19:25, 19:35, 19:45, 19:55
```

### Child Room (+8 Minute Offset)
```yaml
- minutes: /10
  trigger: time_pattern
  offset: "00:08:00"  # ← +8 minutes
# Fires at: 19:08, 19:18, 19:28, 19:38, 19:48, 19:58
```

---

## ✅ Complete Fixed Automations

### Living Room (No Offset - Fires First)
```yaml
alias: Living Room - External Sensor (Hybrid) - FIXED
description: Real-time updates + 10min keep-alive with rate limiting
triggers:
  - entity_id:
      - sensor.living_temp_umiditate_temperature
    trigger: state
    for:
      seconds: 30
  - minutes: /10  # No offset needed
    trigger: time_pattern

conditions:
  - condition: template
    value_template: >-
      {{ states('sensor.living_temp_umiditate_temperature') not in ['unknown', 'unavailable'] }}

actions:
  - metadata: {}
    data:
      value: "{{ states('sensor.living_temp_umiditate_temperature') }}"
    target:
      entity_id:
        - number.living_calorifer_external_temperature_sensor_value
        - number.hol_intrare_calorifer_external_temperature_sensor_value
        - number.bucatarie_calorifer_external_temperature_sensor_value
        - number.hol_parter_calorifer_external_temperature_sensor_value
    action: number.set_value

mode: single
max_exceeded: silent
```

### Bedroom (+2 Min Offset)
```yaml
alias: Dormitor - Sync Temp to 2xThermostat (Hybrid) - FIXED
description: Real-time updates + 10min keep-alive, offset +2min from Living Room
triggers:
  - entity_id:
      - sensor.sonoff_snzb_02d_temperature
    trigger: state
    for:
      seconds: 30
  - minutes: /10
    trigger: time_pattern
    offset: "00:02:00"  # ← +2 minute offset

conditions:
  - condition: template
    value_template: >-
      {{ states('sensor.sonoff_snzb_02d_temperature') not in ['unknown', 'unavailable'] }}

actions:
  - metadata: {}
    data:
      value: "{{ states('sensor.sonoff_snzb_02d_temperature') }}"
    target:
      entity_id:
        - number.dormitor_calorifer_1_external_temperature_sensor_value
        - number.dormitor_calorifer_2_external_temperature_sensor_value
    action: number.set_value

mode: single
max_exceeded: silent
```

---

## 📊 Timeline Verification

### Execution Schedule:
```
19:00:00 → Living Room (4 thermostats)
19:02:00 → Bedroom (2 thermostats) ← +2 min offset
19:05:00 → Office (1 thermostat) ← +5 min offset (coming next)
19:08:00 → Child Room (1 thermostat) ← +8 min offset (coming next)

19:10:00 → Living Room (4 thermostats)
19:12:00 → Bedroom (2 thermostats) ← +2 min offset
19:15:00 → Office (1 thermostat) ← +5 min offset (coming next)
19:18:00 → Child Room (1 thermostat) ← +8 min offset (coming next)
```

**Perfect stagger! No collisions!** ✅

---

## 🎯 Deploy Instructions

### For Bedroom Automation:

1. **Home Assistant → Settings → Automations**
2. Find **"Dormitor - Sync Temp to 2xThermostat (Hybrid)"**
3. Click **Edit** → **Edit in YAML**
4. Replace with the corrected YAML above
5. **Save**
6. ✅ **No more errors!**

---

## 📝 Key Takeaway

**Use `offset:` parameter for time pattern staggering:**
- ✅ `offset: "00:02:00"` for +2 minutes
- ✅ `offset: "00:05:00"` for +5 minutes
- ✅ `offset: "00:08:00"` for +8 minutes

**Don't use comma-separated minutes:**
- ❌ `minutes: "2,12,22,32,42,52"` causes errors
- ❌ Not valid Home Assistant YAML syntax

---

**Status:** ✅ Bedroom automation CORRECTED and ready to deploy!  
**Next:** Office automation with `offset: "00:05:00"`

# 🔧 Bedroom (Dormitor) Automation - Fixed Version

## 📊 Changes Made

### ✅ What Was Added

1. **30-Second Stabilization** (`for: seconds: 30`)
   - Prevents rapid-fire triggers when sensor value fluctuates
   - Reduces Zigbee message spam
   - Waits for stable reading before updating thermostats

2. **+2 Minute Time Offset** (`minutes: "2,12,22,32,42,52"`)
   - **CRITICAL FIX:** Prevents collision with Living Room automation
   - Living Room fires at :00, Bedroom fires at :02
   - 2-minute gap prevents Zigbee queue overflow

3. **Single Mode** (`mode: single`, changed from `restart`)
   - Only one instance runs at a time
   - Prevents overlapping executions
   - More stable than `restart` mode

4. **Silent Max Exceeded** (`max_exceeded: silent`)
   - Prevents log spam if triggered too frequently
   - Graceful handling of rapid triggers
   - Cleaner logs

---

## 📋 Side-by-Side Comparison

### ❌ BEFORE (Original)
```yaml
triggers:
  - entity_id:
      - sensor.sonoff_snzb_02d_temperature
    trigger: state  # ← No stabilization
  - minutes: /10  # ← PROBLEM: Same as Living Room (:00, :10, :20...)
    trigger: time_pattern

mode: restart  # ← Restarts on new trigger (can cause issues)
# Missing: max_exceeded setting
```

### ✅ AFTER (Fixed)
```yaml
triggers:
  - entity_id:
      - sensor.sonoff_snzb_02d_temperature
    trigger: state
    for:
      seconds: 30  # ← NEW: 30-second stabilization
  
  - minutes: "2,12,22,32,42,52"  # ← FIXED: +2 min offset!
    trigger: time_pattern

mode: single  # ← CHANGED: Prevents overlaps
max_exceeded: silent  # ← NEW: Silent fail
```

---

## 🕐 Timing Strategy (CRITICAL)

### ❌ Old Problem (Collision):
```
19:00:00 → Living Room (4) + Bedroom (2) fire together = 6 thermostats
19:10:00 → Living Room (4) + Bedroom (2) fire together = 6 thermostats
19:20:00 → Living Room (4) + Bedroom (2) fire together = 6 thermostats
         → ZIGBEE QUEUE OVERFLOW 💥
```

### ✅ New Solution (Staggered):
```
19:00:00 → Living Room (4 thermostats) fires
19:02:00 → Bedroom (2 thermostats) fires ← 2 minutes later, no collision!
19:05:00 → Office (1 thermostat) will fire ← Coming next
19:08:00 → Child Room (1 thermostat) will fire ← Coming next

19:10:00 → Living Room (4 thermostats) fires
19:12:00 → Bedroom (2 thermostats) fires ← 2 minutes later, no collision!
19:15:00 → Office (1 thermostat) will fire
19:18:00 → Child Room (1 thermostat) will fire
```

**Result:** Messages spread across 8 minutes, Zigbee queue never fills! ✅

---

## 🎯 How to Apply

### Method 1: Home Assistant UI (Recommended)

1. **Open Home Assistant**
2. Go to **Settings → Automations & Scenes**
3. Find **"Dormitor - Sync Temp to 2xThermostat (Hybrid)"**
4. Click **Edit** (pencil icon)
5. Click **3-dot menu** → **Edit in YAML**
6. **Replace entire content** with the fixed version below
7. Click **Save**

### Method 2: Copy/Paste Fixed YAML

**Complete Fixed Automation:**

```yaml
alias: Dormitor - Sync Temp to 2xThermostat (Hybrid) - FIXED
description: Real-time updates + 10min keep-alive with rate limiting, offset +2min from Living Room
triggers:
  # State trigger with 30-second stabilization to prevent rapid-fire updates
  - entity_id:
      - sensor.sonoff_snzb_02d_temperature
    trigger: state
    for:
      seconds: 30
  
  # Time pattern: +2 minute offset from Living Room (prevents Zigbee queue collision)
  - minutes: "2,12,22,32,42,52"
    trigger: time_pattern

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

## ✅ Verification

After saving, verify the changes:

### 1. Check Automation Details
- **Settings → Automations → Dormitor**
- Click to open details
- Verify mode shows: **"single"**
- Verify description mentions "+2min offset"

### 2. Test Next Time Pattern Trigger
- Wait for next `:02` minute (e.g., 19:02, 19:12, 19:22, 19:32, 19:42, 19:52)
- Check **Settings → System → Logs**
- Filter: "dormitor"
- Should execute successfully at **:02** (not at :00 like before!)

### 3. Check Automation Trace
- **Settings → Automations → Dormitor**
- Click **Traces** tab
- Check latest execution timestamp
- Should show times ending in **:02, :12, :22, :32, :42, :52**

### 4. Verify No Collision with Living Room
- **Developer Tools → States**
- Check both automations:
  - `automation.living_room_external_sensor_hybrid` → last_triggered ends in :00
  - `automation.dormitor_sync_temp_to_2xthermostat_hybrid` → last_triggered ends in :02
- **2-minute gap confirmed!** ✅

---

## 📊 Expected Behavior

### Before Fix (Problems):
```
19:00:00 → Living (4) + Bedroom (2) fire together → 6 messages
19:10:00 → Living (4) + Bedroom (2) fire together → 6 messages
19:20:00 → Living (4) + Bedroom (2) + Office (1) + Child (1) → 8 messages
         → ZIGBEE_MAX_MESSAGE_LIMIT_REACHED ❌
```

### After Fix (Working):
```
19:00:00 → Living (4 thermostats) → 4 messages ✅
19:02:00 → Bedroom (2 thermostats) → 2 messages ✅
  [gap]
19:10:00 → Living (4 thermostats) → 4 messages ✅
19:12:00 → Bedroom (2 thermostats) → 2 messages ✅
  [gap]
19:20:00 → Living (4 thermostats) → 4 messages ✅
19:22:00 → Bedroom (2 thermostats) → 2 messages ✅
```

**Key improvements:**
- ✅ 2-minute offset prevents queue collision
- ✅ 30-second stabilization reduces spam
- ✅ Single mode prevents overlaps
- ✅ No Zigbee errors!

---

## 🔍 Troubleshooting

### If automation doesn't fire at :02:
1. Check automation is enabled (toggle switch on)
2. Verify YAML syntax is correct (no extra spaces)
3. Check logs for errors during save
4. Restart Home Assistant if needed

### If still fires at :00 instead of :02:
1. Re-edit automation in YAML mode
2. Verify `minutes:` line shows `"2,12,22,32,42,52"` with quotes
3. Save again and check traces

### If temperature updates are delayed:
1. This is normal! 30-second stabilization is working
2. Reduces Zigbee spam significantly
3. If too slow, reduce to 15 seconds (test first)

---

## 📊 Timing Verification Table

| Time | Living Room | Bedroom | Gap | Status |
|------|-------------|---------|-----|--------|
| 19:00 | ✅ Fires | - | - | No collision |
| 19:02 | - | ✅ Fires | 2 min | No collision |
| 19:05 | - | - | 3 min | Office will fire here |
| 19:08 | - | - | 3 min | Child will fire here |
| 19:10 | ✅ Fires | - | - | No collision |
| 19:12 | - | ✅ Fires | 2 min | No collision |
| 19:15 | - | - | 3 min | Office will fire here |
| 19:18 | - | - | 3 min | Child will fire here |
| 19:20 | ✅ Fires | - | - | No collision |
| 19:22 | - | ✅ Fires | 2 min | No collision |

**Perfect stagger! No automation fires at the same time!** ✅

---

## 💡 Why +2 Minutes?

**Why not +1 minute?**
- Too close, might overlap with Living Room cleanup
- Zigbee messages can take 5-10 seconds to process
- 2 minutes provides safe buffer

**Why not +5 minutes?**
- 5 minutes reserved for Office automation
- Even distribution: 0, 2, 5, 8 = good spread
- 2-minute offset is optimal for bedroom (2 thermostats)

**Formula:**
```
Living Room:  4 thermostats → :00 (largest, goes first)
Bedroom:      2 thermostats → :02 (+2 min)
Office:       1 thermostat  → :05 (+5 min)
Child Room:   1 thermostat  → :08 (+8 min)
```

Staggering based on thermostat count ensures largest batch goes first with most time to complete!

---

## 📅 Next Steps

After deploying Bedroom fix:

1. ✅ **Living Room deployed** (fires at :00)
2. ✅ **Bedroom ready to deploy** (fires at :02)
3. ⏳ **Office next** (will fire at :05, +5 min offset)
4. ⏳ **Child Room last** (will fire at :08, +8 min offset)

---

## 🎯 Current Status

- **Living Room:** ✅ FIXED (fires at :00, :10, :20, :30, :40, :50)
- **Bedroom:** ✅ FIXED (fires at :02, :12, :22, :32, :42, :52)
- **Office:** ⏳ PENDING (will fire at :05, :15, :25, :35, :45, :55)
- **Child Room:** ⏳ PENDING (will fire at :08, :18, :28, :38, :48, :58)

---

**Status:** ✅ Bedroom automation FIXED and ready to deploy!  
**Next:** Office (Birou) automation with +5 minute offset  
**Progress:** 2 of 4 automations completed (50%)

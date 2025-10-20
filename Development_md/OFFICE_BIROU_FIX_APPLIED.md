# 🔧 Office (Birou) Automation - Fixed Version

## 📊 Changes Made

### ✅ What Was Added

1. **30-Second Stabilization** (`for: seconds: 30`)
   - Prevents rapid-fire triggers when sensor value fluctuates
   - Reduces Zigbee message spam
   - Waits for stable reading before updating thermostat

2. **+5 Minute Template Trigger**
   ```yaml
   {{ now().minute % 10 == 5 and now().second == 0 }}
   ```
   - **CRITICAL FIX:** Prevents collision with Living Room and Bedroom
   - Fires at :05, :15, :25, :35, :45, :55
   - 5-minute gap after Living Room, 3-minute gap after Bedroom

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
      - sensor.birou_temperature
    trigger: state  # ← No stabilization
  - minutes: /10  # ← PROBLEM: Same as Living Room and Bedroom
    trigger: time_pattern

mode: restart  # ← Can cause issues
# Missing: max_exceeded setting
```

### ✅ AFTER (Fixed)
```yaml
triggers:
  - entity_id:
      - sensor.birou_temperature
    trigger: state
    for:
      seconds: 30  # ← NEW: 30-second stabilization
  
  # Template trigger for :05, :15, :25, :35, :45, :55
  - trigger: template
    value_template: >-
      {{ now().minute % 10 == 5 and now().second == 0 }}

mode: single  # ← CHANGED: Prevents overlaps
max_exceeded: silent  # ← NEW: Silent fail
```

---

## 🕐 Timing Strategy (CRITICAL)

### ❌ Old Problem (All Collision):
```
19:00:00 → Living (4) + Bedroom (2) + Office (1) fire together = 7 thermostats
19:10:00 → Living (4) + Bedroom (2) + Office (1) fire together = 7 thermostats
19:20:00 → All 4 automations fire together = 8 thermostats
         → ZIGBEE QUEUE OVERFLOW 💥
```

### ✅ New Solution (Perfect Stagger):
```
19:00:00 → Living Room (4 thermostats) fires
19:02:00 → Bedroom (2 thermostats) fires ← +2 min
19:05:00 → Office (1 thermostat) fires ← +5 min, no collision!
19:08:00 → Child Room (1 thermostat) will fire ← +8 min (coming next)

19:10:00 → Living Room (4 thermostats) fires
19:12:00 → Bedroom (2 thermostats) fires
19:15:00 → Office (1 thermostat) fires ← +5 min, no collision!
19:18:00 → Child Room (1 thermostat) will fire

19:20:00 → Living Room (4 thermostats) fires
19:22:00 → Bedroom (2 thermostats) fires
19:25:00 → Office (1 thermostat) fires ← +5 min, no collision!
19:28:00 → Child Room (1 thermostat) will fire
```

**Result:** Messages spread across 8 minutes, Zigbee queue never fills! ✅

---

## 🎯 How to Apply

### Copy/Paste Fixed YAML

**Complete Fixed Automation:**

```yaml
alias: Birou - Sync External Temperature (Hybrid) - FIXED
description: Real-time updates + 10min keep-alive with rate limiting, offset +5min from Living Room
triggers:
  # State trigger with 30-second stabilization to prevent rapid-fire updates
  - entity_id:
      - sensor.birou_temperature
    trigger: state
    for:
      seconds: 30
  
  # Template trigger for :05, :15, :25, :35, :45, :55
  - trigger: template
    value_template: >-
      {{ now().minute % 10 == 5 and now().second == 0 }}

conditions:
  - condition: template
    value_template: "{{ states('sensor.birou_temperature') not in ['unknown', 'unavailable'] }}"

actions:
  - metadata: {}
    data:
      value: "{{ states('sensor.birou_temperature') }}"
    target:
      entity_id:
        - number.birou_temperature
    action: number.set_value

mode: single
max_exceeded: silent
```

---

## ✅ Verification

After saving, verify the changes:

### 1. Check Automation Details
- **Settings → Automations → Birou**
- Click to open details
- Verify mode shows: **"single"**
- Verify description mentions "+5min offset"

### 2. Test Next Time Pattern Trigger
- Wait for next `:05` minute (e.g., 19:05, 19:15, 19:25, 19:35, 19:45, 19:55)
- Check **Settings → System → Logs**
- Filter: "birou"
- Should execute successfully at **:05** (not at :00!)

### 3. Check Automation Trace
- **Settings → Automations → Birou**
- Click **Traces** tab
- Check latest execution timestamp
- Should show times ending in **:05, :15, :25, :35, :45, :55**

### 4. Verify No Collision with Other Automations
- **Developer Tools → States**
- Check all automations:
  - `automation.living_room...` → last_triggered ends in :00
  - `automation.dormitor...` → last_triggered ends in :02
  - `automation.birou...` → last_triggered ends in :05
- **Perfect spacing confirmed!** ✅

---

## 📊 Expected Behavior

### Before Fix (Problems):
```
19:00:00 → Living (4) + Bedroom (2) + Office (1) → 7 messages
19:10:00 → Living (4) + Bedroom (2) + Office (1) → 7 messages
19:20:00 → All 4 automations fire → 8 messages
         → ZIGBEE_MAX_MESSAGE_LIMIT_REACHED ❌
```

### After Fix (Working):
```
19:00:00 → Living (4 thermostats) → 4 messages ✅
19:02:00 → Bedroom (2 thermostats) → 2 messages ✅
19:05:00 → Office (1 thermostat) → 1 message ✅
  [gap]
19:10:00 → Living (4 thermostats) → 4 messages ✅
19:12:00 → Bedroom (2 thermostats) → 2 messages ✅
19:15:00 → Office (1 thermostat) → 1 message ✅
```

**Key improvements:**
- ✅ 5-minute offset prevents queue collision
- ✅ 30-second stabilization reduces spam
- ✅ Single mode prevents overlaps
- ✅ Only 1 thermostat, minimal Zigbee load
- ✅ No errors!

---

## 🧮 Template Explanation

### Formula: `now().minute % 10 == 5`

**What it does:**
- `now().minute` = Current minute (0-59)
- `% 10` = Modulo 10 (remainder after dividing by 10)
- `== 5` = Check if remainder equals 5
- `and now().second == 0` = Only trigger at second :00

**Examples:**
```
Time 19:05:00 → minute=5, 5%10=5, 5==5 ✅ TRIGGERS
Time 19:15:00 → minute=15, 15%10=5, 5==5 ✅ TRIGGERS
Time 19:25:00 → minute=25, 25%10=5, 5==5 ✅ TRIGGERS
Time 19:02:00 → minute=2, 2%10=2, 2≠5 ❌ Doesn't trigger
```

---

## 📊 Complete Timing Table

| Time | Living | Bedroom | Office | Child | Gap | Status |
|------|--------|---------|--------|-------|-----|--------|
| :00 | ✅ (4) | - | - | - | - | No collision |
| :02 | - | ✅ (2) | - | - | 2 min | No collision |
| :05 | - | - | ✅ (1) | - | 3 min | No collision |
| :08 | - | - | - | ✅ (1) | 3 min | No collision |
| :10 | ✅ (4) | - | - | - | 2 min | No collision |
| :12 | - | ✅ (2) | - | - | 2 min | No collision |
| :15 | - | - | ✅ (1) | - | 3 min | No collision |
| :18 | - | - | - | ✅ (1) | 3 min | No collision |
| :20 | ✅ (4) | - | - | - | 2 min | No collision |
| :22 | - | ✅ (2) | - | - | 2 min | No collision |
| :25 | - | - | ✅ (1) | - | 3 min | No collision |
| :28 | - | - | - | ✅ (1) | 3 min | No collision |

**Numbers in () = thermostat count**

**Perfect distribution! Maximum 4 thermostats at any time, minimum 2-minute gaps!** ✅

---

## 💡 Why +5 Minutes?

**Strategic placement:**
- Living Room (4 thermostats) needs most time → Goes first at :00
- Bedroom (2 thermostats) → :02 (+2 min, quick follow-up)
- **Office (1 thermostat) → :05 (+5 min, middle of interval)**
- Child Room (1 thermostat) → :08 (+8 min, end of interval)

**Benefits:**
- Even spacing: 2 min, 3 min, 3 min, 2 min gaps
- Office in middle allows Living Room to fully complete
- Small load (1 thermostat) minimizes impact
- 3-minute gap from Bedroom ensures no overlap

---

## 🔍 Troubleshooting

### If automation doesn't fire at :05:
1. Check automation is enabled (toggle switch on)
2. Verify template syntax is exact (no typos)
3. Check logs for template evaluation errors
4. Restart Home Assistant if needed

### If still fires at :00 instead of :05:
1. Re-edit automation in YAML mode
2. Verify template trigger is present
3. Check `now().minute % 10 == 5` (not 0 or 2)
4. Save again and check traces

### If temperature updates are delayed:
1. This is normal! 30-second stabilization is working
2. Reduces Zigbee spam significantly
3. Office only has 1 thermostat, very lightweight

---

## 📅 Next Steps

After deploying Office fix:

1. ✅ **Living Room deployed** (fires at :00)
2. ✅ **Bedroom deployed** (fires at :02)
3. ✅ **Office ready to deploy** (fires at :05)
4. ⏳ **Child Room next** (will fire at :08, +8 min offset)

**Progress: 75% Complete (3 of 4 automations!)** 🎉

---

## 🎯 Current Status

- ✅ **Living Room** - FIXED (fires at :00, :10, :20, :30, :40, :50)
- ✅ **Bedroom** - FIXED (fires at :02, :12, :22, :32, :42, :52)
- ✅ **Office** - FIXED (fires at :05, :15, :25, :35, :45, :55)
- ⏳ **Child Room** - PENDING (will fire at :08, :18, :28, :38, :48, :58)

---

**Status:** ✅ Office automation FIXED and ready to deploy!  
**Next:** Child Room (Cam Luca) automation with `minute % 10 == 8`  
**Almost done!** 🚀

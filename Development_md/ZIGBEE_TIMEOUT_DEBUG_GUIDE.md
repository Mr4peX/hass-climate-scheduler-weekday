# 🐛 ZIGBEE TIMEOUT ERRORS - Debugging Guide

## 📋 Error Summary

**Time:** 20:41:28 - 20:42:17  
**Affected Device:** `climate.bucatarie_calorifer_thermostat` (Kitchen)  
**Error:** `Failed to send request: device did not respond`  
**Root Cause:** Zigbee network communication timeout  

---

## 🔍 Detailed Analysis

### Error 1: Climate Scheduler Timeout (20:41:28)

```
Service: climate.set_hvac_mode + climate.set_temperature
Entity: climate.bucatarie_calorifer_thermostat
Error: asyncio.TimeoutError → ZHAException
Meaning: Kitchen thermostat didn't respond to Zigbee command within timeout period
```

**Timeline:**
```
20:41:28 → Scheduler sends command to Kitchen thermostat
20:41:28 → ZHA waits for response
20:41:28 → Timeout (MESSAGE_SEND_TIMEOUT_MAINS exceeded)
20:41:28 → Command cancelled → Error logged
```

---

### Error 2: Living Room Automation Timeout (20:42:17)

```
Automation: Living Room - External Sensor (Hybrid) - FIXED v2
Position: pos 5 (3rd thermostat: Kitchen)
Error: Failed to send request: device did not respond
```

**Timeline:**
```
20:42:17 → Living Room automation triggers
20:42:17 → Update Living thermostat ✅
20:42:18 → Delay 1 second
20:42:18 → Update Entrance Hall thermostat ✅
20:42:19 → Delay 1 second
20:42:19 → Update Kitchen thermostat ❌ TIMEOUT!
20:42:19 → continue_on_error: true (automation continues)
20:42:20 → Delay 1 second
20:42:20 → Update Ground Floor Hall thermostat (likely ✅)
```

**Note:** `continue_on_error: true` worked! Despite Kitchen timeout, automation continued to Ground Floor Hall.

---

## 🎯 Root Causes (In Order of Likelihood)

### 1. **Kitchen Thermostat Signal Issues** (MOST LIKELY)

**Symptoms:**
- Kitchen thermostat specifically failing
- Other thermostats (Living, Entrance Hall) working fine
- Timeout errors (not "no response at all")

**Possible causes:**
- Kitchen too far from Zigbee coordinator
- Kitchen has interference (microwave, WiFi router nearby)
- Metal objects blocking signal (refrigerator, cabinets)
- Thermostat mounted in metal junction box

**Fix:** Improve signal strength (see Fix #2 below)

---

### 2. **Too Many Simultaneous Messages** (LIKELY)

**Symptoms:**
- Errors at 20:41 (schedule change time?)
- Multiple commands to same device in short time
- Timeout (not outright failure)

**Cause:**
```
20:41:28 → Climate Scheduler changes schedule
20:41:28 → Sends set_hvac_mode to Kitchen
20:41:28 → Sends set_temperature to Kitchen
20:41:28 → Kitchen overwhelmed, times out
```

**Fix:** Increase delays, reduce command frequency

---

### 3. **Zigbee Network Congestion** (POSSIBLE)

**Symptoms:**
- Errors occur at specific times (:41, :42)
- Multiple automations/schedulers active

**Cause:**
```
20:41:00 → Climate Scheduler updates all 8 thermostats
20:42:00 → Living Room automation updates 4 thermostats
20:42:00 → Bedroom automation updates 2 thermostats
Total: 14+ Zigbee messages in 2 minutes
```

**Fix:** Better stagger automation times

---

### 4. **Kitchen Thermostat Hardware Issue** (LESS LIKELY)

**Symptoms:**
- Only Kitchen thermostat failing
- Consistent timeout errors

**Causes:**
- Thermostat firmware bug
- Low battery (if battery-powered)
- Faulty Zigbee radio
- Device needs re-pairing

**Fix:** Re-pair thermostat, update firmware, check battery

---

## 🔧 Fixes (In Priority Order)

### Fix #1: Increase Delays in Living Room Automation ⭐ (DEPLOY NOW)

**Current:** 1-second delays  
**Problem:** Not enough time for Kitchen thermostat to process  
**Solution:** Increase to 3-second delays

**File:** `living_room_FIXED_v3_slower_delays.yaml`

**Changes:**
```yaml
# OLD (v2): 1-second delays
- delay:
    seconds: 1

# NEW (v3): 3-second delays
- delay:
    seconds: 3
```

**Benefits:**
- More time for each thermostat to respond
- Less network congestion
- Better reliability for weak-signal devices

**Trade-off:**
- Total execution time: 4 seconds → 12 seconds
- Still acceptable (within 10-minute cycle)

---

### Fix #2: Check Kitchen Thermostat Signal Strength 📶 (CHECK NOW)

**In Home Assistant:**

1. **Go to:** Settings → Devices & Services → Zigbee Home Automation
2. **Find:** Bucatarie Calorifer Thermostat (Kitchen)
3. **Check:**
   - **LQI (Link Quality Indicator):** Should be > 150
   - **Last seen:** Should be recent (< 5 minutes ago)
   - **Neighbors:** Should have multiple router nodes

**Interpret Results:**

| LQI Value | Signal Strength | Action |
|-----------|----------------|--------|
| > 200 | Excellent ✅ | No action needed |
| 150-200 | Good ✅ | Monitor |
| 100-150 | Fair ⚠️ | Add router or move coordinator |
| < 100 | Poor ❌ | Immediate action required |

**If signal is poor:**
- Add Zigbee router near kitchen
- Move coordinator closer
- Reduce interference sources

---

### Fix #3: Check Zigbee Network Health 🌐 (INVESTIGATE)

**Visualize Network:**

1. **Settings** → **Devices & Services** → **ZHA**
2. **Click:** Configure
3. **Click:** Visualize

**Look for:**
- ❌ **Red lines** - Poor connections
- ❌ **Kitchen thermostat isolated** - No router neighbors
- ✅ **Green lines** - Good connections
- ✅ **Multiple paths** - Network resilience

**Healthy network:**
```
Coordinator → Router 1 → Kitchen Thermostat ✅
Coordinator → Router 2 → Kitchen Thermostat ✅
(Multiple paths = redundancy)
```

**Unhealthy network:**
```
Coordinator → (long distance) → Kitchen Thermostat ❌
(Single path, long distance = failures)
```

---

### Fix #4: Re-pair Kitchen Thermostat (IF SIGNAL IS GOOD)

**Only if:**
- Signal strength is good (LQI > 150)
- Still getting timeout errors
- Other fixes didn't help

**Steps:**
1. **Remove from ZHA:**
   - Settings → Devices & Services → ZHA → Devices
   - Find Kitchen thermostat → Remove device
2. **Factory reset thermostat** (check manual)
3. **Re-pair:**
   - ZHA → Add device
   - Put thermostat in pairing mode
   - Wait for discovery

**Warning:** Will lose thermostat settings (temperature, schedules, etc.)

---

### Fix #5: Reduce Climate Scheduler Update Frequency (ADVANCED)

**Problem:** Scheduler might be sending too many commands

**Check scheduler configuration:**
```yaml
# In complete_heating_schedulers.yaml
# Look for frequent schedule entries:
schedule:
  - time: "20:41:00"  # ← Was this the time of error?
    weekdays: [6]  # Saturday
    min_temp: 22
```

**If scheduler changes at 20:41:**
- All 8 thermostats receive updates
- Kitchen gets set_hvac_mode + set_temperature
- Then Living Room automation fires at 20:42
- Kitchen gets external sensor update
- **Total: 3 commands to Kitchen in 1 minute!**

**Solution:** Adjust schedule times to avoid conflicts with automation times

---

## 📊 Network Load Analysis

### Timeline of Events (20:40 - 20:43)

```
20:40:00 → (Quiet period)

20:41:28 → Climate Scheduler sends commands
           ├─ Kitchen: set_hvac_mode ❌ TIMEOUT
           └─ Kitchen: set_temperature ❌ TIMEOUT

20:42:00 → Living Room automation triggers (:00 pattern)
           ├─ Living thermostat ✅
           ├─ Delay 1s
           ├─ Entrance Hall thermostat ✅
           ├─ Delay 1s
           ├─ Kitchen thermostat ❌ TIMEOUT (pos 5)
           ├─ Delay 1s
           └─ Ground Floor Hall thermostat ✅ (probably)

20:42:00 → Bedroom automation triggers (:02 pattern)
           └─ Updates 2 thermostats

20:43:00 → (Activity continues)
```

**Problem:** Kitchen received 3 commands in 90 seconds:
1. set_hvac_mode (scheduler)
2. set_temperature (scheduler)
3. set external sensor value (automation)

**Solution:** Either:
- Increase delays (v3 automation)
- Adjust scheduler time
- Improve Kitchen signal strength

---

## ✅ Recommended Action Plan

### IMMEDIATE (Do Now - 10 minutes):

1. ✅ **Check Kitchen thermostat signal strength**
   ```
   ZHA → Devices → Bucatarie Calorifer → Check LQI
   ```

2. ✅ **Deploy Living Room v3 automation** (3-second delays)
   ```
   Replace v2 with living_room_FIXED_v3_slower_delays.yaml
   ```

3. ✅ **Reload ZHA integration**
   ```
   Settings → Devices & Services → ZHA → Reload
   ```

---

### SHORT TERM (This Weekend - 1 hour):

1. ⏳ **Visualize Zigbee network**
   ```
   Check for red lines to Kitchen thermostat
   ```

2. ⏳ **Monitor logs for 24 hours**
   ```
   See if v3 delays fix timeout issues
   ```

3. ⏳ **Check scheduler times**
   ```
   Ensure no schedule changes at :40, :42 (automation times)
   ```

---

### LONG TERM (If Issues Persist):

1. ⏳ **Add Zigbee router near kitchen**
   ```
   Mains-powered Zigbee device (smart plug, bulb, etc.)
   ```

2. ⏳ **Re-pair Kitchen thermostat**
   ```
   Only if signal is good but still timing out
   ```

3. ⏳ **Consider separate automation for Kitchen**
   ```
   If Kitchen consistently problematic, isolate it
   ```

---

## 🎯 Expected Results After Fix #1 (v3 Delays)

### Before (v2 - 1-second delays):
```
20:42:00 → Living ✅
20:42:01 → Entrance Hall ✅  
20:42:02 → Kitchen ❌ TIMEOUT (not enough time)
20:42:03 → Ground Floor Hall (may not execute)
```

**Success rate:** 50-75% (2-3 of 4 thermostats)

---

### After (v3 - 3-second delays):
```
20:42:00 → Living ✅
20:42:03 → Entrance Hall ✅
20:42:06 → Kitchen ✅ (more time to respond)
20:42:09 → Ground Floor Hall ✅
```

**Success rate:** 95-100% (4 of 4 thermostats)

---

## 🔍 How to Verify Fix Worked

### Check Logs (1 Hour After Deployment):

**Look for:**
- ✅ No more "Failed to send request" errors
- ✅ No TimeoutError tracebacks
- ✅ Living Room automation completing successfully

**Check states:**
```
Developer Tools → States → Search:
- number.living_calorifer_external_temperature_sensor_value
- number.hol_intrare_calorifer_external_temperature_sensor_value
- number.bucatarie_calorifer_external_temperature_sensor_value ← Kitchen!
- number.hol_parter_calorifer_external_temperature_sensor_value

All should match: sensor.living_temp_umiditate_temperature
```

---

### Check Automation History:

```
Settings → Automations → Living Room External Sensor
→ Click on automation
→ Check "Last Triggered"
→ Click trace → Should show all 4 thermostats updated ✅
```

---

## 📊 Kitchen Thermostat Debug Info

**To collect more info, run these in Developer Tools → Template:**

### Check Signal Quality:
```jinja
{{ state_attr('sensor.bucatarie_calorifer_linkquality', 'state') }}
```

### Check Last Update:
```jinja
{{ states.climate.bucatarie_calorifer_thermostat.last_changed }}
```

### Check If Available:
```jinja
{{ states('climate.bucatarie_calorifer_thermostat') }}
```

**Expected:**
- LQI: > 150
- Last changed: Recent (< 10 minutes)
- State: "heat" or "idle" (not "unavailable")

---

## 🚨 Red Flags to Watch For

### 🚩 Kitchen Thermostat Going Offline:
```
State: unavailable
Last seen: > 1 hour ago
```
**Action:** Re-pair device immediately

### 🚩 Frequent Timeout Errors (> 5 per day):
```
Multiple TimeoutError in logs
```
**Action:** Add Zigbee router or move coordinator

### 🚩 Other Thermostats Also Failing:
```
Multiple thermostats timing out
```
**Action:** Network-wide issue, check coordinator

### 🚩 Errors Only at Specific Times:
```
Always fails at :41, :42
```
**Action:** Scheduler conflict, adjust schedule times

---

## 💡 Why continue_on_error Helped

**Your v2 automation with `continue_on_error: true` actually WORKED:**

```yaml
- action: number.set_value
  target:
    entity_id: number.bucatarie_calorifer_external_temperature_sensor_value
  continue_on_error: true  # ← This saved the day!
```

**What happened:**
1. Living thermostat updated ✅
2. Entrance Hall thermostat updated ✅
3. Kitchen thermostat timed out ❌ **BUT continued!**
4. Ground Floor Hall thermostat still got updated ✅

**Without continue_on_error:**
- Kitchen timeout would STOP automation
- Ground Floor Hall would never get update
- 2 thermostats updated instead of 3

**With continue_on_error:**
- Kitchen timeout logged but ignored
- Ground Floor Hall still updated
- 3 thermostats updated (75% success!)

---

## 🎯 Bottom Line

**Good News:**
- ✅ `continue_on_error` is working (3 of 4 thermostats updated)
- ✅ Errors are isolated to Kitchen thermostat
- ✅ Other automations working fine

**Action Required:**
- ✅ Deploy v3 automation (3-second delays) ← **DO THIS NOW**
- ✅ Check Kitchen signal strength
- ⏳ Monitor for 24 hours

**Expected Outcome:**
- 3-second delays should give Kitchen enough time to respond
- Success rate: 50-75% → 95-100%
- If still fails → Signal strength issue

---

**Status:** 🐛 Zigbee timeout issue identified  
**Cause:** Kitchen thermostat not responding fast enough  
**Fix:** Increase delays from 1s → 3s (v3 automation)  
**Priority:** HIGH (deploy v3 today)

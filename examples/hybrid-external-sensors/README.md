# 🔄 Hybrid External Sensor Automations

## Problem: 2-Hour Thermostat Timeout

### What Happens Without This Solution:

**The Issue:**
- TuYa Zigbee thermostats have a **2-hour timeout** for external sensors
- If no temperature update received for 2+ hours → Falls back to internal sensor
- Internal sensor reads different temperature than room sensor
- Result: **Single thermostat activates** while others don't
- **Inefficient**: Central heating runs for just one heater

**Real-World Example:**
> Living room has 4 thermostats sharing 1 external sensor. Temperature is stable at 20.5°C for 3 hours. Original automation only triggers on state change, so no updates sent. After 2 hours, one thermostat (hol_parter) times out, uses internal sensor (reads 18°C), activates alone. Central boiler runs for single radiator = wasted energy!

---

## ✅ Solution: Hybrid Trigger System

### How It Works:

**Dual-Trigger Approach:**

1. **State Trigger** - Real-time response
   - Fires when temperature changes
   - Immediate update to all thermostats
   - Maintains comfort

2. **Time Pattern Trigger** - Keep-alive
   - Fires every 10 minutes regardless
   - Prevents 2-hour timeout
   - Maintains synchronization

**Result:** Thermostats NEVER timeout, always stay synchronized!

---

## 📊 Technical Details

### Thermostat Specifications:

**Device:** TuYa Zigbee TRV (TRVZB)  
**External Sensor Timeout:** 2 hours  
**Fallback Behavior:** Uses internal sensor  
**Problem:** Internal sensor ≠ Room sensor  

### Solution Specifications:

**Update Frequency:** Every 10 minutes  
**Safety Margin:** 12x (10 min vs 120 min timeout)  
**Real-time Response:** Immediate on temp change  
**Network Impact:** ~144 messages/day (negligible)  
**Battery Impact:** Zero (mains-powered devices)  

---

## 🏠 Example Configurations

This folder contains **4 real-world examples**:

### 1. Living Room (4 Thermostats + 1 Sensor)

**File:** `living_room_4_thermostats.yaml`

**Scenario:**
- Large room with 4 radiators
- 1 central temperature sensor
- Most critical (highest thermostat count)
- Most prone to single-heater activation

**Entities:**
- Sensor: `sensor.living_temp_umiditate_temperature`
- 4 external sensor values (number entities)

---

### 2. Bedroom (2 Thermostats + 1 Sensor)

**File:** `bedroom_2_thermostats.yaml`

**Scenario:**
- Two radiators in bedroom
- 1 Sonoff temperature/humidity sensor
- Needs perfect sync for comfortable sleep

**Entities:**
- Sensor: `sensor.sonoff_snzb_02d_temperature`
- 2 external sensor values

---

### 3. Office (1 Thermostat + 1 Sensor)

**File:** `office_1_thermostat.yaml`

**Scenario:**
- Single thermostat
- 1 dedicated room sensor
- Less critical but still uses hybrid approach

**Entities:**
- Sensor: `sensor.birou_temperature`
- 1 external sensor value

---

### 4. Child's Room (1 Thermostat + 1 Sensor)

**File:** `child_room_1_thermostat.yaml`

**Scenario:**
- Single thermostat
- 1 temperature/humidity sensor
- Keep-alive ensures reliability

**Entities:**
- Sensor: `sensor.luca_temp_umiditate_temperature`
- 1 external sensor value

---

## 🚀 Implementation Guide

### Step 1: Choose Your Room Files

Select the files that match your setup:
- Multiple thermostats per room? → Use living_room example
- Two thermostats? → Use bedroom example
- Single thermostat? → Use office or child_room example

### Step 2: Customize Entity IDs

**Open the YAML file and replace:**

```yaml
# Replace sensor entity
entity_id:
  - sensor.YOUR_ROOM_TEMPERATURE_SENSOR

# Replace external sensor number entities
entity_id:
  - number.YOUR_THERMOSTAT_EXTERNAL_SENSOR_VALUE
```

### Step 3: Deploy to Home Assistant

**Method 1: Via Studio Code Server (Recommended)**
1. Copy code from example file
2. Paste in Home Assistant → Settings → Automations → Create → Edit in YAML

**Method 2: Via automations.yaml**
1. Copy code to `/config/automations.yaml`
2. Reload automations: Developer Tools → YAML → Automations

### Step 4: Verify Working

**Check 1: Immediate (0-5 minutes)**
- Automation appears in list
- Status: Enabled ✅

**Check 2: First trigger (within 10 minutes)**
- Developer Tools → States
- Find external sensor number entity
- Check "Last Updated" timestamp
- Should be recent (within 10 min)

**Check 3: Real-time response**
- Change room temperature (open window)
- External sensor value updates immediately

**Check 4: Keep-alive (10 minutes)**
- Wait 10 minutes
- Check "Last Updated" again
- Should have updated even if temp stable

**Check 5: Long-term stability (3 hours)**
- Let run for 3+ hours
- Verify all thermostats still synchronized
- No single-heater activation

---

## 🧪 Testing Guide

### Test 1: Time Pattern Trigger (15 minutes)

1. **Deploy automation**
2. **Note current time:** e.g., 14:23
3. **Wait for next 10-minute mark:** 14:30
4. **Check external sensor "Last Updated":** Should be ~14:30
5. **Wait another 10 minutes:** 14:40
6. **Check again:** Should be ~14:40

✅ **Pass:** Time pattern trigger working

---

### Test 2: State Trigger (5 minutes)

1. **Note current external sensor value:** e.g., 20.5
2. **Note "Last Updated" timestamp**
3. **Change room temperature:**
   - Open window
   - Turn on/off heater
   - Use hairdryer near sensor
4. **Watch sensor value update**
5. **Check external sensor "Last Updated":**
   - Should update immediately (within seconds)

✅ **Pass:** State trigger working (real-time)

---

### Test 3: Condition Check (2 minutes)

1. **Go to automation**
2. **Click three dots → Test condition**
3. **Should show:** ✅ Condition passed

If sensor is offline/unknown:
4. **Disconnect sensor** (unplug or remove battery)
5. **Wait for sensor to show "unavailable"**
6. **Test condition again**
7. **Should show:** ❌ Condition failed

✅ **Pass:** Condition prevents "unknown" updates

---

### Test 4: Multi-Thermostat Sync (3+ hours)

**Only for rooms with multiple thermostats:**

1. **Developer Tools → States**
2. **Find all external sensor number entities** for the room
3. **Verify all have same value:** e.g., all show 20.5
4. **Verify all have same "Last Updated":** Within a few seconds
5. **Let system run 3+ hours**
6. **Check all external sensors again:**
   - Still same value?
   - Still recent "Last Updated"?
   - No timeouts?

✅ **Pass:** No single-heater activation

---

## 🎯 Before vs After Comparison

### Before (State-Only Automation):

```yaml
triggers:
  - trigger: state
    entity_id:
      - sensor.room_temperature
```

**Problems:**
- ❌ Only triggers when temperature changes
- ❌ Stable temp = No updates
- ❌ 2+ hours stable = Timeout
- ❌ Single heater activation
- ❌ Inefficient heating

**Update Frequency:**
- When temp changes: Immediate
- When temp stable: Never
- After 2 hours stable: Timeout!

---

### After (Hybrid Automation):

```yaml
trigger:
  - platform: state
    entity_id:
      - sensor.room_temperature
  
  - platform: time_pattern
    minutes: "/10"
```

**Benefits:**
- ✅ Real-time on temp change
- ✅ Keep-alive every 10 min
- ✅ Never timeout (12x safety)
- ✅ All thermostats synchronized
- ✅ Efficient heating

**Update Frequency:**
- When temp changes: Immediate
- When temp stable: Every 10 min
- After 2 hours: Still updating!

---

## 📊 Network Impact Analysis

### Messages Per Day:

**Time Pattern Updates:**
- 6 updates/hour × 24 hours = 144 updates/day

**State Updates (estimated):**
- Temperature changes: ~100-200/day
- Typical residential variance

**Total Messages:**
- ~300 messages/day per room
- All rooms: ~1,200 messages/day (4 rooms)

### Zigbee Network Load:

**Theoretical Capacity:**
- Zigbee: 250 kbps
- Our usage: ~0.004% (negligible)

**Impact:**
- ✅ Zero network congestion
- ✅ Zero interference with other devices
- ✅ Scales to dozens of rooms

### Battery Impact:

**All devices mains-powered:**
- Temperature sensors: Mains or USB powered
- Thermostats: Powered by heating system (24V)
- Coordinator: Mains powered

**Result:**
- ✅ Zero battery drain
- ✅ Can update as frequently as needed
- ✅ No power concerns

---

## 🐛 Troubleshooting

### Issue: Automation not triggering every 10 minutes

**Symptoms:**
- "Last Triggered" shows > 10 minutes ago
- External sensor "Last Updated" not recent

**Checks:**
1. Automation is Enabled? (toggle on)
2. Check automation Traces → Any errors?
3. Verify `minutes: "/10"` syntax correct (with quotes!)

**Solution:**
```yaml
# Correct syntax:
- platform: time_pattern
  minutes: "/10"

# Wrong syntax:
- platform: time_pattern
  minutes: /10  # Missing quotes!
```

---

### Issue: External sensor showing "unknown"

**Symptoms:**
- External sensor number entity = "unknown"
- Thermostat showing errors

**Checks:**
1. Room sensor online? (Developer Tools → States)
2. Room sensor value valid? (not unknown/unavailable)
3. Condition working? (Test condition button)

**Solution:**
- Check sensor battery (if battery-powered)
- Check Zigbee network connectivity
- Verify sensor entity ID correct
- Check automation condition:
  ```yaml
  condition:
    - condition: template
      value_template: >-
        {{ states('sensor.room_temp') not in ['unknown', 'unavailable'] }}
  ```

---

### Issue: Thermostats still timing out after 2 hours

**Symptoms:**
- One thermostat activates alone
- Others don't heat
- After 2+ hours of stable temperature

**Checks:**
1. Hybrid automation deployed and enabled?
2. Time pattern trigger firing? (Check Traces)
3. External sensors updating? (Check "Last Updated")
4. All thermostats using external sensor? (Check thermostat config)

**Solution:**
1. Verify automation exists in Automations list
2. Check "Last Triggered" timestamp (should be < 10 min)
3. Check automation Traces → Should show entries every 10 min
4. Verify external sensor number entities correct
5. Check thermostat settings → External sensor enabled

---

### Issue: Updates happen but thermostats don't respond

**Symptoms:**
- External sensor updates correctly
- "Last Updated" shows recent
- But thermostats still use internal sensor

**Checks:**
1. Thermostat external sensor feature enabled?
2. Correct external sensor entity configured?
3. Thermostat firmware up to date?

**Solution:**
- Check thermostat settings in Home Assistant
- Verify external sensor number entity exists
- Test by manually changing external sensor value
- Check thermostat logs for errors

---

### Issue: Temperature not updating in real-time

**Symptoms:**
- State trigger not firing on temp change
- Only time pattern updates happening

**Checks:**
1. State trigger present in automation?
2. Room sensor reporting state changes?
3. Sensor sensitivity too low?

**Solution:**
```yaml
# Verify state trigger exists:
trigger:
  - platform: state
    entity_id:
      - sensor.room_temperature  # Correct entity ID?
```

- Check sensor refresh rate (some sensors update slowly)
- Verify sensor working (check in States)
- Test by forcing temp change (heat near sensor)

---

## 💡 Pro Tips

### Choosing Update Interval

**Available options:**

```yaml
minutes: "/5"   # Every 5 minutes (paranoid)
minutes: "/10"  # Every 10 minutes (recommended)
minutes: "/15"  # Every 15 minutes (balanced)
minutes: "/30"  # Every 30 minutes (minimum safe)
```

**Recommendation:**
- **10 minutes** is optimal (12x safety margin)
- Provides 12 updates before timeout
- Negligible network impact
- Real-time response still works

**When to adjust:**
- 5 min: If you experience timeouts with 10 min
- 15 min: If network very congested (unlikely)
- 30 min: If absolutely necessary (risky!)

---

### Monitoring Your System

**Developer Tools → States:**
- Filter: "external_temperature"
- Watch "Last Updated" column
- Should update every ~10 minutes

**Automation Traces:**
- Settings → Automations → Your automation → Traces
- Shows every execution with timestamp
- Should see entries every 10 minutes
- Plus immediate entries when temp changes

**Logbook:**
- History → Logbook
- Filter by external sensor entity
- See all updates in chronological order
- Verify frequency correct

---

### Multi-Thermostat Rooms

**Critical for:**
- Living rooms with multiple radiators
- Large bedrooms
- Open-plan areas

**Without hybrid automation:**
- Thermostats desynchronize after 2 hours
- One activates based on internal sensor
- Others don't activate (use external sensor)
- Inefficient heating

**With hybrid automation:**
- All thermostats always synchronized
- All use external sensor reading
- All activate/deactivate together
- Efficient heating

**Pro tip:** Check all external sensor values periodically (daily/weekly) to ensure all match room sensor.

---

## 🎉 Success Criteria

You'll know it's working when:

✅ **Automation triggers every 10 minutes**
- Check Traces → See entries every 10 min
- Check "Last Triggered" → Within last 10 min

✅ **Real-time updates when temp changes**
- Open window → Immediate update
- Temperature change → Immediate update

✅ **External sensors always current**
- "Last Updated" always < 10 min
- Values match room sensor
- Never "unknown" or "unavailable"

✅ **Multi-thermostat rooms synchronized**
- All external sensors have same value
- All "Last Updated" within seconds
- No single-heater activation

✅ **System stable 24/7**
- 3+ hours → No timeouts
- 24+ hours → Still synchronized
- 1+ week → Bulletproof reliability

✅ **No timeout after stable periods**
- Temperature stable for hours
- Thermostats still synchronized
- All using external sensor

---

## 📚 Additional Resources

### Testing Guide

See: `testing_guide.md` for comprehensive testing procedures

### Deployment Guide

See: `../DEPLOYMENT_GUIDE.md` for complete installation

### Troubleshooting

See: This README troubleshooting section

### Example Files

- `living_room_4_thermostats.yaml` - Most complex example
- `bedroom_2_thermostats.yaml` - Two thermostats
- `office_1_thermostat.yaml` - Single thermostat
- `child_room_1_thermostat.yaml` - Single thermostat

---

## 🔄 Update Schedule

**Time Pattern Options:**

| Interval | Updates/Hour | Safety Margin | Recommendation |
|----------|--------------|---------------|----------------|
| 5 min | 12 | 24x | Paranoid |
| **10 min** | **6** | **12x** | **Recommended** ✅ |
| 15 min | 4 | 8x | Balanced |
| 30 min | 2 | 4x | Minimum safe |
| 60 min | 1 | 2x | Risky! |

**Network Impact:**

| Interval | Messages/Day | Impact |
|----------|--------------|--------|
| 5 min | 288 | 0.002% |
| **10 min** | **144** | **0.001%** ✅ |
| 15 min | 96 | 0.0007% |
| 30 min | 48 | 0.0003% |

**All options have negligible network impact!**

---

**Ready to prevent timeout issues?**

**Start with:** `living_room_4_thermostats.yaml` (if you have multiple thermostats)

**Or:** Choose the file that matches your room setup!

**Questions?** Check the troubleshooting section!

**Happy Synchronizing! 🔄🔥**

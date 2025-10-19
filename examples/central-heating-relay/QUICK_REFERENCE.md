# 🔥 Central Heating Relay - Quick Reference

## 📋 Three Versions Available

### 1. CURRENT (Your Original)
**File:** Your current automation  
**Status:** ✅ Functionally correct  
**Pros:** Simple, works reliably  
**Cons:** No optimization, potential for redundant commands  

**When to use:** If current version works perfectly with no issues

---

### 2. IMPROVED (Recommended)
**File:** `central_heating_relay_IMPROVED.yaml`  
**Status:** ⭐ **RECOMMENDED**  
**Changes:**
- ✅ Relay state checking (no redundant commands)
- ✅ 10-second stabilization delay
- ✅ Simplified template conditions
- ✅ `max_exceeded: silent` (clean logs)

**When to use:** Deploy this for better efficiency and reduced Zigbee traffic

---

### 3. ADVANCED (Production Grade)
**File:** `central_heating_relay_ADVANCED.yaml`  
**Status:** 🏆 Production-ready  
**Changes:** Everything from IMPROVED plus:
- ✅ 5-minute minimum ON duration (relay wear protection)
- ✅ 8-hour maximum ON duration (safety timeout)
- ✅ Optional notification on safety trigger

**When to use:** For maximum reliability and relay longevity

---

## 🎯 Quick Comparison

| Feature | CURRENT | IMPROVED | ADVANCED |
|---------|---------|----------|----------|
| Works correctly | ✅ | ✅ | ✅ |
| Redundant commands | ❌ | ✅ Prevented | ✅ Prevented |
| Stabilization delay | ❌ | ✅ 10 seconds | ✅ 10 seconds |
| Relay state check | ❌ | ✅ Yes | ✅ Yes |
| Simplified logic | ❌ | ✅ Template | ✅ Template |
| Wear protection | ❌ | ❌ | ✅ 5 min minimum |
| Safety timeout | ❌ | ❌ | ✅ 8 hour max |
| Complexity | Simple | Medium | Advanced |

---

## 📊 Performance Impact

### Scenario: Schedule Change (All 8 Thermostats Heat)

**CURRENT:**
```
19:54:00 → All 8 thermostats → heating
19:54:01 → 8 automation triggers
19:54:01 → 8 relay ON commands sent
Result: 8 Zigbee messages (7 redundant)
```

**IMPROVED:**
```
19:54:00 → All 8 thermostats → heating
19:54:01 → 8 automation triggers
19:54:01 → First delays 10s, others queued & dropped
19:54:11 → Check: Any heating? YES
19:54:11 → Check: Relay ON already? NO
19:54:11 → Send ONE relay ON command
Result: 1 Zigbee message
```

**Savings:** 87.5% fewer messages! (7 messages saved)

---

## 🚀 Deployment Steps

### For IMPROVED Version (5 minutes):

1. **Backup current automation**
   ```
   Settings → Automations → Central Heating ON → ⋮ → Download
   ```

2. **Edit automation**
   ```
   Settings → Automations → Central Heating ON → Edit
   ```

3. **Switch to YAML mode**
   ```
   ⋮ (three dots) → Edit in YAML
   ```

4. **Replace entire content**
   - Copy from `central_heating_relay_IMPROVED.yaml`
   - Paste into editor
   - **IMPORTANT:** Change relay entity_id if different!

5. **Save and test**
   ```
   Save → Test by changing thermostat temperature
   ```

---

## ✅ Testing Checklist

After deployment:

### Test 1: Single Thermostat Heating
- [ ] Set one thermostat to high temp (above current)
- [ ] Wait 10 seconds
- [ ] Verify relay turns ON
- [ ] Check logs: Should see ONE ON command

### Test 2: All Thermostats Idle
- [ ] Set all thermostats to low temp (below current)
- [ ] Wait 10 seconds
- [ ] Verify relay turns OFF
- [ ] Check logs: Should see ONE OFF command

### Test 3: Idempotency (Relay Already ON)
- [ ] Manually turn ON relay
- [ ] Trigger automation (change thermostat)
- [ ] Check logs: Should NOT see redundant ON command

### Test 4: Stabilization (Schedule Change)
- [ ] Wait for next schedule change
- [ ] All thermostats should call for heat
- [ ] Verify only ONE relay command sent
- [ ] Logs should show "max_exceeded: silent" for queued triggers

---

## 🔧 Customization Options

### Change Stabilization Delay

**Current:** 10 seconds
```yaml
- delay:
    seconds: 10  # ← Change this value
```

**Recommendations:**
- **5 seconds:** Fast response, some risk of redundant commands
- **10 seconds:** ⭐ Balanced (recommended)
- **15 seconds:** Very stable, slower response

---

### Add Freeze Protection (ADVANCED version)

```yaml
# Add before CASE 1 conditions:
- conditions:
    # Emergency: Any room below 15°C → Force relay ON
    - condition: template
      value_template: >-
        {{ [
          states('sensor.living_temp_umiditate_temperature')|float(20),
          states('sensor.sonoff_snzb_02d_temperature')|float(20),
          states('sensor.birou_temperature')|float(20),
          states('sensor.luca_temp_umiditate_temperature')|float(20)
        ] | min < 15 }}
  sequence:
    - action: switch.turn_on
      target:
        entity_id: switch.sonoff_1002540801_1
```

---

### Enable Safety Notifications (ADVANCED version)

Uncomment lines in CASE 3:
```yaml
# Send notification when safety timeout triggers
- action: notify.notify  # or notify.mobile_app_your_phone
  data:
    title: "Central Heating Safety Timeout"
    message: "Relay ON for 8+ hours. Forcing OFF."
```

---

## 🐛 Troubleshooting

### Relay Not Turning ON

**Check:**
1. Are any thermostats in "heating" state?
   ```
   Developer Tools → States → Search: hvac_action
   ```
2. Is relay entity_id correct?
   ```
   Check: switch.sonoff_1002540801_1
   ```
3. Is automation enabled?
   ```
   Settings → Automations → Check toggle
   ```

---

### Relay Not Turning OFF

**Check:**
1. Are ALL thermostats in "idle" state?
2. If using ADVANCED: Has relay been ON for at least 5 minutes?
3. Check automation last triggered time

---

### Multiple ON Commands Still Sent

**Check:**
1. Is stabilization delay present? (10 seconds)
2. Is `max_exceeded: silent` set?
3. Is relay state check in conditions?
4. Check logs for "Already ON" messages

---

## 📊 Expected Log Messages

### IMPROVED Version Logs:

**Normal operation:**
```
[automation.central_heating_on] Triggered by state of sensor.living_calorifer_hvac_action
[automation.central_heating_on] Executing step delay
[automation.central_heating_on] Executing step choose (choice 1)
[automation.central_heating_on] Executing step turn_on switch.sonoff_1002540801_1
```

**Idempotent (relay already correct state):**
```
[automation.central_heating_on] Triggered by state of sensor.living_calorifer_hvac_action
[automation.central_heating_on] Executing step delay
[automation.central_heating_on] Conditions not met, skipping
```

**Max exceeded (multiple triggers during delay):**
```
[automation.central_heating_on] Already running, skipping execution (mode: single, max_exceeded: silent)
```

---

## 🎯 Recommendations by Use Case

### If Everything Works Fine:
→ Keep CURRENT version (no need to change)

### If You Notice:
- Multiple relay commands in logs → Deploy IMPROVED
- Zigbee queue warnings → Deploy IMPROVED
- Relay clicking rapidly → Deploy ADVANCED
- Want production-grade reliability → Deploy ADVANCED

### If You Experience:
- Relay failure/wear → Deploy ADVANCED (wear protection)
- Runaway heating concerns → Deploy ADVANCED (safety timeout)
- Need monitoring → Deploy ADVANCED (with notifications)

---

## 📁 Related Files

- `central_heating_relay_ANALYSIS.md` - Detailed technical analysis
- `central_heating_relay_IMPROVED.yaml` - ⭐ Recommended version
- `central_heating_relay_ADVANCED.yaml` - Production-grade version

---

## 🎉 Summary

**Your current automation is functionally correct!** ✅

**Improvements available:**
- 87.5% fewer Zigbee messages
- No redundant relay commands
- Better relay longevity
- Production-grade safety features

**Recommendation:** Deploy IMPROVED version for immediate benefits with minimal complexity.

---

**Status:** ✅ Analysis complete  
**Risk Level:** Low (changes are optimizations)  
**Time to Deploy:** 5 minutes  
**Expected Benefit:** 87.5% fewer relay commands

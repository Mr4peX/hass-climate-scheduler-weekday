# 🔧 SIMPLE FIX: Remove default_fan_mode from Climate Scheduler

## ⚡ The Problem

Your Climate Scheduler has `default_fan_mode: "auto"` set, but your Sonoff TRV thermostats **don't support fan_mode** (they're radiator valves, not HVAC units with fans).

This causes errors every time the scheduler tries to update thermostats.

---

## ✅ The Fix (2 Minutes)

### Find and Remove default_fan_mode

**File to edit:** The scheduler configuration you're actually using in Home Assistant

**Likely location:** One of these:
- `configuration.yaml`
- `packages/heating.yaml`  
- Custom integration configuration

### What to Change:

```yaml
# ❌ BEFORE (WRONG - Causes errors):
- platform: climate_scheduler
  name: Living Room Heating
  default_profile: "Weekly Schedule"
  climate_entities:
    - climate.living_calorifer_thermostat
  profiles:
    - id: "Weekly Schedule"
      default_hvac_mode: "heat"
      default_fan_mode: "auto"  # ← REMOVE THIS LINE!
      default_min_temp: 20

# ✅ AFTER (CORRECT - No errors):
- platform: climate_scheduler
  name: Living Room Heating
  default_profile: "Weekly Schedule"
  climate_entities:
    - climate.living_calorifer_thermostat
  profiles:
    - id: "Weekly Schedule"
      default_hvac_mode: "heat"
      # default_fan_mode: "auto"  ← Removed (thermostats don't support this)
      default_min_temp: 20
```

---

## 📝 Step-by-Step Instructions

### Step 1: Find Your Active Scheduler Configuration

**Option A: Check configuration.yaml**
```yaml
# Look in configuration.yaml for:
switch:
  - platform: climate_scheduler
```

**Option B: Check packages folder**
```yaml
# Look in packages/*.yaml for:
switch:
  - platform: climate_scheduler
```

**Option C: Check custom_components**
```
custom_components/climate_scheduler/
```

### Step 2: Edit the File

1. Open the file containing your scheduler configuration
2. Find each `- platform: climate_scheduler` block
3. Look for `default_fan_mode: "auto"` in each profile
4. Remove or comment out the line

### Step 3: Apply to ALL 4 Schedulers

You have 4 schedulers (from your 4 rooms):
- [ ] Living Room Heating
- [ ] Bedroom (Dormitor) Heating
- [ ] Office (Birou) Heating  
- [ ] Child Room (Cam Luca) Heating

**Each one needs `default_fan_mode` removed!**

### Step 4: Restart Home Assistant

```
Settings → System → Restart
```

---

## 🔍 Quick Search Method

**In VS Code:**
1. Press `Ctrl+Shift+F` (Find in Files)
2. Search for: `default_fan_mode`
3. Results will show all locations
4. Edit each file
5. Remove the line

**In Terminal:**
```powershell
# Find all files with default_fan_mode
cd Z:\TradingView\Kiwwi\HomeAsistentGreen
grep -r "default_fan_mode" .
```

---

## ✅ Verification

After restart, check logs:

### ✅ Should SEE:
- Climate schedulers loading successfully
- Temperature updates working
- Schedule changes executing cleanly

### ❌ Should NOT SEE:
- `Entity does not support action climate.set_fan_mode`
- `ServiceNotSupported` errors
- Fan mode related errors

---

## 📊 Why This Happens

### HVAC vs TRV Thermostats

**HVAC Systems (with fans):**
```yaml
climate.hvac_unit:
  - supports: temperature, hvac_mode, fan_mode ✅
  - has: furnace, air handler, fan
```

**TRV Radiator Valves (no fans):**
```yaml
climate.sonoff_trvzb_thermostat:
  - supports: temperature, hvac_mode ✅
  - does NOT support: fan_mode ❌
  - reason: It's a valve, not an HVAC unit
```

Your Sonoff TRVs are **radiator valves** - they open/close water flow. They don't have fans, so `fan_mode` doesn't exist!

---

## 🎯 Complete Fix Checklist

- [ ] Find active scheduler configuration file
- [ ] Remove `default_fan_mode: "auto"` from Living Room scheduler
- [ ] Remove `default_fan_mode: "auto"` from Bedroom scheduler
- [ ] Remove `default_fan_mode: "auto"` from Office scheduler
- [ ] Remove `default_fan_mode: "auto"` from Child Room scheduler
- [ ] Save all files
- [ ] Restart Home Assistant
- [ ] Check logs → No fan_mode errors ✅
- [ ] Test schedule change → Works cleanly ✅

---

## 🚨 Still Getting Zigbee Overflow?

If you still see `ZIGBEE_MAX_MESSAGE_LIMIT_REACHED` after removing `fan_mode`, it means the scheduler is still updating all 8 thermostats simultaneously.

**This is a separate issue** that requires modifying the Climate Scheduler integration code itself, OR creating a custom wrapper script with delays (as described in CLIMATE_SCHEDULER_OVERFLOW_ISSUE.md).

---

## 📞 Need Help Finding the File?

Share the output of:

```powershell
cd Z:\TradingView\Kiwwi\HomeAsistentGreen
grep -r "platform: climate_scheduler" .
```

This will show exactly which files contain your active scheduler configuration!

---

**Status:** ✅ **Fix identified!**  
**Action:** Remove `default_fan_mode` from all schedulers  
**Time:** 2 minutes  
**Difficulty:** Easy

# 📅 Three-Profile Heating System

## Overview

This example demonstrates a complete three-profile heating system for maximum flexibility and energy savings:

- **Weekly Schedule** - Normal daily routine (comfort when home, eco when away)
- **Away Mode** - Energy-saving when house is empty for days
- **Holiday Mode** - Special schedules for weekends and holidays

---

## 🏠 Real-World System Specs

**Hardware:**
- 8 TuYa Zigbee thermostats (TRVZB, mains-powered 24V)
- 4 rooms (Living, Dormitor, Birou, Cam Luca)
- 1 central relay (Sonoff) for boiler control
- Zigbee coordinator (built-in Home Assistant Green)

**Software:**
- Climate Scheduler with weekday validation fix
- 252 schedule entries across 4 rooms
- 3 profiles per room (Weekly/Away/Holiday)

---

## 📂 Files in This Folder

### 1. `complete_heating_schedulers.yaml` (1004 lines)

**Full configuration with all 252 schedule entries:**
- Living Room: 4 thermostats, 9 time slots/day × 7 days = 63 entries
- Birou (Office): 1 thermostat, 9 time slots/day × 7 days = 63 entries  
- Dormitor (Bedroom): 2 thermostats, 9 time slots/day × 7 days = 63 entries
- Cam Luca: 1 thermostat, 9 time slots/day × 7 days = 63 entries

**Copy this to:** `/config/climate_scheduler.yaml`

---

## 🎯 Profile System Explained

### Profile 1: Weekly Schedule (Default)

**Purpose:** Normal daily routine

**Living Room Example:**
```yaml
Monday-Friday (Workdays):
- 00:00-06:30 → 20°C (sleeping, eco)
- 06:30-08:00 → 22°C (morning, comfort)
- 08:00-17:00 → 20°C (away at work, eco)
- 17:00-21:30 → 22°C (evening, comfort)
- 21:30-00:00 → 20°C (sleeping, eco)

Saturday-Sunday (Weekend):
- 06:30-23:00 → 22°C (home all day, comfort)
- Other times → 20°C (sleeping, eco)
```

**Bedroom Example:**
```yaml
All days:
- 06:30-08:00 → 22°C (morning warmth)
- 08:00-13:30 → 21°C (Saturday/Sunday comfort)
- Other times → 20°C (sleeping temperature)
```

---

### Profile 2: Away Mode

**Purpose:** House empty for multiple days

**Temperatures:**
- All rooms: **15°C** (minimum safe temperature)
- Prevents freezing
- Prevents mold/humidity issues
- Massive energy savings

**When to use:**
- Business trips (2-7 days)
- Vacations
- Weekend getaways
- House empty for 24+ hours

**Energy savings:** ~70% compared to Weekly Schedule

---

### Profile 3: Holiday Mode

**Purpose:** Special schedules for holidays/weekends when routines change

**Custom schedules per room:**
- Living Room: Extended comfort hours
- Bedrooms: Relaxed morning schedules
- Office: Lower temps (not in use)
- Child's room: Adjusted for school holidays

**When to use:**
- Public holidays
- Christmas/New Year
- School holidays
- Extended weekends
- Work-from-home days

---

## 🔧 How to Install

### Step 1: Copy Configuration

```bash
# Copy example to your Home Assistant config
cp complete_heating_schedulers.yaml /config/climate_scheduler.yaml
```

### Step 2: Customize Entity IDs

**Find your climate entities:**
```yaml
# Go to Developer Tools → States
# Search for: climate.
# Copy your thermostat entity IDs
```

**Replace in YAML:**
```yaml
climate_entities:
  - climate.YOUR_LIVING_ROOM_1    # Replace these
  - climate.YOUR_LIVING_ROOM_2
  - climate.YOUR_BEDROOM
  - climate.YOUR_OFFICE
```

### Step 3: Add to Configuration.yaml

```yaml
# Add this line to configuration.yaml
climate_scheduler: !include climate_scheduler.yaml
```

### Step 4: Restart Home Assistant

```bash
# Configuration → System → Restart
# Or use Developer Tools → YAML → Restart
```

### Step 5: Verify Entities Created

**Check in Developer Tools → States:**
- `switch.living_room_heating` (on/off control)
- `select.living_room_heating_profile` (profile selector)
- `switch.dormitor_heating`
- `select.dormitor_heating_profile`
- `switch.birou_heating`
- `select.birou_heating_profile`
- `switch.cam_luca_heating`
- `select.cam_luca_heating_profile`

---

## 🎮 How to Use Profiles

### Manual Profile Switching

**Via UI:**
1. Go to Settings → Devices & Services → Entities
2. Find: `select.living_room_heating_profile`
3. Click and choose: Weekly Schedule / Away Mode / Holiday Mode

**Via Automation:**
```yaml
# Switch to Away Mode when leaving
- service: select.select_option
  target:
    entity_id:
      - select.living_room_heating_profile
      - select.dormitor_heating_profile
      - select.birou_heating_profile
      - select.cam_luca_heating_profile
  data:
    option: "Away Mode"
```

---

### Automatic Profile Switching (Recommended)

**Example: Auto-Away when nobody home**

```yaml
automation:
  - alias: "Heating - Auto Away Mode"
    trigger:
      - platform: state
        entity_id: group.family
        to: "not_home"
        for:
          hours: 2
    action:
      - service: select.select_option
        target:
          entity_id:
            - select.living_room_heating_profile
            - select.dormitor_heating_profile
            - select.birou_heating_profile
            - select.cam_luca_heating_profile
        data:
          option: "Away Mode"
      - service: notify.mobile_app
        data:
          message: "🏠 Heating switched to Away Mode (nobody home for 2h)"
```

**Example: Auto-Return to Weekly Schedule**

```yaml
automation:
  - alias: "Heating - Auto Return from Away"
    trigger:
      - platform: state
        entity_id: group.family
        to: "home"
    condition:
      - condition: state
        entity_id: select.living_room_heating_profile
        state: "Away Mode"
    action:
      - service: select.select_option
        target:
          entity_id:
            - select.living_room_heating_profile
            - select.dormitor_heating_profile
            - select.birou_heating_profile
            - select.cam_luca_heating_profile
        data:
          option: "Weekly Schedule"
      - service: notify.mobile_app
        data:
          message: "🏠 Welcome home! Heating back to Weekly Schedule"
```

---

## 📊 Schedule Structure

### Living Room (4 Thermostats)

**Entities controlled:**
- `climate.living_calorifer_thermostat`
- `climate.bucatarie_calorifer_thermostat`
- `climate.hol_intrare_calorifer_thermostat`
- `climate.hol_parter_calorifer_thermostat`

**Schedule logic:**
- **Weekdays (Mon-Fri):** Eco during work hours (08:00-17:00)
- **Weekends (Sat-Sun):** Extended comfort (06:30-23:00)
- **Morning boost:** 06:30-08:00 @ 22°C
- **Evening comfort:** 17:00-21:30 @ 22°C

---

### Birou / Office (1 Thermostat)

**Entity controlled:**
- `climate.birou_calorifer_thermostat`

**Schedule logic:**
- **Weekdays:** High comfort (17:00-23:00 @ 22°C)
- **Weekends:** All-day comfort (06:30-00:00 @ 22°C)
- **Night:** Eco mode (00:00-06:30 @ 20°C)

---

### Dormitor / Bedroom (2 Thermostats)

**Entities controlled:**
- `climate.dormitor_calorifer_1_thermostat`
- `climate.dormitor_calorifer_2_thermostat`

**Schedule logic:**
- **Stable temperature:** 20°C most of the day
- **Morning boost:** 06:30-08:00 @ 22°C (weekdays) / 21°C (weekends)
- **Focus on sleep quality:** Lower temps at night

---

### Cam Luca / Child's Room (1 Thermostat)

**Entity controlled:**
- `climate.sonoff_trvzb_thermostat`

**Schedule logic:**
- **Variable by day:** Different schedules Mon-Sun
- **Evening priority:** 19:00-21:30 @ 21-22°C
- **Bedtime:** 21:30-23:00 @ 22°C (comfort for sleep)
- **Homework time:** 16:00-19:00 adjusted by weekday

---

## 🧪 Testing Your Configuration

### Test 1: Profile Switching (2 minutes)

**Steps:**
1. Note current room temperature
2. Switch profile: Weekly → Away Mode
3. Wait 30 seconds
4. Check thermostat target changed to 15°C
5. Switch back: Away → Weekly Schedule
6. Verify target restored to scheduled temperature

**Success:** Target temperature changes immediately ✅

---

### Test 2: Schedule Activation (5 minutes)

**Steps:**
1. Wait for next scheduled time change
2. Watch thermostat target temperature
3. Should change automatically at scheduled time
4. Check in logbook: "Living Room Heating changed to XX°C"

**Success:** Temperature changes at exact scheduled times ✅

---

### Test 3: Multi-Room Sync (1 hour)

**Steps:**
1. Switch all rooms to Away Mode
2. Wait 5 minutes
3. Verify ALL 8 thermostats show 15°C target
4. Switch back to Weekly Schedule
5. Verify each room follows its own schedule

**Success:** All rooms synchronized, then independent ✅

---

### Test 4: 24-Hour Stability (1 day)

**Steps:**
1. Enable Weekly Schedule on all rooms
2. Let system run for 24 hours
3. Check logbook for all temperature changes
4. Count changes = 9 per room × 4 rooms = 36 changes

**Success:** All scheduled changes executed ✅

---

## 🔍 Troubleshooting

### Issue: Profile selector not showing

**Check:**
```bash
# In terminal or Developer Tools → Template
{{ states('select.living_room_heating_profile') }}

# Should return: "Weekly Schedule" or "Away Mode" or "Holiday Mode"
# If "unavailable" → Configuration error
```

**Fix:**
1. Check `climate_scheduler.yaml` syntax
2. Verify `profiles:` section has all 3 IDs
3. Restart Home Assistant

---

### Issue: Temperature not changing at scheduled time

**Check:**
```yaml
# Developer Tools → States
# Find: switch.living_room_heating
# Should be: "on"
```

**Fix:**
1. Turn on the switch: `switch.living_room_heating`
2. Check if schedule has entry for current weekday
3. Verify time format is "HH:MM:SS"

---

### Issue: Away Mode not saving energy

**Check:**
```yaml
# Verify Away Mode profile exists in YAML:
- id: "Away Mode"
  default_min_temp: 15  # Should be 15°C
```

**Verify actual target:**
```bash
# Developer Tools → States
# Find your thermostat, check attribute: temperature
# Should be 15 when Away Mode active
```

---

### Issue: One room not responding

**Check:**
```bash
# Verify entity ID exists:
{{ states('climate.your_thermostat') }}

# Should NOT return: "unknown"
```

**Fix:**
1. Go to Developer Tools → States
2. Search for your climate entities
3. Copy exact entity ID to YAML
4. Restart Home Assistant

---

## 📈 Energy Savings Calculation

### Weekly Schedule (Baseline)

**Average temperature:**
- Comfort hours: 22°C (8h/day)
- Eco hours: 20°C (16h/day)
- Weighted average: 20.67°C

**Energy consumption:** 100% (baseline)

---

### Away Mode

**Temperature:** 15°C constant

**Energy savings:**
- Temperature reduction: 5.67°C
- Savings: ~70% (rough estimate)
- €€€ saved per day away

---

### Holiday Mode

**Variable by room and time**

**Energy impact:**
- Some rooms: Higher comfort (more energy)
- Other rooms: Lower usage (less energy)
- Overall: ~10-20% more than Weekly Schedule
- But: Increased comfort and happiness! 😊

---

## 🎯 Pro Tips

### 1. Start Simple

**First week:**
- Use only Weekly Schedule
- Monitor actual vs target temperatures
- Fine-tune time slots if needed

**Second week:**
- Test Away Mode when leaving for a day
- Verify energy savings
- Adjust return automation timing

**Third week:**
- Create Holiday Mode schedules
- Test on a long weekend
- Refine based on comfort needs

---

### 2. Customize Temperatures

**Everyone is different!**

Some prefer:
- Cooler bedrooms (18-19°C)
- Warmer living rooms (23-24°C)
- Consistent temperatures (no eco mode)

**To adjust:**
1. Edit `complete_heating_schedulers.yaml`
2. Find your room section
3. Change `min_temp:` values
4. Restart Home Assistant

---

### 3. Use CSV for Planning

**Human-friendly approach:**
1. Open `heating_schedule_template.csv` (in schedules folder)
2. Edit in Excel/LibreOffice
3. See visual table of all times
4. Plan weekly patterns easily
5. Convert to YAML when happy

**Much easier than editing 252 YAML entries!** 📊

---

### 4. Monitor Energy Usage

**Track your savings:**
```yaml
# Add to configuration.yaml
sensor:
  - platform: template
    sensors:
      heating_profile_active:
        friendly_name: "Active Heating Profile"
        value_template: >
          {{ states('select.living_room_heating_profile') }}
```

**Create energy dashboard:**
1. Settings → Dashboards → Energy
2. Add your boiler power sensor
3. Compare consumption by profile
4. See real savings!

---

### 5. Seasonal Adjustments

**Winter (Dec-Feb):** Higher comfort temps
```yaml
min_temp: 22  # Was 20
```

**Spring/Fall (Mar-May, Sep-Nov):** Medium temps
```yaml
min_temp: 20  # Standard
```

**Summer (Jun-Aug):** Lower temps or off
```yaml
default_state: false  # Turn off heating
```

---

## 🎁 Bonus: Automation Ideas

### Smart Preheating

**Heat house before arrival:**
```yaml
automation:
  - alias: "Heating - Preheat Before Arrival"
    trigger:
      - platform: zone
        entity_id: person.you
        zone: zone.home
        event: enter
    action:
      # 30 minutes before arrival, start heating
      - delay:
          minutes: -30
      - service: select.select_option
        target:
          entity_id: select.living_room_heating_profile
        data:
          option: "Weekly Schedule"
```

---

### Weather-Based Adjustments

**Lower temps on sunny days:**
```yaml
automation:
  - alias: "Heating - Reduce on Sunny Days"
    trigger:
      - platform: numeric_state
        entity_id: sensor.outdoor_temperature
        above: 15
    condition:
      - condition: sun
        after: sunrise
        before: sunset
    action:
      - service: climate.set_temperature
        target:
          entity_id: climate.living_calorifer_thermostat
        data:
          temperature: "{{ state_attr('climate.living_calorifer_thermostat', 'temperature') | float - 1 }}"
```

---

### Vacation Mode

**Ultra-low temps for extended absence:**
```yaml
# Create 4th profile in YAML:
- id: "Vacation Mode"
  default_hvac_mode: "heat"
  default_min_temp: 10  # Absolute minimum
  schedule: []  # No schedule, constant 10°C
```

---

## 📚 Related Examples

- **Hybrid External Sensors** → See `../hybrid-external-sensors/` folder
  - Prevents 2-hour timeout issue
  - Must-have for external temperature sensors

- **Schedule Examples** → See `../schedules/` folder  
  - CSV template for easy editing
  - Individual room examples
  - How to convert CSV → YAML

- **Deployment Guide** → See `../DEPLOYMENT_GUIDE.md`
  - Complete installation steps
  - Testing procedures
  - Troubleshooting guide

---

## ✅ Success Criteria

Your three-profile system is working correctly when:

1. ✅ All 8 climate scheduler entities created
2. ✅ Profile selectors show 3 options each
3. ✅ Manual profile switching works instantly
4. ✅ Away Mode sets all thermostats to 15°C
5. ✅ Weekly Schedule follows time slots
6. ✅ Temperatures change at exact scheduled times
7. ✅ Auto-away automation triggers correctly
8. ✅ Auto-return restores Weekly Schedule
9. ✅ Energy consumption decreases in Away Mode
10. ✅ System runs 24/7 without errors

---

## 🎓 What You'll Learn

By implementing this system:

- **Profile concept:** Multiple schedules per room
- **Weekday scheduling:** Different temps by day of week
- **Energy management:** Balance comfort vs cost
- **Automation design:** Smart switching logic
- **Home Assistant:** Climate control fundamentals
- **YAML structure:** Complex configuration patterns

---

## 🚀 Next Steps

1. **Deploy basic system** (1 hour)
   - Copy complete_heating_schedulers.yaml
   - Customize entity IDs
   - Restart HA
   - Test manual switching

2. **Add auto-away** (30 minutes)
   - Create presence group
   - Add automation
   - Test leaving house

3. **Add auto-return** (15 minutes)
   - Create return automation
   - Test coming home
   - Fine-tune delay timings

4. **Add hybrid sensors** (45 minutes)
   - See `../hybrid-external-sensors/`
   - Deploy keep-alive automations
   - Prevent timeout issues

5. **Monitor and optimize** (ongoing)
   - Track energy usage
   - Adjust temperatures
   - Refine schedules
   - Add more automations

---

## 🎉 Success Story

**Our real system:**
- 8 thermostats
- 252 schedule entries  
- 3 profiles per room
- Hybrid external sensors
- Auto-away/auto-return
- Running 24/7 since deployment
- Zero timeout issues
- Comfortable home
- Lower energy bills

**You can achieve the same!** 💪

---

## 📞 Need Help?

**Issues with this example?**
- Check `DEPLOYMENT_GUIDE.md` for installation help
- See `PROGRESS.md` for implementation status
- Open issue on GitHub if bugs found

**Want to contribute?**
- Share your schedules
- Improve documentation
- Add more automation examples
- Help other users

---

**Happy heating! 🔥**

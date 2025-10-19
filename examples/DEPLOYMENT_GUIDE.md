# 🚀 Climate Scheduler - Complete Deployment Guide

> **Step-by-step guide to implement a professional heating automation system**
>
> From installation to full operation in under 2 hours!

---

## 📋 Table of Contents

1. [System Overview](#system-overview)
2. [Prerequisites](#prerequisites)
3. [Installation Steps](#installation-steps)
4. [Configuration](#configuration)
5. [Testing](#testing)
6. [Troubleshooting](#troubleshooting)

---

## 🎯 System Overview

### What You'll Build

A complete heating automation system with:

**✅ Three Operating Profiles:**
- **Weekly Schedule** - Personalized per room/day/time
- **Away Mode** - Energy-saving constant temperature
- **Holiday Mode** - Relaxed weekend schedule all week

**✅ External Sensor Integration:**
- Real-time temperature updates
- 10-minute keep-alive (prevents timeout)
- Multi-thermostat synchronization

**✅ Advanced Scheduling:**
- Multiple rooms
- Different times per day
- Weekday vs weekend variations
- Easy CSV-based creation

---

## ✅ Prerequisites

### Required:

- [ ] Home Assistant 2024.6.0 or newer
- [ ] Climate entities (thermostats) configured
- [ ] HACS installed
- [ ] Basic YAML knowledge
- [ ] 30-60 minutes free time

### Recommended:

- [ ] Studio Code Server addon (file editing)
- [ ] External temperature sensors (for hybrid automation)
- [ ] Home Assistant Green or always-on system
- [ ] Zigbee coordinator (if using Zigbee thermostats)

### Optional:

- [ ] Presence detection (for auto-away)
- [ ] Energy monitoring (track savings)
- [ ] Mobile app (notifications)

---

## 📥 Installation Steps

### Step 1: Install Climate Scheduler via HACS

1. **Open HACS** (Home Assistant Community Store)
2. Click **Integrations**
3. Click **+ Explore & Download Repositories**
4. Search: **"climate scheduler"**
5. Select **Climate Scheduler**
6. Click **Download**
7. **Restart Home Assistant**

### Step 2: Add Custom Repository (Enhanced Version)

**If using the enhanced weekday-specific version:**

1. **HACS → Integrations → ⋮ → Custom repositories**
2. **Repository URL:** `https://github.com/Mr4peX/hass-climate-scheduler-weekday`
3. **Category:** Integration
4. **Add**
5. **Download** the integration
6. **Restart Home Assistant**

---

## ⚙️ Configuration

### Step 3: Choose Your Implementation Path

**Choose based on your needs:**

| Path | What You Get | Complexity | Time |
|------|--------------|------------|------|
| **Basic** | Simple scheduling, 1-2 rooms | Easy | 30 min |
| **Intermediate** | Three-profile system, multiple rooms | Medium | 1 hour |
| **Advanced** | Full system with external sensors | Advanced | 2 hours |

---

### Path A: Basic Implementation (Quick Start)

**Perfect for:** Testing, single room, simple needs

#### 1. Create Configuration File

Navigate to: `/config/switches/heating_schedulers.yaml`

```yaml
- platform: climate_scheduler
  name: "Living Room Heating"
  climate: climate.living_room_thermostat
  schedules:
    - name: "Weekly Schedule"
      schedule:
        - time: "06:00:00"
          temperature: 20
          weekdays: [mon, tue, wed, thu, fri]
        - time: "08:00:00"
          temperature: 17
          weekdays: [mon, tue, wed, thu, fri]
        - time: "18:00:00"
          temperature: 21
          weekdays: [mon, tue, wed, thu, fri]
        - time: "23:00:00"
          temperature: 18
          weekdays: [mon, tue, wed, thu, fri]
        - time: "08:00:00"
          temperature: 20
          weekdays: [sat, sun]
        - time: "23:00:00"
          temperature: 18
          weekdays: [sat, sun]
```

#### 2. Add to configuration.yaml

```yaml
switch: !include_dir_list switches/
```

#### 3. Restart Home Assistant

**Settings → System → Restart**

#### 4. Verify

**Developer Tools → States**

Search for: `switch.living_room_heating_weekly_schedule`

---

### Path B: Intermediate (Three-Profile System)

**Perfect for:** Multiple rooms, vacation home, regular travel

#### 1. Copy Example Configuration

From: `examples/three-profile-system/heating_schedulers_config.yaml`

To: `/config/switches/heating_schedulers.yaml`

#### 2. Customize Entity IDs

Replace with YOUR thermostat entities:

```yaml
climate: climate.your_thermostat_here
```

#### 3. Customize Temperatures & Times

Adjust to your preferences:

```yaml
- time: "06:00:00"  # Your wake-up time
  temperature: 20    # Your preferred temp
```

#### 4. Create Schedules for Each Room

Use the CSV template from `examples/schedules/template_schedule.csv`

Convert to YAML using guide in `CSV_to_YAML_guide.md`

#### 5. Add Configuration

```yaml
# configuration.yaml
switch: !include_dir_list switches/
```

#### 6. Restart & Verify

**Check for these entities:**
- `switch.living_room_heating_weekly_schedule`
- `switch.living_room_heating_away`
- `switch.living_room_heating_holiday`
- `select.living_room_heating_profile`

---

### Path C: Advanced (Full System + External Sensors)

**Perfect for:** Complete solution, multiple thermostats per room, external sensors

#### 1. Complete Path B First

Get three-profile system working.

#### 2. Add Hybrid External Sensor Automations

**For each room with external sensor:**

**Example: Living Room**

From: `examples/hybrid-external-sensors/living_room_4_thermostats.yaml`

Deploy to: **Settings → Automations → Create Automation → Edit in YAML**

**Customize:**
```yaml
# Your sensor entity
entity_id: sensor.your_room_temperature

# Your thermostat external sensor entities
entity_id:
  - number.your_thermostat_1_external_sensor
  - number.your_thermostat_2_external_sensor
```

#### 3. Deploy for All Rooms

Repeat for:
- Bedroom (if using external sensor)
- Office (if using external sensor)
- Other rooms (if using external sensor)

#### 4. Verify Hybrid Automations Working

**Developer Tools → States**

Check "Last Updated" timestamp on external sensor values:
- Should update every ~10 minutes
- Should update immediately when temp changes

#### 5. Test 3-Hour Stability

Let system run for 3+ hours to verify no timeout issues.

---

## 🧪 Testing

### Test 1: Profile Switching (5 minutes)

1. **Go to:** Settings → Devices & Services → Entities
2. **Find:** `select.living_room_heating_profile`
3. **Click** on entity
4. **Select:** "Away"
5. **Verify:** Thermostat target temp changes to Away temp (e.g., 19°C)
6. **Switch back:** "Weekly Schedule"
7. **Verify:** Thermostat returns to scheduled temp

✅ **Pass:** Profile switching works immediately

---

### Test 2: Schedule Time Changes (30 minutes)

1. **Note current time:** e.g., 14:30
2. **Check:** Current scheduled temperature
3. **Wait for next schedule change:** e.g., 18:00
4. **Verify:** Temperature changes automatically

✅ **Pass:** Schedule triggers at specified times

---

### Test 3: External Sensor Updates (20 minutes)

**Only if using hybrid external sensor automations:**

1. **Developer Tools → States**
2. **Find:** `number.your_thermostat_external_sensor_value`
3. **Note:** "Last Updated" timestamp
4. **Wait 10 minutes**
5. **Check:** "Last Updated" timestamp updated
6. **Change room temperature** (e.g., open window)
7. **Verify:** External sensor value updates immediately

✅ **Pass:** Both time_pattern and state triggers working

---

### Test 4: Multi-Thermostat Sync (3 hours)

**Only for rooms with multiple thermostats:**

1. **Developer Tools → States**
2. **Check all thermostats** in same room
3. **Verify:** All using same external sensor value
4. **Wait 3 hours** (past 2-hour timeout)
5. **Verify:** All thermostats still synchronized

✅ **Pass:** No single-thermostat activation

---

## 🐛 Troubleshooting

### Issue: "Platform not found: climate_scheduler"

**Cause:** Component not installed or Home Assistant not restarted

**Solution:**
1. Verify HACS installation successful
2. Check `/config/custom_components/climate_scheduler/` folder exists
3. Restart Home Assistant
4. Check logs for errors

---

### Issue: "Entity not available"

**Cause:** Climate entity doesn't exist or wrong name

**Solution:**
1. Developer Tools → States
2. Search for your thermostat
3. Copy exact entity ID (case-sensitive!)
4. Update configuration with correct entity ID
5. Reload switches: Developer Tools → YAML → Switches

---

### Issue: "Invalid config"

**Cause:** YAML syntax error

**Solution:**
1. Developer Tools → YAML → Check Configuration
2. Look for specific error message
3. Common issues:
   - Indentation wrong (use spaces, not tabs)
   - Missing colons
   - Quotes missing around times
   - Entity IDs misspelled

---

### Issue: Schedule not switching at specified time

**Cause:** Time format wrong or weekday not specified

**Solution:**
1. Verify time format: `"HH:MM:SS"` (with quotes!)
2. Verify weekdays: `[mon, tue, wed, thu, fri]`
3. Check logs for errors
4. Test manually: Turn switch on/off

---

### Issue: External sensor not updating

**Cause:** Automation not triggering or sensor offline

**Solution:**
1. Check automation enabled
2. Check automation traces (Settings → Automations → Traces)
3. Verify sensor entity ID correct
4. Check sensor battery (if battery-powered)
5. Test automation manually: Run actions button

---

### Issue: Thermostat timeout after 2 hours

**Cause:** Hybrid automation not deployed or not working

**Solution:**
1. Verify hybrid automation exists and enabled
2. Check automation traces - should fire every 10 minutes
3. Verify `platform: time_pattern` trigger present
4. Verify `minutes: "/10"` syntax correct
5. Reload automations: Developer Tools → YAML → Automations

---

## 📊 Verification Checklist

### Basic System:

- [ ] Climate scheduler installed via HACS
- [ ] Configuration file created
- [ ] Entities visible in States
- [ ] Profile switching works
- [ ] Schedule changes at specified times
- [ ] No errors in logs

### Three-Profile System:

- [ ] 3 switches per room (Weekly/Away/Holiday)
- [ ] 1 profile selector per room
- [ ] Profile switching works immediately
- [ ] Each profile has correct temperatures
- [ ] Away mode saves energy
- [ ] Holiday mode uses Saturday schedule

### External Sensor System:

- [ ] Hybrid automation deployed
- [ ] External sensor updates every 10 minutes
- [ ] Real-time updates when temp changes
- [ ] All thermostats in room synchronized
- [ ] No timeout after 3+ hours
- [ ] Condition prevents "unknown" values

---

## 🎉 Success Criteria

### You'll know it's working when:

✅ **Profile switching is instant**
- Select "Away" → Thermostat changes within seconds

✅ **Schedules are automatic**
- No manual adjustments needed
- Temperature changes at specified times

✅ **External sensors reliable**
- Values always current (never timeout)
- Multi-thermostat rooms stay synchronized

✅ **Energy is saved**
- Away mode lowers temperature
- Night setback reduces heating
- 20-40% typical savings

✅ **Comfort is maintained**
- Wake to warm room
- House at desired temp when home
- No cold spots or overheating

---

## 📞 Next Steps

### After Successful Deployment:

1. **Monitor for 48 hours** - Verify stability
2. **Fine-tune temperatures** - Adjust to preferences
3. **Add more rooms** - Expand system
4. **Create auto-return** - Never come home to cold house
5. **Track energy savings** - Measure results

### Optional Enhancements:

**Automatic Away Mode:**
```yaml
# When everyone leaves
- trigger: state
  entity_id: group.all_persons
  to: "not_home"
  for: "00:15:00"
```

**Automatic Return:**
```yaml
# Before arriving home
- trigger: time
  at: "17:00:00"
condition:
  - condition: time
    weekday: [sun]
```

**Weather-Based Adjustments:**
```yaml
# Lower temps when sunny
condition:
  - condition: numeric_state
    entity_id: sensor.outdoor_temperature
    above: 15
```

---

## 🎓 Learning Resources

### Included Documentation:

- `three-profile-system/README.md` - Three-profile guide
- `hybrid-external-sensors/README.md` - External sensor guide
- `schedules/README.md` - Scheduling concepts
- Each YAML file has inline comments

### External Resources:

- [Climate Integration Docs](https://www.home-assistant.io/integrations/climate/)
- [Automation Guide](https://www.home-assistant.io/docs/automation/)
- [YAML Syntax](https://www.home-assistant.io/docs/configuration/yaml/)
- [Home Assistant Forums](https://community.home-assistant.io/)

---

## ⏱️ Time Estimates

| Task | Time | Difficulty |
|------|------|------------|
| Install HACS | 10 min | Easy |
| Install Climate Scheduler | 5 min | Easy |
| Basic configuration (1 room) | 15 min | Easy |
| Three-profile system (4 rooms) | 45 min | Medium |
| Hybrid external sensors (4 rooms) | 30 min | Medium |
| Testing & verification | 30 min | Easy |
| **Total (Full System)** | **2-3 hours** | **Medium** |

**After initial setup:** System runs automatically with no maintenance!

---

## 🏁 Quick Start Checklist

### Absolute Minimum (30 minutes):

- [ ] Install Climate Scheduler via HACS
- [ ] Create basic configuration (1 room)
- [ ] Test profile switching
- [ ] Verify schedule changes
- [ ] Done! ✅

### Recommended (1 hour):

- [ ] Install Climate Scheduler
- [ ] Three-profile configuration (2-3 rooms)
- [ ] Customize temperatures/times
- [ ] Test all profiles
- [ ] Monitor for 24 hours
- [ ] Done! ✅

### Complete System (2 hours):

- [ ] Install Climate Scheduler
- [ ] Three-profile configuration (all rooms)
- [ ] Deploy hybrid external sensor automations
- [ ] Test everything
- [ ] 3-hour stability test
- [ ] Create auto-return automation
- [ ] Done! ✅

---

**Ready to get started?**

**Next:** Choose your path (Basic/Intermediate/Advanced) and follow the steps!

**Questions?** Check the README.md in each example folder!

**Happy Heating! 🏡🔥**

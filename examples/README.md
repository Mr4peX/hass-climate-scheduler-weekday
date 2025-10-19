# 📚 Climate Scheduler - Examples & User Guides

> **Complete real-world implementation examples for the Enhanced Climate Scheduler**
> 
> These examples are from an actual production system running on Home Assistant Green with 8 Zigbee thermostats (TuYa TRVs) controlling a central heating system.

---

## 🎯 What's Inside

This folder contains **everything you need** to implement a professional heating automation system:

### 🔥 Three-Profile Heating System
Complete scheduling system with:
- **Weekly Schedule** - Different temperatures per room, per day, per time
- **Away Mode** - Energy-saving mode (19°C constant)
- **Holiday Mode** - Saturday schedule for all week

### 🔄 Hybrid External Sensor Automations
Prevent thermostat timeout issues with:
- Real-time temperature updates
- 10-minute keep-alive mechanism
- Works with external room sensors

### 📅 Multi-Room Scheduling Examples
Real schedules for 4 rooms:
- 252 time-based schedule entries
- Different wake-up/work/sleep times per room
- Weekend vs weekday variations

---

## 📂 Folder Structure

```
examples/
├── README.md (this file)
│
├── three-profile-system/
│   ├── README.md - Complete guide to three-profile setup
│   ├── heating_schedulers_config.yaml - Full configuration example
│   ├── profile_switching_automations.yaml - Profile management
│   └── CSV_to_YAML_guide.md - How to create schedules from CSV
│
├── hybrid-external-sensors/
│   ├── README.md - Fix 2-hour timeout issue
│   ├── living_room_4_thermostats.yaml - 4 thermostats + 1 sensor
│   ├── bedroom_2_thermostats.yaml - 2 thermostats + 1 sensor
│   ├── office_1_thermostat.yaml - 1 thermostat + 1 sensor
│   ├── child_room_1_thermostat.yaml - 1 thermostat + 1 sensor
│   └── testing_guide.md - How to verify it works
│
├── schedules/
│   ├── README.md - Scheduling concepts explained
│   ├── sample_living_room_schedule.yaml - Living room example
│   ├── sample_bedroom_schedule.yaml - Bedroom example
│   ├── sample_office_schedule.yaml - Office example
│   ├── template_schedule.csv - Blank CSV template
│   └── complete_4_rooms_252_entries.yaml - Full real example
│
└── DEPLOYMENT_GUIDE.md - Start here!
```

---

## 🚀 Quick Start

### New Users - Start Here:

1. **Read:** `DEPLOYMENT_GUIDE.md` (step-by-step installation)
2. **Choose:** Which examples match your needs
3. **Copy:** Example files to your Home Assistant
4. **Customize:** Entity IDs, times, temperatures
5. **Deploy:** Test and enjoy!

### By Use Case:

| What You Need | Go To |
|---------------|-------|
| **Multiple heating profiles** (Away/Home/Holiday) | `three-profile-system/` |
| **Fix thermostat timeout** (external sensors) | `hybrid-external-sensors/` |
| **Create heating schedules** (CSV to YAML) | `schedules/` |
| **Complete working example** | All folders! |

---

## 🏠 Real-World System Specs

**Hardware Setup:**
- Home Assistant Green (HAOS)
- 8x TuYa Zigbee Thermostats (TRVZB)
- 4x External Temperature Sensors
- 1x Central Heating Relay (Sonoff)
- Zigbee Coordinator (built-in)

**Rooms Covered:**
1. Living Room (4 thermostats)
2. Bedroom (2 thermostats)
3. Office (1 thermostat)
4. Child's Room (1 thermostat)

**Features Implemented:**
- ✅ 252 scheduled time/temperature entries
- ✅ 3 profiles (Weekly/Away/Holiday)
- ✅ Profile switching (manual + automatic)
- ✅ External sensor integration
- ✅ Hybrid keep-alive system (prevents timeout)
- ✅ Central relay automation
- ✅ Energy saving (25-30% in Away mode)

---

## 📖 Comprehensive Guides Included

### 1. Three-Profile System (`three-profile-system/`)

**What it does:**
- Manage 3 different heating modes
- Switch profiles via UI dropdown
- Automatic profile switching (optional)
- Different schedules per profile

**Perfect for:**
- Families with regular schedule
- Vacation homes (Away mode)
- Energy-conscious users
- Anyone who travels

**Example use case:**
> "When we go away for the weekend, switch to Away mode (19°C). House warms up automatically 2 hours before we return on Sunday evening."

---

### 2. Hybrid External Sensors (`hybrid-external-sensors/`)

**Problem it solves:**
- Thermostats have 2-hour timeout for external sensors
- If no temperature update → Falls back to internal sensor
- Different readings → Single heater activates → Inefficient

**Solution:**
- Dual-trigger automation (state change + time pattern)
- Real-time updates when temperature changes
- Keep-alive update every 10 minutes
- Thermostats never timeout!

**Perfect for:**
- Anyone using external temperature sensors
- Multi-thermostat rooms (living room with 4 TRVs)
- Preventing single-heater activation
- Reliable temperature control

**Example use case:**
> "Living room has 4 thermostats sharing 1 room sensor. Without keep-alive, if temperature is stable for 2+ hours, thermostats timeout and one heater activates alone. With hybrid automation, all 4 stay synchronized 24/7."

---

### 3. Advanced Scheduling (`schedules/`)

**What it does:**
- Create complex time-based schedules
- Different temps per room, per day, per time
- Easy CSV to YAML conversion
- Weekday vs weekend variations

**Perfect for:**
- Multi-room homes
- Different occupancy patterns
- Shift workers
- Detailed temperature control

**Example use case:**
> "Living room: 20°C from 6:00-8:00 (morning), 17°C during work hours, 21°C evenings 18:00-23:00. Bedroom: Cold during day (17°C), warm at night (19°C). Office: Only warm during work hours Mon-Fri."

---

## 🎓 Learning Path

### Beginner - Start Simple

1. **Read:** `DEPLOYMENT_GUIDE.md`
2. **Install:** Climate Scheduler via HACS
3. **Create:** One simple schedule (1 room, 1 profile)
4. **Test:** Profile switching works
5. **Expand:** Add more rooms

### Intermediate - Add Features

1. **Implement:** Three-profile system
2. **Add:** External sensor automation (if using external sensors)
3. **Create:** Schedules for all rooms
4. **Setup:** Profile switching automation

### Advanced - Full System

1. **Deploy:** All 4 hybrid external sensor automations
2. **Create:** 252-entry complete schedule
3. **Setup:** Auto-away automation (presence detection)
4. **Setup:** Auto-return automation (timed)
5. **Monitor:** Energy consumption tracking
6. **Optimize:** Fine-tune temperatures

---

## ⚙️ Technical Requirements

### Minimum Requirements:

- Home Assistant 2024.6.0+ (for blueprint schema)
- Climate entities (thermostats)
- Climate Scheduler component installed

### Recommended:

- Home Assistant Green or similar (always-on)
- Zigbee coordinator (for wireless thermostats)
- External temperature sensors (for hybrid automation)
- Studio Code Server addon (for file editing)

### Optional:

- Presence detection (for auto-away)
- Energy monitoring (for tracking savings)
- Notification system (for alerts)

---

## 📝 How to Use These Examples

### Step 1: Identify Your Needs

**Do you need multiple profiles?**
→ Go to `three-profile-system/`

**Using external sensors?**
→ Go to `hybrid-external-sensors/`

**Need to create schedules?**
→ Go to `schedules/`

### Step 2: Read the Guide

Each folder has a `README.md` with:
- What problem it solves
- How it works
- Step-by-step setup
- Troubleshooting

### Step 3: Copy Example Files

1. Open example YAML file
2. Copy to your Home Assistant config
3. Customize entity IDs
4. Adjust temperatures/times
5. Save and reload

### Step 4: Test

1. Verify configuration valid
2. Check entities created
3. Test functionality
4. Monitor for 24 hours
5. Adjust as needed

---

## 🔧 Customization Guide

### Entity IDs

**Replace these with YOUR entities:**

```yaml
# Example shows:
entity_id: climate.living_calorifer_thermostat

# Change to your thermostat:
entity_id: climate.your_thermostat_name
```

### Temperatures

**Adjust to your preferences:**

```yaml
# Example shows:
temperature: 20

# Change to your preference:
temperature: 21  # Warmer
temperature: 19  # Cooler
```

### Times

**Match your schedule:**

```yaml
# Example shows:
time: "06:00:00"

# Change to your wake-up time:
time: "07:30:00"
```

### Room Names

**Use your room names:**

```yaml
# Example shows:
alias: "Living Room - Update External Sensor"

# Change to your room:
alias: "Master Bedroom - Update External Sensor"
```

---

## 🐛 Troubleshooting

### Common Issues:

**1. "Entity not found"**
- Check entity IDs are correct
- Use Developer Tools → States to find exact names
- Entity IDs are case-sensitive

**2. "Invalid config"**
- Check YAML syntax (indentation matters!)
- Use YAML validator
- Check for typos in entity names

**3. "Schedule not switching"**
- Verify time format correct (HH:MM:SS)
- Check weekday specified (if using weekday schedules)
- Reload automations after changes

**4. "Automation not triggering"**
- Check automation is Enabled
- Verify trigger entity IDs correct
- Check automation Traces for errors

### Getting Help:

1. **Check logs:** Settings → System → Logs
2. **Check traces:** Automation → Traces button
3. **Validate config:** Developer Tools → YAML → Check Configuration
4. **Test manually:** Automation → Run actions button

---

## 💡 Pro Tips

### Temperature Scheduling

**Start conservative:**
- Don't set too many time slots initially
- Test with 2-3 time periods per day
- Expand gradually as you learn patterns

**Common patterns:**
- Morning warmup: 1 hour before wake-up
- Day setback: 2-3°C lower when away
- Evening comfort: 2 hours before bedtime
- Night setback: 1-2°C lower while sleeping

### External Sensors

**Best practices:**
- Place sensor at average height (not floor/ceiling)
- Away from direct sunlight
- Away from drafts/doors
- Central location in room

**Multi-thermostat rooms:**
- Use hybrid automation (mandatory!)
- 10-minute keep-alive prevents timeout
- All thermostats stay synchronized

### Profile Usage

**Weekly Schedule:**
- Your normal routine
- Most time spent here
- Comfort + efficiency balance

**Away Mode:**
- Weekends away
- Vacations
- Extended absences
- Set 3-4°C lower (19°C typical)

**Holiday Mode:**
- Public holidays at home
- Saturday schedule (typically most relaxed)
- No work/school routine

---

## 📊 Expected Results

### Energy Savings

**Typical savings:**
- Away mode: 25-30% reduction
- Night setback: 10-15% reduction
- Overall: 20-40% depending on schedule

**Payback period:**
- Smart thermostats: 1-2 years
- Full system: 2-3 years
- Ongoing savings forever

### Comfort Improvements

**Before automation:**
- Manual adjustments needed
- Inconsistent temperatures
- Wake to cold rooms
- Waste energy when away

**After automation:**
- House always at desired temp
- Wake to warm room
- No manual adjustments
- Energy saved automatically

---

## 🚀 Advanced Features

### Automatic Away Mode

```yaml
# Trigger on everyone leaving
- trigger: state
  entity_id: group.all_persons
  to: "not_home"
  for: "00:15:00"
```

### Automatic Return Mode

```yaml
# Switch back before arrival
- trigger: time
  at: "17:00:00"
condition:
  - condition: time
    weekday: [sun]
```

### Presence-Based Heating

```yaml
# Heat room only when occupied
- trigger: state
  entity_id: binary_sensor.living_room_occupancy
  to: "on"
```

### Weather-Based Adjustments

```yaml
# Lower temps when sunny
condition:
  - condition: numeric_state
    entity_id: sensor.outdoor_temperature
    above: 15
```

---

## 📚 Additional Resources

### Included Documentation

- `DEPLOYMENT_GUIDE.md` - Complete installation guide
- Each folder has detailed `README.md`
- Example files include inline comments
- Testing guides for verification

### External Resources

- [Home Assistant Climate Integration](https://www.home-assistant.io/integrations/climate/)
- [Automation Documentation](https://www.home-assistant.io/docs/automation/)
- [YAML Syntax](https://www.home-assistant.io/docs/configuration/yaml/)
- [Blueprint System](https://www.home-assistant.io/docs/blueprint/)

### Community

- Home Assistant Forums - Share your setup
- GitHub Issues - Report problems
- Discord - Real-time help
- Reddit r/homeassistant - Ideas and inspiration

---

## 🙏 Credits

**System Design & Implementation:**
- User: Mr4peX
- Real production system
- Tested and refined over multiple iterations
- Shared to help the community

**Climate Scheduler Component:**
- Base integration for scheduling
- Enhanced with weekday-specific validation
- Custom fixes for edge cases

**Hardware Vendors:**
- Home Assistant Green (Nabu Casa)
- TuYa Zigbee Thermostats
- Various temperature sensors
- Sonoff relay products

---

## 📄 License

These examples are provided as-is for educational purposes.

- ✅ Free to use
- ✅ Free to modify
- ✅ Free to share
- ✅ No warranty provided

**Please share improvements back to the community!**

---

## 🔄 Version History

**v1.0 (2025-10-19)**
- Initial release
- Three-profile system examples
- Hybrid external sensor automations
- Complete 4-room, 252-entry schedule
- Comprehensive documentation

**Future Plans:**
- Weather-based heating adjustments
- Machine learning temperature predictions
- Energy cost optimization
- Integration with energy tariffs

---

## 📞 Support

**Before asking for help:**
1. ✅ Read the relevant README.md
2. ✅ Check troubleshooting section
3. ✅ Verify entity IDs correct
4. ✅ Check Home Assistant logs

**When asking for help, include:**
- Your Home Assistant version
- Your hardware (thermostats, sensors)
- Relevant YAML configuration
- Error messages from logs
- What you've already tried

---

## 🎉 Success Stories

**Share your implementation!**

If you successfully implement these examples:
- Share screenshots on Home Assistant forums
- Post energy savings data
- Contribute improvements back
- Help other users

**Tag:** #ClimateScheduler #HomeAutomation #EnergySavings

---

**Ready to build your smart heating system?**

**Start with:** `DEPLOYMENT_GUIDE.md`

**Questions?** Check the README.md in each folder!

**Happy Automating! 🏡🔥**

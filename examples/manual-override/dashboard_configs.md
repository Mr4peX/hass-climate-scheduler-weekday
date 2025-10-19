# Dashboard Configuration for Heating Control

## Quick Access Scheduler Toggles

Add this card to your Home Assistant dashboard for easy manual override control:

```yaml
type: entities
title: 🔥 Heating Schedulers
entities:
  - entity: switch.living_room_heating_scheduler
    name: Living Room
    icon: mdi:sofa
  - entity: switch.bedroom_dormitor_heating_scheduler
    name: Bedroom
    icon: mdi:bed
  - entity: switch.office_birou_heating_scheduler
    name: Office
    icon: mdi:desk
  - entity: switch.child_room_cam_luca_heating_scheduler
    name: Child Room
    icon: mdi:human-male-child
show_header_toggle: true  # Toggle all schedulers at once
```

---

## Alternative: Button Cards (More Visual)

```yaml
type: grid
columns: 2
cards:
  - type: button
    entity: switch.living_room_heating_scheduler
    name: Living Room
    icon: mdi:sofa
    tap_action:
      action: toggle
    hold_action:
      action: more-info
    show_state: true
  
  - type: button
    entity: switch.bedroom_dormitor_heating_scheduler
    name: Bedroom
    icon: mdi:bed
    tap_action:
      action: toggle
    hold_action:
      action: more-info
    show_state: true
  
  - type: button
    entity: switch.office_birou_heating_scheduler
    name: Office
    icon: mdi:desk
    tap_action:
      action: toggle
    hold_action:
      action: more-info
    show_state: true
  
  - type: button
    entity: switch.child_room_cam_luca_heating_scheduler
    name: Child Room
    icon: mdi:human-male-child
    tap_action:
      action: toggle
    hold_action:
      action: more-info
    show_state: true
```

---

## Complete Heating Control Dashboard

Includes schedulers, current temperatures, and quick climate controls:

```yaml
type: vertical-stack
cards:
  # Scheduler Controls
  - type: entities
    title: 🔥 Heating Schedulers
    entities:
      - entity: switch.living_room_heating_scheduler
        name: Living Room
      - entity: switch.bedroom_dormitor_heating_scheduler
        name: Bedroom
      - entity: switch.office_birou_heating_scheduler
        name: Office
      - entity: switch.child_room_cam_luca_heating_scheduler
        name: Child Room
    show_header_toggle: true
  
  # Current Temperatures
  - type: entities
    title: 🌡️ Current Temperatures
    entities:
      - entity: sensor.living_temp_umiditate_temperature
        name: Living Room
      - entity: sensor.sonoff_snzb_02d_temperature
        name: Bedroom
      - entity: sensor.birou_temperature
        name: Office
      - entity: sensor.luca_temp_umiditate_temperature
        name: Child Room
  
  # Quick Climate Controls
  - type: entities
    title: 🎚️ Quick Controls
    entities:
      - entity: climate.living_calorifer_thermostat
        name: Living Room
      - entity: climate.dormitor_calorifer_1_thermostat
        name: Bedroom
      - entity: climate.birou_calorifer_thermostat
        name: Office
      - entity: climate.sonoff_trvzb_thermostat
        name: Child Room
```

---

## With Profile Selectors

If you want to quickly change profiles:

```yaml
type: entities
title: 🔥 Heating Control
entities:
  # Living Room
  - type: section
    label: Living Room
  - entity: switch.living_room_heating_scheduler
    name: Schedule Active
  - entity: select.living_room_heating_profile
    name: Profile
  - entity: climate.living_calorifer_thermostat
    name: Temperature
  
  # Bedroom
  - type: section
    label: Bedroom
  - entity: switch.bedroom_dormitor_heating_scheduler
    name: Schedule Active
  - entity: select.bedroom_dormitor_heating_profile
    name: Profile
  - entity: climate.dormitor_calorifer_1_thermostat
    name: Temperature
  
  # Office
  - type: section
    label: Office
  - entity: switch.office_birou_heating_scheduler
    name: Schedule Active
  - entity: select.office_birou_heating_profile
    name: Profile
  - entity: climate.birou_calorifer_thermostat
    name: Temperature
  
  # Child Room
  - type: section
    label: Child Room
  - entity: switch.child_room_cam_luca_heating_scheduler
    name: Schedule Active
  - entity: select.child_room_cam_luca_heating_profile
    name: Profile
  - entity: climate.sonoff_trvzb_thermostat
    name: Temperature
```

---

## Usage Instructions

### For Manual Override:

1. **Need to increase temperature temporarily?**
   - Turn OFF scheduler switch for that room
   - Adjust thermostat to desired temperature
   - Temperature will stay at your setting
   - Turn scheduler ON when you want to resume schedule

2. **Example:**
   ```
   18:00 → Turn OFF Living Room scheduler
   18:01 → Set Living Room to 24°C
   20:00 → Temperature still at 24°C
   20:30 → Turn ON Living Room scheduler
   20:45 → Scheduler resumes (next 15-min update)
   ```

---

## Status Indicators

**Scheduler States:**

| Switch State | Meaning | Temperature Control |
|-------------|---------|---------------------|
| **ON** ✅ | Schedule active | Follows schedule |
| **OFF** ❌ | Manual mode | Your setting stays |

**Visual Feedback:**
- Switch **ON** (blue/green) = Schedule controlling temperature
- Switch **OFF** (gray) = Manual control active

---

## Mobile App Quick Actions

Add to mobile app for fast access:

```yaml
# In configuration.yaml
homeassistant:
  customize:
    switch.living_room_heating_scheduler:
      friendly_name: "Living Room Schedule"
      icon: mdi:radiator
    
    switch.bedroom_dormitor_heating_scheduler:
      friendly_name: "Bedroom Schedule"
      icon: mdi:bed
    
    switch.office_birou_heating_scheduler:
      friendly_name: "Office Schedule"
      icon: mdi:desk
    
    switch.child_room_cam_luca_heating_scheduler:
      friendly_name: "Child Room Schedule"
      icon: mdi:human-male-child
```

---

## Tips

1. **Quick Toggle All:** Use header toggle to turn all schedulers ON/OFF at once
2. **Visual Status:** Color-coded switches show active/inactive state
3. **Long Press:** Hold button for more details (current schedule, next change time)
4. **Organize:** Group related controls together (scheduler + thermostat per room)

---

**Status:** ✅ Dashboard configurations ready  
**Recommendation:** Start with "Quick Access Scheduler Toggles"  
**Complexity:** Low (copy-paste YAML to dashboard)

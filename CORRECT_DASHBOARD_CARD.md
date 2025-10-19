# 🔥 CORRECT Dashboard Card for Your Heating Schedulers

## ✅ Copy This to Your Dashboard

Based on your actual scheduler names, use this exact configuration:

```yaml
type: entities
title: 🔥 Heating Schedulers
entities:
  - entity: switch.living_room_heating
    name: Living Room
    icon: mdi:sofa
  - entity: switch.dormitor_heating
    name: Bedroom (Dormitor)
    icon: mdi:bed
  - entity: switch.birou_heating
    name: Office (Birou)
    icon: mdi:desk
  - entity: switch.luca_room_heating
    name: Child Room (Luca)
    icon: mdi:human-male-child
show_header_toggle: true
```

---

## 📋 Entity Name Mapping

Your scheduler names convert to entity IDs like this:

| Scheduler Name | Entity ID |
|---------------|-----------|
| "Living Room Heating" | `switch.living_room_heating` |
| "Dormitor Heating" | `switch.dormitor_heating` |
| "Birou Heating" | `switch.birou_heating` |
| "Luca Room Heating" | `switch.luca_room_heating` |

**Note:** Home Assistant automatically converts spaces to underscores and removes special characters.

---

## 🔍 How to Verify Entity Names

If you want to double-check the exact entity names:

### Method 1: Developer Tools

1. **Go to:** Developer Tools → States
2. **Search:** "heating" or "scheduler"
3. **Look for:** Entities starting with `switch.`
4. **Copy:** Exact entity IDs

### Method 2: Settings

1. **Go to:** Settings → Devices & Services → Entities
2. **Filter:** Type "heating"
3. **Find:** Your scheduler switches
4. **Click:** Entity to see full entity_id

---

## 🎨 Alternative Card Designs

### Option 1: With Profile Selectors

```yaml
type: entities
title: 🔥 Heating Control
entities:
  # Living Room
  - type: section
    label: Living Room
  - entity: switch.living_room_heating
    name: Schedule Active
  - entity: select.living_room_heating_profile
    name: Profile
  
  # Bedroom
  - type: section
    label: Bedroom (Dormitor)
  - entity: switch.dormitor_heating
    name: Schedule Active
  - entity: select.dormitor_heating_profile
    name: Profile
  
  # Office
  - type: section
    label: Office (Birou)
  - entity: switch.birou_heating
    name: Schedule Active
  - entity: select.birou_heating_profile
    name: Profile
  
  # Child Room
  - type: section
    label: Child Room (Luca)
  - entity: switch.luca_room_heating
    name: Schedule Active
  - entity: select.luca_room_heating_profile
    name: Profile
```

---

### Option 2: Button Grid (More Visual)

```yaml
type: grid
columns: 2
cards:
  - type: button
    entity: switch.living_room_heating
    name: Living Room
    icon: mdi:sofa
    tap_action:
      action: toggle
    show_state: true
  
  - type: button
    entity: switch.dormitor_heating
    name: Bedroom
    icon: mdi:bed
    tap_action:
      action: toggle
    show_state: true
  
  - type: button
    entity: switch.birou_heating
    name: Office
    icon: mdi:desk
    tap_action:
      action: toggle
    show_state: true
  
  - type: button
    entity: switch.luca_room_heating
    name: Child Room
    icon: mdi:human-male-child
    tap_action:
      action: toggle
    show_state: true
```

---

### Option 3: Complete Control Dashboard

```yaml
type: vertical-stack
cards:
  # Scheduler Toggles
  - type: entities
    title: 🔥 Scheduler Control
    entities:
      - entity: switch.living_room_heating
        name: Living Room
      - entity: switch.dormitor_heating
        name: Bedroom
      - entity: switch.birou_heating
        name: Office
      - entity: switch.luca_room_heating
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
  
  # Quick Climate Access
  - type: entities
    title: 🎚️ Thermostats
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

## 🚀 How to Add to Dashboard

### Step-by-Step:

1. **Open Dashboard** in edit mode (three dots → Edit Dashboard)

2. **Click** "Add Card"

3. **Choose** "Manual" (at bottom) or "Entities"

4. **If Manual:**
   - Delete example code
   - Paste one of the YAML codes above
   - Click "Save"

5. **If Entities:**
   - Click "Show Code Editor"
   - Delete example code
   - Paste one of the YAML codes above
   - Click "Save"

6. **Position card** where you want it

7. **Save dashboard**

---

## ✅ Testing Your Card

After adding the card:

### Test 1: Toggle Switch
- **Click** one of the scheduler switches
- **Expected:** Switch turns ON/OFF
- **Check logs:** No errors

### Test 2: Header Toggle (if using entities card)
- **Click** the card title/header
- **Expected:** ALL schedulers toggle together
- **Verify:** All switches change state

### Test 3: Manual Override
1. **Turn OFF** Living Room scheduler
2. **Go to:** Climate control
3. **Change** Living Room temperature to 24°C
4. **Wait** 15 minutes
5. **Check:** Temperature still at 24°C ✅
6. **Turn ON** scheduler
7. **Wait** 15 minutes
8. **Check:** Temperature follows schedule ✅

---

## 🐛 Troubleshooting

### Problem: "Entity not available"

**Possible causes:**
1. Scheduler not loaded yet (restart Home Assistant)
2. Entity name typo
3. Scheduler configuration not active

**Fix:**
```
Settings → Developer Tools → States
Search: "heating"
Find your scheduler entities
Copy exact entity_id
```

---

### Problem: Profile selector not found

**Entity names for profile selectors:**
- `select.living_room_heating_profile`
- `select.dormitor_heating_profile`
- `select.birou_heating_profile`
- `select.luca_room_heating_profile`

**If not found:**
Check if your scheduler configuration has multiple profiles defined.

---

### Problem: Card not updating

**Reload dashboard:**
- Press `F5` or `Ctrl+R`
- Or three dots → "Refresh"

---

## 📊 Expected Behavior

### When Scheduler is ON:
- Switch shows **ON** (blue/green)
- Temperature follows schedule
- Updates every 15 minutes (your new setting)

### When Scheduler is OFF:
- Switch shows **OFF** (gray)
- Manual temperature control active
- Scheduler updates are paused

### Header Toggle:
- **Click header** → ALL schedulers toggle together
- Useful for: "Manual mode for entire house!"

---

## 💡 Pro Tips

### Tip 1: Quick Access Script

Create a script for common scenarios:

```yaml
# scripts.yaml
heating_manual_mode:
  alias: "All Heating Manual Mode"
  sequence:
    - service: switch.turn_off
      target:
        entity_id:
          - switch.living_room_heating
          - switch.dormitor_heating
          - switch.birou_heating
          - switch.luca_room_heating

heating_auto_mode:
  alias: "All Heating Auto Mode"
  sequence:
    - service: switch.turn_on
      target:
        entity_id:
          - switch.living_room_heating
          - switch.dormitor_heating
          - switch.birou_heating
          - switch.luca_room_heating
```

Then add buttons to dashboard:
```yaml
type: button
entity: script.heating_manual_mode
name: Manual Mode
icon: mdi:hand-pointing-up

type: button
entity: script.heating_auto_mode
name: Auto Mode
icon: mdi:clock-check
```

---

### Tip 2: Conditional Color

Make switches more visual:

```yaml
type: entities
entities:
  - entity: switch.living_room_heating
    name: Living Room
    icon: mdi:sofa
    card_mod:
      style: |
        :host {
          --paper-item-icon-color: {% if is_state('switch.living_room_heating', 'on') %} green {% else %} red {% endif %};
        }
```

---

### Tip 3: Notifications

Get notified when manual mode active:

```yaml
# automations.yaml
- alias: "Heating Manual Mode Reminder"
  trigger:
    - platform: time
      at: "22:00:00"  # 10 PM reminder
  condition:
    - condition: state
      entity_id: switch.living_room_heating
      state: "off"
  action:
    - service: notify.notify
      data:
        title: "Heating Reminder"
        message: "Living Room scheduler is OFF. Turn ON for automatic control."
```

---

## 🎯 Summary

**Your correct entity names are:**
- ✅ `switch.living_room_heating`
- ✅ `switch.dormitor_heating`
- ✅ `switch.birou_heating`
- ✅ `switch.luca_room_heating`

**NOT:**
- ❌ `switch.living_room_heating_scheduler`
- ❌ `switch.bedroom_dormitor_heating_scheduler`
- ❌ etc.

**Copy the first YAML code at the top of this file** - it's ready to use! 🚀

---

**Status:** ✅ Correct entity names identified  
**Action:** Copy YAML from top of file to your dashboard  
**Time:** 2 minutes to add card

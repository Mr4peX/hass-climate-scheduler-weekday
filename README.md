# Home Assistant Climate Scheduler with Weekday Support

A [Home Assistant](https://www.home-assistant.io/) component to facilitate the automation of climate entities with **weekday-specific scheduling**. A scheduler controls its assigned climate entities based on user defined profiles and schedules. Each scheduler is represented as a switch entity which can be toggled on or off. Each scheduler registers an input_select entity which can be used to switch between profiles.

## ✨ Enhanced Features

This is an enhanced version of [hass-climate-scheduler](https://github.com/FrancisLab/hass-climate-scheduler) with added **weekday-specific scheduling support**:

- 🗓️ **Different schedules for different days** - Monday ≠ Saturday
- 📅 **Full weekday control** - Specify which days each schedule applies to
- 🔄 **Backward compatible** - Existing configurations still work
- 🎯 **Flexible scheduling** - Workday vs weekend patterns, specific days only

### Example: Weekday-Specific Scheduling

```yaml
schedule:
  - time: "06:30:00"
    weekdays: [1, 2, 3, 4, 5]  # Monday-Friday only
    min_temp: 22
  - time: "08:00:00"
    weekdays: [6, 7]  # Saturday-Sunday only
    min_temp: 22
```

![Climate Scheduler UI](https://github.com/FrancisLab/hass-climate-scheduler/blob/main/climate_scheduler.png)

## Installation

Install the custom component using [HACS](https://hacs.xyz/) (recommended), or manually by copying `custom_components\climate_scheduler` to your `config\custom_components` directory.

[![hacs_badge](https://img.shields.io/badge/HACS-Custom-orange.svg?style=for-the-badge)](https://github.com/custom-components/hacs)

## Configuration

### Component Configuration

```yaml
climate_scheduler:
  update_interval: "00:10:00"
```

| Variable        | Description                                                     | Type                            | Default  |
| --------------- | --------------------------------------------------------------- | ------------------------------- | -------- |
| update_interval | How often schedulers should attempt to update climate entities. | Optional Positive Time HH:MM:SS | 00:15:00 |

### Scheduler Configuration

Each scheduler is registered as a switch entity. Each scheduler must be assigned at least one profile.

**Schedulers**

```yaml
switch:
  - platform: climate_scheduler
    name: Bedroom
    default_state: True
    default_profile: "Override"
    climate_entities:
      - climate.bedroom
    profiles:
      # See Profiles section
```

| Variable         | Description                       | Type                    | Default             |
| ---------------- | --------------------------------- | ----------------------- | ------------------- |
| **profiles**     | Climate profiles of the scheduler | Required List[Profiles] |                     |
| name             | Name of the scheduler             | Optional String         | "Climate Scheduler" |
| default_state    | Initial state of scheduler        | Optional Bool           | False               |
| default_profile  | Initial profile of scheduler      | Optional String         | Id of 1st profile   |
| climate_entities | Climate entities to control       | Optional List[String]   | []                  |

**Profiles**

Profiles define when and how to configure climate entities. At least one must be provided per scheduler. Multiple profiles can be defined to make it easy to swith between different presets via UI or automations.

```yaml
    ...
    profiles:
    - id: "Bedroom Heating"
      default_hvac_mode: "heat"
      default_fan_mode: "auto"
      default_swing_mode: "auto"
      default_min_temp: 22
      schedule:
        # See Scheuldes section
```

| Variable           | Description                                                                 | Type                     | Default |
| ------------------ | --------------------------------------------------------------------------- | ------------------------ | ------- |
| **id**             | Name of the profile. Must be unique in list.                                | Required String          |         |
| default_hvac_mode  | HVAC mode to use when none specified by schedule entry                      | Optional String          | None    |
| default_fan_mode   | Fan mode to use when none specified by schedule entry                       | Optional String          | None    |
| default_swing_mode | Swing mode to use when none specified by schedule entry                     | Optional String          | None    |
| default_min_temp   | Default min temperature to set when none specified by schedule entry        | Optional Float           | None    |
| default_max_temp   | Default max temperature to set when none specified by schedule entry        | Optional Float           | None    |
| schedule           | List of schedule entries defining climate changes to apply at certain times | Optional List[Schedules] | None    |

**Schedules**

A schedule is an optional list of times at which the target climate changes. If none are provided the scheduler will only set default values if any are presence. Having a single entry will cause the scheduler to set the desired values at all time. Having multiple entries will have the scheduler update climate entities at the specific time.

```yaml
    ...
    schedule:
      # Heat quietly during morning. Rely on default temp
      - time: "06:30:00"
        fan_mode: "superQuiet"
        swing_mode: "vertical"
      # Heat during the day. Relying on default values
      - time: "07:30:00"
      # Lower temperature during the night with quiet heating
      - time: "20:30:00"
        fan_mode: "superQuiet"
        swing_mode: "vertical"
        min_temp: 17.5
```

| Variable   | Description                                                  | Type                                | Default                                           |
| ---------- | ------------------------------------------------------------ | ----------------------------------- | ------------------------------------------------- |
| **time**   | Time where the climate must be updated                       | Required Time (HH:MM:SS) < 24:00:00 |                                                   |
| **weekdays** | 🆕 Days of week this schedule applies to (1=Mon, 7=Sun)     | Optional List[Integer 1-7]          | [1,2,3,4,5,6,7] (all days)                        |
| hvac_mode  | HVAC mode to set at time                                     | Optional String                     | Default value from profile if any, otherwise none |
| fan_mode   | Fan mode to set at time                                      | Optional String                     | Default value from profile if any, otherwise none |
| swing_mode | Swing mode to set at time                                    | Optional String                     | Default value from profile if any, otherwise none |
| min_temp   | Min temperature to set at time. Use with relevant HVAC modes | Optional Float                      | Default value from profile if any, otherwise none |
| max_temp   | Max temperature to set at time. Use with relevant HVAC modes | Optional Float                      | Default value from profile if any, otherwise none |

### 🆕 Weekday-Specific Scheduling Examples

**Workday vs Weekend Pattern:**
```yaml
schedule:
  # Weekday morning (Mon-Fri)
  - time: "06:30:00"
    weekdays: [1, 2, 3, 4, 5]
    min_temp: 22
  # Weekday evening
  - time: "17:00:00"
    weekdays: [1, 2, 3, 4, 5]
    min_temp: 22
  # Weekend all-day comfort (Sat-Sun)
  - time: "08:00:00"
    weekdays: [6, 7]
    min_temp: 22
  # Night setback (all days)
  - time: "22:00:00"
    min_temp: 18
```

**Work-From-Home Days Only:**
```yaml
schedule:
  # Only heat on Mon, Wed, Fri (WFH days)
  - time: "08:00:00"
    weekdays: [1, 3, 5]
    min_temp: 23
  # Cool down on other days
  - time: "08:00:00"
    weekdays: [2, 4, 6, 7]
    min_temp: 19
```

**Different Schedule Each Day:**
```yaml
schedule:
  # Monday early start
  - time: "06:00:00"
    weekdays: [1]
    min_temp: 22
  # Tuesday-Thursday normal
  - time: "07:00:00"
    weekdays: [2, 3, 4]
    min_temp: 22
  # Friday relaxed
  - time: "08:00:00"
    weekdays: [5]
    min_temp: 22
  # Weekend late start
  - time: "09:00:00"
    weekdays: [6, 7]
    min_temp: 22
```

**Weekday Numbers:**
- 1 = Monday
- 2 = Tuesday
- 3 = Wednesday
- 4 = Thursday
- 5 = Friday
- 6 = Saturday
- 7 = Sunday

**Note:** Omitting `weekdays` makes the schedule apply to all days (backward compatible with original version).

## Tips & Tricks

### Organizing & Sharing Profiles

You can define profiles in their own separate yaml files and share them across schedulers.

In `configuration.yaml`:

```yaml
- platform: climate_scheduler
  name: Bedroom
  default_profile: "Override"
  climate_entities:
    - climate.bedroom
  profiles:
    - !include climate_profiles/bedroom/heating.yaml
    - !include climate_profiles/bedroom/cooling.yaml
    - !include climate_profiles/common/override.yaml
    - !include climate_profiles/common/away.yaml
    - !include climate_profiles/common/vacation.yaml
- platform: climate_scheduler
  name: Living Room
  default_profile: "Override"
  climate_entities:
    - climate.living_room
  profiles:
    - !include climate_profiles/common/heating.yaml
    - !include climate_profiles/common/cooling.yaml
    - !include climate_profiles/common/override.yaml
    - !include climate_profiles/common/away.yaml
    - !include climate_profiles/common/vacation.yaml
```

Example content of `climate_profiles\bedroom\heating.yaml`

```yaml
# Climate profile for winter. Only allows heating.
id: "Bedroom Heating"
default_hvac_mode: "heat"
default_fan_mode: "auto"
default_swing_mode: "auto"
default_min_temp: 22
schedule:
  # Heat quietly during morning. Rely on default temp
  - time: "06:30:00"
    fan_mode: "superQuiet"
    swing_mode: "vertical"
  # Heat during the day. Relying on default values
  - time: "07:30:00"
  # Lower temperature during the night with quiet heating
  - time: "20:30:00"
    fan_mode: "superQuiet"
    swing_mode: "vertical"
    min_temp: 17.5
```

### Overrides

It's often useful to disable a scheduler and assume manual control over climate entities. This can either be done by turning off the climate scheduler entity directly, or by defining and choosing an empty profile. In both cases, the scheduler won't make any changes to its assigned climate entities.

Override profile example:

```yaml
  ...
  profiles:
    - id: "Override"
```

### Static Profiles

Profiles with a constant climate configuration can be defined either with default values or with a single schedule entry.

Default Values:

```yaml
  ...
  profiles:
    - id: "Away"
      default_hvac_mode: "heat_cool"
      default_fan_mode: "auto"
      default_swing_mode: "auto"
      default_min_temp: 17
      default_max_temp: 27
```

Single Schedule Entry:

```yaml
  ...
  profiles:
    - id: "Away"
      schedule:
        - time: "00:00:00"
          hvac_mode: "heat_cool"
          fan_mode: "auto"
          swing_mode: "auto"
          min_temp: 17
          max_temp: 27
```

### Controlling Schedulers via UI

You can add the climate scheduler entity and its profile picker entity to your LoveLace UI to manually control them.

![Climate Scheduler UI](https://github.com/FrancisLab/hass-climate-scheduler/blob/main/climate_scheduler.png)

### Controlling Schedulers via Automation

Each scheduler registers a switch entity. The switch can be controlled with switch service calls:

- switch.toggle
- switch.turn_on
- switch.turn_off

Each scheduler registers a select_input entity for picking profiles. The profiles can be picked with input_select service calls:

- input_select.select_option
- And other less relevant input_select service calls. (Using input_select.set_options to change the content of the picker might result in undefined behavior)

---

## 🙏 Acknowledgments & Credits

### Original Project

This component is based on the excellent work by **[FrancisLab](https://github.com/FrancisLab)** with their original [hass-climate-scheduler](https://github.com/FrancisLab/hass-climate-scheduler) project.

**Special thanks to FrancisLab** for creating such a robust and well-designed climate scheduling system for Home Assistant. The original codebase provided an excellent foundation with clean architecture, comprehensive documentation, and solid implementation of profile-based scheduling.

### What This Fork Adds

The weekday-specific scheduling feature was developed to address a specific need: **different heating schedules for weekdays versus weekends**. Many households have different routines during the work week compared to weekends, and this enhancement makes it possible to define those patterns naturally within a single profile.

**Key enhancement:**
- Added `weekdays` parameter to schedule entries, allowing specification of which days (1=Monday through 7=Sunday) each schedule applies to
- Maintained 100% backward compatibility - existing configurations continue to work without modification
- Follows the same design patterns and coding standards as the original project

### Why This Fork Exists

This fork was created to solve a real-world heating automation challenge: managing 8 thermostats across 4 zones in a home with distinctly different schedules for weekdays (work/school routines) versus weekends (family time). The original scheduler handled time-based scheduling beautifully, but needed the ability to differentiate between days of the week.

Rather than creating an entirely new component, this enhancement respects and builds upon FrancisLab's excellent work. **All credit for the core functionality goes to the original author.**

### Contributing Back

If you find this weekday scheduling feature useful and would like to see it in the official hass-climate-scheduler, please consider:
- Opening a discussion in the [original repository](https://github.com/FrancisLab/hass-climate-scheduler)
- Submitting a pull request with this feature
- Supporting FrancisLab's work

### License

This fork maintains the same MIT License as the original project, ensuring the code remains open and accessible to the Home Assistant community.

---

## 📞 Support & Community

**For issues specific to weekday scheduling:** Please open an issue in this repository.

**For general scheduler functionality:** Refer to the [original project](https://github.com/FrancisLab/hass-climate-scheduler) and its excellent documentation.

**Home Assistant Community:** Join the discussion at [https://community.home-assistant.io/](https://community.home-assistant.io/)

---

## ⭐ Show Your Support

If this enhanced version helps you automate your home heating:
- Give this repository a star ⭐
- Give the [original repository](https://github.com/FrancisLab/hass-climate-scheduler) a star ⭐⭐
- Share your configuration examples with the community
- Report bugs and suggest improvements

**Thank you to FrancisLab and the entire Home Assistant community for making home automation accessible and powerful!** 🏠🔥

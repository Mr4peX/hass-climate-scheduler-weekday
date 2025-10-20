# Changelog

All notable changes to this project will be documented in this file.

## [1.0.6] - 2025-10-20

### Fixed
- **CRITICAL**: Replaced deprecated `async_track_state_change` with `async_track_state_change_event`
  - This API was deprecated and will be removed in Home Assistant 2025.5
  - Updated callback function to use event-based API
  - No functional changes - backwards compatible with current HA versions

### Technical Details
- Updated imports: Added `Event` from `homeassistant.core`
- Modified `_async_on_profile_selector_change` to accept `Event` instead of separate state parameters
- Changed state tracker from `async_track_state_change` to `async_track_state_change_event`

---

## [1.0.5] - 2025-10-20

### Fixed
- Restored `input_select` dependency in manifest.json (was accidentally removed in v1.0.4)
- Added safety check for `_profile_selector` attribute access

---

## [1.0.4] - 2025-10-20

### Fixed
- Fixed HACS `iot_class` format (changed from array to string "calculated")
- Removed outdated `info.md` file (HACS now uses README.md)

### Breaking
- ⚠️ **CRITICAL BUG**: Accidentally removed `input_select` dependency causing integration failure
- This version should NOT be used - upgrade to v1.0.5 or later

---

## [1.0.3] - 2025-10-20

### Changed
- Updated HACS configuration

---

## [1.0.2] - 2025-10-20

### Changed
- Updated HACS configuration

---

## [1.0.1] - 2025-10-20

### Changed
- Initial HACS compatibility fixes

---

## [1.0.0] - 2025-10-20

### Added
- Initial versioned release
- Added 3 new Holiday heating profiles (22°C, 23°C, 24°C)

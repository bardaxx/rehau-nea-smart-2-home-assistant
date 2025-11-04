# Changelog

## [2.0.2] - 2025-11-04

### Fixed
- Re-publish MQTT discovery configs on every zone reload to ensure persistence
- Outside temperature sensor discovery now refreshes periodically
- Zone climate and sensor discovery configs refresh every 5 minutes (configurable)
- Improved resilience to MQTT broker restarts

## [2.0.1] - 2025-11-04

### Fixed
- Fixed MQTT authentication handling for optional credentials
- Improved Mosquitto broker compatibility
- Better handling of empty MQTT username/password

### Changed
- Updated documentation with MQTT broker setup requirements

## [2.0.0] - 2025-11-04

### Added
- Full TypeScript implementation with strict typing
- Configurable intervals via environment variables
  - Zone reload interval (default: 5 minutes)
  - Token refresh interval (default: 6 hours)
  - Referentials reload interval (default: 24 hours)
- Fresh login on every boot (no token persistence)
- Automatic token refresh with fallback to fresh login
- Optimistic mode for instant UI feedback
- Separate temperature and humidity sensors per zone
- Outside temperature sensor
- Installation-wide mode control

### Changed
- Converted from JavaScript to TypeScript
- Improved error handling and logging
- Better MQTT connection management

## [1.0.0] - 2025-11-03

### Added
- Initial release
- REHAU NEA SMART 2.0 authentication
- MQTT bridge between REHAU and Home Assistant
- Home Assistant MQTT Climate integration
- REST API for direct control
- Automatic MQTT discovery
- Real-time temperature and mode updates

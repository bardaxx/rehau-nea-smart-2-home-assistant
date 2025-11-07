# REHAU NEA SMART MQTT Bridge

TypeScript-based MQTT bridge for REHAU NEA SMART 2.0 heating systems with Home Assistant integration.

> **🚨 CRITICAL: Version 2.3.3 REQUIRES CLEAN REINSTALL**
>
> This version fixes a critical bug but requires complete removal and reinstallation.
> See [Migration Guide](#-migration-guide-v233) below.

## 🚨 BREAKING CHANGES - Version 2.3.3

### What's Fixed

**Critical Bug:** Zones with duplicate numbers across different controllers were overwriting each other's data.

**Example Problem:**
- Controller 0, Zone 0 → "Living Room" (temperature: 20°C)
- Controller 1, Zone 0 → "Bedroom" (temperature: 18°C)
- **BUG:** Both zones shared the same MQTT topic, causing temperature readings to alternate

**Solution:** Each zone now uses its unique MongoDB ObjectId for identification.

### MQTT Topic Changes

| Version | Topic Format | Example |
|---------|-------------|----------|
| **< 2.3.3** | `homeassistant/climate/rehau_{installId}_zone_{zoneNumber}/...` | `homeassistant/climate/rehau_6ba02d11..._zone_0/current_temperature` |
| **≥ 2.3.3** | `homeassistant/climate/rehau_{zoneId}/...` | `homeassistant/climate/rehau_6595d1d5cceecee9ce9772e1/current_temperature` |

**Impact:** All MQTT topics have changed. Old entities will become unavailable.

---

## 📋 Migration Guide (v2.3.3)

### ⚠️ REQUIRED STEPS - DO NOT SKIP

#### Step 1: Backup Your Configuration
```bash
# Backup automations and scripts that use REHAU entities
# You'll need to update entity IDs after migration
```

#### Step 2: Remove Old Entities

**Option A: Via Home Assistant UI (Recommended)**
1. Go to **Settings** → **Devices & Services** → **MQTT**
2. Find all REHAU devices
3. Click each device → **Delete Device**
4. Repeat for all REHAU zones

**Option B: Via MQTT Explorer/CLI**
```bash
# Delete all REHAU discovery topics
mosquitto_pub -h localhost -t "homeassistant/climate/rehau_+/config" -n -r
mosquitto_pub -h localhost -t "homeassistant/sensor/rehau_+/config" -n -r
mosquitto_pub -h localhost -t "homeassistant/light/rehau_+/config" -n -r
mosquitto_pub -h localhost -t "homeassistant/lock/rehau_+/config" -n -r
```

#### Step 3: Uninstall Add-on
1. Go to **Settings** → **Add-ons** → **REHAU NEA SMART MQTT Bridge**
2. Click **Uninstall**
3. Wait for complete removal

#### Step 4: Reinstall Add-on
1. Go to **Settings** → **Add-ons** → **Add-on Store**
2. Find **REHAU NEA SMART MQTT Bridge**
3. Click **Install**
4. Configure with your REHAU credentials
5. Start the add-on

#### Step 5: Verify New Entities
1. Go to **Settings** → **Devices & Services** → **MQTT**
2. New REHAU devices should appear automatically
3. Check that all zones are present and showing correct temperatures

#### Step 6: Update Automations & Scripts
- Entity IDs have changed (see table below)
- Update all references in automations, scripts, and dashboards

---

## 📊 Entity Naming in Home Assistant

### Climate Entities

| Zone Name | Entity ID | MQTT Topic |
|-----------|-----------|------------|
| Living Room | `climate.rehau_xxx_ground_floor_living_room` | `homeassistant/climate/rehau_6595d1d5cceecee9ce9772e1/...` |
| Kitchen | `climate.rehau_xxx_ground_floor_kitchen` | `homeassistant/climate/rehau_6595d1d71cf174839175074b/...` |
| Bedroom 1 | `climate.rehau_xxx_first_floor_bedroom_1` | `homeassistant/climate/rehau_6595d1e16c9645c4cf338302/...` |

### Temperature Sensors

| Zone Name | Entity ID | MQTT Topic |
|-----------|-----------|------------|
| Living Room Temperature | `sensor.rehau_ground_floor_living_room_temperature` | `homeassistant/sensor/rehau_6595d1d5cceecee9ce9772e1_temperature/state` |
| Living Room Humidity | `sensor.rehau_ground_floor_living_room_humidity` | `homeassistant/sensor/rehau_6595d1d5cceecee9ce9772e1_humidity/state` |

### Control Entities

| Entity Type | Entity ID | MQTT Topic |
|-------------|-----------|------------|
| Ring Light | `light.rehau_xxx_ground_floor_living_room_ring_light` | `homeassistant/light/rehau_6595d1d5cceecee9ce9772e1_ring_light/...` |
| Lock | `lock.rehau_xxx_ground_floor_living_room_lock` | `homeassistant/lock/rehau_6595d1d5cceecee9ce9772e1_lock/...` |

### Entity ID Structure

```
climate.rehau_{installation}_{group}_{zone}
sensor.rehau_{group}_{zone}_{type}
light.rehau_{installation}_{group}_{zone}_ring_light
lock.rehau_{installation}_{group}_{zone}_lock
```

**Notes:**
- `{installation}` = Sanitized installation name (lowercase, underscores)
- `{group}` = Sanitized group name (lowercase, underscores)
- `{zone}` = Sanitized zone name (lowercase, underscores)
- `{type}` = `temperature` or `humidity`

### MQTT Topic Structure (v2.3.3+)

```
# Climate entity
homeassistant/climate/rehau_{zoneId}/
  ├─ config                    # Discovery config
  ├─ availability              # Online/offline status
  ├─ current_temperature       # Current temp reading
  ├─ target_temperature        # Target setpoint
  ├─ current_humidity          # Humidity reading
  ├─ mode                      # off/heat/cool
  ├─ mode_command              # Command topic
  ├─ preset                    # comfort/away
  └─ preset_command            # Command topic

# Separate sensors
homeassistant/sensor/rehau_{zoneId}_temperature/
  ├─ config
  ├─ state
  └─ availability

homeassistant/sensor/rehau_{zoneId}_humidity/
  ├─ config
  ├─ state
  └─ availability

# Ring light
homeassistant/light/rehau_{zoneId}_ring_light/
  ├─ config
  ├─ state
  └─ command

# Lock
homeassistant/lock/rehau_{zoneId}_lock/
  ├─ config
  ├─ state
  └─ command
```

**Key Change:** Topics now use `{zoneId}` (MongoDB ObjectId) instead of `{installId}_zone_{zoneNumber}`

---

## 🔧 Configuration

### Required Settings

```yaml
rehau:
  email: your.email@example.com
  password: your_password
mqtt:
  host: core-mosquitto
  port: 1883
  username: mqtt_user
  password: mqtt_password
```

### Optional Settings

```yaml
api_port: 3000                          # REST API port (default: 3000)
log_level: info                         # debug|info|warn|error (default: info)
zone_reload_interval: 300               # Seconds between HTTPS polls (default: 300)
token_refresh_interval: 21600           # Seconds between token refresh (default: 21600)
referentials_reload_interval: 86400     # Seconds between referentials reload (default: 86400)
use_group_in_names: false               # Include group in display names (default: false)
```

---

## 🐛 Troubleshooting

### Entities Not Appearing

1. **Check add-on logs** for connection errors
2. **Verify MQTT broker** is running and accessible
3. **Check REHAU credentials** are correct
4. **Restart MQTT integration** in Home Assistant

### Wrong Temperature Readings

1. **Check zone mapping** in add-on logs (set `log_level: debug`)
2. **Verify zone IDs** match between REHAU app and Home Assistant
3. **Restart add-on** to refresh all data

### Old Entities Still Visible

1. **Delete old MQTT devices** manually from Home Assistant
2. **Clear MQTT retained messages** (see Step 2 Option B above)
3. **Restart Home Assistant**

---

## 📝 Changelog

See [CHANGELOG.md](CHANGELOG.md) for detailed version history.

---

## 📄 License

MIT License - See LICENSE file for details

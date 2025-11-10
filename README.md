# Phyn Smart Water Assistant Integration for Home Assistant

[![GitHub Release][releases-shield]][releases]
[![GitHub Activity][commits-shield]][commits]
[![License][license-shield]](LICENSE)
[![hacs][hacsbadge]][hacs]
[![Community Forum][forum-shield]][forum]

**A Home Assistant custom integration for Phyn and Kohler H2Wise+ smart water monitoring devices.**

Monitor your water usage, control shutoff valves, enable away mode, and receive leak detection alerts—all from Home Assistant.

---

## 🌟 Features

- **Real-time Monitoring** (Phyn Plus)
  - Current water flow rate
  - Water temperature and pressure
  - Instant leak detection alerts
  - MQTT-based updates for immediate notifications

- **Daily Water Usage Tracking**
  - Compatible with Home Assistant Energy dashboard
  - Historical consumption data
  - Track usage patterns

- **Smart Controls**
  - Remote shutoff valve control
  - Away mode activation
  - Automatic shutoff configuration
  - Scheduled leak test management

- **Device Support**
  - Phyn Plus (PP1, PP2) - Full smart monitoring with valve control
  - Phyn Classic (PC1) - Monitoring and valve control
  - Phyn Water Sensor (PW1) - Environmental monitoring

- **Multi-Brand Support**
  - Phyn Smart Water Assistant
  - Kohler H2Wise+ by Phyn

---

## 📋 Requirements

- **Home Assistant** 2024.2.0 or newer
- **Python** 3.11 or 3.12
- **Phyn Account** with username and password
- **Supported Device** (PP1, PP2, PC1, or PW1)

---

## 🚀 Installation

### HACS (Recommended)

1. **Ensure HACS is installed.** If not, follow the [HACS installation guide](https://hacs.xyz/docs/installation/manual).

2. **Add this repository to HACS:**
   - Open HACS in Home Assistant
   - Click on **Integrations**
   - Click the three dots (⋮) in the top right
   - Select **Custom repositories**
   - Add repository URL: `https://github.com/jordanruthe/homeassistant-phyn`
   - Category: **Integration**
   - Click **Add**

3. **Install the integration:**
   - Find "Phyn Smart Water Assistant" in HACS
   - Click **Download**
   - Restart Home Assistant

### Manual Installation

1. **Download the integration:**
   ```bash
   cd /config
   git clone https://github.com/jordanruthe/homeassistant-phyn.git
   ```

2. **Copy to custom_components:**
   ```bash
   cp -r homeassistant-phyn/custom_components/phyn custom_components/
   ```

3. **Restart Home Assistant**

---

## ⚙️ Configuration

### Initial Setup

1. **Add the integration:**
   - Navigate to **Settings** → **Devices & Services**
   - Click **+ Add Integration**
   - Search for "Phyn"
   - Click to start configuration

2. **Enter your credentials:**
   - **Username**: Your Phyn account email (case-sensitive)
   - **Password**: Your Phyn account password
   - **Brand**: Select "Phyn" or "Kohler"

3. **Wait for setup:**
   - Configuration may take 2-3 minutes
   - The integration will discover your devices automatically

### Reconfiguration

If you need to change credentials or switch brands:
1. Go to **Settings** → **Devices & Services**
2. Find the Phyn integration
3. Click the three dots (⋮) → **Reconfigure**
4. Enter new credentials

### Reauthorization

If credentials expire:
1. You'll receive a notification
2. Go to **Settings** → **Devices & Services**
3. Click **Authenticate** on the Phyn integration
4. Enter your credentials

---

## 📊 Entities

### Phyn Plus (PP1/PP2)

#### Sensors
- **Daily Water Usage** - Total gallons used today (Energy dashboard compatible)
- **Total Water Usage** - Cumulative usage counter
- **Current Flow Rate** - Gallons per minute (GPM)
- **Water Temperature** - Current water temperature
- **Water Pressure** - Current water pressure (PSI)
- **Water Flow State** - Flow status indicator

#### Binary Sensors
- **Leak Test Running** - Active leak test indicator
- **Leak Test Warning** - Leak test warning status
- **Leak Detected** - Active leak detection alert
- **Firmware Update Available** - New firmware notification

#### Controls (Switches)
- **Away Mode** - Enable enhanced leak sensitivity when away
- **Autoshutoff Enabled** - Automatic water shutoff on leak detection
- **Scheduled Leak Test Enabled** - Automatic periodic leak testing

#### Valve
- **Shutoff Valve** - Remote water main control (open/close)

#### Update
- **Firmware Update** - View and manage device firmware

### Phyn Classic (PC1)

- **Daily Water Usage**
- **Average Hot Water Temperature**
- **Average Cold Water Temperature**
- **Average Hot Water Pressure**
- **Average Cold Water Pressure**
- **Firmware Update Available**
- **Firmware Update Entity**

### Phyn Water Sensor (PW1)

- **Air Temperature**
- **Humidity**
- **Battery Level**
- **High Humidity Alert**
- **Low Humidity Alert**
- **Low Temperature Alert**
- **Water Detected Alert**
- **Firmware Update Entity**

---

## 🔧 Services

### `phyn.leak_test`

Run a leak test on your Phyn Plus device.

**Parameters:**
- `entity_id` (required): The valve entity to test
- `extended` (optional): Run extended test (default: false)

**Example:**
```yaml
service: phyn.leak_test
data:
  entity_id: valve.phyn_plus_shutoff_valve
  extended: true
```

---

## 🏡 Automation Examples

### Leak Detection Alert

```yaml
automation:
  - alias: "Water Leak Detected"
    trigger:
      - platform: state
        entity_id: binary_sensor.phyn_plus_leak_detected
        to: "on"
    action:
      - service: notify.mobile_app
        data:
          title: "🚨 Water Leak Detected!"
          message: "Phyn has detected a leak. Check your home immediately."
          data:
            priority: high
      - service: valve.close
        target:
          entity_id: valve.phyn_plus_shutoff_valve
```

### Away Mode Automation

```yaml
automation:
  - alias: "Enable Away Mode"
    trigger:
      - platform: state
        entity_id: person.home_owner
        to: "not_home"
        for:
          hours: 4
    action:
      - service: switch.turn_on
        target:
          entity_id: switch.phyn_plus_away_mode

  - alias: "Disable Away Mode"
    trigger:
      - platform: state
        entity_id: person.home_owner
        to: "home"
    action:
      - service: switch.turn_off
        target:
          entity_id: switch.phyn_plus_away_mode
```

### High Water Usage Alert

```yaml
automation:
  - alias: "High Water Usage Alert"
    trigger:
      - platform: numeric_state
        entity_id: sensor.phyn_plus_daily_water_usage
        above: 300  # gallons
    action:
      - service: notify.mobile_app
        data:
          message: "High water usage detected today: {{ states('sensor.phyn_plus_daily_water_usage') }} gallons"
```

### Weekly Leak Test

```yaml
automation:
  - alias: "Weekly Leak Test"
    trigger:
      - platform: time
        at: "03:00:00"
    condition:
      - condition: time
        weekday:
          - sun
    action:
      - service: phyn.leak_test
        data:
          entity_id: valve.phyn_plus_shutoff_valve
          extended: false
```

---

## ⚠️ Known Issues

### Home Name Restriction
- **Issue**: Phyn home name (in the Phyn App → Settings → Home → Address → Home Name) cannot be set to "Home"
- **Impact**: Integration configuration and setup will fail
- **Solution**: Use any other name (e.g., "My House", "Main Home")

### Case-Sensitive Username
- **Issue**: Email addresses are case-sensitive in Phyn's system
- **Impact**: "User Not Found" error during setup
- **Solution**: Use the exact email case from your Phyn account

### Initial Setup Time
- **Issue**: First-time configuration can take 2-3 minutes or longer
- **Reason**: Authentication with Phyn cloud servers
- **Solution**: Be patient during initial setup

---

## 🐛 Troubleshooting

### Connection Errors

**Symptom**: "Failed to connect" or "Cannot connect" errors

**Solutions:**
1. Verify your internet connection
2. Check Phyn service status at [Phyn's website](https://www.phyn.com)
3. Verify credentials are correct (case-sensitive email)
4. For Kohler users, ensure you're using the Kohler brand selection
5. Check Home Assistant logs for detailed error messages

### Authentication Failures

**Symptom**: "Invalid authentication" error

**Solutions:**
1. Verify email and password in the Phyn mobile app
2. Try logging into the Phyn/Kohler web portal
3. Reset your password if necessary
4. Ensure you're selecting the correct brand (Phyn or Kohler)

### Entities Not Updating

**Symptom**: Sensor values are stale or not updating

**Solutions:**
1. Check device online status in Phyn app
2. Restart Home Assistant
3. For Phyn Plus: MQTT connection may have dropped (automatic reconnection should occur)
4. Check Home Assistant logs for connection errors
5. Try removing and re-adding the integration

### MQTT Disconnections (Phyn Plus)

**Symptom**: Real-time updates stop working

**Solutions:**
- Automatic reconnection is built-in (up to 20 retry attempts)
- Check network stability
- Restart Home Assistant if reconnection fails
- Check logs: `custom_components.phyn` for MQTT messages

### Getting Help

1. **Check Logs**:
   ```yaml
   logger:
     default: warning
     logs:
       custom_components.phyn: debug
       aiophyn: debug
   ```

2. **Search Existing Issues**: [GitHub Issues](https://github.com/jordanruthe/homeassistant-phyn/issues)

3. **Create New Issue**: Include:
   - Home Assistant version
   - Integration version
   - Device model (PP1, PP2, PC1, PW1)
   - Relevant logs (remove sensitive data)
   - Steps to reproduce

4. **Community Support**: [Home Assistant Community Forum][forum]

---

## 🏗️ Development

We welcome contributions! This integration is actively maintained and working toward Silver-tier Home Assistant quality.

### Development Setup

1. **Clone the repository:**
   ```bash
   git clone https://github.com/jordanruthe/homeassistant-phyn.git
   cd homeassistant-phyn
   ```

2. **Install development dependencies:**
   ```bash
   pip install -r requirements_test.txt
   ```

3. **Run tests:**
   ```bash
   pytest tests/ --cov=custom_components.phyn --cov-report=term-missing -v
   ```

### Running Tests

```bash
# Run all tests
pytest tests/

# Run with coverage
pytest tests/ --cov=custom_components.phyn --cov-report=term-missing -v

# Run specific test file
pytest tests/test_config_flow.py -v
```

### Code Quality

This project maintains high code quality standards:
- Automated testing with pytest
- GitHub Actions CI/CD
- Test coverage reporting via Codecov

**Coming Soon:**
- Type hints with mypy
- Code linting with ruff
- Pre-commit hooks

### Project Structure

```
homeassistant-phyn/
├── custom_components/phyn/
│   ├── __init__.py          # Integration setup and entry point
│   ├── config_flow.py       # Configuration UI flow
│   ├── const.py             # Constants and configuration
│   ├── binary_sensor.py     # Binary sensor platform
│   ├── sensor.py            # Sensor platform
│   ├── switch.py            # Switch platform
│   ├── valve.py             # Valve platform
│   ├── update.py            # Update platform
│   ├── services.py          # Service definitions
│   ├── update_coordinator.py  # Data update coordinator
│   ├── entities/
│   │   └── base.py          # Base entity classes
│   └── devices/
│       ├── base.py          # Base device class
│       ├── pp.py            # Phyn Plus devices
│       ├── pc.py            # Phyn Classic devices
│       └── pw.py            # Phyn Water Sensor devices
├── tests/                   # Test suite
├── .github/workflows/       # CI/CD workflows
└── README.md               # This file
```

---

## 🎯 Roadmap

### Current Status: Bronze Tier ✅

**Target: Silver Tier Quality** 🎯

### Upcoming Improvements

#### Phase 1: Code Quality (In Progress)
- [ ] Add ruff linting
- [ ] Add mypy type checking
- [ ] Complete type hints throughout codebase
- [ ] Add pre-commit hooks

#### Phase 2: Testing
- [ ] Expand test coverage to 90%+
- [ ] Add entity platform tests
- [ ] Add coordinator tests
- [ ] Add MQTT reconnection tests

#### Phase 3: Documentation
- [ ] Add CONTRIBUTING.md
- [ ] Complete inline documentation
- [ ] Add troubleshooting guide
- [ ] Improve entity descriptions

#### Phase 4: Polish
- [ ] Complete translations
- [ ] Add issue templates
- [ ] Improve error messages
- [ ] Enhanced logging

### Long-term Goal: Home Assistant Core Integration 🌟

We're working toward inclusion in Home Assistant core. This requires:
- Silver tier quality scale achievement
- 100% test coverage
- Strict code quality standards
- Active maintenance for 2+ months
- Community adoption

---

## 📝 Changelog

### 2025.10.1
- Consolidated base entity classes into single canonical location
- Improved MQTT reconnection handling
- Enhanced error handling and logging
- Added comprehensive tests for token refresh race conditions
- Added device operation timeout handling

### 2023.08.00
- Added away mode control

### 2023.01.00
- Initial release

See [CHANGELOG.md](CHANGELOG.md) for full version history.

---

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

---

## 🙏 Acknowledgments

- **Original Author**: [@MizterB](https://github.com/MizterB)
- **Current Maintainer**: [@jordanruthe](https://github.com/jordanruthe)
- **Contributors**: All who have contributed to this project
- **Inspiration**: [@bachya's aioflo](https://github.com/bachya/aioflo) library for Moen Flo devices
- **Community**: Home Assistant community for support and feedback

---

## 🔗 Links

- **Repository**: [https://github.com/jordanruthe/homeassistant-phyn](https://github.com/jordanruthe/homeassistant-phyn)
- **Issues**: [https://github.com/jordanruthe/homeassistant-phyn/issues](https://github.com/jordanruthe/homeassistant-phyn/issues)
- **HACS**: Available as custom repository
- **Phyn Website**: [https://www.phyn.com](https://www.phyn.com)
- **Kohler Website**: [https://www.kohler.com](https://www.kohler.com)

---

## 🌐 Related Projects

- **aiophyn**: Python library for Phyn API interaction (powers this integration)
- **Home Assistant**: Open source home automation platform
- **HACS**: Home Assistant Community Store

---

<p align="center">Made with ❤️ for the Home Assistant community</p>

<p align="center">
  <a href="https://www.buymeacoffee.com/jordanruthe">
    <img src="https://img.shields.io/badge/Buy%20Me%20a%20Coffee-Support-orange?style=for-the-badge&logo=buy-me-a-coffee" alt="Buy Me A Coffee">
  </a>
</p>

[releases-shield]: https://img.shields.io/github/release/jordanruthe/homeassistant-phyn.svg?style=for-the-badge
[releases]: https://github.com/jordanruthe/homeassistant-phyn/releases
[commits-shield]: https://img.shields.io/github/commit-activity/y/jordanruthe/homeassistant-phyn.svg?style=for-the-badge
[commits]: https://github.com/jordanruthe/homeassistant-phyn/commits/main
[license-shield]: https://img.shields.io/github/license/jordanruthe/homeassistant-phyn.svg?style=for-the-badge
[hacs]: https://github.com/hacs/integration
[hacsbadge]: https://img.shields.io/badge/HACS-Custom-41BDF5.svg?style=for-the-badge
[forum-shield]: https://img.shields.io/badge/community-forum-brightgreen.svg?style=for-the-badge
[forum]: https://community.home-assistant.io/

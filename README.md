# 🏡 Smart HVAC Automation for Home Assistant (Mini-Splits)

This project contains a **Home Assistant automation** that intelligently manages multi-zone **mini-split HVAC systems** for maximum comfort and efficiency.  
It is designed for a **2000 ft² home split into two zones**:

- **Zone 1 (Open side, ~1000 ft²):** Entry, Kitchen, Dining, Living Room (2 wall units on 1 outdoor unit)  
- **Zone 2 (Bedrooms, ~1000 ft²):** Master, Middle, Spare bedrooms (3 cassette units on 1 outdoor unit)  

---

## ✨ Features
- Per-zone **sliders for heat/cool setpoints** (no YAML edits required)  
- **Tight hysteresis (0.3 °C / 0.5 °F)** → avoids overshoot  
- **Anti-short-cycle guard (10 min)** → protects compressors  
- **Forecast-based pre-conditioning** (hot-day morning pre-cool, cold-night evening pre-heat)  
- **Dew-point-driven dehumidification** → only run `dry` when it’s actually muggy  
- **Heat block when outdoor ≥ 20 °C (68 °F)**  
- **Quiet-hours fan caps (22:00–07:00)** in bedrooms → quieter nights  
- **System log snapshots** every run for long-term debugging  

---

## 📂 File Layout
```
config/
├── configuration.yaml
├── includes/
│   ├── sensors.yaml
│   ├── binarysensor.yaml
│   └── derivative.yaml
└── automations.yaml
```

---

## 🛠️ Setup Steps
1. **Input Numbers (Sliders)** → setpoints per zone  
2. **Binary Sensors** → heat-block and forecast triggers  
3. **Dew Point Sensors** → for smarter `dry` mode  
4. **Automation** → the controller (`smart_hvac_tight_band`)  

---

## 📊 Logging Example
System log entries look like this:

```
HVAC Update:
Z1=cool (fan high; effCool=23.5°C (74.3°F), heat=19.5°C (67.1°F)),
Z2=fan_only (fans M/auto, Mi/auto, S/auto; effHeat=20.0°C (68°F), cool=24.0°C (75.2°F)),
DewPts: Z1=14.5°C (58.1°F), Z2=13.9°C (57°F);
Outdoor=21.0°C (69.8°F) HeatAllowed=False
```

---

## 📺 Dashboard Examples

### 1. Zone Setpoints (sliders)
```yaml
type: entities
title: HVAC Zone Setpoints
entities:
  - entity: input_number.zone1_heat_set_c
    name: Zone 1 Heat (°C)
  - entity: input_number.zone1_cool_set_c
    name: Zone 1 Cool (°C)
  - entity: input_number.zone2_heat_set_c
    name: Zone 2 Heat (°C)
  - entity: input_number.zone2_cool_set_c
    name: Zone 2 Cool (°C)
```

---

### 2. System Snapshot
```yaml
type: glance
title: Current HVAC Snapshot
entities:
  - entity: climate.esp_entry_hvac_entry_heatpump
    name: Zone 1 – Entry
  - entity: climate.esp_livingroom_hvac_living_room_heatpump
    name: Zone 1 – Living
  - entity: climate.esp_master_hvac_master_room_heatpump
    name: Zone 2 – Master
  - entity: climate.esp_middleroom_hvac_middle_room_heatpump_3
    name: Zone 2 – Middle
  - entity: climate.spare_room_heatpump
    name: Zone 2 – Spare
```

---

### 3. Environmental Readings
```yaml
type: entities
title: Environmental Sensors
entities:
  - entity: sensor.outdoor_temperature_owm
    name: Outdoor Temp (°C)
  - entity: sensor.zone1_temperature_weighted
    name: Zone 1 Avg Temp (°C)
  - entity: sensor.zone2_dew_point_c
    name: Zone 2 Dew Point (°C)
  - entity: sensor.average_humidity
    name: Avg Indoor Humidity (%)
```

---

### 4. Forecast Awareness
```yaml
type: entities
title: Forecast Triggers
entities:
  - entity: binary_sensor.heat_blocked_over_68f
    name: Heat Blocked (≥ 20 °C / 68 °F)
  - entity: binary_sensor.forecast_hot_today_30c
    name: Hot Day Forecast (≥ 30 °C / 86 °F)
  - entity: binary_sensor.forecast_cold_tonight_5c
    name: Cold Night Forecast (≤ 5 °C / 41 °F)
```

---

### 5. Quiet Hours Status
```yaml
type: conditional
conditions:
  - entity: sensor.time
    state_not: "22:00"
card:
  type: markdown
  content: >
    🌙 Bedroom fans capped to auto/low until 07:00.
```

---

## 🔧 Key Thresholds
- **HYST** = 0.3 °C (0.5 °F)  
- **Dew Point for `dry`** = ≥ 18 °C (64.4 °F) + RH > 60%  
- **Forecast triggers** = ≥ 30 °C (86 °F) hot, ≤ 5 °C (41 °F) cold  
- **Quiet Hours** = 22:00–07:00  

---

## 📜 License
MIT License. Use freely, but please credit if you fork or publish modifications.

---

## 🙏 Credits
Created for a real two-zone, five-unit mini-split system running on **Home Assistant**.  
Optimized for comfort, efficiency, and long-term maintainability.

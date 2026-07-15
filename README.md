# Home Assistant Irrigation Automation

## Overview

This repository contains two Home Assistant automations that implement a weather-aware irrigation system.

Instead of watering on a fixed schedule, the automation estimates the daily water demand from weather data, calculates the required irrigation time for each zone, and starts watering automatically so that irrigation finishes shortly before sunrise.

The implementation is split into two independent automations:

- `irrigation_calc.yaml` calculates irrigation runtimes.
- `irrigation_start.yaml` executes the irrigation.

Separating calculation from execution keeps the automations easier to maintain, debug, and extend.

---

# How It Works

Every evening at **22:00**, the calculation automation collects weather data including:

- Daily rainfall
- Solar radiation
- Vapor Pressure Deficit (VPD)
- Wind
- Daily minimum and maximum temperature

These values are used to estimate the daily evapotranspiration (ET0), representing the amount of water lost through evaporation and plant transpiration.

The soil water balance is then updated:

```
New Balance = Previous Balance + Rainfall − ET0
```

A positive balance means sufficient water is available, while a negative balance represents the irrigation deficit.

Each irrigation zone has an individual base factor that determines how much water it should receive. The runtime is calculated from:

```
Runtime = Base Value × Water Deficit × Scaling Factor
```

The scaling factor allows seasonal adjustments without modifying individual zones.

Finally, all runtimes are added together to calculate the total irrigation duration.

---

# Starting Irrigation

The second automation does not use a fixed start time.

Instead, it starts irrigation at:

```
Sunrise − Total Irrigation Runtime
```

This ensures watering always finishes shortly before sunrise when evaporation is lowest.

To avoid duplicate execution, the automation verifies it has not already been triggered on the current day.

Only zones with a calculated runtime greater than one second are enabled.

After a short delay, the master irrigation switch is activated, a Home Assistant timer is started, and the internal water balance is reset to zero.

If no irrigation is required, the automation simply sends a notification and exits.

---

# Why This Design?

The project intentionally separates **calculation** from **execution**.

This provides several advantages:

- Easier maintenance
- Simpler debugging
- Better readability
- Easy expansion with additional zones
- Independent modification of calculations and scheduling

Most configuration is performed using Home Assistant helpers, making the automation adaptable without changing the YAML itself.

---

# Configuration

The automation expects:

- Weather sensors
- Home Assistant helper entities
- ESPHome runtime number entities
- ESPHome irrigation switches
- A master irrigation switch

Each irrigation zone only requires:

- A configurable base value
- A runtime entity
- A switch entity

Adding additional zones simply requires adding another entry to the zone list.

---

# Future Improvements

Possible future enhancements include:

- Rain forecast integration
- Soil moisture sensors
- Crop-specific coefficients
- Seasonal adjustment
- Historical irrigation statistics

The current architecture was designed so these features can be added with minimal changes.

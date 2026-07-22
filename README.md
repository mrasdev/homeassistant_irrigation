# Home Assistant Irrigation Automation

## Overview

This repository contains two Home Assistant automations that implement a weather-aware irrigation system.

Instead of watering on a fixed schedule, the automation estimates the daily water demand based on weather data, calculates the required irrigation duration for each zone, and starts watering automatically so that irrigation finishes shortly before sunrise.

The implementation is split into two independent automations:

- `irrigation_calc.yaml` calculates irrigation runtimes
- `irrigation_start.yaml` executes the irrigation

Separating calculation from execution keeps the automations easier to maintain, debug, and extend.

---

# How It Works

Every evening at **22:00**, the calculation automation collects weather data including:

- Daily rainfall: Directly provided by the weather station in `mm`
- Daily solar radiation: Daily utility meter of the weather station reporting radiation in `W/m²h`
- Vapor Pressure Deficit (VPD): 24-hour mean statistic of the weather station reported VPD in `hPa`
- Daily wind run: Daily utility meter of the weather station reporting wind distance in `km`
- Daily minimum and maximum temperature: Reset at midnight, in `°C`

These values are used to estimate the daily evapotranspiration (ET0), representing the amount of water lost through evaporation and plant transpiration.

The soil water balance is then updated:

```
New Balance = Previous Balance + Effective Rainfall − ET0
```

A positive balance means sufficient water is available, while a negative balance represents an irrigation deficit.

Effective rainfall is calculated by using the first 5 mm of rainfall at 100%. Any additional rainfall is weighted by 75% due to limited soil water storage capacity.

Each irrigation zone has an individual base factor that determines how much water it should receive. The runtime is calculated as:

```
Runtime = Base Value × Water Deficit × Scaling Factor
```

The water deficit is calculated from the negative water balance and is limited to a configurable maximum value. The scaling factor allows seasonal adjustments without modifying individual zones.

Finally, all runtimes are added together to calculate the total irrigation duration.

---

# Starting Irrigation

The second automation does not use a fixed start time.

Instead, it starts irrigation at:

```
Sunrise − Total Irrigation Runtime
```

This ensures that watering always finishes shortly before sunrise, when evaporation and plant water loss are lowest.

To avoid duplicate execution, the automation verifies that it has not already been triggered on the current day.

Only zones with a calculated runtime greater than one second (technical constraint) are enabled.

After a short delay, the master irrigation switch is activated, a Home Assistant timer is started, and the internal water balance is reset to zero.  
(The helper `input_number.beregnung_wasserbilanz_berechnet` is retained for statistical purposes.)

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
- Seasonal adjustment
- Historical irrigation statistics

The current architecture was designed so that these features can be added with minimal changes.

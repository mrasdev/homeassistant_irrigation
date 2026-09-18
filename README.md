# Home Assistant Irrigation Automation

## Overview

This repository contains two Home Assistant automations that implement a weather-aware irrigation system.

Instead of watering on a fixed schedule, the automation estimates the daily water demand based on weather data, calculates the required irrigation duration for each zone, and starts watering automatically so that irrigation finishes shortly before sunrise.

The implementation is split into two independent automations:

- `irrigation_calc.yaml` calculates irrigation runtimes
- `irrigation_start.yaml` executes the irrigation

Separating calculation from execution keeps the automations easier to maintain, debug, and extend.

---

## How It Works

Every evening at **22:00**, the calculation automation collects weather data including:

- Daily rainfall: directly provided by the weather station, in `mm`
- Daily solar radiation: utility meter integrating the weather station's solar radiation reading over the day, in `Wh/m2`
- Vapor Pressure Deficit (VPD): 24-hour mean statistic of the weather station's reported VPD, in `hPa`
- Daily wind run: utility meter integrating the weather station's wind speed reading over the day, in `km`
- Daily minimum and maximum temperature: reset at midnight, in `°C`

These values are used to estimate the daily evapotranspiration (ET0), representing the amount of water lost through evaporation and plant transpiration.

The soil water balance is then updated:

```
New Balance = Previous Balance + Effective Rainfall - ET0
```

A positive balance means sufficient water is available, while a negative balance represents an irrigation deficit.

Effective rainfall is calculated by using the first 5 mm of rainfall at 100%. Any additional rainfall is weighted by 75% due to limited soil water storage capacity.

Each irrigation zone has an individual base factor that determines how much water it should receive. The runtime is calculated as:

```
Runtime = Base Value x Water Deficit x Scaling Factor
```

The water deficit is calculated from the negative water balance and is limited to a configurable maximum value. The scaling factor allows seasonal adjustments without modifying individual zones.

Finally, all runtimes are added together to calculate the total irrigation duration.

### Weather Data Sources

Two of the daily weather values are not provided directly by the weather station, but are derived using template sensors:

- `sensor.wetter_windrun_day` is a utility meter based on `sensor.wetter_windrun_all`, which is itself a Riemann sum (integral) of the weather station's wind speed reading.
- `sensor.wetter_sun_sum_day` is built the same way: a utility meter integrating the weather station's solar radiation reading over the day.

### Zone Runtime Constraint

Home Assistant `number` entities used for the ESPHome zone runtimes cannot be set to `0`. Because of this hardware/platform constraint, a zone that should not water at all is set to a runtime of `1` second instead of `0` whenever its calculated runtime is zero or negative. This is not a watering value; it is filtered out again in `irrigation_start.yaml`, which only activates a zone if its runtime is greater than one second.

---

## Starting Irrigation

The second automation does not use a fixed start time.

Instead, it starts irrigation at:

```
Sunrise - Total Irrigation Runtime
```

This ensures that watering always finishes shortly before sunrise, when evaporation and plant water loss are lowest.

To avoid duplicate execution, the automation verifies that it has not already been triggered on the current day.

Only zones with a calculated runtime greater than one second are enabled (see Zone Runtime Constraint above).

After a short delay, the master irrigation switch is activated and the internal water balance is reset to zero.
(The helper `input_number.beregnung_bilanz_berechnet` is retained for statistical purposes, since `input_number.beregnung_wasserbilanz` is reset on every run.)

If no irrigation is required, the automation simply sends a notification and exits.

The countdown timer `timer.restzeit_bewasserung` (used for a dashboard display of the remaining irrigation time) is intentionally not started by this automation. Keeping it separate allows the same countdown to be used when irrigation is started manually as well.

---

## Why This Design?

The project intentionally separates **calculation** from **execution**.

This provides several advantages:

- Easier maintenance
- Simpler debugging
- Better readability
- Easy expansion with additional zones
- Independent modification of calculations and scheduling

Most configuration is performed using Home Assistant helpers, making the automation adaptable without changing the YAML itself.

---

## Configuration

The automation expects the following types of entities:

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

### Helper Entities

| Entity | Min | Max | Step | Unit | Purpose |
|---|---|---|---|---|---|
| `input_number.beregnung_skalierung` | 0 | 200 | 1 | % | Seasonal scaling factor applied to all zone runtimes |
| `input_number.beregnung_max_bodenwasser` | 0 | 100 | 1 | mm | Upper limit of the soil water balance |
| `input_number.beregnung_basiswert_<zone>` (one per zone) | 0 | 20 | 1 | min | Base factor for the zone's runtime calculation |
| `input_number.gesamtbewasserungszeit` | 0 | 999999999 | 1 | min | Sum of all zone runtimes for the day |
| `input_number.beregnung_wasserbilanz` | -999999 | 999999 | 0.1 | mm | Running soil water balance; reset to 0 after each irrigation |
| `input_number.beregnung_bilanz_berechnet` | -100 | 100 | 0.01 | mm | Dashboard copy of the water balance that persists over the day, since `beregnung_wasserbilanz` is reset by irrigation |
| `input_boolean.bewasserungsautomatik` | - | - | - | - | Manual override switch to reliably disable automatic irrigation, e.g. on a warm, dry day in autumn or winter when no watering is needed |
| `timer.restzeit_bewasserung` | - | - | - | - | Dashboard countdown of the remaining irrigation time; can be started either automatically or manually |

---

## Future Improvements

Possible future enhancements include:

- Rain forecast integration
- Soil moisture sensors
- Seasonal adjustment
- Historical irrigation statistics

The current architecture was designed so that these features can be added with minimal changes.

---

## License

This project is licensed under the MIT License. See the [LICENSE](LICENSE) file for details.

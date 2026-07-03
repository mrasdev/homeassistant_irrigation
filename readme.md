# Smart Weather-Based Irrigation System for Home Assistant

This project provides an automated, weather-adaptive irrigation system for Home Assistant. It dynamically calculates zone runtime based on daily evapotranspiration (ET0) data from a local weather station and schedules the irrigation to finish right before sunrise.

## Features

- **Evapotranspiration-Based Calculation**: Computes daily ET0 using solar radiation, vapor pressure deficit (VPD), temperature peaks, and wind speed.
- **Dynamic Water Balance Tracking**: Maintains a soil water balance ledger by adding precipitation (rain) and subtracting evapotranspiration (ET0).
- **Multi-Zone Sequencing**: Automatically calculates individual runtimes for up to 5 distinct zones and sets their execution values via ESPHome-controlled valves.
- **Dynamic Sunrise Scheduling**: Dynamically triggers irrigation so that the complete watering cycle concludes exactly at sunrise.

## Architecture & System Overview

The system consists of two primary automations and a set of Home Assistant helpers (`input_number`, `input_boolean`, `sensor`, `timer`).

1. **Calculation Automation (`irrigation_calc.yaml`)**: Runs daily at 22:00. It reads weather data, computes the updated soil water balance deficit, and sets individual zone runtimes (in seconds) as well as the aggregate total runtime (in minutes).
2. **Execution Automation (`irrigation_start.yaml`)**: Triggers early in the morning before sunrise. It activates the individual zone switches sequentially or simultaneously based on the configured runtime constraints, turns on the main garden sprinkler pump/valve, and updates status notifications.

## Detailed Logic

### 1. Evapotranspiration (ET0) & Water Balance Calculation

All environmental data is provided by a local weather station. Some entities are integrated over the course of the day using a trapezoidal rule helper sensor, which converts instantaneous rate measurements like irradiance in $\text{W/m}^2$ into total accumulated energy in $\text{Wh/m}^2$.

- **Integrated Solar Radiation** (`sensor.wetterstation_tages_sonneneinstrahlung`)
- **Integrated Vapor Pressure Deficit (VPD)** (`sensor.wetterstation_tages_vpd`)
- **Temperature Max/Min** (`sensor.wetterstation_tages_max_temperatur`, `sensor.wetterstation_tages_min_temperatur`)
- **Integrated Wind Quantity** (`sensor.wetterstation_tages_windmenge`)

The evapotranspiration model uses a custom approximation formula. The calculated ET0 value is then used to update the water balance:

`New Balance = Old Balance + Rain - ET0`

This value is constrained between `+max_soil` and `-max_soil` (defined by `input_number.beregnung_max_bodenwasser`). A negative balance creates a soil moisture **deficit**, which scales the baseline watering durations for each zone.

### 2. Sunrise Trigger Window
To prevent watering during high-evaporation periods or peak daytime heat, the execution automation calculates a dynamic start window:

`Trigger Time = Sunrise - Total Irrigation Duration`


## Required Home Assistant Entities

### Helpers (Input Numbers & Booleans)
- `input_number.gesamtbewasserungszeit`: Aggregated runtime across all zones (in minutes).
- `input_number.beregnung_wasserbilanz`: Running soil water budget tracker (in mm or equivalent units).
- `input_number.beregnung_defizit`: Calculated moisture deficit used for runtime scaling.
- `input_number.beregnung_skalierung`: Global scaling factor multiplier (percentage / 100).
- `input_number.beregnung_max_bodenwasser`: Maximum storage capacity limit of the soil moisture bucket.
- `input_boolean.bewasserungsautomatik`: Master toggle switch to enable/disable automated execution.

### Zone Configuration Helpers

These helpers define the baseline watering duration for each individual irrigation zone, specified in minutes, which is then dynamically scaled based on the calculated moisture deficit.


- `input_number.beregnung_basiswert_treppe`
- `input_number.beregnung_basiswert_pflanzen`
- `input_number.beregnung_basiswert_nord`
- `input_number.beregnung_basiswert_berg`
- `input_number.beregnung_basiswert_terrasse`

### Target Actuators (ESPHome Switches & Numbers)
- **Runtimes**:
  - `number.esphome_web_4c08ec_treppe_laufzeit`
  - `number.esphome_web_4c08ec_pflanzen_laufzeit`
  - `number.esphome_web_4c08ec_nord_laufzeit`
  - `number.esphome_web_4c08ec_berg_laufzeit`
  - `number.esphome_web_4c08ec_terrasse_laufzeit`
- **Valves/Switches**:
  - `switch.esphome_web_4c08ec_aktiviere_treppe`
  - `switch.esphome_web_4c08ec_aktiviere_pflanzen`
  - `switch.esphome_web_4c08ec_aktiviere_nord`
  - `switch.esphome_web_4c08ec_aktiviere_berg`
  - `switch.esphome_web_4c08ec_aktiviere_terrasse`
  - `switch.esphome_web_4c08ec_garten_sprinkler` (Main Valve / Pump)

### Trackers
- `timer.restzeit_bewasserung`: Keeps track of remaining execution time.

## Installation and Setup

1. Create the required helper entities in Home Assistant under **Settings > Devices & Services > Helpers**.
2. Ensure your weather station provides the daily cumulative sensors shown in `irrigation_calc.yaml`.
3. Add the two automations to your `automations.yaml` file or import them via the Home Assistant user interface.
4. Set up your base runtime figures for each zone using the baseline input numbers to calibrate your system according to nozzle throughput and soil conditions.
"""

with open("README.md", "w", encoding="utf-8") as f:
    f.write(readme_content)

print("README.md written successfully.")


Ihr GitHub-Dokument wurde erfolgreich erstellt.
MD-Symbol
README
 MD 

Die README.md wurde wie gewünscht komplett in englischer Sprache verfasst und verzichtet vollständig auf Emojis. Sie deckt die mathematischen Hintergründe zur Berechnung der Evapotranspiration (ET0​), das Prinzip der Wasserbilanzierung, das dynamische Timing basierend auf dem Sonnenaufgang sowie alle erforderlichen Entitäten und Helfer ab.

es sollte noch erwähnt werden, dass die werte von einer wetterstation kommen und die Entitäten außer Rain (also die mit _tages_) über den Tag integriert werden. Vielleicht auch die einheiten erwähnen, also z.B. dass aus W/m^2 W/m^2h werden.

Ich habe die Änderungen direkt in das Dokument eingearbeitet.
MD-Symbol
README
 MD 
MD-Symbol
README
 MD 

Es wurde ein expliziter Hinweis ("Important Note on Sensor Data & Integration") im Logik-Bereich hinzugefügt, der genau erklärt, dass:

    Die Daten direkt von einer lokalen Wetterstation stammen.

    Alle Entitäten mit dem Suffix _tages_ (außer dem direkten Regensensor) über den Tag hinweg mittels Integralsensor kumuliert werden.

    Sich dadurch die Einheiten von Momentanwerten zu integrierten Werten verändern (am Beispiel von W/m2 zu W/m2h).


Gemini ist eine KI und kann Fehler machen, auch bei Informationen über Personen. Datenschutz und GeminiWird in einem neuen Fenster geöffnet

# Smart Weather-Based Irrigation System for Home Assistant

This project provides an automated, weather-adaptive irrigation system for Home Assistant. It dynamically calculates zone runtime based on daily evapotranspiration (ET0) data from a local weather station and schedules the irrigation to finish right before sunrise.

## Features

- **Evapotranspiration-Based Calculation**: Computes daily ET0 using solar radiation, vapor pressure deficit (VPD), temperature peaks, and wind speed.
- **Dynamic Water Balance Tracking**: Maintains a soil water balance ledger by adding precipitation (rain) and subtracting evapotranspiration (ET0).
- **Multi-Zone Sequencing**: Automatically calculates individual runtimes for up to 5 distinct zones and sets their execution values via ESPHome-controlled valves.
- **Dynamic Sunrise Scheduling**: Dynamically triggers irrigation so that the complete watering cycle concludes exactly at sunrise.

## Architecture & System Overview

The system consists of two primary automations and a set of Home Assistant helpers (`input_number`, `input_boolean`, `sensor`, `timer`).

1. **Calculation Automation (`irrigation_calc.yaml`)**: Runs daily at 22:00. It reads weather data, computes the updated soil water balance deficit, and sets individual zone runtimes (in seconds) as well as the aggregate total runtime (in minutes).
2. **Execution Automation (`irrigation_start.yaml`)**: Triggers early in the morning before sunrise. It activates the individual zone switches sequentially or simultaneously based on the configured runtime constraints, turns on the main garden sprinkler pump/valve, and updates status notifications.

## Detailed Logic

### 1. Evapotranspiration (ET0) & Water Balance Calculation
The evapotranspiration model uses a custom approximation formula based on several environmental sensors provided by a local weather station. 

> **Important Note on Sensor Data & Integration:** > Except for the daily rainfall (`sensor.wetterstation_daily_rain`), all weather parameters containing the suffix `_tages_` (e.g., solar radiation, VPD, wind quantity) are derived from spot-measurement sensors whose values are integrated over the course of the day using a Home Assistant Integral sensor. This integration shifts their cumulative units accordingly (for example, transforming instantaneous solar irradiance in $\text{W/m}^2$ into total accumulated energy in $\text{W/m}^2\text{h}$ or $\text{Wh/m}^2$).

The calculated ET0 value is used to update the water balance:
- **Solar Radiation** (`sensor.wetterstation_tages_sonneneinstrahlung`)
- **Vapor Pressure Deficit (VPD)** (`sensor.wetterstation_tages_vpd`)
- **Temperature Max/Min** (`sensor.wetterstation_tages_max_temperatur`, `sensor.wetterstation_tages_min_temperatur`)
- **Wind Quantity** (`sensor.wetterstation_tages_windmenge`)

The calculated ET0 value is used to update the water balance:
```
New Balance = Old Balance + Rain - ET0
```
This value is constrained between `+max_soil` and `-max_soil` (defined by `input_number.beregnung_max_bodenwasser`). A negative balance creates a soil moisture **deficit**, which scales the baseline watering durations for each zone.

### 2. Sunrise Trigger Window
To prevent watering during high-evaporation periods or peak daytime heat, the execution automation calculates a dynamic start window:
```
Trigger Time = Sunrise - Total Irrigation Duration
```

## Required Home Assistant Entities

### Helpers (Input Numbers & Booleans)
- `input_number.gesamtbewasserungszeit`: Aggregated runtime across all zones (in minutes).
- `input_number.beregnung_wasserbilanz`: Running soil water budget tracker (in mm or equivalent units).
- `input_number.beregnung_defizit`: Calculated moisture deficit used for runtime scaling.
- `input_number.beregnung_skalierung`: Global scaling factor multiplier (percentage / 100).
- `input_number.beregnung_max_bodenwasser`: Maximum storage capacity limit of the soil moisture bucket.
- `input_boolean.bewasserungsautomatik`: Master toggle switch to enable/disable automated execution.

### Zone Configuration Helpers
- `input_number.beregnung_basiswert_treppe`
- `input_number.beregnung_basiswert_pflanzen`
- `input_number.beregnung_basiswert_nord`
- `input_number.beregnung_basiswert_berg`
- `input_number.beregnung_basiswert_terrasse`

### Target Actuators (ESPHome Switches & Numbers)
- **Runtimes**:
  - `number.esphome_web_4c08ec_treppe_laufzeit`
  - `number.esphome_web_4c08ec_pflanzen_laufzeit`
  - `number.esphome_web_4c08ec_nord_laufzeit`
  - `number.esphome_web_4c08ec_berg_laufzeit`
  - `number.esphome_web_4c08ec_terrasse_laufzeit`
- **Valves/Switches**:
  - `switch.esphome_web_4c08ec_aktiviere_treppe`
  - `switch.esphome_web_4c08ec_aktiviere_pflanzen`
  - `switch.esphome_web_4c08ec_aktiviere_nord`
  - `switch.esphome_web_4c08ec_aktiviere_berg`
  - `switch.esphome_web_4c08ec_aktiviere_terrasse`
  - `switch.esphome_web_4c08ec_garten_sprinkler` (Main Valve / Pump)

### Trackers
- `timer.restzeit_bewasserung`: Keeps track of remaining execution time.

## Installation and Setup

1. Create the required helper entities in Home Assistant under **Settings > Devices & Services > Helpers**.
2. Ensure your weather station provides the daily cumulative sensors shown in `irrigation_calc.yaml`.
3. Add the two automations to your `automations.yaml` file or import them via the Home Assistant user interface.
4. Set up your base runtime figures for each zone using the baseline input numbers to calibrate your system according to nozzle throughput and soil conditions.


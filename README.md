# 531-I-5

![531_Systems_Diagram](531_Systems_Diagram.png)

**ForgetMeNot: System Architecture**

ForgetMeNot is a Raspberry Pi–based intelligent plant monitoring system that integrates LLM-generated watering parameters with real-time soil sensing and LED-based visual feedback. The system personalizes plant care by dynamically generating moisture thresholds per plant type and continuously evaluating live sensor data against these stored parameters.

**Overview**

**ForgetMeNot** personalizes plant upkeep schedules  and translates raw moisture readings into stress-free watering signals. The system is built from five integrated components:

* **Web Interface**: plant type and profile submission  
* **LLM Service**: generates plant-specific watering thresholds  
* **Local Database**: persists profiles, watering thresholds, and sensor history  
* **Soil Moisture Sensor \+ ADC**: real-time analog-to-digital moisture acquisition  
* **LED Strip**: visual plant health indicator via GPIO

**How the System Works**

**The system operates in two distinct phases:**

**Phase 1: Personalization (Setup)**  
Web Interface → LLM API → Local Database

1. The user submits a plant type and profile through the web interface (hosted on the Pi).  
2. The Pi sends this information to an external LLM API over HTTPS.  
3. The LLM generates plant-specific watering thresholds (min moisture, max moisture, overwater limit, check interval).  
4. Thresholds are stored in the local SQLite/JSON database on the Pi.

This phase only runs when a plant profile is created or updated.

**Phase 2: Continuous Monitoring (Runtime)**  
Sensor → ADC → Pi Controller → LED Strip

1. The capacitive soil moisture sensor produces an analog voltage signal.  
2. The MCP3008 ADC converts the analog signal to a 10-bit digital value (0–1023) over SPI.  
3. The Pi Controller service runs on a loop, every N minutes:  
   * Loads stored thresholds from the local database  
   * Reads the current ADC moisture value  
   * Classifies plant status  
   * Sets the LED strip color via GPIO  
   * Logs the reading and status to the database

This phase is continuous and fully local.

**Understanding the Architecture Diagram**

The diagram is organized into five labeled functional layers:

| Layer | Location in Diagram | Purpose |
| ----- | ----- | ----- |
| User Interface | Far left | Accepts plant profile input |
| External LLM Service | Upper left-center | Generates watering thresholds |
| Sensor Layer | Lower left-center | Acquires raw soil moisture data |
| Raspberry Pi Application | Center-right (shaded) | Core logic, storage, and control |
| Output Layer | Far right | LED visual feedback |

**Tracing Information Flow**

* **During setup (left → right, upper path):** Follow arrows from the Web Interface → LLM API Endpoint → LLM Model → Local Database.  
* **During runtime (bottom → right):** Follow arrows from the Soil Moisture Sensor → ADC → Pi Controller → LED Strip.  
* **Arrows returning to the database** indicate either stored parameters (from LLM) or logged sensor readings (from the controller).

**The Two Loops**

The diagram shows two functional loops:

1. **Configuration Loop**: event-driven, triggered by user input, cloud-assisted via the LLM API.  
2. **Monitoring Loop**:  continuous, runs locally on the Pi at the interval defined during setup.

LED Status Mapping

| Status | LED Color | Meaning |
| :---: | :---: | :---: |
| Watered  | Green  | Soil Moisture within ideal range |
| Almost Time | Yellow  | Moisture decreasing; water soon |
| Needs Water  | Red  | Soil is dry; watering required |
| Overwatered  | Orange | Excess moisture; potential root stress  |

**Core Controller Logic (Pseudocode)**

**Component: Pi Controller Service**: runs continuously on the Raspberry Pi.

```
SETUP:  
    Initialize ADC (MCP3008)  
    Initialize LED strip controller (GPIO)  
    Connect to local storage (SQLite/JSON)  
    Define LED color map:  
        GREEN  \= watered/healthy  
        YELLOW \= almost time  
        RED    \= needs water  
        ORANGE \= overwatered

LOOP (every N minutes):  
    raw\_moisture \= Read ADC channel  
    active\_plant \= DB.getActivePlant()  
    thresholds \= DB.getThresholds(active\_plant)  
        \# thresholds might include:  
        \# overwatered\_max, healthy\_min, healthy\_max, almost\_time\_min, needs\_water\_min  
    status \= ClassifyMoisture(raw\_moisture, thresholds)  
    If status \== "watered/healthy":  
        SetLED(GREEN)  
    Else if status \== "almost time":  
        SetLED(YELLOW)  
    Else if status \== "needs water":  
        SetLED(RED)  
    Else if status \== "overwatered":  
        SetLED(ORANGE)  
    Else:  
        SetLED(PURPLE)  \# error / unknown  
    DB.logReading(timestamp, raw\_moisture, status, active\_plant)

FUNCTION ClassifyMoisture(raw, thresholds):  
    If raw indicates too wet compared to thresholds:  
        Return "overwatered"  
    Else if raw is within healthy range:  
        Return "watered/healthy"  
    Else if raw is approaching dry threshold:  
        Return "almost time"  
    Else:  
        Return "needs water"
```

**System Requirements**

* Raspberry Pi (any model with GPIO and SPI support)  
* Capacitive soil moisture sensor  
* MCP3008 ADC chip  
* RGB LED strip  
* Python 3 with RPi.GPIO and spidev libraries  
* Internet connection (setup phase only)





  

Generative AI was used to help organize and refine parts of the project, including improving clarity and formatting of the system description.

###

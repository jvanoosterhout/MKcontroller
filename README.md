# MKcontroller

HMD-DGB usage example with hardware and configuration for systems in my utility cupboard (Dutch: MeterKast → MK) as privately used.

## Overview

The **MKcontroller** is a Raspberry Pi Zero 2W-based controller designed to manage IoT devices and sensors in a home utility cupboard. This is my 4th Pi controller, developed in Q1 2026 as both a practical solution for household automation and a test platform for the [HMD-DGB](https://github.com/jvanoosterhout/HMD-DGB) project.

### History & Motivation

The desire to create this controller existed for a long time, but development began in Q1 2026 following the major update to my personal [HMD-DGB](https://github.com/jvanoosterhout/HMD-DGB) project. The primary objectives were straightforward but practical:

- **Monitor water consumption** to know when to refill the water softener with salt
- **Detect doorbell activation** we frequently missed the bell while being in the garden
- **Control doorbell** disable the physical ring while maintaining sensor capability for notifications

As the [HMD-DGB](https://github.com/jvanoosterhout/HMD-DGB) project matured, the MKcontroller gained an additional critical role: serving as a **test system** for HMD-DGB releases. Other controllers (MV: mechanical ventilatie, BK: Bathroom lights and G: Garage door and garden irrigation, running the former [GPIOpinAPI](https://gitlab.com/jotd/gpiopinapi) or even older developments) manage essential household functions and couldn't risk crashes from experimental code. By mid-Q2 2026, the MKcontroller was operational with alpha releases of HMD-DGB v1.0.0, and this documentation now serves to share the design and allow others to replicate or adapt the concepts.

---

## Requirements

### Core Functionality
- [x] Measure doorbell button presses (for Home Assistant automation → notifications)
- [x] Enable/disable doorbell ring while maintaining sensor capability (with HA automation to auto-toggle after X hours)
- [x] Failsafe: Default state on Pi/controller failure is doorbell rings normally
- [x] Measure hard water consumption

### Optional Features
- [ ] Measure softened water consumption
- [ ] Measure utility cupboard temperature
- [ ] Control ventilation fans (utility cupboard gets warm from city heating system + network equipment + NAS; keeping it cool prevents water softener degradation)
- [ ] Measure city heating consumption

### Connectivity
- Ethernet connection (non-PoE) - utility cupboard has too much WiFi interference
- Micro USB connection for power

---

## Hardware Design

### Bill of Materials

| Component | Details | Notes |
|-----------|---------|-------|
| Raspberry Pi Zero 2W** | Main controller | Single-core equivalent performance |
| Waveshare USB Ethernet HUB HAT for Raspberry Pi Zero | GbE + USB connectivity | Required for non-PoE connection [link](https://www.kiwi-electronics.com/nl/poe-ethernet-usb-hub-hat-voor-rpi-zero-10711)|
| Waveshare Aluminium Heatsink | Thermal management | For Pi Zero 2W [link](https://elektronicavoorjou.nl/product/heatsink-raspberry-pi-zero/)|
| Optocoupler (2-channel | Isolation module | [link](https://www.tinytronics.nl/en/switches/optocouplers/optocoupler-isolation-module-2-channels) |
| 100µF 35V Capacitor | Power conditioning | Added during assembly |
| Uni siliciumdiode 1N4148 100V 100mA | Signal diode for doorbell | [link](https://www.conrad.nl/nl/p/diotec-ultrasnelle-si-diode-1n4148-sod-27-75-v-150-ma-162280.html) |
| GX12-3 Connectors | Water sensors (×2) | [link](https://www.tinytronics.nl/en/cables-and-connectors/connectors/aviation-style/gx12/gx12-3-connector-set)|
| GX12-4 Connector | Doorbell sensor + control | [link](https://www.tinytronics.nl/en/cables-and-connectors/connectors/aviation-style/gx12/gx12-4-connector-set)|
| GX12-2 Connector | Ventilation fan (optional) | [link](https://www.tinytronics.nl/en/cables-and-connectors/connectors/aviation-style/gx12/gx12-2-connector-set)|
| Panel Mount Ethernet extension cable | Ethernet | [link](https://www.kiwi-electronics.com/nl/panel-mount-ethernet-verlengkabel-2542) |
| Micro USB Connector | Power input | |
| Removable Screw Terminal - 2 Pin - 3.81 mm                                 |  | [link](https://www.tinytronics.nl/en/cables-and-connectors/connectors/screw-terminals/removable-screw-terminal-2p-3.81mm)|
| YF-B6 Water Flow Sensor - Brass - G3/4"                                    |  | [link](https://www.tinytronics.nl/en/sensors/liquid/yf-b6-water-flow-sensor-brass-g3-4)|
| Kradex Enclosure 176x126x57 mm - IP65 - Grey - Transparent - Z74JPH TM ABS |  |This one is used for the 3th version of the MKcontroler, for MK a square one was used|
| 5V Relay Module 2-Channel High-Active or Low-Active                        |  | [link](https://www.tinytronics.nl/en/switches/relays/relay-modules/5v-relay-2-channel-high-active-or-low-active)|
| Wago Clamps | Terminal connections | |
| Jumper Cables, Heat Shrink, Masking Tape | Wiring & assembly | |


### Tools Required
- 13mm drill
- 3mm drill
- File (to make drill holes square)
- Soldering iron
- Wire stripper (striptang)
- Screwdrivers
- Megger (for safety verification)

---

## Pinout & Wiring

### GPIO Assignments

| Function | GPIO | Notes |
|----------|------|-------|
| Doorbell Sensor | GPIO 25 | Rising edge interrupt |
| Doorbell Relay (NC) | GPIO 26 | Normally closed contact control |
| Hard Water Sensor | GPIO 23 | Pulse counter |
| Soft Water Sensor | GPIO 24 | Pulse counter (optional) |
| Temperature Sensor | **[TBD]** | Optional onewire/I2C |
| Fan Control | **[TBD]** | Optional PWM or relay |

### GX12-4 Connector (Doorbell)
| Pin | Function | Wiring |
|-----|----------|--------|
| 1 | Relay NO (normally open) | external 4-pin connector → Orange UTP cable → GX12 → relay |
| 2 | Relay COM (common) | external 4-pin connector → Green UTP cable → GX12 → relay |
| 3 | Sensor Signal | external 4-pin connector → Brown UTP cable → GX12 → rectifier → Optocoupler → GPIO 25 |
| 4 | Sensor Power | external 4-pin connector → Blue UTP cable → GX12 → 3.3V rail |

**External 4-wire Connector Layout:**
- Wires 1-2: To doorbell power supply (2-wire)
- Wires 3-4: From doorbell button (2-wire)

### GX12-3 Connectors (Water Sensors)

**Water 1 (Hard) & Water 2 (Soft - optional):**
| Pin | Function | Wiring |
|-----|----------|--------|
| 1 | GND | Black wire |
| 2 | +3.3V Power | Red wire |
| 3 | Pulse Signal | Yellow wire → GPIO 23 (water 1) / GPIO 24 (water 2) |

### Relay 
- Relay Board GND → Pi GND
- Relay Board PWR → Pi 3.3V
- Relay 0 Control → GPIO 26

---

## Assembly Instructions

### Preparation
1. **Prepare casing**
   - Cut internal edges away
   - Mark hole positions on the sides for connectors
   
2. **Fabricate holes**
   - Drill 13mm holes for GX12 connectors
   - Drill 3mm holes for component mounting
   - File edges smooth

3. **Prepare wiring**
   - Pre-cut and strip all internal & external jumpers
   - Pre-tin connector pins and Pi GPIO headers

### Soldering & Assembly
4. **Solder components** (in this order)
   - Pi GPIO headers to Pi Zero 2W
   - GX12 connectors
   - Bridge rectifier + capacitor to power rail
   - Relay board to Pi via 4-pin header

5. **Heat shrink preparation**
   - Slide (but do not shrink yet) heat shrink tubes over all joints

6. **Mount to internal board**
   - Mount Pi + relay board + optocoupler on internal mounting plate
   - Connect all signal and power wires

7. **Testing (before final sealing)**
   - Verify with megger (high voltage safety test)
   - Test basic GPIO functionality with HMD-DGB config
   - Measure optocoupler isolation resistance
   - **Better to catch mistakes now than after shrinking & sealing**

### Final Assembly
8. **Mount in enclosure**
   - Mount internal board assembly
   - Mount all external connectors
   - Route wires neatly with Wago clamps

9. **Connect wiring**
   - Connect all external connectors to board
   - Re-test with megger before shrinking

10. **Finalize**
    - Heat shrink all joints
    - Label all connectors (optional but recommended)
    - Final functional test with HMD-DGB

---

## Software Setup

### First Boot Script
- [**[TODO: Link to script]**]

### Configuration
- [**[TODO: Link to configuration template]**]
- [**[TODO: Link to environment variables documentation]**]

### Home Assistant Integration
- Doorbell notification automation example: [**[TODO]**]
- Doorbell auto-toggle (enable/disable after X hours) example: [**[TODO]**]
- Water consumption history & alerts: [**[TODO]**]

---

## Failsafe Behavior

On power loss or Pi crash, the doorbell operates in its **normal state** (rings) due to the relay design:
- Relay is wired as **Normally Closed (NC)** on the doorbell trigger
- When relay is de-energized → doorbell rings normally
- When relay is energized (via GPIO 26) → doorbell is silenced

This ensures that even if the Pi is completely offline, the doorbell remains functional.

---

## Related Projects

- **[HMD-DGB](https://github.com/jvanoosterhout/HMD-DGB)** - Home automation control platform
- **Other Controllers:**
  - MV (Mechanical Ventilation)
  - BK (Bathroom Lights)
  - G (Garage Door & Garden Irrigation)

---

## License & Disclaimer

This is a personal home automation project shared as-is for educational purposes. Use at your own risk. Electrical work involving relays and/or high voltage should be performed safely and verified before deployment.

---

## TODO

- [ ] Add first boot script reference
- [ ] Add configuration file examples
- [ ] Add Home Assistant automation examples
- [ ] Add wiring diagram/schematic
- [ ] Add photos of assembled controller
- [ ] Document temperature sensor integration (optional)
- [ ] Document fan control logic (optional)
- [ ] Add troubleshooting section
- [ ] Add power consumption specs

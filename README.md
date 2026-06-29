# Anti-Theft Tracking CompacTag

## Project Overview
This project focuses on the design of a high-performance, ultra-thin asset protection and anti-theft tracking tag. The objective is to implement commercial-grade Ultra-Wideband (UWB) indoor positioning and proximity tracking capabilities within a highly compact, low-profile tracker envelope. 

The design represents a direct hardware optimization of the original [ESD_uwb Tracking System](https://github.com/Krish-S25/ESD_uwb). While the original development iteration validated room-scale UWB tracking using a broad footprint and discrete evaluation modules, this configuration consolidates the architecture into an ultra-low-profile 4-layer PCB utilizing an integrated System-in-Package (SiP), a dedicated low-profile bulk capacitor power delivery network, and an optimized buck-boost regulation topology.

---

## Architectural Evolution

| Design Metric | Original System (ESD_uwb) | Next-Gen CompacTag |
| :--- | :--- | :--- |
| **Microcontroller Core** | ESP32 Devkit v1 (DevBoard) | STM32WBA5MMG (Ultra-low-power BLE 5.4 SiP Module) |
| **UWB RF Module** | Ai-Thinker BU03 (Standard Pinout) | Ai-Thinker BU03 / Qorvo DWM3000 (Optimized Bus Interface) |
| **Power Source** | USB / LiPo Battery | CR2032 Coin Cell |
| **Power Regulation** | AMS1117 / MCP1700 Linear LDOs | TPS63900 High-Efficiency Buck-Boost Converter |
| **Form Factor** | Desktop Development Board Target | Ultra-low profile, 4-layer "Sticker" PCB Layout |

---

## Technical Specifications & Capabilities

* **Ranging Protocol:** Programmable Double-Sided Two-Way Ranging (DS-TWR) operating on UWB Channel 5.
* **Proximity Perimeter:** Monitors hardware ranging thresholds, executing asset alert routines immediately upon breach of a strict 30-meter boundary.
* **Alert Mechanism:** High-output Piezo buzzer driven using a **6.6V Differential Drive configuration** toggled via microcontroller hardware timers on pins `PB1` and `PB2` to achieve maximum acoustic output from a 3.3V rail.
* **Host Interconnect:** High-speed SPI1 hardware bus lines routed systematically to maintain clean transmission line matching and minimal signal cross-talk.

---

## Engineering Design Decisions & Explanations

### 1. Power Delivery Network (PDN) Engineering
Sourcing a peak-heavy UWB RF transmitter from a lithium coin cell (CR2032) introduces severe voltage stability constraints. During transient UWB transmission bursts, the module demands peak currents up to **176mA**. Without targeted local mitigation, the high internal resistance of the coin cell causes a profound voltage drop, inducing immediate microcontroller brownout resets.

* **Regulation Topology:** To stabilize the system voltage rail across the complete discharge curve of the coin cell, a high-efficiency **TPS63900 Buck-Boost converter** regulates the input power path to a stable 3.3V supply.
* **Bulk Reservoir Optimization:** To sustain high transient current bursts while conforming to a strict low-profile z-height requirement, standard electrolytic capacitors were omitted. Instead, a **304µF bulk reservoir** is placed directly at the UWB transceiver $V_{CC}$ inputs. This is implemented via low-profile **Tantalum-Polymer surface-mount capacitors (1206 footprint, 1.8mm maximum seated height)** featuring an ultra-low ESR of $35\text{m}\Omega$ @ $100\text{kHz}$.

### 2. Battery Life Analytics & Power Profiling
The device alternates strictly between ultra-low-power sleep states and transient wake periods, completing a full DS-TWR interaction within an optimized ~3ms active window.

* **Continuous 1Hz Polling Lifecycle:**
  * Operating continuously under a standard 1Hz tracking profile utilizing a standard $220\text{mAh}$ capacity cell, the average current loop calculations yield an estimated operational battery life of **1.4 Years**.
* **Daily Intermittent Duty-Cycling:**
  * For storage environments where the high-power UWB transceiver is actively polling for only **2 minutes per day** (relying on ultra-low-power BLE sleeping state frames for the remaining duration), the battery lifecycle extends close to the natural shelf-life of the lithium cell chemistry.

### 3. High-Frequency RF Layout Strategy
Integrating low-power Bluetooth (2.4GHz) and high-frequency Ultra-Wideband (6.5GHz) copper structures into a miniature board real estate requires strict isolation routing boundaries:
* **Antenna Keepout Zone:** A strict 4-layer copper voiding area was defined directly beneath the STM32 integrated module antenna. All internal traces, vias, ground fills, and surface-mount components are strictly banned from this region to prevent tuning distortion or reflection losses.
* **SPI Peripheral Remapping:** To avoid high-speed data line cross-talk, the design maps the hardware SPI1 peripheral bus straight to the lower-right cluster of the STM32 module module (**PB4, PB3, PA15, and PA12**). This keeps trace runs direct and short, completely separating high-speed logic from sensitive power paths.
* **Switching Node Isolation:** A localized 3-via grounding loop was laid out tightly around the TPS63900 buck-boost switching lines to block high-frequency switching harmonics from leaking into the primary RF ground planes.

---

Send Mail to **newk2505@gmail.com** for further inquiry

## PCB Layout Constraints (Manufacturing Rules)

The physical board is validated against the following electrical and mechanical constraints to guarantee clean trace signals and reliable fabrication:

```text
[Clearance Constraints]
Minimum Track-to-Track Clearance: 0.15 mm
Minimum Track-to-Copper Pour Clearance: 0.20 mm
Microcontroller Pad-to-Pour Isolated Clearance: 0.15 mm
UWB Transmission Line Edge Clearance: > 0.50 mm

[Via Constraints]
Minimum Hole Diameter: 0.3 mm
Minimum Annular Ring Width: 0.15 mm
Thermal Relief Via Connection Width: 0.25 mm

[Fabrication Edge Constraints]
Edge-Cuts Layer Margin Keepout: 0.35 mm
Mounting/PTH Pad Strain Relief Spacing: 1.20 mm
Maximum Component Component Seated Height: 1.80 mm (Sticker Envelope)

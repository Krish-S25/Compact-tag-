# Project 30TH (30-Meter Threshold) — Ultra-Thin Anti-Theft UWB Sticker Tag

## Project Overview
Project **30TH** is a high-performance, ultra-thin asset protection and anti-theft "sticker" tag designed to bring commercial-grade Ultra-Wideband (UWB) indoor positioning capabilities into a compact, low-profile tracker envelope. 

This project represents the direct architectural evolution of the original [ESD_uwb Tracking System](https://github.com/Krish-S25/ESD_uwb). While the original system validated room-scale UWB tracking using discrete components on larger footprints, Project 30TH moves the design forward by consolidating components into a highly integrated System-in-Package (SiP), implementing severe power delivery networks, and shifting to an ultra-thin 4-layer form-factor.

---

## Architectural Evolution

| Design Metric | Original System (ESD_uwb) | Next-Gen Sticker Tag (30TH) |
| :--- | :--- | :--- |
| **Microcontroller Core** | ESP32 Devkit v1 | STM32WBA5MMG (Ultra-low-power BLE 5.4 SiP Module)    |
| **UWB RF Module** | Ai-Thinker BU03 (Standard Pinout) | Ai-Thinker BU03 / Qorvo DWM3000 (Optimized Bus Layout Interface) |
| **Power Source** | USB / LiPo | CR2032 Coin Cell via TPS63900 |
| **Power Regulation** | AMS1117 / MCP1700 | TPS63900 Buck-Boost Converter |
| **Form Factor** | Desktop Development Board Target | Ultra-low profile, 4-layer "Sticker" PCB Layout |

---

## Technical Specifications & Capabilities

* **Ranging Protocol:** 1Hz Double-Sided Two-Way Ranging (DS-TWR) operating on UWB Channel 5.
* **Proximity Perimeter:** Tracks proximity thresholds, triggering asset alert routines immediately upon breach of a strict 30-meter boundary.
* **Alert Mechanism:** High-output Piezo buzzer driven using a **6.6V Differential Drive configuration** toggled via microcontroller hardware timers on pins `PB1` and `PB2` to squeeze maximum acoustic output from a 3.3V rail.
* **Host Interconnect:** High-speed SPI1 hardware bus lines systematically routed to maintain clean transmission line matching and minimal signal cross-talk.

---

## Engineering Design Decisions & Explanations

### 1. Advanced Power Delivery Network (PDN) Mitigation
Sourcing a peak-heavy UWB RF transmitter from a lightweight lithium coin cell (CR2032) introduces severe voltage stability constraints. During short UWB transmission bursts, the module demands peak currents up to **176mA**. Without local mitigation, the internal resistance of the coin cell causes a massive voltage dip, inducing immediate microcontroller brownout resets.

* **The Architecture:** To stabilize the system, a high-efficiency **TPS63900 Buck-Boost converter** regulates the power rail, ensuring a reliable 3.3V out even as the coin cell sags.
* **Bulk Reservoir Optimization:** To meet peak current bursts while preserving the flat sticker envelope, standard bulky electrolytic capacitors were completely avoided. Instead, a massive **304µF bulk reservoir** is mapped directly at the BU03 $V_{CC}$ inputs. This is implemented via ultra-low profile **Tantalum-Polymer surface-mount capacitors (1206 footprint, 1.8mm max seated height)** boasting an ultra-low ESR of $35\text{m}\Omega$ @ $100\text{kHz}$. 

### 2. Battery Life Analytics & Lifecycle Profiling
The device alternates strictly between ultra-low-power sleep states and transient wake periods, completing a full DS-TWR interaction within an optimized ~3ms active window.

* **Continuous 1Hz Polling Lifecycle:**
  * Operating continuously under a standard 1Hz tracking profile utilizing a standard $220\text{mAh}$ capacity cell, the average current loop calculations yield an estimated operational battery life of **1.4 Years**.
* **Daily Intermittent Duty-Cycling:**
  * For storage environments where the high-power UWB transceiver is actively polling for only **2 minutes per day** (relying on ultra-low-power BLE sleeping state frames for the remaining duration), the battery lifecycle extends to near the natural shelf-life of the lithium cell chemistry.

### 3. High-Frequency RF Layout Strategy
Packing low-power Bluetooth (2.4GHz) and high-frequency Ultra-Wideband (6.5GHz) signals into a miniature board real estate requires aggressive noise control boundaries:
* **Antenna Keepout Zone:** A strict 4-layer copper voiding area was defined directly beneath the STM32 integrated module antenna. All internal traces, vias, ground fills, and surface-mount components are strictly banned from this region to prevent tuning distortion or reflection losses.
* **SPI Peripheral Remapping:** To avoid high-speed data line cross-talk, the design maps the hardware SPI1 peripheral bus straight to the lower-right cluster of the STM32 module (**PB4, PB3, PA15, and PA12**). This keeps trace runs direct and short, completely separating high-speed logic from sensitive power paths.
* **Switching Node Isolation:** A localized 3-via grounding loop was laid out tightly around the TPS63900 buck-boost switching lines to block high-frequency switching harmonics from leaking into the primary RF ground planes.

---

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

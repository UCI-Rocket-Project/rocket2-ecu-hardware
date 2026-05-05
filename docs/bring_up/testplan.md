# ECU Hardware Bring-Up Test Plan

This page outlines the test plan for the ECU hardware from initially powering the board to firmware implementation.

---

## Note: Mandatory Testing Setup: Barrel Jack & GSE+
All circuits utilizing GSE+ as an input (including the 5V Insta360 regulator and the REDS Voltage Sense dividers) require the Barrel Jack connector to be populated and tested. Refer to schematic to ensure that the test can be conducted in a controlled manner please!

**Required Test Hardware:**
* Use a Barrel Jack-to-pigtail connector --> DC Jack to dual-wire Power/GND 
* Connect the wires to a DC Power Supply to do controlled voltage sweeps and current limiting!

---

## Power Subsystem Verification

### USB-C (Insta360) / 5V Regulator
*Note: Input sourced from GSE+ via Barrel Jack/D3.*

**Procedure:**

* Connect Anker power bank to USB-C (or bench supply via Barrel Jack). Or any other fast-charging USB-C item that can do the handshake! Power bank was used because it has a fast-charging (lightning indicator), but a safer option could include: 
* Confirm fast charging (LED status on board + lightning symbol on power bank).
* Monitor for excessive heat/smoke immediately after connection.
* Sweep Input Voltage: Start at 5V and increment to 24V using a DC power supply 

**Expected Voltages per Pin:**

| Pin | Expected Reading | Notes |
| :--- | :--- | :--- |
| **Vin** | Vin - &approx;0.5V | Accounting for diode forward voltage drop (D3). |
| **Vout** | 5.0V | Target regulated output. |
| **SW** | &approx; Average of Vin & GND | e.g., if Vin is 12V, SW should see &approx;5V-6V. |
| **BST** | SW + &approx;5.0V | If BST = SW, FET is not switching correctly. |

**Warning:** If BST is above or below 5V relative to the SW pin, the regulator circuit is non-functional.

### Six Single-Cell Chargers
**Procedure:**

* Solder all six single-cell charger ICs.
* **Visual Inspection:** Verify no bridges, lifted pads, or missing passives.
* **Input Sweep:** Using DC bench supply via Barrel Jack, sweep from 3.3V &rarr; 24V in 1V increments.
* **Monitoring:**
    * Measure 3.3V rail at each increment.
    * Monitor total input current draw.
    * Confirm blinking LEDs (indicates "no battery" state).
* **Load Test:** Connect a LiPo battery and verify charging current.

### Barrel Jack (AC Power / 24V)
**Procedure:**

* Connect 24V supply to barrel jack—**Verify polarity first**.
* Confirm 3.3V regulator output stability.
* Confirm all six single-cell chargers remain in the expected idle/standby state.
* **GSE Sense Check:** Verify voltage divider outputs at `REDs_V_SENSE1` match the 1V : 7.67V ratio.

---

## Firmware & Sensor Bring-Up

### Preliminary Checks

* Check for shorts across all data lines.
* Confirm resistance values match the schematic/BOM.
* Solder connectors onto the board prior to full firmware integration.

### Sensor Integration Order
Flash the test firmware and bring up the 3.3V rail to verify Ground Station communication.

* **IMU**: Initialize and verify raw data stream.
* **Altimeter**: Verify I2C/SPI address acknowledgement.
* **Magnetometer**: (Completed - Pending Review)
* **Flash Memory**: (Completed - Pending Review)
* **GPS**: Verify NEMA sentence reception.

### Peripheral Controls

* **PTs & TCs**: Analog front-end verification.
* **Solenoids**: Test firing with current limiting enabled.
* **Ethernet**:
    * Check hardware for shorts.
    * Initialize PHY in firmware (Verify Link LED).
    * Perform Ping test to Ground Station.

---

## General JTAG & Debugging Reference
*Note: Debugging interfaces are located on the **bottom layer (L6)**.*
Please **refer to schematic** for additional test points or connection clarification!!

### JTAG (J5) Pinout
| Pin | Signal | Pin | Signal |
| :--- | :--- | :--- | :--- |
| 1 | **3.3V (VREF)** | 2 | **SWDIO** |
| 3 | **GND** | 4 | **SWCLK** |
| 5 | **GND** | 6 | **SWO** |
| 7 | **KEY** | 8 | **NC** |
| 9 | **GND** | 10 | **nRESET** |

## Hardware Debug Log (April 2026)

### 5V Regulator Input Instability
* **Observations:**
    * Detected unstable resistance readings across the **Schottky diode (D3)** at the 5V regulator input.
    * **Reverse bias:** 1MΩ.
    * **Forward voltage:** 20v (likely faulty diode)
* **Steps Taken:**
    * Desoldered D3 and installed a jumper wire across the pads to bypass the diode for testing.
    * **Result:** Regulator circuit now displays stable, expected voltage values. 
    * **Validation:** Verified successful USBC handshake and fast-charging.
* **Next: May 2026:**
    * Parallel pathing for flashing **REDs firmware**.
    * Verifying **Ethernet communication** tests to verify Radio functionality and mirrored Main circuitry. 

### Issue: Balance Connector Mismatch
* Issue: balance connector currently on the board does not match the existing battery inventory.
* Solution Complete: Buy new batteries with the JST-style balance connector that maches the pcb footprint.

---

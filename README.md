# CAN-Based Dynamic Traction Control (TCS) System

## Overview

This repository contains a Software-in-the-Loop (SIL) simulation of an automotive Traction Control System (TCS) developed in **Vector CANoe**. The system continuously calculates real-time wheel slip across a simulated 500 kbps CAN bus. By evaluating driver inputs against dynamic road friction coefficients (Mu), the TCS Master Node autonomously derates engine torque and applies brake pressure to prevent loss of vehicle control in adverse weather conditions.

Additionally, the system features a fully integrated **ISO 14229 Unified Diagnostic Services (UDS)** stack, allowing simulated scan tools to perform security access, manipulate Data Identifiers (DIDs), and read/clear Diagnostic Trouble Codes (DTCs).

## Key Features

* **Continuous Vehicle Physics:** Replaces static speed steps with continuous CAPL-driven acceleration and deceleration physics based on brake pressure and surface friction.
* **Dynamic Environment Modeling:** Supports four real-time driving scenarios with varying grip thresholds: Dry (Mu=0.80), Wet (Mu=0.60), Snow (Mu=0.35), and Ice (Mu=0.20).
* **Automated Safety Interventions:** 50% engine torque derating and active brake pumping triggered within a 20ms cyclic loop when wheel slip exceeds a 15% safety threshold.
* **Interactive HMI Dashboard:** Built with Panel Designer to provide live telemetry (Slip Ratio, RPM, Speed), driver input sliders, and simulated hardware fault injection switches.
* **Automated Logic Validation:** Includes a dedicated CAPL Test Module (`TCS_TestCase.can`) that executes automated fault and fail-safe validations with a 100% pass rate.

## Network Architecture

The simulation runs on a Classical CAN network (ISO 11898-2) at 500 kbit/s, consisting of the following simulated ECUs:

1. **Driver_Input_ECU:** Transmits throttle, brake, steering, and ignition states.
2. **Wheel_Speed_Module:** Transmits individual wheel RPM and vehicle reference speed.
3. **Powertrain_Brake_ECU:** Receives torque derate requests and brake pump commands.
4. **TCS_Master_Node:** The central controller executing the closed-loop slip algorithm and managing UI telemetry.
5. **Diagnostic_Tester:** A virtual scan tool for sending ISO 14229 UDS requests.

## UDS Diagnostics Implementation

The network supports industry-standard diagnostic interrogation and fault memory management:

* **0x10 Diagnostic Session Control:** Enforces Default and Extended session states, protected by an S3 timeout fail-safe.
* **0x27 Security Access:** Implements a cryptographic Seed-Key algorithm (Seed XOR 0xA5C3) to unlock the ECU.
* **0x22 Read Data By Identifier:** Supports both single-frame and ISO-TP multi-frame (e.g., 17-character VIN) data retrieval.
* **0x2E Write Data By Identifier:** Allows configuration changes (e.g., writing to DID 0x0100) strictly when the ECU is in an unlocked Extended Session.
* **0x19 Read DTC Information & 0x14 Clear Diagnostic Information:** Logs and clears simulated hardware faults (e.g., DTC C0035 for a severed wheel speed sensor).

## Tools & Technologies Used

* **Vector CANoe** (Simulation & Testing)
* **CAPL** (Communication Access Programming Language)
* **Vector CANdb++** (Database design and signal mapping)
* **Vector Panel Designer** (HMI/UI Creation)

## How to Run

1. Clone the repository and open the `.cfg` file in Vector CANoe.
2. Ensure the `Elc23019.dbc` database is properly linked in the Simulation Setup window.
3. Compile all `.can` node scripts via the CAPL Browser.
4. Start the measurement. Use the HMI panel to interact with the vehicle pedals, change road scenarios, and inject wheel sensor faults to observe the system's dynamic interventions.

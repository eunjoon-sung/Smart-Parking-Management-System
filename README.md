# Distributed Smart Parking Management System (ATmega128)

## 1. Project Overview
[cite_start]This project implements a multi-node, real-time smart parking management system using three ATmega128 microcontrollers. [cite_start]Designed with a Master-Slave distributed architecture, the system automates vehicle entry detection, parking slot allocation, parking duration tracking, and fee calculation[cite: 3028, 3030, 3045]. [cite_start]Communication between nodes is handled via a custom asynchronous UART protocol, ensuring synchronized state management across the parking lot facility[cite: 3032, 3050].

## 2. System Architecture & Key Features
* [cite_start]**Master MCU (Central Hub):** Manages the global parking state, records entry/exit timestamps using Timer Interrupts (RTC simulation), calculates parking fees, and broadcasts real-time available slot data to slaves[cite: 3084, 3407].
* **Slave A MCU (Entry Node):** Utilizes ultrasonic sensors to detect incoming vehicles. [cite_start]Interfaces with a 4x4 matrix keypad to capture the 4-digit vehicle ID, displays inputs on a 7-segment display, and visualizes the available parking slots using an 8-LED array[cite: 3175].
* [cite_start]**Slave B MCU (Exit Node):** Detects exiting vehicles, captures the vehicle ID, and requests fee calculation from the Master[cite: 3489, 3801]. [cite_start]Displays the parking duration and calculated fee on a 16x2 LCD, processing the final "payment" signal to free up the parking slot[cite: 3802, 3803].
* [cite_start]**Custom UART Protocol:** Defined a strict communication protocol using fixed-length payloads and newline (`\n`) delimiters to ensure robust data framing between the Master and Slaves[cite: 3789].

## 3. Critical Troubleshooting Log (Deep Dive)

### Issue: Asynchronous Event Handling in Blocking Loops (Data Synchronization Failure)
* [cite_start]**Symptom:** When a vehicle exited via Slave B and the Master updated the parking state, Slave A failed to immediately reflect the new available slot on its LED/LCD interface.
* [cite_start]**Root Cause Analysis:** Verified that the Master was transmitting the correct ASCII payload and Slave A's UART RX Interrupt Service Routine (ISR) was successfully receiving it and setting the `parking_state_updated` flag. [cite_start]The core issue lay in the main execution flow: Slave A's main function was trapped in nested, blocking polling loops (e.g., `while(is_full())` or waiting for keypad inputs)[cite: 3351, 3357]. [cite_start]Although the ISR fired, the main loop could not evaluate the updated flag until the current blocking routine naturally terminated[cite: 3362].
* **Resolution:** Refactored the rigid polling architecture into a preemptive flag-checking system. [cite_start]Injected `if (parking_state_updated) break;` conditions into all nested blocking loops. [cite_start]This forced the main execution thread to immediately exit any idle or polling state upon receiving a UART interrupt, allowing it to jump back to the top-level state machine to refresh the LCD and LED interfaces in real-time.

## 4. Future Architecture Upgrade
* [cite_start]**Hardware Expansion:** Integration of DC motors with PWM control to simulate physical barrier gates upon successful entry/payment[cite: 3839].
* [cite_start]**Data Integrity:** Implementation of Parity bit checks for UART packets to prevent communication errors, and adding an EEPROM-based backup system to prevent data loss of parked vehicle IDs during power failures[cite: 3037, 3838].

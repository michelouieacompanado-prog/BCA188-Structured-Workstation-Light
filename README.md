# Structured Workstation Light

This repository contains the firmware and documentation for a structured workstation light controller using an ESP32. 

## Overview
The system implements a structured Input-Process-Output design to control a PWM LED (workstation light) and a status LED. The system requires the user to press and hold a safety button to enable the light, while a potentiometer controls the brightness. When the button is released, the light instantly turns off, demonstrating safe workstation practices.

## Hardware Requirements
| Component | ESP32 Pin | Details |
| :--- | :--- | :--- |
| Push Button | GPIO 23 | Active LOW (INPUT_PULLUP) |
| Potentiometer | GPIO 34 | Wiper to GPIO 34, Outer pins to 3.3V and GND |
| Status LED | GPIO 18 | Indicates if output is enabled |
| PWM Light LED | GPIO 19 | Brightness controlled via PWM |

## System Logic
1. **Read:** Obtains the button state and potentiometer analog value.
2. **Process:** Scales the 12-bit ADC input to an 8-bit PWM value. Decides whether to apply the duty cycle based on the button state.
3. **Write:** Updates the status LED and writes the PWM duty cycle to the brightness LED.

## Setup Instructions
1. Wire the components according to the pinout above.
2. Ensure the potentiometer is wired with the left pin to GND and right pin to 3.3V to increase brightness when turned clockwise.
3. Flash `workstationLight.ino` using the Arduino IDE.

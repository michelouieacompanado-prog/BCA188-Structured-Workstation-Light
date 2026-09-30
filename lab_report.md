
#  Laboratory Report

**Course:** BCA188 - Programming For Internet of Things                                    
**Activity:** Laboratory Activity 5 — Structured Workstation Light  
**Development Environment:** Arduino IDE  
**Target Board:** ESP32 Microcontroller  

---

## 1. Activity Objectives

1. Develop modular firmware following an Input-Processing-Output (IPO) structure in the Arduino IDE.
2. Implement a dedicated scaling function: `int scaleToDuty(int raw)`.
3. Implement a dead-man enable control using a pushbutton.
4. Confirm that rotating the potentiometer while the button is released does not light the output.
5. Record experimental observations for low, middle, and high potentiometer positions.

---

## 2. Hardware Connections & Circuit Description

The circuit uses two inputs and two outputs connected to the ESP32:

* **Pushbutton (Digital Input):** Connected to GPIO 23 and GND. Uses the internal `INPUT_PULLUP` resistor.
* **Potentiometer (Analog Input):** Middle wiper connected to GPIO 34. Outer legs connected to GND and 3.3V.
* **Status LED (Digital Output):** Anode connected to GPIO 18 (via current-limiting resistor), cathode to GND.
* **PWM LED (PWM Output):** Anode connected to GPIO 19 (via current-limiting resistor), cathode to GND.

---

## 3. Explanation of Software Concepts

* **Constants (`const uint8_t`):** Assign human-readable names to microcontroller pins. Using `const` prevents pin numbers from changing accidentally at runtime.
* **Boolean State Variables (`bool`):** Hold binary states. `buttonPressed` tracks the physical button state, and `pwmReady` tracks if the hardware PWM driver initialized successfully.
* **Numeric Reading Variables (`int`):** `rawInput` holds the raw 12-bit ADC value, `requestedDuty` holds the calculated 8-bit mapped value, and `appliedDuty` gates the final output depending on the button state.
* **Scaling Calculation (`scaleToDuty`):** This function extracts the mapping logic. It converts the 12-bit ADC reading (0 to 4095) into an 8-bit PWM duty cycle (0 to 255) using Arduino's `map()` and `constrain()` functions.

---

## 4. System Firmware Source Code

```cpp
#include <Arduino.h>

const uint8_t BUTTON_PIN = 23;
const uint8_t POT_PIN = 34;
const uint8_t STATUS_LED_PIN = 18;
const uint8_t PWM_LED_PIN = 19;

bool buttonPressed = false;
bool pwmReady = false;
int rawInput = 0;
int requestedDuty = 0;
int appliedDuty = 0;

void readInputs();
void processInputs();
void updateOutputs();
int scaleToDuty(int raw);

void setup() {
  Serial.begin(115200);
  
  pinMode(BUTTON_PIN, INPUT_PULLUP);
  
  pinMode(STATUS_LED_PIN, OUTPUT);
  digitalWrite(STATUS_LED_PIN, LOW);
  
  pinMode(PWM_LED_PIN, OUTPUT);
  digitalWrite(PWM_LED_PIN, LOW);
  
  analogReadResolution(12);
  analogSetPinAttenuation(POT_PIN, ADC_11db);
  
  pwmReady = ledcAttach(PWM_LED_PIN, 5000, 8);
  if (pwmReady) {
    ledcWrite(PWM_LED_PIN, 0);
  } else {
    Serial.println("PWM setup failed.");
  }
}

void loop() {
  readInputs();
  processInputs();
  updateOutputs();
  delay(20);
}

void readInputs() {
  buttonPressed = (digitalRead(BUTTON_PIN) == LOW);
  rawInput = analogRead(POT_PIN);
}

int scaleToDuty(int raw) {
  return constrain(map(raw, 0, 4095, 0, 255), 0, 255);
}

void processInputs() {
  requestedDuty = scaleToDuty(rawInput);
  
  if (buttonPressed && pwmReady) {
    appliedDuty = requestedDuty;
  } else {
    appliedDuty = 0;
  }
}

void updateOutputs() {
  digitalWrite(STATUS_LED_PIN, (buttonPressed && pwmReady) ? HIGH : LOW);
  
  if (pwmReady) {
    ledcWrite(PWM_LED_PIN, appliedDuty);
  }
}
```

---

## 5. Experimental Documentation (Drawings, Photos, & Video)

**Wiring Diagram:**
*<img width="2048" height="1536" alt="2fe56a70-5da9-48dd-a42c-35046b2ea66b" src="https://github.com/user-attachments/assets/42741d27-85b3-4151-b171-171eaaf92c80" />*

**Recorded Results for Knob Positions:**
* **Low Position (Fully CCW):** When the button is held, the Status LED turns ON, but the PWM LED remains OFF (0% brightness). *<img width="2048" height="1536" alt="7661614c-82aa-4547-882e-36737a1c0b77" src="https://github.com/user-attachments/assets/38f34e81-2e69-47d7-bfd7-a3ecf4828055" />*


* **Middle Position (~Center):** When the button is held, the Status LED turns ON, and the PWM LED lights up at medium brightness (~50%). *<img width="2048" height="1536" alt="ce44726b-81f7-4f8e-b9d2-62776c8dd7c1" src="https://github.com/user-attachments/assets/8ef8820a-82fc-47d9-84db-f762a4489d9a" />*


* **High Position (Fully CW):** When the button is held, the Status LED turns ON, and the PWM LED reaches full brightness (100%). *<img width="2048" height="1536" alt="30891fc4-41b9-4c3a-bb6f-1c22dfca3299" src="https://github.com/user-attachments/assets/aa0f8594-5ba7-4e3c-9968-e62088aa0b6b" />*

**Demonstration Video:**
*<video src="https://github.com/user-attachments/assets/2fd43dbd-7b66-40b2-a631-a5dda76aaf93" width="100%" controls></video>*

---

## 6. Table of Expected vs. Observed Behavior

| Test Sequence | Required Condition | Expected Behavior | Observed Behavior | Test Status |
| :--- | :--- | :--- | :--- | :---: |
| **1. Startup / Idle** | Reset with button released | Both LEDs off; system remains disabled. | Both LEDs remained OFF. | PASS |
| **2. Enable Output** | Hold button (at high/mid setting) | Status LED turns ON; PWM LED lights up. | Status LED turned ON; PWM LED turned ON. | PASS |
| **3. Variable Control** | Rotate knob while held | PWM LED brightness changes smoothly. | Brightness scaled properly as knob turned. | PASS |
| **4. Safety Shutoff** | Release button (at high setting) | Both LEDs turn off immediately. | Both LEDs turned OFF instantly. | PASS |
| **5. Disabled State**| Rotate knob while released | Both LEDs stay off; knob changes ignored. | Both LEDs stayed OFF during rotation. | PASS |

---

## 7. Conclusion

Laboratory Activity 5 successfully demonstrated a modular, state-controlled workstation light application on the ESP32. By isolating scaling math inside `scaleToDuty()` and keeping input, processing, and output tasks in separate functions, the firmware structure remains clean and easy to modify. The dead-man control logic ensures that the light source can only operate while the button is actively held, providing a dependable safety control mechanism.

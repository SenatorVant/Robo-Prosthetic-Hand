# **Robotic Hand Project – README**
## **Overview**

This project uses **five flex sensors** to control **ten servos** (two per finger) on a robotic hand that was machined and assembled by hand.  
An Arduino reads the sensor data, smooths it, detects gestures, and maps finger motion to servo rotation.

### **This README contains:**

- **Detailed project description**  
- **Wiring diagrams**  
- **Step-by-step build + setup instructions**  
- **Arduino code reference**  


## **CAD Images**

<img width="710" height="718" alt="Image1" src="https://github.com/user-attachments/assets/f89b7d17-734f-4e8d-a58e-162b6d9b3166" />
<img width="690" height="519" alt="Image3" src="https://github.com/user-attachments/assets/89ff6632-bb52-471c-a30d-90b25c0d24a7" />
<img width="623" height="590" alt="Image2" src="https://github.com/user-attachments/assets/f8757ba6-b1ce-469f-9ee9-726479eb34d7" />


## **Features**

- Five fully articulated fingers  
- Two-servo linkage per finger for realistic curved movement  
- Flex-sensor glove input  
- Gesture detection (**FIST**, **OPEN**, **POINT**, **PINCH**, **NONE**)  
- Built-in signal smoothing and calibration arrays  
- Fully open and modifiable Arduino code  

## **Required Materials**

- **Arduino Uno or Mega**  
- **10 servo motors**  
- **5 flex sensors**  
- **5V external power supply**  
- **Machined robotic hand**  
- Jumper wires  
- Breadboard (optional)  
- Zip ties  
- 3D-printed or machined glove sensor mounts (optional)

## **Wiring Diagrams**

### **Flex Sensor Wiring**

**Flex Sensor → Voltage Divider → Arduino Analog Input**

 +5V
  |
  |
[ Flex ]
|
|-------> A0 (Thumb)
|
[10kΩ]
|
GND

Repeat the same divider for pins **A1–A4**.

**Pin assignment:**

- **Thumb:** A0  
- **Index:** A1  
- **Middle:** A2  
- **Ring:** A3  
- **Pinky:** A4  

### **Servo Wiring**

Each finger uses **two servos**:

Finger: Servo Pins
Thumb: 2, 3
Index: 4, 5
Middle: 6, 7
Ring: 8, 9
Pinky: 10, 11


**Servo power layout:**

Servo Power Rail:

Red -> 5V (external supply)
Brown -> GND
Orange -> Arduino PWM Pin

**IMPORTANT:** Connect **Arduino GND** to **Servo Power GND**.

### **Full Servo Pin Mapping**

- **Thumb** → Pins 2, 3  
- **Index** → Pins 4, 5  
- **Middle** → Pins 6, 7  
- **Ring** → Pins 8, 9  
- **Pinky** → Pins 10, 11  

## **Step-by-Step Build Guide**

### **1. Machine or Print the Hand**

- Fabricate the palm, knuckles, and finger segments.  
- Ensure each finger supports a dual-driven linkage path.  
- Install pivots, screws, or bushings.
<img width="2240" height="1260" alt="image" src="https://github.com/user-attachments/assets/34e0c9bc-8930-4ddd-a499-93d41e370b53" />

<img width="1024" height="659" alt="image" src="https://github.com/user-attachments/assets/d5ec9f71-e790-488c-a127-f9b499d8e448" />

### **2. Assemble the Servos**

- Mount two servos per finger.  
- Prevent mechanical interference.  
- Test each servo before final mounting.


### **3. Build the Flex-Sensor Glove**

- Attach one flex sensor to each glove finger.  
- Secure with heat-shrink, tape, resin channels, or stitched pockets.  
- Wire each sensor into a **10kΩ voltage divider**.

<img width="497" height="252" alt="image" src="https://github.com/user-attachments/assets/5790986c-7742-44be-a1bf-06a6d236ace4" />

### **4. Wire the Electronics**

- Connect flex sensors to analog pins **A0–A4**.  
- Connect servo signal lines to pins **2–11**.  
- Use an external 5V supply for servos.  
- Connect all grounds together.

<img width="756" height="506" alt="image" src="https://github.com/user-attachments/assets/60423890-7159-4bb6-b8bf-37e87449f9a0" />

<img width="601" height="674" alt="image" src="https://github.com/user-attachments/assets/2d8c41e5-0c5d-4778-8b39-3d63230ec43e" />

### **5. Upload the Arduino Code**

 ```
#include <Servo.h>

const int NUM_FINGERS = 5;
const int SERVOS_PER_FINGER = 2;

// Flex sensor pins
int flexPins[NUM_FINGERS] = {A0, A1, A2, A3, A4};

// Servos per finger (each finger controls two servos)
int servoPins[NUM_FINGERS][SERVOS_PER_FINGER] = {
  {2, 3},   // Thumb
  {4, 5},   // Index
  {6, 7},   // Middle
  {8, 9},   // Ring
  {10, 11}  // Pinky
};

Servo servos[NUM_FINGERS][SERVOS_PER_FINGER];

// Calibration (adjust with Serial Monitor)
int flexMin[NUM_FINGERS] = {300, 300, 300, 300, 300};
int flexMax[NUM_FINGERS] = {600, 600, 600, 600, 600};

// Smoothing parameters
const int NUM_SAMPLES = 10;
int flexHistory[NUM_FINGERS][NUM_SAMPLES];
int historyIndex = 0;

// Gesture thresholds
int fistThreshold = 500;
int openThreshold = 350;

void setup() {
  Serial.begin(115200);

  // Attach servos
  for (int f = 0; f < NUM_FINGERS; f++) {
    for (int s = 0; s < SERVOS_PER_FINGER; s++) {
      servos[f][s].attach(servoPins[f][s]);
    }
  }

  // Initialize smoothing buffers
  for (int f = 0; f < NUM_FINGERS; f++) {
    for (int i = 0; i < NUM_SAMPLES; i++) {
      flexHistory[f][i] = flexMin[f];
    }
  }
}

int smoothFlex(int finger) {
  flexHistory[finger][historyIndex] = analogRead(flexPins[finger]);

  long sum = 0;
  for (int i = 0; i < NUM_SAMPLES; i++) {
    sum += flexHistory[finger][i];
  }

  return sum / NUM_SAMPLES;
}

void moveFingerServos(int finger, int angle) {
  // Servo 1 follows the angle fully
  servos[finger][0].write(angle);

  // Servo 2 follows partially for realistic finger motion
  int secondaryAngle = map(angle, 0, 180, 0, 120);
  servos[finger][1].write(secondaryAngle);
}

// Detect gestures
String detectGesture(int flexValues[]) {
  bool allBent = true;
  bool allStraight = true;

  for (int f = 0; f < NUM_FINGERS; f++) {
    if (flexValues[f] < fistThreshold) allBent = false;
    if (flexValues[f] > openThreshold) allStraight = false;
  }

  if (allBent) return "FIST";
  if (allStraight) return "OPEN";

  if (flexValues[1] < 450 && flexValues[0] > 450) return "POINT";

  if (flexValues[0] < 450 && flexValues[1] < 450) return "PINCH";

  return "NONE";
}

void loop() {
  int flexValues[NUM_FINGERS];

  historyIndex = (historyIndex + 1) % NUM_SAMPLES;

  for (int f = 0; f < NUM_FINGERS; f++) {
    flexValues[f] = smoothFlex(f);

   int angle = map(flexValues[f], flexMin[f], flexMax[f], 0, 180);
    angle = constrain(angle, 0, 180);

   moveFingerServos(f, angle);
  }

  String gesture = detectGesture(flexValues);

  Serial.print("Gesture: ");
  Serial.println(gesture);

  delay(10);
}

 ```

## **Finished Project Images**

<img width="345" height="505" alt="image" src="https://github.com/user-attachments/assets/d22d3b31-c260-48d1-bb47-45514658b63a" />

<img width="540" height="673" alt="image" src="https://github.com/user-attachments/assets/f3137d92-07f7-4644-ad9d-97a984b64c89" />

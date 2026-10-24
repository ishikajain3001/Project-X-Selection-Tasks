> note:- you just need to compete tasks and [Meshmerize_PS.pdf](https://github.com/the-robotronics-club/Project-X-Selection-Tasks/blob/dad5236d8f27beb7fc49fb64453f889b1e6ad99e/Line-Maze-Solver/Meshmerize_PS.pdf) is just to give you an idea of what competition actually is. 

# LMS Learning Resources

### Track Overview

1. **Sensors, Centroid Math & PID Control** (Tasks 1, 3, & 9)
2. **Motors, Drivers, Datasheets & Wiring** (Tasks 2, 4, & 6)
3. **Batteries, Power Budgets & Runtime** (Task 5)
4. **Maze Solving & Wireless Telemetry** (Tasks 7 & 8)

---

## 1. Sensors, Centroid Math & PID Control

**Applies to:** Task 1 (Centroid & Motor Clamping), Task 3 (PID Control Theory), Task 9 (ESP32 Line Follower Implementation).

### Essential Technical Concepts

- **Sensor Operating Principle:** The array uses five infrared (IR) reflectance sensors facing the surface. Light surfaces reflect IR light back (producing a lower analog reading); dark lines absorb IR light (producing a higher analog reading).
- **Line Position Estimation (Weighted Centroid):** Do not estimate line position using simple discrete `if/else` checks. Compute the weighted average across all calibrated readings:
  $$\text{Position} = \frac{\sum_{i=0}^{4} (w_i \times S_i)}{\sum_{i=0}^{4} S_i} = \frac{(-2 \times S_1) + (-1 \times S_2) + (0 \times S_3) + (1 \times S_4) + (2 \times S_5)}{S_1 + S_2 + S_3 + S_4 + S_5}$$
  - A centered line yields an output of `0`.
  - Drifting toward the right yields a positive value; drifting left yields a negative value.
- **Handling the All-Zero Edge Case:** If the robot leaves the track, every sensor reading drops to zero. This makes the denominator zero. Your function must catch this condition before performing division (`if (sum == 0)`). Return a dedicated status code or retain the last valid position sign so your steering routine turns toward the lost line.
- **PID Control for Steering:**
  - **Proportional ($P$):** Applies a corrective turn proportional to current displacement error. High values cause violent side-to-side hunting.
  - **Derivative ($D$):** Counteracts rapid changes in error ($\Delta\text{error} / \Delta t$). This acts as a damper to stabilize straight-line tracking.
  - **Integral ($I$):** Corrects steady-state mechanical bias over time. For high-speed line followers, $I$ is usually set to zero because accumulated error causes steering overshoot on sharp turns.
- **Motor Speed Differential:** Convert the PID output into left and right motor commands:

  ```cpp
  int leftSpeed  = baseSpeed + correction;
  int rightSpeed = baseSpeed - correction;
  ```

  Clamp these commands to valid PWM boundaries (`0` to `255`) before writing to the motor driver pins.

### Curated Study Material

- **Video — What is PID Control? (Brian Douglas / MATLAB Tech Talks):**
  [Watch on YouTube](https://www.youtube.com/watch?v=UR0hOmjaHp0)  
  *Why study this:* Uses clear physical analogies of vehicle lane-keeping to explain how P, I, and D work together, avoiding unnecessary mathematical clutter.
- **Video — Line Follower Robot with PID Controller (Robotistan):**
  [Watch on YouTube](https://www.youtube.com/watch?v=7u7P8c_vj_A)  
  *Why study this:* Demonstrates live manual tuning on physical hardware. Shows what happens when $K_p$ is set too high and how adding $K_d$ smooths the robot's line tracking.
- **Reference — Pololu 3pi Line Following Guide (Section 7.c):**
  [Read Documentation](https://www.pololu.com/docs/0J21/7.c)  
  *Why study this:* The standard industry reference for converting multi-sensor reflectance readings into calibrated position variables and steering differentials.
- **Reference — Arduino Math & Clamping Reference:**
  [Read Documentation](https://www.arduino.cc/reference/en/language/functions/math/constrain/)  
  *Why study this:* Shows how to bound motor control commands using `constrain()`.

---

## 2. Motors, Drivers, Datasheets & Wiring

**Applies to:** Task 2 (Datasheet Literacy), Task 4 (BOM Preparation under ₹10,000), Task 6 (Electronics, Wiring & Troubleshooting).

### Essential Technical Concepts

- **The Role of the Motor Driver:** An ESP32 GPIO pin provides 3.3V and a maximum of ~20mA. A DC motor requires 6V–12V and draws between 300mA and 1500mA. Connecting a motor directly to a microcontroller pin will destroy the chip. The motor driver uses low-current logic signals to switch high-current battery power.
- **Comparing Driver Architectures (L298N vs TB6612FNG):**
  - The **L298N** uses bipolar Darlington transistors. It introduces an internal voltage drop between 1.8V and 3.2V, dissipating battery power as waste heat and requiring a metal heatsink.
  - The **TB6612FNG** uses modern MOSFETs with low on-resistance ($R_{\text{DS(on)}} \approx 0.5\,\Omega$). It drops roughly 0.5V, stays cool, and runs on a much smaller board footprint.
- **Extracting Datasheet Specifications:** When evaluating driver ICs for your BOM, identify three critical parameters:
  1. *Motor Supply Voltage ($V_M$ / $V_{CC2}$):* Must safely exceed your battery's maximum fully charged voltage.
  2. *Continuous Current per Channel:* Must exceed the motor's operating current under normal forward drive.
  3. *Logic Input Voltage ($V_{CC}$ / $V_{\text{IN}}$):* Must recognize 3.3V logic levels from the ESP32.
- **Common Ground Topology:** You must connect the battery negative terminal, ESP32 GND, and motor driver GND together. Voltage is a potential difference between two conductors; without a shared ground reference, logic signals float and cause random motor triggers.
- **Diagnosing Controller Brownout Resets:** If your ESP32 reboots the moment motors start, the cause is supply voltage sag. When motors start from a dead stop, they pull their full stall current. This momentary current surge causes the battery voltage to dip below the dropout voltage of the 3.3V regulator, triggering the ESP32's hardware brownout detector.
  - *Mitigation:* Place a bulk electrolytic capacitor (100–470µF) across the motor driver power inputs, solder 0.1µF ceramic noise-suppression capacitors across motor terminals, and isolate logic power tracks from motor power tracks.

### Curated Study Material

- **Video — TB6612FNG Dual Motor Controller with Arduino (DroneBot Workshop):**
  [Watch on YouTube](https://www.youtube.com/watch?v=di_GkF_GqO4) / [Read Written Guide](https://dronebotworkshop.com/tb6612fng-h-bridge/)  
  *Why study this:* Details pin assignments (`PWMA`, `AIN1`, `AIN2`, `STBY`), demonstrates breadboard wiring, and directly contrasts performance against the older L298N.
- **Video — L298N Motor Driver with Arduino (DroneBot Workshop):**
  [Watch on YouTube](https://www.youtube.com/watch?v=da_vX_DeRJ8)  
  *Why study this:* Covers the on-board 5V regulator jumper, common grounding, and dual-motor wiring.
- **Guide — How to Read a Datasheet (SparkFun Learn):**
  [Read Tutorial](https://learn.sparkfun.com/tutorials/how-to-read-a-datasheet)  
  *Why study this:* Explains where to find maximum current, operating voltage, pinouts, and thermal ratings without getting bogged down in manufacturing specs.
- **Application Note — Dealing with Motor Noise (Pololu):**
  [Read Application Note](https://www.pololu.com/docs/0J15/9)  
  *Why study this:* Explains why brushed DC motors emit high-frequency electrical noise and how to install 0.1µF ceramic snubber capacitors.
- **Recommended Component Suppliers (India):**
  - [Robu.in](https://robu.in/) — ESP32 boards, N20 metal gearmotors, TB6612FNG breakout boards, sensor arrays, chassis kits.
  - [ElectronicsComp.com](https://www.electronicscomp.com/) — Passives, capacitors, connectors, wiring accessories.

---

## 3. Batteries, Power Budgets & Runtime

**Applies to:** Task 5 (Power Budget & Battery Selection).

### Essential Technical Concepts

- **Runtime Calculation:**
  $$\text{Runtime (hours)} = \frac{\text{Battery Capacity (mAh)}}{\text{Average System Current (mA)}} \times \text{Derating Factor (0.75)}$$
  *Example:* A system drawing an average of 450mA powered by a 1200mAh pack provides an estimated real-world runtime of:
  $$\frac{1200\,\text{mAh}}{450\,\text{mA}} \times 0.75 \approx 2.0\,\text{hours}$$
- **Current Summation Categories:**
  - *Idle Current:* ESP32 active (~80mA) + IR sensor array LEDs (~25mA) + driver quiescent current (~5mA) $\approx 110\,\text{mA}$.
  - *Nominal Running Current:* Logic current + two DC motors operating on a flat surface (~150mA to 300mA each) $\approx 410\text{--}710\,\text{mA}$.
  - *Worst-Case Stall Current:* Logic current + two stalled motors (~1.0A to 1.5A each) $\approx 2.1\text{--}3.1\,\text{A}$.
- **C-Rating and Peak Discharge:** C-rating indicates maximum continuous discharge current relative to capacity. A 1000mAh battery with a 25C rating can safely supply:
  $$1.0\,\text{Ah} \times 25\text{C} = 25.0\,\text{Amps}$$
- **Battery Chemistry Comparison:**
  - **Lithium Polymer (LiPo pouch):** High C-ratings (20C–50C), very low internal resistance, high power-to-weight ratio. Vulnerable to physical puncture.
  - **Lithium-Ion (18650 cylindrical cells):** Durable steel housing, higher energy density, but lower C-ratings (typically 1C–3C for budget cells) and heavier overall weight.

### Curated Study Material

- **Video — Battery Type Comparison (GreatScott!):**
  [Watch on YouTube](https://www.youtube.com/watch?v=AUZIYQFe4KgCjhGKrfDADYplGflGs49ZQqgIZF8HZGUaeLItlrUVGFGQPLdNgeKibN1-RNwSgdcB1TXeoTSYCn4n1PUep1XPOYgJ4uDJuFDRkur7u03H1dKyAMnSuYWxv63bam2q)  
  *Why study this:* Practical lab comparison showing weight, capacity, and current discharge capabilities across different battery chemistries.
- **Reference — Battery University (Isidor Buchmann):**
  - [BU-402: What Is C-rate?](https://batteryuniversity.com/article/bu-402-what-is-c-rate)  
  - [BU-205: Types of Lithium-ion](https://batteryuniversity.com/article/bu-205-types-of-lithium-ion)  
  *Why study this:* Provides clear explanations of discharge rates, nominal voltages, and safety guidelines.

---

## 4. Maze Solving & Wireless Telemetry

**Applies to:** Task 7 (Wireless Sensor Diagnostics), Task 8 (Maze-Solving Algorithms).

### Essential Technical Concepts

#### A. Maze Traversal & Path Simplification

- **Exploration Heuristic (The Left-Hand Rule):** On simple mazes without loops, an autonomous robot can reach the exit by following a strict priority at every intersection:
  1. Turn Left if a path exists.
  2. Else go Straight if a path exists.
  3. Else turn Right if a path exists.
  4. Else Turn Around (Dead End).
- **Recording Traversal Sequences:** As the robot explores, it stores each intersection turn in memory using single-character codes (`L` for Left, `R` for Right, `S` for Straight, `B` for Back/Dead End).
- **Algebraic Path Reduction for the Fast Run:** Entering and exiting a dead end always produces an unnecessary loop containing `B`. Your code simplifies these turn sequences before the replay run:
  - `L` + `B` + `L` $\rightarrow$ `S` (Left into dead end, back, then left out = Straight)
  - `L` + `B` + `S` $\rightarrow$ `R` (Left into dead end, back, then straight out = Right)
  - `L` + `B` + `R` $\rightarrow$ `S`
  - `S` + `B` + `L` $\rightarrow$ `R`
  - `S` + `B` + `S` $\rightarrow$ `B`
  - `R` + `B` + `L` $\rightarrow$ `B`

#### B. Wireless Real-Time Diagnostics

- **Protocol Selection on ESP32:**
  - **Bluetooth Low Energy (BLE) or Bluetooth Serial:** Lowest implementation overhead and minimal battery penalty compared to running full Wi-Fi network stacks. Connects directly to mobile devices or laptops without an external router.
- **Distinguishing Physical Bumps from Line Changes:**
  - *Legitimate line shift:* Analog values transition smoothly across neighboring sensors ($S_2 \rightarrow S_3 \rightarrow S_4$) due to physical geometry.
  - *Intermittent hardware fault (e.g., loose pin or track bump):* A single sensor channel drops abruptly to zero or pegs to 4095 while adjacent sensors show steady readings.
- **Data Transmission Format:** Transmit clean, comma-separated frames over Serial or Bluetooth:

  ```cpp
  Serial.printf("%d,%d,%d,%d,%d\n", s1, s2, s3, s4, s5);
  ```

### Curated Study Material

- **Guide — Autonomous Maze-Solving Robot (Instructables):**
  [Read Guide](https://www.instructables.com/Autonomous-Maze-Solving-Robot/)  
  *Why study this:* Provides clear diagrams explaining intersection detection, junction geometry, and turn-string simplification.
- **Reference — Pololu 3pi Maze Solving Guide (Sections 8.a through 8.e):**
  [Read Documentation](https://www.pololu.com/docs/0J21/8.a)  
  *Why study this:* The standard reference code for implementing Left-Hand-on-the-Wall rules and dead-end string reduction.
- **Tool — Serial Bluetooth Terminal (Android App):**
  [View on Google Play](https://play.google.com/store/apps/details?id=de.kai_morich.serial_bluetooth_terminal)  
  *Why study this:* Simple utility for connecting an Android phone to an ESP32 Bluetooth stream to view and log live telemetry.
- **Guide — Arduino IDE Serial Plotter Tutorial:**
  [Read Tutorial](https://docs.arduino.cc/software/ide-v2/tutorials/ide-v2-serial-plotter/)  
  *Why study this:* Explains how formatting serial output with commas automatically renders live, multi-channel line graphs on a PC.

---


# RoboReach — Mobile Robot with Robotic Arm

## Overview

RoboReach is a mobile robot designed to navigate an obstacle course and pick up objects using a robotic arm and a parallel-jaw gripper. The project combines mechanical design, electronics, power management, wireless control, and safety features to create a robot capable of performing pickup and navigation tasks.

## Project Structure

```text
Solution/
├── TASK 1.docx
├── Task2.docx
├── Task 3/
│   ├── Task 3.docx
│   └── Task 3_ckt design.svg
└── Task4/
    ├── Assembly 1 (1).step
    └── RoboReach_EXPLODED (1).step
```

## Features

- **Four-wheel drive:** Skid-steer mechanism with four geared DC motors for mobility and stability.
- **Robotic arm:** Multi-joint arm designed to reach objects up to 450 mm above the ground.
- **Parallel-jaw gripper:** Rack-and-pinion mechanism designed to grip cubes, cylindrical objects, cones, markers, and loop handles.
- **Wireless control:** ESP32-based controller for receiving commands.
- **Motor control:** L298N dual H-bridge motor driver for the drive motors.
- **Power management:** Separate regulated power rails for the servos and logic electronics.
- **Safety mechanism:** Planned communication-loss response, gripper position holding, fuse protection, and an accessible kill switch.
- **3D CAD models:** STEP files for the robot assembly and exploded view.

## Hardware Components

The proposed hardware includes:

- ESP32 DevKit V1
- L298N motor driver module
- Four 12 V geared DC motors
- Four MG996R servos for the robotic arm
- One MG90S micro servo for the gripper
- 6 V and 5 V DC-DC buck converters
- 3S 11.1 V LiPo battery
- Fuse, kill switch, wiring, connectors, and supporting components

## Design Specifications

| Parameter | Specification |
|---|---|
| Starting envelope | 300 × 200 × 300 mm |
| Required object-centre reach | 450 mm above ground |
| Minimum ground clearance | 50 mm |
| Wheel diameter | 100 mm |
| Approximate chassis footprint | 250 × 200 mm |
| Gripper opening | Approximately 7–76 mm |
| Maximum permitted voltage | 24 V DC |
| Estimated project budget | ₹11,825, including estimated shipping and contingency |

## Project Documentation

### Task 1 — Requirements and Safety
Covers size and reach constraints, communication-loss safety, battery voltage limits, and the example obstacle-course score calculation.

### Task 2 — Mechanical Design
Explains the selection of the drivetrain, robotic arm, and gripper, including design comparisons and sizing calculations.

### Task 3 — Circuit Design and Power Budget
Contains the circuit diagram, electronics bill of materials, power consumption estimates, battery selection, wiring considerations, and troubleshooting procedures.

### Task 4 — CAD Models
Contains the STEP assembly and exploded-view models for examining the robot's mechanical arrangement.

## Safety Considerations

The design uses a 3S LiPo battery with a maximum fully charged voltage of 12.6 V, below the project's 24 V limit.

The intended communication-loss behaviour is to stop the drive motors, prevent further arm movement, and maintain the gripper's last commanded position until a valid command is received.

**Important:** Verify the L298N current limits, motor stall current, power-converter ratings, wiring, fusing, and emergency-stop behaviour before operating the robot. The hardware and safety behaviour must be tested; the documentation alone does not establish that they work as intended.

## Current Status

The project package includes design documents, a circuit drawing, and 3D CAD files. Firmware source code and a software build configuration are not included in the supplied files.

## Future Improvements

- Implement and test the ESP32 wireless-control firmware.
- Validate motor-current protection and power stability.
- Test the robotic arm's reach and gripper performance.
- Verify safe behaviour during wireless disconnection.
- Integrate and test the complete robot on the obstacle course.

## Conclusion

RoboReach combines mobile navigation, robotic manipulation, electronics, and mechanical design in one project. Its documentation provides the basis for constructing, testing, and improving the robot.

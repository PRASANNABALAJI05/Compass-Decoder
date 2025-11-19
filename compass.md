## CompassDecoder

A simple **Verilog module** designed to decode digital compass directions based on raw X and Y sensor inputs from a 2-axis magnetometer. This repository includes both the core decoding logic and a testbench for comprehensive simulation.

---

## Overview

The `compassdecoder` module takes signed 2-axis magnetometer data (`x`, `y`) and activates a corresponding set of directional LEDs (8 total outputs, 2 per main direction).

* **Core Function:** Calculates the approximate **compass heading** using basic integer arithmetic, similar to the $\text{atan2}$ function, to determine the quadrant/octant.
* **Output:** Generates an 8-bit output (`direction_led`) that maps directly to the four cardinal directions (North, East, South, West).
* **Language:** Written in **Verilog HDL**.

---

##  Repository Contents

| File Name | Description |
| :--- | :--- |
| `compassdecoder.v` | The main **Verilog module** containing the decoding logic. |
| `compassdecoder_tb.v` | The **Testbench** used for simulating the module's behavior. |
| `image.jpg` | Sample simulation waveform output. |

---

##  Usage

### 1. Simulation

To verify the logic, use a standard Verilog simulator (e.g., **ModelSim**, **Vivado Simulator**, or **Icarus Verilog**).

The `compassdecoder_tb.v` file runs several test patterns, systematically varying the `x` and `y` inputs to cover all four main directions and boundary conditions. The resulting output is printed to the console.

### 2. Example Waveform

The following image shows a sample of the input signals (`x`, `y`) and the resulting `direction_led` output during simulation.



---

##  Direction LED Mapping

The 8-bit output (`direction_led`) is mapped as follows (using two adjacent bits per direction for visual clarity or redundancy).

| Direction | Condition | LED Output (8 bits) |
| :--- | :--- | :--- |
| **North** | $X \approx 0, Y > 0$ | `11000000` |
| **East** | $X > 0, Y \approx 0$ | `00110000` |
| **South** | $X \approx 0, Y < 0$ | `00001100` |
| **West** | $X < 0, Y \approx 0$ | `00000011` |
| **Undefined** | e.g., $X=0, Y=0$ | `00000000` |

> **Note:** The order of bits in the 8-bit output is typically mapped: `N1, N2, E1, E2, S1, S2, W1, W2`.

---

##  How to Extend

This module is designed to be a starting point. Here are a few ways it can be enhanced:

* **Accuracy:** Replace the current integer-math heading equation with a hardware-efficient algorithm like **CORDIC** for a more accurate $\text{atan2}$ approximation, or integrate a higher-resolution lookup table.
* **Integration:** Add modules to interface directly with an **I2C/SPI sensor** to provide the `x` and `y` inputs.
* **Hardware Mapping:** Map the `direction_led` output to actual physical LEDs on an **FPGA** or a custom PCB.

---

##  License

This project is open-source and available under the **MIT License**. See the `LICENSE` file for more details.

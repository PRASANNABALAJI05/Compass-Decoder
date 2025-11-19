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
## Verilog HDL Code
```
// CompassDisplay.v
module compassdecoder(
    input signed [15:0] x,    // Raw sensor X-axis input
    input signed [15:0] y,    // Raw sensor Y-axis input
    output reg [7:0] direction_led // LEDs for N, E, S, W (2 LEDs each)
);

// Internal heading representation 0-359 degrees
reg [8:0] heading; 

// Simple heading calculation (approximate arctan2)
always @(*) begin
    if (x == 0 && y == 0) 
        heading = 0; // No valid direction
    else if (x == 0)
        heading = (y > 0) ? 90 : 270;
    else begin
        if (x > 0 && y >= 0)
            heading = (y * 90) / x; // 0 to 90 degrees
        else if (x <= 0 && y > 0)
            heading = 180 - ((y * 90) / (-x)); // 90 to 180 degrees
        else if (x < 0 && y <= 0)
            heading = 180 + ((-y * 90) / (-x)); // 180 to 270 degrees
        else if (x >= 0 && y < 0)
            heading = 360 - (((-y) * 90) / x); // 270 to 360 degrees
        else
            heading = 0;
    end
end

// Direction decode based on heading angle
always @(*) begin
    case (1)
        (heading < 45 || heading >= 315): direction_led = 8'b11000000; // North (2 LEDs)
        (heading >= 45 && heading < 135): direction_led = 8'b00110000; // East  (2 LEDs)
        (heading >= 135 && heading < 225): direction_led = 8'b00001100; // South (2 LEDs)
        (heading >= 225 && heading < 315): direction_led = 8'b00000011; // West  (2 LEDs)
        default: direction_led = 8'b00000000; // Undefined
    endcase
end

endmodule

// Testbench for CompassDisplay
module compassdecoder_tb;

reg signed [15:0] x_tb;
reg signed [15:0] y_tb;
wire [7:0] direction_led_tb;

compassdecoder uut (
    .x(x_tb),
    .y(y_tb),
    .direction_led(direction_led_tb)
);

initial begin
    $monitor("Time=%0t | x=%0d | y=%0d | direction_led=%b", $time, x_tb, y_tb, direction_led_tb);
    
    // Test pattern generation
    x_tb = 0; y_tb = 0; #10;       // Undefined
    x_tb = 100; y_tb = 0; #10;     // East
    x_tb = 0; y_tb = 100; #10;     // North
    x_tb = -100; y_tb = 0; #10;    // West
    x_tb = 0; y_tb = -100; #10;    // South

    // Random test values
    repeat (10) begin
        x_tb = $random % 200 - 100; // -100 to 99
        y_tb = $random % 200 - 100;
        #10;
    end

    $finish;
end

endmodule

```
# Output

<img width="1915" height="1079" alt="PROJECT OP" src="https://github.com/user-attachments/assets/e0b337e6-41df-4ff2-9ac2-bfd33beb6074" />


##  How to Extend

This module is designed to be a starting point. Here are a few ways it can be enhanced:

* **Accuracy:** Replace the current integer-math heading equation with a hardware-efficient algorithm like **CORDIC** for a more accurate $\text{atan2}$ approximation, or integrate a higher-resolution lookup table.
* **Integration:** Add modules to interface directly with an **I2C/SPI sensor** to provide the `x` and `y` inputs.
* **Hardware Mapping:** Map the `direction_led` output to actual physical LEDs on an **FPGA** or a custom PCB.

---

##  License

This project is open-source and available under the **MIT License**. See the `LICENSE` file for more details.

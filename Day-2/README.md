# Verilog RTL Design & Synthesis Workshop - Day 2

This repository contains the materials and concepts for Day 2, focusing on *Timing Libraries, contrasting **Hierarchical vs. Flattened Synthesis* approaches, and establishing *Efficient Flip-Flop Coding Styles*.

-----

## ⏰ Timing Libraries and SKY130 PDK

The *Process Design Kit (PDK), specifically the open-source **SKY130*, provides the foundational models for IC design. The .lib file is critical as it contains all the timing and power characteristics of the physical standard cells (gates) that Yosys uses during synthesis.

### Decoding the Library Name

The file name sky130_fd_sc_hd__tt_025C_1v80.lib contains crucial information about the operating conditions:

| Component | Meaning | Description |
| :--- | :--- | :--- |
| *tt* | *Process Corner* | *Typical **T*ypical (standard transistor performance). |
| *025C* | *Temperature* | $\mathbf{25^\circ \text{C}}$ (room temperature operating condition). |
| *1v80* | *Voltage* | $\mathbf{1.8\text{V}}$ (the core operating voltage). |

The synthesis tool uses this library to ensure the resulting netlist meets timing specifications under these specific Process, Voltage, and Temperature (PVT) conditions.

### Exploring the .lib File

The .lib file is a plain text file containing detailed definitions of every standard cell, including:

  * *Pin Capacitances:* The load presented by each input pin.
  * *Timing Arcs:* The delay from an input pin transition to the corresponding output pin change (e.g., cell_delay, rise_transition, fall_transition).
  * *Boolean Function:* The logic performed by the cell.

You can inspect the contents using a text editor:

bash
gedit sky130_fd_sc_hd__tt_025C_1v80.lib


-----

## 🌳 Hierarchical vs. Flattened Synthesis

The synthesis approach significantly impacts optimization potential, runtime, and debuggability of the final netlist.

| Aspect | 🏗 Hierarchical Synthesis | 📏 Flattened Synthesis |
| :--- | :--- | :--- |
| *Definition* | Synthesizes and maintains the module structure defined in the RTL. | Collapses all modules into a single, unified netlist. |
| *Optimization* | Limited to *module-level* (local optimizations). Cross-module paths are harder to optimize. | *Whole-design* (global optimizations) are maximized, potentially leading to better area/speed. |
| *Runtime* | *Faster* for large designs due to modular, separate processing. | *Slower* for large designs; higher memory usage. |
| *Debugging* | *Easier* as the gate netlist preserves the RTL module names, allowing direct trace-back. | *Harder* due to the loss of RTL module boundaries. |
| *Yosys Command* | Uses the hierarchy command implicitly/explicitly. | Uses the flatten command. |
| *Use Case* | Complex, multi-team projects; easier integration with formal verification and timing analysis tools. | Small-to-medium designs where maximum optimization is the primary goal. |

-----

## 💾 Efficient Flip-Flop Coding Styles

Flip-flops (FFs) are the essential *sequential elements* that store state. How you describe their reset behavior in Verilog determines how the synthesis tool implements them.

### 1\. Asynchronous Reset D Flip-Flop

The reset signal (async_reset) is included in the always sensitivity list and overrides the clock, resetting the FF immediately.

verilog
module dff_asyncres (input clk, input async_reset, input d, output reg q);
  always @ (posedge clk, posedge async_reset) // Reset is asynchronous
    if (async_reset)
      q <= 1'b0; // Immediate reset
    else
      q <= d;
endmodule


### 2\. Asynchronous Set D Flip-Flop

Similar to the reset, the set signal (async_set) is asynchronous and immediately sets the FF output to '1'.

verilog
module dff_async_set (input clk, input async_set, input d, output reg q);
  always @ (posedge clk, posedge async_set) // Set is asynchronous
    if (async_set)
      q <= 1'b1; // Immediate set
    else
      q <= d;
endmodule


### 3\. Synchronous Reset D Flip-Flop

The reset signal (sync_reset) is *not* in the sensitivity list. The reset only takes effect on the active clock edge, ensuring all state changes are synchronized.

verilog
module dff_syncres (input clk, input sync_reset, input d, output reg q);
  always @ (posedge clk) // Reset is synchronous
    if (sync_reset)
      q <= 1'b0; // Reset takes effect only on posedge clk
    else
      q <= d;
endmodule


-----

## 💻 Simulation and Synthesis Workflow

The general design flow involves simulation for functional verification, followed by synthesis to generate the gate-level implementation.

### Icarus Verilog Simulation

Use iverilog to verify the functionality of the coded flip-flops before synthesis.

bash
# 1. Compile
iverilog dff_asyncres.v tb_dff_asyncres.v 
# 2. Run
./a.out 
# 3. View Waveform
gtkwave tb_dff_asyncres.vcd


### Yosys Synthesis for Flip-Flops

Synthesis requires an additional step, **dfflibmap**, to ensure the Verilog-defined flip-flop is correctly mapped to a physical FF cell from the library that supports the required reset/set configuration.

bash
yosys

# 1. Read Liberty Library
read_liberty -lib /address/to/your/sky130/file/sky130_fd_sc_hd__tt_025C_1v80.lib

# 2. Read Verilog Code
read_verilog /path/to/dff_asyncres.v

# 3. Synthesize and Optimize
synth -top dff_asyncres

# 4. Map Flip-Flops to Library Cells (crucial for sequential elements)
dfflibmap -liberty /address/to/your/sky130/file/sky130_fd_sc_hd__tt_025C_1v80.lib

# 5. Technology Mapping (maps logic to standard gates)
abc -liberty /address/to/your/sky130/file/sky130_fd_sc_hd__tt_025C_1v80.lib

# 6. Visualize the Final Gate-Level Netlist
show

day2

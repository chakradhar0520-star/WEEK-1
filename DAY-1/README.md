# Verilog RTL Design & Synthesis Workshop - Day 1

This repository contains the materials and labs for Day 1 of the RTL Workshop, covering the fundamentals of *Verilog RTL design, **open-source simulation* with *Icarus Verilog (iverilog), and the basics of **logic synthesis* using *Yosys*.

-----

## 🚀 Workshop Overview

Today's focus is on building a strong foundation in digital design by:

  * Understanding the roles of a *Simulator, **Design, and **Testbench*.
  * Running a practical simulation using the open-source toolchain.


-----

## 🧪 Lab 1: Simulating a 2-to-1 Multiplexer

This lab walks you through the complete simulation flow for a simple 2-to-1 Multiplexer (good_mux.v).

### Verilog Code Analysis (good_mux.v)

The design uses an always @ (*) block for combinatorial logic and an if-else structure to describe the multiplexer functionality:

verilog
module good_mux (input i0, input i1, input sel, output reg y);
    always @ (*) begin
        if(sel)
            y <= i1;
        else 
            y <= i0;
    end
endmodule


*Logic:*

  * *Inputs:* i0, i1 (data inputs), sel (select line).
  * *Output:* y (the selected output).
  * *Function:* If sel is $\mathbf{1}$, output $\mathbf{y}$ is connected to $\mathbf{i1}$. If sel is $\mathbf{0}$, output $\mathbf{y}$ is connected to $\mathbf{i0}$.

### Simulation Steps

1.  *Compile:* Use iverilog to compile both the design (good_mux.v) and the testbench (tb_good_mux.v). This produces an executable file, usually named a.out.

    bash
    iverilog good_mux.v tb_good_mux.v
    

2.  *Run Simulation:* Execute the compiled file. This runs the testbench logic, which generates test inputs and writes the resulting signals to a Value Change Dump (.vcd) file.

    bash
    ./a.out
    

3.  *View Waveform:* Use *GTKWave* to open the generated .vcd file and visualize the signals (inputs i0, i1, sel, and output y).

    bash
    gtkwave tb_good_mux.vcd
    

-----
<img width="1135" height="758" alt="Image" src="https://github.com/user-attachments/assets/d1fb1fdb-af3b-4f3f-88a1-3942096daed9" />
## 🔩 Introduction to Yosys & Logic Synthesis

*Yosys* is the open-source tool we use to convert the behavioral Verilog RTL code into an actual hardware blueprint, known as a *gate-level netlist*.

### Key Synthesis Concepts

  * *Synthesis:* The process of translating HDL (Verilog) into a logic circuit using standard cells.
  * *Gate-Level Netlist:* The final output of synthesis, a description of how standard library gates (AND, OR, NOT, etc.) are connected to implement the design.
  * *Technology Mapping:* The step where the abstract logic is mapped to specific, physical gate "flavors" from a *Gate Library* (.lib file).

### Why Do Gate Libraries Have "Flavors"?

A *gate library* (.lib file) contains multiple versions (or "flavors") of the same basic gate (e.g., an AND gate). The synthesis tool selects the most appropriate flavor based on design constraints:

  * *Performance:* Faster, higher-power gates for critical speed paths.
  * *Power:* Slower, low-power gates for non-critical paths to save energy.
  * *Area:* Smaller gates for compact layouts.
  * *Drive Strength:* Gates with different output strengths to drive varying loads (fan-out).

-----

## ⚙ Lab 2: Synthesis with Yosys

This lab demonstrates the fundamental synthesis flow for the good_mux design using the *Yosys* tool and a sample *sky130* gate library.

*Note:* Ensure you have the correct path to the *sky130* library file (sky130_fd_sc_hd__tt_025C_1v80.lib).

### Yosys Synthesis Flow

Start the Yosys command-line interface:

bash
yosys


Execute the following commands within the Yosys prompt:

1.  *Read Gate Library:* Loads the timing and physical characteristics of the target technology's standard cells.

    tcl
    read_liberty -lib /address/to/your/sky130/file/sky130_fd_sc_hd__tt_025C_1v80.lib
    

2.  *Read Verilog Design:* Imports the RTL code to be synthesized.

    tcl
    read_verilog /home/vsduser/VLSI/sky130RTLDesignAndSynthesisWorkshop/verilog_files/good_mux.v
    

3.  *Synthesize:* Converts the RTL into a generic logic structure, performs initial optimizations, and selects the top module.

    tcl
    synth -top good_mux
    

4.  *Technology Mapping (ABC):* Maps the generic logic to specific physical gates from the loaded library.

    tcl
    abc -liberty /address/to/your/sky130/file/sky130_fd_sc_hd__tt_025C_1v80.lib
    

5.  *Visualize:* Generates a graphical representation of the resulting gate-level netlist.

    tcl
    show

     <img width="872" height="721" alt="Image" src="https://github.com/user-attachments/assets/accc706c-847b-4d6d-9822-49e63d9292b8" /> 

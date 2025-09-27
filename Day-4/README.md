# Verilog RTL Design & Synthesis Workshop - Day 4

This day focuses on crucial post-synthesis verification, understanding the impact of *Blocking vs. Non-Blocking assignments, and recognizing sources of **Synthesis-Simulation Mismatch*.

-----

## 1\. Gate-Level Simulation (GLS)

*Gate-Level Simulation (GLS)* is a verification step performed after the synthesis tool has converted the Register Transfer Level (RTL) code into a *gate-level netlist*.

### Purpose of GLS

| Verification Aspect | Description |
| :--- | :--- |
| *Synthesis Validation* | Confirms that the netlist functionally matches the original RTL description. |
| *Timing Verification* | Checks the circuit's operation with real-world *delays* (annotated from an *SDF* file), allowing detection of setup/hold time violations. |
| *Testability Check* | Verifies that design-for-test (DFT) structures, like scan chains, are correctly implemented and functional. |
| *Power Estimation* | Provides a more accurate estimate of power consumption using gate-level activity. |

### GLS Flow

1.  *Synthesize* the RTL into a netlist (.v).
2.  *Extract Timing:* Perform Physical Design (Place and Route) to get wire and cell delays.
3.  *Generate SDF (Standard Delay Format) file:* Contains all timing information.
4.  *Run Simulation:* Simulate the netlist (.v) along with the *SDF file* and the *technology library* (e.g., sky130_fd_sc_hd.v).

-----

## 2\. Synthesis-Simulation Mismatch

A *mismatch* occurs when the behavior observed in the RTL simulation differs from the behavior of the synthesized netlist (or the actual hardware).

### Common Causes

| Cause | Description | Best Practice |
| :--- | :--- | :--- |
| *Non-Synthesizable Constructs* | Using initial blocks, #delays, $display (outside of the testbench), or other constructs not supported by hardware. | Limit non-synthesizable code to the testbench. |
| *Ambiguous Combinational Logic* | Forgetting an **else clause** or failing to assign an output in all possible conditions, leading to unintended latches. | Always use always @(*) and assign all outputs in every branch of if/case statements. |
| *Improper Assignment Usage* | Using *non-blocking* assignments (<=) for combinational logic or *blocking* assignments (=) for sequential logic (the most common cause). | Follow the recommended guidelines (Section 3). |
| *Incomplete Sensitivity List* | Using always @(a, b) instead of the automatic always @(*) in combinational blocks. | Use always @(*) exclusively for combinational logic. |

-----

## 3\. Blocking vs. Non-Blocking Assignments in Verilog

The correct usage of assignment operators is fundamental to writing synthesizable and unambiguous RTL.

| Feature | Blocking Assignment ($=$) | Non-Blocking Assignment ($\mathbf{<=}$) |
| :--- | :--- | :--- |
| *Operator* | = (Equals sign) | \<= (Less-than-or-equals sign) |
| *Execution* | *Sequential/Immediate:* Updates happen instantly within the always block's execution order. | *Concurrent/Scheduled:* Updates are scheduled to occur simultaneously at the end of the current time step. |
| *Hardware Inference* | *Combinational Logic* (gates, wires, temporary variables). | *Sequential Logic* (flip-flops, registers). |
| *Recommended Use* | **always @(*)** blocks for combinational circuits. | **always @(posedge clk)** blocks for sequential circuits. |
| *Caveat (Blocking)* | *Order matters\!* Assignments must be ordered correctly, as shown in Lab 6. | *Required* to prevent race conditions in sequential logic. |

-----

## 4\. Labs: Pitfalls and Best Practices

### Lab 4 & 5: Bad MUX Example (Mismatch)

The **bad_mux** code demonstrates two classic mismatch pitfalls:

1.  *Incomplete Sensitivity List:* always @(sel) misses inputs i0 and i1. In simulation, this leads to a mismatch (output doesn't update when i0 or i1 changes). Synthesis tools usually ignore the list and treat it as always @(*).
2.  *Incorrect Assignment:* Using non-blocking (<=) in a combinational block. The synthesis tool correctly infers a *latch* or *combinational logic*, but the simulation behavior may not accurately reflect the intended combinational behavior.

### Lab 6 & 7: Blocking Assignment Caveat

The **blocking_caveat** module highlights why *order matters* for blocking assignments:

*Incorrect Code (Original):*

verilog
always @ (*) begin
    d = x & c; // d uses the old value of x
    x = a | b; // x is updated after d is calculated
end


*Corrected Code:*

verilog
always @ (*) begin
    x = a | b; // x is updated first
    d = x & c; // d now uses the newly calculated value of x
end


*Conclusion:* When using blocking assignments for combinational logic, variables must be assigned before they are used by subsequent statements in the same always block.

-----

## 📝 Synthesis and GLS Flow Summary

The following command demonstrates the overall flow for verifying the synthesized netlist (Lab 3):

bash
# General GLS Command (assuming paths are set)
iverilog <path>/primitives.v <path>/sky130_fd_sc_hd.v synthesized_netlist.v testbench.v
# Run simulation (generates .vcd)
./a.out
# View waveform
gtkwave tb_mux.vcd


*Always* ensure that the final gate-level simulation result matches the initial RTL simulation result.
day4

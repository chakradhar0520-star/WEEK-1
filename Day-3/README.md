# Verilog RTL Design & Synthesis Workshop - Day 3

This repository covers Day 3 of the workshop, focusing on critical *Optimization Techniques* for both combinational and sequential circuits to improve chip *performance, **area, and **power efficiency*.

-----

## 💡 Core Optimization Techniques

Modern synthesis tools like Yosys employ numerous strategies to refine the RTL netlist. Here are four fundamental techniques:

### 1\. Constant Propagation

*Concept:* A compiler optimization where variables known to have a *constant value* are replaced by that constant throughout the code.

*Mechanism:* The synthesis tool analyzes the data flow and simplifies the logic connected to the constant value.

*Benefits:*

  * *Reduced Logic:* Simplifies complex gates (e.g., an AND gate with an input tied to '0' becomes '0').
  * *Area & Power Savings:* Fewer gates and simpler routing are required.
  * *Performance:* Reduced path delays through the simplified logic.

### 2\. State Optimization

*Concept:* Techniques used to enhance the efficiency of *Finite State Machines (FSMs)*.

*Mechanisms:*

  * *State Reduction:* Merging states that have equivalent functionality, minimizing the total number of states.
  * *State Encoding:* Assigning binary codes to the remaining states optimally (e.g., one-hot, binary) to minimize the required control logic.
  * *Logic Minimization:* Using algorithms (like Karnaugh maps or Quine-McCluskey) to simplify the next-state and output logic equations.

### 3\. Cloning

*Concept:* Duplicating a logic cell or module to *balance load* or *reduce delay* on a critical path.

*Mechanism:* When a single driver (output) is connected to a large number of loads (inputs), it can slow down the signal. Cloning the driver and splitting the loads across the two outputs reduces the *fan-out* for each driver, improving the overall timing.

*Goal:* To mitigate high fan-out issues and reduce parasitic capacitance from long, heavily-loaded wires.

### 4\. Retiming

*Concept:* A sequential optimization technique that moves the position of *registers (flip-flops)* in a circuit without altering its functional behavior.

*Mechanism:*

  * Registers are moved across combinational logic blocks.
  * The total number of registers along any functional path remains constant.

*Goal:* To minimize the *clock period* by balancing the delay of combinational logic between registers, ensuring no single path is excessively long (the critical path).

-----

## 🔬 Practical Optimization Labs (Verilog Code Analysis)

The following labs demonstrate how simple Verilog constructs can be optimized by synthesis tools, often resulting in much simpler gates than initially expected.

### Combinational Logic Optimization

| Lab | Verilog Code | RTL Functionality | Optimized/Resulting Logic | Optimization Concept |
| :--- | :--- | :--- | :--- | :--- |
| *Lab 1* | assign y = a ? b : 0; | If $a=1, y=b$; If $a=0, y=0$. | $\mathbf{y = a \text{ AND } b}$ | Logic Simplification |
| *Lab 2/3* | assign y = a ? 1 : b; | If $a=1, y=1$; If $a=0, y=b$. | $\mathbf{y = a \text{ OR } b}$ | Logic Simplification |
| *Lab 4* | assign y = a ? (b ? (a & c) : c) : (!c); | Simplifies to $y = a \cdot c + \bar{a} \cdot \bar{c}$ | $\mathbf{y = a \text{ XNOR } c}$ | Boolean Algebra / Logic Simplification |

*Yosys Optimization Command:* For combinational labs, the opt_clean -purge command is often inserted in the synthesis flow to aggressively simplify the logic:

tcl
read_verilog <file>.v
synth -top <module>
opt_clean -purge # Aggressive simplification and cleanup
abc -liberty ...

<img width="872" height="656" alt="Image" src="https://github.com/user-attachments/assets/85ba3ecb-0681-446a-8ff6-fad320742ede" />


<img width="904" height="819" alt="Image" src="https://github.com/user-attachments/assets/9632ac64-1631-4fb1-8de4-c0dfd7401d13" />
-----

### Sequential Logic (Flip-Flop) Optimization

These labs illustrate how the synthesis tool handles flip-flops that are configured to load a constant value.

| Lab | Verilog Code (Key Logic) | RTL Functionality | Optimized/Resulting Logic | Optimization Concept |
| :--- | :--- | :--- | :--- | :--- |
| *Lab 5* | else q <= 1'b1; | *Asynchronous Reset* to 0; *Synchronously* loads $\mathbf{1}$. | If the FF always loads '1' (except during reset), it is simplified to a *Set-Reset Latch* or a simple *wire* if the reset is tied off. | Constant Propagation |
| *Lab 6* | q <= 1'b1; and q <= 1'b1; | *Asynchronously Reset* to 1; *Synchronously* loads $\mathbf{1}$. | The output *q is always 1. The sequential element (FF) is **removed* and replaced by a simple $\mathbf{1}$ (VCC wire). | Constant Propagation (Full removal) |

<img width="888" height="868" alt="Image" src="https://github.com/user-attachments/assets/63630e2a-cb63-4cbb-a659-ac323df44e9d" />

<img width="858" height="943" alt="Image" src="https://github.com/user-attachments/assets/544201f1-5f70-44f0-a3cb-1cc3e7eaa056" />

# Verilog RTL Design & Synthesis Workshop - Day 5

This day focuses on writing *scalable and correct RTL code* by mastering *conditional statements* (if-else and case), utilizing *constructs for repetition* (for loops and generate blocks), and strictly avoiding *inferred latches* during synthesis.

-----

## 1\. Conditional Statements & Latch Avoidance

### If-Else Statements and Latch Inference

In combinational logic (always @(*)), the synthesis tool infers a *latch* when an output variable is *not assigned a value* in all possible branches of a conditional statement. Latches are generally undesirable as they can introduce difficult-to-analyze timing hazards.

| Lab Example | Code Snippet | Implied Logic When Condition Fails | Synthesis Result |
| :--- | :--- | :--- | :--- |
| *Lab 1: Incomplete If* | if (i0) y <= i1; | When $i0=0$, y must *hold its previous value. | **Inferred Latch*  |
| *Lab 3: Nested If-Else* | if (i0) y <= i1; else if (i2) y <= i3; | When $i0=0$ *and* $i2=0$, y is *not assigned. | **Inferred Latch* |
| *Lab 5: Complete Case* | case(sel) ... default : y = i2; | All sel values (including unlisted ones) are handled by default. | *Pure Combinational Logic* (MUX) |

### Solution: Always Assign a Default Value

To guarantee pure combinational logic (wires and gates) and avoid latches, ensure the variable is assigned:

1.  **Use a default case** in case statements.
2.  **Use an else branch** in if statements.
3.  *Use a pre-assignment* before the if/case structure.

*Corrected Example (Pre-assignment):*

verilog
always @(*) begin
    y = 1'b0; // Default assignment
    if (sel == 1'b1)
        y = a;
end


### Case Statement Pitfalls (Lab 7 & 8)

  * *Lab 7 (Incomplete Case):* If the case statement does not cover all possible values of the select signal and lacks a default case, a latch is inferred for the outputs not assigned for the missing values.
  * *Lab 8 (Partial Assignments):* If a variable (x or y) is not assigned in every case item, a latch is inferred for that specific variable. *All variables must be assigned in all branches.*

-----

## 2\. Scalable Hardware Generation

### For Loops in Verilog

A for loop executes statements multiple times. When used in synthesis, it serves as an efficient way to *replicate logic* a fixed number of times.

*Synthesizability Rule:* The loop bounds (*initialization* and *condition) must be **constant* and resolvable at compile time.

*Lab 9 (4-to-1 MUX):*
The for loop iterates exactly 4 times, generating the hardware equivalent of four logic branches (or a single $\mathbf{4}$-to-$\mathbf{1}$ *MUX*) within a combinational block.

verilog
// Effectively generates a 4-to-1 MUX structure:
// if (0 == sel) y = i_int[0];
// else if (1 == sel) y = i_int[1];
// ... and so on.
for (k = 0; k < 4; k = k + 1) begin
    if (k == sel)
        y = i_int[k];
end


### Generate Blocks

The generate block is a powerful structural construct used to instantiate modules or create logic conditionally and iteratively at *compile time*. It is ideal for creating regular, scalable hardware architectures.

*Keywords:*

  * genvar: Must be used for the loop variable inside a generate block.
  * generate ... endgenerate: Encloses the conditional or iterative code.
  * : gen_loop: A required label for the loop.

### Ripple Carry Adder (RCA) with Generate (Lab 12)

The *Ripple Carry Adder (RCA)* is a binary adder built by chaining multiple *Full Adder (FA)* modules, where the carry-out ($C_{out}$) of one FA feeds the carry-in ($C_{in}$) of the next.

*Lab 12* uses a generate block to instantiate *7* identical FA modules, creating a scalable 8-bit adder structure.

verilog
// Instantiates 7 FAs, connecting the carry chain:
for (i = 1; i < 8; i = i + 1) begin
    // Carry-out from previous stage (i-1) is carry-in to current stage (i)
    fa u_fa_1 (.c(int_co[i-1]), .co(int_co[i]), ...); 
end


This demonstrates how generate blocks replace tedious, manual instantiation of repetitive logic.
day 5

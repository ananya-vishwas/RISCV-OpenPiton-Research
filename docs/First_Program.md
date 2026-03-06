# Creating a Custom Instruction for custom -3 Factorial program

## PHASE - 1 HARDWARE DESIGNING
## Step 1: Update the Decoder

You must modify the decoder to recognize the Custom-3 opcode ($01111011$ or $0x7B$)
In the directory : openpiton/piton/design/chip/tile/ariane/core/

File: decoder.sv

Locate the main opcode case statement. You need to add a branch for riscv::OpcodeCustom3. This branch must signal the control logic to route the instruction to the correct functional unit and identify that it uses one source register (rs1) and one destination register (rd).

This is where to locate 
```verilog
if (~ex_i.valid) begin
            case (instr.rtype.opcode)
```
 under case (instr.rtype.opcode) (+1 identation add the following)

```verilog
riscv::OpcodeCustom3: begin
    instruction_o.fu        = ALU; 
    instruction_o.op        = ariane_pkg::FACT; 
    
    // Remove the "instruction_o." prefix here
    use_rs1_o               = 1'b1; 
    use_rd_o                = 1'b1;
    
    // Ensure this matches the fix from our last step
    illegal_instr           = 1'b0; 
end
```
The Next Critical Step: Defining riscv::FACT because the compiler doesn't know what that word means yet. We need to define it in the Ariane Package file so the whole processor understands this new operation.

In same directory, inside core/include find ariane_pkg.sv

Inside that file locate to : 
```verilog
 // ---------------
    // EX Stage
    // ---------------

    typedef enum logic [7:0] { // basic ALU op
```
Add FACT to the end of that list. It will look something like this:

```verilog
typedef enum logic [6:0] {
    ADD, SUB, MUL, ... ,
    FACT  // <--- Add this here!
} alu_op;
```
successfully mapped the Custom-3 major opcode to a specific functional unit (ALU) and assigned a unique internal operator signal (FACT) within the CVA6 instruction decode stage.

## Step 2 — ALU Implementation

You now need to modify the Arithmetic Logic Unit (ALU) to handle the new FACT operator.

Look for alu.sv in Directory: /home/misty/openpiton/piton/design/chip/tile/ariane/core/
Since factorials grow very quickly, a 64-bit register can only hold up to $20!$. For your initial test ($5!$), we can implement a simple combinational lookup or a small loop.

Locate : 
```verilog
    // -----------
    // Result MUX
    // -----------
    always_comb begin
        result_o   = '0;
        unique case (fu_data_i.operator)
```
Under Unique case add : 

```verilog
ariane_pkg::FACT: begin
            // Operand_a_i contains the number we want to factorial
            case (operand_a_i[3:0])
                4'h0: result_o = 64'd1;        // 0!
                4'h1: result_o = 64'd1;        // 1!
                4'h2: result_o = 64'd2;        // 2!
                4'h3: result_o = 64'd6;        // 3!
                4'h4: result_o = 64'd24;       // 4!
                4'h5: result_o = 64'd120;      // 5!
                default: result_o = 64'hBAD;   // Error flag for higher numbers
            endcase
        end
```
## PHASE - 2 Building the Hardware Model
Before you can build your new hardware, you need to source the OpenPiton environment settings. This will tell your terminal where the sims command lives.

Run these commands in your terminal:
Add the Piton tools directory directly to your PATH
```bash
export PATH=$PATH:~/openpiton/piton/tools/bin
which sims
```
Then now that sims is working, you must re-compile your processor model. This is the most important step today because it takes the Factorial logic you added to the ALU and actually puts it into the virtual chip.

Verification: Type which sims. If it returns a path inside your openpiton folder, you are ready to go.

Since you have modified the Verilog "blueprint" of the processor, you must re-compile the simulation executable. This takes your new alu.sv, decoder.sv, and ariane_pkg.sv and builds a new virtual chip.

Run this command in your terminal:
```bash
# 1. Manually set the main OpenPiton directory
export PITON_ROOT=~/openpiton

# 2. Add ALL possible tool directories at once
export PATH=$PATH:$PITON_ROOT/piton/tools/bin:$PITON_ROOT/piton/tools/x86_64/bin

# 3. Use the absolute path to source your settings
source $PITON_ROOT/piton/piton_settings.bash

# 4. Check if configsrch is actually there
ls $PITON_ROOT/piton/tools/bin/configsrch
```
Then run the sims command

```Bash
cd /home/misty/openpiton/build
sims -sys=manycore -vlt_build -ariane
```
Timing Fix: The --no-timing flag tells Verilator not to worry about the #20 delay found in the monitor files, which it usually dislikes in modern versions.

## STEP - 1 Run the Simulation
Now you can finally run your test program to see the hardware calculate $5! = 120$. Run this command in your terminal:

```Bash
cd ~/openpiton/build
sims -sys=manycore -vlt_run -ariane -vcd -asm_diag_name=fact_test.riscv
```
-vlt_run: Tells the script to execute the simulation we just built.-vcd: This is critical—it records all the "wires" so you can see them in GTKWave.

You need to create a fact_test.c file in piton/verif/diag/c folder

```c
int main() {
    int input = 5;
    int result;

    // This inline assembly triggers your custom FACT instruction (opcode 0x7b)
    // It takes 'input' from a register and puts the 'result' into another
    asm volatile (
        ".word 0x0005057b\n\t" 
        "mv %0, a0"
        : "=r" (result)
        : "r" (input)
        : "a0"
    );

    /? If result is 120 (0x78), the hardware works!
    if (result == 120) {
        return 0; // Success
    } else {
        return 1; // Failure
    }
}
```
Then compile: 

```bash
riscv64-unknown-elf-gcc -march=rv64imafdc -mabi=lp64d -static -mcmodel=medany \
-fvisibility=hidden -nostdlib -nostartfiles fact_test.c -o fact_test.riscv
```

Now you can finally run the simulation to verify your custom FACT logic. Run this command from your build directory:


```bash
# Create a custom assembly directory if it doesn't exist
mkdir -p /home/misty/openpiton/piton/verif/diag/assembly/fact_test/
```

# Move your binary there
```bash
mv /home/misty/openpiton/piton/verif/diag/c/fact_test.riscv /home/misty/openpiton/piton/verif/diag/assembly/fact_test/
```





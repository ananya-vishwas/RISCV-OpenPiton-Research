# Project Report: Side-Channel Prevention via FLUSHX
### Objective: Implement a hardware-based "Clean Slate" instruction in the Ariane (CVA6) RISC-V core to ensure temporal isolation and prevent secret data leakage.

define your flush command
<img width="761" height="905" alt="image" src="https://github.com/user-attachments/assets/12e638e1-a04b-4d55-9165-3e0c234e6bd6" />

Map its decoder: 

```verilog
riscv::OpcodeCustom3: begin //find this
                    // Assign the Functional Unit to the ALU
                    instruction_o.fu  = ariane_pkg::ALU;
                    // Assign the specific custom operator we defined in ariane_pkg
                    instruction_o.op  = ariane_pkg::ZKP_FLUSH;
                    
                    // Standard register mapping for a custom R-type instruction
                    instruction_o.rs1 = instr.rtype.rs1;
                    instruction_o.rs2 = instr.rtype.rs2;
                    instruction_o.rd  = instr.rtype.rd;
                    
                    // Ensure the instruction is marked as valid (not illegal)
                    illegal_instr = 1'b0;
end
```

In alu.sv
<img width="799" height="903" alt="image" src="https://github.com/user-attachments/assets/3517b2e0-6ac4-40f2-8ff2-341e037e6ea2" />

```verilog
ariane_pkg::ZKP_FLUSH:begin
                  // Securely wipe the ALU output to prevent side-channel leakage
                  result_o = 64'b0; 
                  // In a real FLUSHX, this would also signal the Cache to clear
end
```
Then bash these commands : 

```bash
# 1. Environment Setup
export PITON_ROOT=~/openpiton
export PATH=$PATH:$PITON_ROOT/piton/tools/bin:$PITON_ROOT/piton/tools/x86_64/bin
source $PITON_ROOT/piton/piton_settings.bash
export VERILATOR_FLAGS="--trace"

# 2. Clean stale metadata
cd $PITON_ROOT/build
rm -rf manycore/rel-0.1/obj_dir

# 3. Build the core
sims -sys=manycore -vlt_build -ariane
```
Now that the hardware is successfully built with the ZKP_FLUSH logic and tracing enabled, we need to write a C program to prove it works.

### The Goal: Run a calculation to put data in the ALU, then execute our custom ZKP_FLUSH instruction, and verify in the waveform that the data is physically wiped from the pipeline.




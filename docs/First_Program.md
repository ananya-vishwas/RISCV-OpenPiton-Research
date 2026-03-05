# Creating a Custom Instruction for custom -3 Factorial program

## Step 1: Update the Decoder

You must modify the decoder to recognize the Custom-3 opcode ($01111011$ or $0x7B$)
In the directory : openpiton/piton/design/chip/tile/ariane/core/

File: decoder.sv

Locate the main opcode case statement. You need to add a branch for riscv::OpcodeCustom3. This branch must signal the control logic to route the instruction to the correct functional unit and identify that it uses one source register (rs1) and one destination register (rd).


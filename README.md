# Environment Setup to clone and run an assembly program
For RISC V Artitecture, i will be using OpenPiton + Ariane core. It's a six stage pipeline and multicore

## Step 1 is to clone the Respositary : 

```bash
git clone https://github.com/PrincetonUniversity/openpiton.git
cd openpiton
```
Set the Root Variable: OpenPiton needs to know where its files are located at all times. This variable (PITON_ROOT) is used by every script in the framework to find design files and simulation tools.

```bash
export PITON_ROOT=`pwd`
```
## Step 2: Initialize the Environment
OpenPiton uses a 64-bit RISC-V core called Ariane. This requires a different set of settings than your previous Ibex projects.

Source the Main Settings:

```Bash
source $PITON_ROOT/piton/piton_settings.bash
```
Source the Ariane Settings:

```Bash
source $PITON_ROOT/piton/ariane_setup.sh
```
Why? These scripts add the necessary tools to your PATH and configure the simulator settings for the Ariane RV64IMAC core.
The image below shows sucessful integration
<img width="925" height="325" alt="image" src="https://github.com/user-attachments/assets/6d95acc6-d904-41ed-bb2a-946dc8962bda" />

## Step 3: Build the Tools (Crucial Step)
Before you can run a C program, you need a 64-bit RISC-V toolchain (compiler) and simulation helpers.

Run the Build Script: Downloading the entire RISC-V toolchain, compiling a C++ cross-compiler, and building Verilator, it is the "heavy lifting" part of the setup

```Bash
./piton/ariane_build_tools.sh
```
Why? This downloads and compiles the RISC-V cross-compiler so you can turn C code into machine code that Ariane understands.

You may run throught error, for best practice follow these verification steps : 

1. Check the Toolchain
Run this command in your terminal:

```Bash
riscv64-unknown-elf-gcc --version
```
If it prints a version: Great! The compiler is built and in your path. For me it was 13.2.0

If it says "command not found": The build failed or the path wasn't set. We need to check if the tools exist in the tmp folder created by the script.

2. Verify Ariane Environment Variables
OpenPiton needs specific variables to find the Ariane core you just built. Check them with:

```Bash
echo $ARIANE_ROOT
```
It should point to: /home/misty/openpiton/piton/design/chip/tile/ariane.

If it is empty, you must run:
```bash
source piton/ariane_setup.sh.
```
Next you can try the following verifications too : 
Re-source everything (just to be safe):

```Bash
cd /home/misty/openpiton
source piton/piton_settings.bash
source piton/ariane_setup.sh
```
I faced an error where the Verilator installation is "broken" or incomplete, likely because the script stopped early due to those previous errors.

How to fix this (Step-by-Step)
We need to manually finish the Verilator build so the system can find verilator_bin.

Navigate to the Verilator build folder:

```Bash
cd /home/misty/openpiton/piton/design/chip/tile/ariane/tmp/verilator-4.014/
```
Manually run the build commands:
Run these one by one. If any of these show "Error", tell me immediately.

```Bash
./configure
make -j$(nproc)
```
Note: nproc uses all your CPU cores to make it faster.

i ran into this error 
<img width="951" height="458" alt="image" src="https://github.com/user-attachments/assets/5201871d-1017-4476-8dd7-4df916f2ec1a" />

To fix this : 
```bash
sudo apt-get update
sudo apt-get install -y autoconf automake autotools-dev curl python3 libmpc-dev libmpfr-dev libgmp-dev gawk build-essential bison flex texinfo gperf libtool patchutils bc zlib1g-dev libexpat-dev
```
Phase B: Clean the "Broken" Builds
We need to remove the failed attempts so the script doesn't get confused by "already exists" errors.
```bash
cd /home/misty/openpiton/piton/design/chip/tile/ariane/
rm -rf tmp/
```
Phase C: Re-run the Build Script properly
Now, we run the build script again, but we will watch it carefully.

```Bash
cd /home/misty/openpiton
source piton/piton_settings.bash
source piton/ariane_setup.sh
./piton/ariane_build_tools.sh
```
My riscv64-unknown-elf-gcc (13.2.0-11ubuntu1+12) 13.2.0 is succesfully done, where as verilator still faces issues

To fix my verilator the issue lied in my Bison version 3.8.2. Older versions of Verilator (like 4.014) are incompatible with Bison 3.7+ because the way Bison generates parser files changed, leading to the YYEMPTY and verilog.h errors

The Fix: Upgrading Verilator
We are going to manually install Verilator 4.106, which is known to be stable with newer Bison versions while still working perfectly with OpenPiton.

Clear the broken folder:

```Bash
cd /home/misty/openpiton/piton/design/chip/tile/ariane/tmp/
rm -rf verilator-4.014
```
Download and Install the New Version:

```Bash
wget https://github.com/verilator/verilator/archive/refs/tags/v4.106.tar.gz
tar -xzf v4.106.tar.gz
cd verilator-4.106
autoconf
./configure
make -j$(nproc)
```
Link it to OpenPiton:
OpenPiton's scripts expect the folder to be named verilator-4.014. We can "trick" it using a symbolic link so you don't have to rewrite any core scripts.

```Bash
cd ..
ln -s verilator-4.106 verilator-4.014
```

Final Environment Check
Once that make finishes, verify your progress:

Verilator: verilator --version (should now show 4.106).

Now that everything is ok, moving ahead with sims build command to create your first Ariane hardware model

first 
```bash
sudo apt-get update
sudo apt-get install -y tcsh csh libx11-dev
```
Stay in the root folder: cd /home/misty/openpiton
Run the builder: mktools

Step 4: Building the Ariane Hardware Model 
Now that your environment has its compiler, its simulator (Verilator), and its internal tools, you can proceed with the hardware build.

```bash
cd /home/misty/openpiton/build
sims -sys=manycore -x_tiles=1 -y_tiles=1 -msm_build -ariane
```
You may face series of error
<img width="953" height="1018" alt="image" src="https://github.com/user-attachments/assets/02e51944-350a-4b2e-94be-209065129d0f" />
<img width="956" height="1012" alt="image" src="https://github.com/user-attachments/assets/a328fdf3-fc14-42fd-a9b0-4e46a8560a88" />

Once you sucesfully run, also deleteing tmp is necessary: 

```bash
cd openpiton/piton/verif/env/manycore
rm -rf *tmp.v
```
```bash
cd openpiton
sims -sys=manycore -vlt_build -ariane
```
It takes about 10 mins...

## Step - 4 
Now that the hardware model is built, you can finally run your first software program on it. Since you are targeting the Ariane core, you can use the -vlt_run flag to launch the simulation.

Run your first "Hello World" (or a basic assembly test) with this command:

```bash
cd /home/misty/openpiton/build
sims -sys=manycore -x_tiles=1 -y_tiles=1 -vlt_run -ariane /home/misty/openpiton/piton/verif/diag/assembly/riscv/rv64ui-p-add.s
```

 1. Fixing the Simulation Script (sims,2.0) 
The main script was missing the command to actually start the simulator and was crashing on minor hardware warnings.

The Problem: The script tried to execute -f as a program name and stopped the build for things like timing delays (STMTDLY) and unpacked structs.

The Fix: You edited line 1571 (and the vlt_build block) to include the command name and "waiver" flags.

The Correct Line: my $cmd = "vlog -Wno-fatal -Wno-STMTDLY -Wno-UNPACKED -f flist";

2. Fixing the Hardware Source Code
You hit a "Hard Error" where the Ariane cache was trying to grab 8 bits from a wire that only had 1 bit.

The Problem: In wt_dcache_wbuffer.sv, line 486 had a bit-slicing mismatch that Verilator refused to ignore.

The Fix: You manually commented out the faulty assignment line since those "User" bits weren't needed for your test.

The Command: nano .../wt_dcache_wbuffer.sv and adding // to the start of line 486.

3. Fixing the C++ Compilation Errors
When the hardware was finally translated to C++, the actual C++ compiler (GCC) failed because of a missing header and a name mismatch.

Type Mismatch: In my_top.cpp, you changed char* to const char* to match the new Verilator standard.

Missing Header: In verilated.cpp, you added #include <limits> because GCC 13 needed it to understand the max() function.

4. Running the Successful Simulation
Finally, the script couldn't find your test file because it didn't like slashes in the filename and the test wasn't compiled yet.

The Solution: You compiled the tests manually and then pointed the script directly to the folder.

The Final Commands:

Build the test: 
```bash
cd $ARIANE_ROOT/tmp/riscv-tests/isa && make rv64ui-p-add
```

Run the simulation:

```Bash
sims -sys=manycore -x_tiles=1 -y_tiles=1 -vlt_run -ariane -precompiled -asm_diag_path=$ARIANE_ROOT/tmp/riscv-tests/isa/ -asm_diag_name=rv64ui-p-add
```
This resulted in the Simulation -> PASS (HIT GOOD TRAP) message you saw at the end!

In very simple terms, you just proved that your "virtual brain" (the Ariane RISC-V processor) can think correctly.Here is exactly what happened during that test:
1. The "Math" TestThe specific test you ran was called rv64ui-p-add.What it did: It gave the processor a series of addition problems (e.g., $1+1=2$).The Goal: It checked if the hardware's internal logic gates actually produced the correct mathematical result for 64-bit numbers.

2. The "Pipeline" TestA processor doesn't just do math; it has to move data around.What it did: The test checked if the core could correctly Fetch an instruction from memory, Decode what it means, and Execute it without getting confused.The Result: You saw lines like Stage 1 status and Stage 2 status in your log. This confirmed the "assembly line" inside the processor was moving data smoothly from one stage to the next.

3. The "Memory" TestThe processor had to talk to the outside world (the Cache and Memory) to get its instructions.What it did: It tested the connection between the Ariane core and the L1.5 Cache.The Result: The log showed L15_REQTYPE_ACKDT_IFILL, which is just a fancy way of saying the memory sent the "Add" instruction to the brain, and the brain received it perfectly.

4. The "Good Trap" (The Finale)In hardware testing, we don't use a screen to say "Success." Instead, we use a Trap.What it did: The program was written so that if every math problem was solved correctly, the processor would jump to a specific secret address called the Good Trap.The Result: Your log printed PASS (HIT GOOD TRAP). This is the ultimate "green light" that your hardware works as intended.


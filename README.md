# RISCV-OpenPiton-Research
For RISC V Artitecture, i will be using OpenPiton + Ariane core. It's a six stage pipeline and multicore

# Environment Setup

First step is to clone the Respositary : 

```bash
git clone https://github.com/PrincetonUniversity/openpiton.git
cd openpiton
```
Set the Root Variable: OpenPiton needs to know where its files are located at all times. This variable (PITON_ROOT) is used by every script in the framework to find design files and simulation tools.

```bash
export PITON_ROOT=`pwd`
```
Step 2: Initialize the Environment
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

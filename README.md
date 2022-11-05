# OpenFASOC
OpenFASoC is a project focused on automated analog generation from user specification to GDSII with fully open-sourced tools. It is led by a team of researchers at the University of Michigan and is inspired from FASoC which sits on proprietary software.

The tool is comprised of analog and mixed-signal circuit generators, which automatically create a physical design based on user specifications.

# Installation
## Installing OpenFASOC
First, cd into a directory of your choice and clone the OpenFASoC repository:

`git clone https://github.com/idea-fasoc/openfasoc`

Now go to the home location of this repository (where the README.rst file is located) and run sudo ./dependencies.sh. 

## Installing OpenROAD
OpenROAD is an integrated chip physical design tool that takes a design from synthesized Verilog to routed layout.

An outline of steps used to build a chip using OpenROAD is shown below:

1. Initialize floorplan - define the chip size and cell rows
2. Place pins (for designs without pads )
3. Place macro cells (RAMs, embedded macros)
4. Insert substrate tap cells
5. Insert power distribution network
6. Macro Placement of macro cells
7. Global placement of standard cells
8. Repair max slew, max capacitance, and max fanout violations and long wires
9. Clock tree synthesis
10. Optimize setup/hold timing
11. Insert fill cells
12. Global routing (route guides for detailed routing)
13. Antenna repair
14. Detailed routing
15. Parasitic extraction
16. Static timing analysis

To install OpenROAD:

``` git clone --recursive https://github.com/The-OpenROAD-Project/OpenROAD.git

  cd OpenROAD

  ./etc/DependencyInstaller.sh

  ./etc/DependencyInstaller.sh -run

  ./etc/DependencyInstaller.sh -dev

  mkdir build

  cd build

  cmake ..

  make
```

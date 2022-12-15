# 1. OpenFASOC
OpenFASoC is a project focused on automated analog generation from user specification to GDSII with fully open-sourced tools. It is led by a team of researchers at the University of Michigan and is inspired from FASoC which sits on proprietary software.

The tool is comprised of analog and mixed-signal circuit generators, which automatically create a physical design based on user specifications.

# 2. Installation
## 2.1 Installing OpenFASOC
First, cd into a directory of your choice and clone the OpenFASoC repository:

`git clone https://github.com/idea-fasoc/openfasoc`

Now go to the home location of this repository (where the README.rst file is located) and run sudo ./dependencies.sh. 

## 2.2 Installing OpenROAD
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
## 2.3 Installing Yosys
Yosys is a framework for Verilog RTL synthesis. It currently has extensive Verilog-2005 support and provides a basic set of synthesis algorithms for various application domains. Selected features and typical applications:

Process almost any synthesizable Verilog-2005 design
Built-in formal methods for checking properties and equivalence
Mapping to ASIC standard cell libraries (in Liberty File Format)
Mapping to Xilinx 7-Series and Lattice iCE40 and ECP5 FPGAs

To install Yosys use the following commands
``` sudo apt-get install build-essential clang bison flex \
   libreadline-dev gawk tcl-dev libffi-dev git \
   graphviz xdot pkg-config python3 libboost-system-dev \
   libboost-python-dev libboost-filesystem-dev zlib1g-dev
   
   git clone https://github.com/YosysHQ/yosys.git

   make

   sudo make install 

   make test
```
## 2.4 Installing Magic
Magic is a venerable VLSI layout tool, written in the 1980's at Berkeley by John Ousterhout, now famous primarily for writing the scripting interpreter language Tcl. Due largely in part to its liberal Berkeley open-source license, magic has remained popular with universities and small companies. The open-source license has allowed VLSI engineers with a bent toward programming to implement clever ideas and help magic stay abreast of fabrication technology. However, it is the well thought-out core algorithms which lend to magic the greatest part of its popularity. Magic is widely cited as being the easiest tool to use for circuit layout, even for people who ultimately rely on commercial tools for their product design flow.

In order to compile Magic on a vanilla installation of Ubuntu, the following additional packages are needed (at a minimum):
```
sudo apt-get install m4
sudo apt-get install tcsh
sudo apt-get install csh
sudo apt-get install libx11-dev
sudo apt-get install tcl-dev tk-dev
sudo apt-get install libcairo2-dev
sudo apt-get install mesa-common-dev libglu1-mesa-dev
sudo apt-get install libncurses-dev
```

To install magic
```
git clone https://github.com/RTimothyEdwards/magic
cd magic/
./configure
sudo make
sudo make install
```
###

## 2.5 Installing KLayout
KLayout is a GDS and OASIS file viewer. Downlaod the latest version of the Klayout from here. Install the following dependencies: qt5-default, qttools5-dev, libqt5xmlpatterns5-dev, qtmultimedia5-dev, libqt5multimediawidgets5 and libqt5svg5-dev.

```
sudo apt-get install -y libqt5widgets5

sudo dpkg -i klayout_0.27.11-1_amd64.deb
```
# 3. Temperature Sensor Auxiliary Cells
An overview of how the Temperature Sensor Generator (temp-sense-gen) works internally in OpenFASoC

## 3.1 Circuit
This generator creates a compact mixed-signal temperature sensor based on the topology from this paper. It consists of a ring oscillator whose frequency is controlled by the voltage drop over a MOSFET operating in subthreshold regime, where its dependency on temperature is exponential.

<p align="center">
  <img width="1000" height="500" src="https://user-images.githubusercontent.com/110079890/200105228-1ba0d839-3f1a-482f-9889-bb2cd1eb5048.png">
</p>

The physical implementation of the analog blocks in the circuit is done using two manually designed standard cells:

1. HEADER cell, containing the transistors in subthreshold operation;

2. SLC cell, containing the Split-Control Level Converter.

The gds and lef files of HEADER and SLC cells are pre-created before the start of the Generator flow.

The layout of the HEADER cell is shown below:
<p align="center">
  <img width="1000" height="500" src="https://user-images.githubusercontent.com/110079890/207322258-561ed913-58b6-433c-bb46-4d8772cb8845.png">
</p>

The layout of the SLC cell is shown below:
<p align="center">
  <img width="1000" height="500" src="https://user-images.githubusercontent.com/110079890/207323939-51a7531f-f1df-4e8f-af5f-6ae798c07a6c.png">
</p>


## 3.2 OpenFASOC flow for Temperature Sensor Generation
To configure circuit specifications, modify the test.json specfile in the generators/temp-sense-gen/ folder.

To run the default generator, cd into openfasoc/generators/temp-sense-gen/ and execute the following command:

`make sky130hd_temp`

The default circuit’s physical design generation can be divided into three parts:

1. Verilog generation

2. RTL-to-GDS flow (OpenROAD)

3. Post-layout verification (DRC and LVS)

### 3.2.1 Verilog Generation
To run verilog generation, type the command
`make sky130hd_temp_verilog`

The generator must first parse the user’s requirements into a high-level circuit description or verilog. User input parsing is implemented by reading from a JSON spec file directly in the temp-sense-gen repository. The JSON allows for specifying power, area, maximum error (temperature result accuracy), an optimization option (to choose which option to prioritize), and an operating temperature range (minimum and maximum operating temperature values). The operating temperature range and optimization must be specified, but other items can be left blank.

The generator uses this model file to automatically determine the number of headers and slc, among other necessary modifications that can be made to meet spec. The generator references the model file in an iterative process until either meeting spec or failing. A verilog description is then produced by substituting specifics into template verilog files.

The test.json file shown in the below screenshot corresponds to the temp_sense_gen.

<p align="center">
  <img width="1000" height="500" src="https://user-images.githubusercontent.com/110079890/200110700-17034822-61a6-4b29-aee5-09888bcde8ae.png">
</p>

In this file temperature is being varied from -20 C to 100 C.Based on the operating temperature range, generator calculates the number of header and inverters to minimize the error.

The generator starts from a Verilog template of the temperature sensor circuit, located in temp-sense-gen/src/. The .v template files have lines marked with @@, which are replaced according to the specifications.

To replace these lines with the correct circuit elements, temp-sense-gen takes cells from the selected technology. The number of inverters in the ring oscillator and of HEADER cells in parallel are optimized using a nearest-neighbor approach with experimental data from models/modelfile.csv.

The generator references the model file in an iterative process until either meeting specifications or failing.
<p align="center">
  <img width="1000" height="500" src="https://user-images.githubusercontent.com/110079890/207328599-f40c124e-06cc-4128-a0f8-1ab3808210be.png">
</p>

As shown in the above picture, the tool is trying to minimize the error iteratively, by varying the number of inverters and headers for the given temperature range.

#### Directory where verilog files are generated
<p align="center">
  <img width="700" height="500" src="https://user-images.githubusercontent.com/110079890/201682391-8468fd05-dfe6-408e-a94d-54da7b8c6a2c.png">
</p>
After running the `make sky130hd_temp_verilog` command the verilog files of counter.v, TEMP_ANALOG_hv.nl.v, TEMP_ANALOG_lv.nl.v are created in the src folder.

Here, using the generic template, extra blocks of counter, TEMP_ANALOG_hv.nl.v, TEMP_ANALOG_lv.nl.v are created in the src folder. For the temperature specification of -20C to 100C, we see that three header files are required.

### 3.2.2 Synthesis

The OpenROAD Flow starts with a flow configuration file config.mk, the chosen platform (sky130hd, for example) and the Verilog files are generated from the previous part.

The synthesis is run using Yosys to find the appropriate circuit implementation from the available cells in the platform. The synthesized verilog netlist is saved as shown in the below figure.
<p align="center">
  <img width="2000" height="400" src="https://user-images.githubusercontent.com/110079890/207364136-3c0d4fe7-7620-4b9c-9329-225b65925228.png">
</p>

### 3.2.3 Floorplan
The floorplan for the physical design is generated with OpenROAD, which requires a description of the power delivery network in pdn.cfg.

The floorplan final power report is shown below:
<p align="center">
  <img width="700" height="500" src="https://user-images.githubusercontent.com/110079890/207365893-86f050f0-037f-4f2f-822f-78f02161d825.png">
</p>

This temperature sensor design implements two voltage domains: one for the VDD that powers most of the circuit, and another for the VIN that powers the ring oscillator and is an output of the HEADER cells. Such voltage domains are created within the floorplan.tcl script, with the following lines of code:

<p align="center">
  <img width="700" height="500" src="https://user-images.githubusercontent.com/110079890/207369337-441cabe9-8ff6-463b-ab31-ca984f2d6061.png">
</p>

In the image, line #34 will create a voltage domain named TEMP_ANALOG with area coordinates as defined in config.mk.

Lines #36 to #38 will initialize the floorplan, as default in OpenROAD Flow, from the die area, core area and place site coordinates from config.mk.

And finally, lines #40 to #42 will source read_domain_instances.tcl, a script that assigns the corresponding instances to the TEMP_ANALOG domain group. It gets the wanted instances from the DOMAIN_INSTS_LIST variable, set to tempsenseInst_domain_insts.txt in config.mk. This will ensure the cells are placed in the correct voltage domain during the detailed placement phase.

The tempsenseInst_domain_insts.txt file contains all instances to be placed in the TEMP_ANALOG domain (VIN voltage tracks). These cells are the components of the ring oscillator, including the inverters whose quantity may vary according to the optimization results. Thus, this file actually gets generated during temp-sense-gen.py.

### 3.2.4 Placement
Placement takes place after the floorplan is ready and has two phases: global placement and detailed placement. The output of this phase will have all instances placed in their corresponding voltage domain, ready for routing.

#### The Global Placement power and area report is shown below:
<p align="center">
  <img width="700" height="500" src="https://user-images.githubusercontent.com/110079890/207367846-cd120f44-d050-44ab-ad64-b3be19542531.png">
</p>

#### The Detail Placement power and area report is shown below:
<p align="center">
  <img width="700" height="500" src="https://user-images.githubusercontent.com/110079890/207370865-49874446-01d1-499b-9ba3-0b531b416b04.png">
</p>

### 3.2.5 Routing
Routing is also divided into two phases: global routing and detailed routing. Right before global routing, OpenFASoC calls pre_global_route.tcl:
<p align="center">
  <img width="1000" height="500" src="https://user-images.githubusercontent.com/110079890/207374779-15b542d2-9b53-4b6d-9870-5c63a7b8d2fe.png">
</p>

This script sources two other files: add_ndr_rules.tcl, which adds an NDR rule to the VIN net to improve routes that connect both voltage domains, and create_custom_connections.tcl, which creates the connection between the VIN net and the HEADER instances.

#### The Global route power and area report is shown below:
<p align="center">
  <img width="700" height="500" src="https://user-images.githubusercontent.com/110079890/207372763-4d811680-c1f7-432d-84e8-b3c7a6f7e8cf.png">
</p>

#### The finished power and area report is shown below:
<p align="center">
  <img width="700" height="500" src="https://user-images.githubusercontent.com/110079890/207377437-e6d78adc-05cb-476f-8751-db3d731d5dc7.png">
</p>

#### Final layout after routing:

<p align="center">
  <img width="700" height="500" src="https://user-images.githubusercontent.com/110079890/207379770-e9cb8f5f-b8ed-4ace-880e-266944286578.png">
</p>


### 3.2.6 Post-layout verification

After generating the design, OpenFASoC runs DRC and LVS to check that the circuit is manufacturable and corresponds to the specified design. In flow/Makefile, the targets magic_drc and netgen_lvs are run using make.
In DRC, Magic takes the generated GDS file and checks for failed constraints. A report is written under temp-sense-gen/flow/reports/ with any errors found.

In LVS, Magic takes the generated GDS file and extracts its netlist to compare with the original circuit netlist, in order to verify if the physical implementation was done correctly. Files generated from the layout extraction are created under temp-sense-gen/flow/objects/.

Netgen is then used to run the comparison, outputting a report under temp-sense-gen/flow/reports/.

# 4. Phase Locked Loop

## 4.1 Circuit
A Phase Locked Loop (PLL) consists of

- Phase Detector
- Charge Pump
- Voltage Controlled Oscillator
- Frequency Divider

### 4.1.2 Phase Detector

Phase detector produces two DC voltages namely UP and DOWN, which is proportional to the phase difference between the input signal(Vref) and feedback (output) signal(Vout). If the Vref phase is lagging with respect to Vout then UP signal remains high to the duration of their phase difference and the DOWN signal remains low. If the Vref phase is leading with respect to Vout then DOWN signal remains high to the duration of their phase difference and the UP signal remains low.

<p align="center">
  <img width="700" height="500" src="https://user-images.githubusercontent.com/110079890/207939908-c1b35785-72c9-4ed6-b1ee-2b0df30c1453.png">
</p>

The Phase detector is constructed using two negative edge triggered D Flip-Flops and a AND gate, which makes it a digital block.

<p align="center">
  <img width="700" height="500" src="https://user-images.githubusercontent.com/110079890/207940974-e5e30949-f028-448e-af3b-838210317e60.png">
</p>

### 4.1.2 Charge Pump
Charge Pump is used to convert the digital measure of the phase difference done in the Phase Detector into a analog control signal, which is used to control the Voltage Controlled Oscillator in the next stage. In the construction of the Charge Pump we use a current steering model which makes it a Analog block. Here when the UP signal is high the current flows from Vdd to output which charges the load capacitance. When the DOWN signal is high the current flows from load capacitance to ground which is discharging of the current.

<p align="center">
  <img width="700" height="700" src="https://user-images.githubusercontent.com/110079890/207941411-888377f7-3f22-4fdd-9813-b85cf2a47f0d.png">
</p>

<p align="center">
  <img width="700" height="800" src="https://user-images.githubusercontent.com/110079890/207941578-c12ef6e4-eb75-43d1-a53c-a87b7badc5c1.png">
</p>

### 4.1.3 Voltage Controlled Oscillator
The Output of the Charge Pump acts as a Control signal to the Voltage Controlled Oscillator.The VCO generates a DC signal, the amplitude of which is proportional to the amplitude of output of Charge Pump Control Signal. Here the adjustment in the output frequency/phase of VCO is made until it shows equivalency with the input signal frequency/phase. The VCO is contructed using two current mirrors and a ring Oscillator which makes it a Analog block. The control signal is used as an input to these current source(mirrors) to control the current supply which in turn control the delay of the circuit. By controlling the delay we are basically controlling the frequency of the Oscillator which makes it frequency flexible.

<p align="center">
  <img width="600" height="500" src="https://user-images.githubusercontent.com/110079890/207942242-20e09acc-7ed0-4b65-a109-69b5d3b02b91.png">
</p>

### 4.1.4 Frequency Divider
Frequency Divider is used to divide the frequency which is otherwise a multiplier in time of the Output voltage from the Voltage Controlled Oscillator and is feedback as an input to the Phase Detector, which is then compared with the Vref input signal in the Phase Detector stage. The Frequency Divider is constructed using a series of Toggle Flip-Flops, which makes it a complete Digital block.

<p align="center">
  <img width="500" height="500" src="https://user-images.githubusercontent.com/110079890/207942515-55ba0ff6-584e-449a-9833-9ab5cb839498.png">
</p>

## 4.2 Auxiliary Cells
The auxiliary cells used in the PLL are charge pump and voltage controlled oscillators. They are present inside the directory shown below:

<p align="center">
  <img width="1500" height="400" src="https://user-images.githubusercontent.com/110079890/207943636-7cf9d98f-59fc-4592-acef-8b59317e475d.png">
</p>

# Author
- Rakshit Bhatia, M.Tech Student at IIIT Bangalore

# Acknowledgements
- Kunal Ghosh, Director, VSD Corp. Pvt. Ltd.
- Madhav Rao, Associate Professor, IIIT Bangalore
- Tejas B N, M.Tech Student at IIIT Bangalore

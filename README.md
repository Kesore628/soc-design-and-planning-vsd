# soc-design-and-planning-vsd
"Code to Chip: An open-source RTL-to-GDSII journey with OpenLANE &amp; Sky130 PDK | VSD SoC Design and Planning Workshop"
# Digital VLSI SoC Design and Planning — RTL to GDSII

> A comprehensive hands-on workshop focused on the complete RTL-to-GDSII design flow for Digital VLSI SoC development. Conducted by **VSD (VLSI System Design)** in association with **NASSCOM**, this repository captures my daily progress, laboratory exercises, implementation results, and the key concepts learned throughout the program.

## Day 1 — Inception of Open-Source EDA, OpenLANE & Sky130 PDK

#### Understanding Chip Packaging

When observing an electronic board, the component commonly referred to as the "chip" is actually the **chip package**. This package protects the delicate silicon inside and provides connections to the external circuitry. The actual silicon die is located at the centre of the package and is connected to the package terminals through **wire bonds**, which are extremely fine metallic wires.

#### Exploring the Internal Structure of a Chip

Within the silicon die, communication with the outside world occurs through **I/O pads** positioned along the edges of the chip. The area surrounded by these pads is known as the **core**, which contains the complete digital circuitry responsible for the chip's functionality. The combination of the core and the surrounding pads constitutes the **die**, the basic unit fabricated during semiconductor manufacturing.

* **Foundry** – A semiconductor fabrication facility where integrated circuits are manufactured.
* **Foundry IPs** – Specialized intellectual property blocks developed using process-specific expertise, such as **SRAM**, **PLL**, and analog interfaces.
* **Macros** – Pre-designed and reusable digital building blocks integrated into larger chip designs.

#### Bridging Software and Hardware Through ISA

The journey of a program from high-level code to physical hardware involves several transformation stages:

1. The source code written in **C** is translated into **RISC-V assembly language** (or another target ISA) using a compiler.
2. The assembler then converts the assembly instructions into **machine code**, represented as binary values.
3. To execute these instructions, the processor requires an **RTL (Register Transfer Level) implementation** that follows the chosen ISA specification.
4. The RTL design is synthesized and passed through the complete **Place and Route (PnR)** process to generate the final chip layout ready for fabrication.

The software toolchain — comprising the **Operating System, Compiler, and Assembler** — serves as the interface between software developers and the underlying hardware, ensuring that human-readable programs can be executed by silicon.

#### Importance of Open-Source EDA Ecosystems

Building a completely open-source ASIC design flow requires three essential components:

1. **RTL Designs** – Open hardware descriptions obtained from repositories and community projects.
2. **EDA Tools** – Software used for synthesis, verification, floorplanning, placement, routing, and timing analysis.
3. **PDKs (Process Design Kits)** – Technology-specific files containing design rules, device models, and standard cell libraries.

For many years, access to PDKs was restricted because semiconductor foundries distributed them only under strict confidentiality agreements. A major breakthrough occurred in **June 2020**, when **Google** partnered with **SkyWater Technology** to release the **Sky130 PDK**, marking the first widely available open-source Process Design Kit and significantly lowering the barriers to ASIC development and education.

#### OpenLANE: Automating the RTL-to-GDSII Design Flow

**OpenLANE** is an open-source ASIC implementation framework that integrates several EDA tools to automate the complete journey from an RTL description to the final **GDSII** file used for chip fabrication. By combining multiple tools into a unified flow, it simplifies the physical design process for digital IC development.

| Design Stage                   | Tool(s)                |
| ------------------------------ | ---------------------- |
| Logic Synthesis                | Yosys, ABC             |
| Floorplanning & Power Planning | OpenROAD               |
| Cell Placement                 | OpenROAD               |
| Clock Tree Synthesis (CTS)     | TritonCTS              |
| Global and Detailed Routing    | FastRoute, TritonRoute |
| Parasitic Extraction           | OpenRCX                |
| GDSII Generation               | Magic, KLayout         |
| Static Timing Analysis         | OpenSTA                |
| DRC and LVS Verification       | Magic, Netgen          |

### Lab Exercise — Executing OpenLANE with `picorv32a`

#### Initializing the OpenLANE Environment

To begin the implementation flow, navigate to the OpenLANE directory and start the tool in **interactive mode**. This mode provides greater control by allowing each stage of the flow to be executed individually and inspected before proceeding to the next step.

```bash
cd /home/vscode/Desktop/OpenLane
make mount
./flow.tcl -interactive
package require openlane 1.0.2
```
#### Initializing the Design Environment

Before starting the synthesis stage, the design environment must be configured properly. This preparation step combines the required **technology LEF** and **cell LEF** files, creates the necessary run directories, and loads the design-specific configuration files needed for the implementation flow.

```tcl
prep -design picorv32a
```

Executing this command prepares the `picorv32a` design for subsequent stages in OpenLANE, ensuring that all technology information and project settings are correctly initialized before synthesis begins.
#### Executing the Synthesis Stage

Once the design setup is complete, the next step is to perform **logic synthesis**. During this stage, the RTL description is translated into a gate-level netlist using the standard cell library specified by the target technology.

```tcl id="7h2kqp"
run_synthesis
```

After synthesis finishes successfully, the generated reports can be used to examine important design statistics. One commonly observed metric is the **flop ratio**, which indicates the proportion of sequential elements present in the design.

```text
Flop Ratio = (Number of D Flip-Flops) / (Total Number of Standard Cells)
           = 1613 / 15762
           ≈ 0.1023
           ≈ 10.23%
```

A flop ratio of approximately **10.23%** suggests that around one-tenth of the synthesized cells are sequential elements, providing a quick sanity check on the overall composition of the design.

## Day 2 — Floorplanning and Introduction to Library Cells

#### Chip Floorplanning — Core Area and Utilisation

Floorplanning is about deciding where everything goes on the chip. Two key parameters drive this:

- **Utilisation Factor** = (Area occupied by Netlist) / (Total Core Area)
  - A utilisation of 0.5–0.6 is typical — you want room for buffers, routing, etc.
- **Aspect Ratio** = Height / Width of the core
  - A ratio of 1 means a square; anything else is a rectangle.

#### Pre-Placed Cells and Decoupling Capacitors

**Pre-placed cells** (like memories, PLLs, and complex IP blocks) are fixed in position before automated placement runs. Their location is determined manually based on connectivity and power intent.

**Decoupling capacitors** are placed around pre-placed cells to act as local charge reservoirs — they compensate for voltage drops caused by switching activity and ensure these blocks see clean power.
#### Power Planning — Establishing a Reliable Power Distribution Network

An efficient power distribution network is essential for maintaining stable chip operation. To achieve this, designers typically employ a combination of **power rings** and a **power mesh** throughout the layout. Power rings are formed around the core region, while the mesh extends across the chip using multiple metal layers to distribute the **VDD** and **VSS** supplies uniformly.

This arrangement ensures that every standard cell has convenient access to power through nearby supply connections, thereby reducing **IR drop**, improving voltage stability, and minimizing the risk of **electromigration** caused by excessive current density.

#### I/O Pin Placement and Cell Placement Restrictions

The placement of **input and output (I/O) pins** is carried out along the boundaries of the chip. Their positions are chosen carefully based on the logical connectivity of the design. Pins that communicate extensively with specific regions of the core are generally placed closer to those areas to reduce routing complexity and improve performance.

In addition, the region between the **core area** and the **die boundary**, commonly referred to as the **I/O ring region**, is reserved for essential circuitry such as pin buffers, ESD protection structures, and related interface components. To preserve this space, automated standard cell placement is restricted within these designated blockage areas.

### Lab — Floorplan and Placement

#### Running Floorplan
#### Executing the Floorplanning Stage

The next step in the OpenLANE flow is to perform **floorplanning**, where the initial physical structure of the chip is established. During this stage, the tool determines the core dimensions, configures the power distribution strategy, defines placement boundaries, and prepares the design for standard cell placement.

```tcl id="9k4twp"
run_floorplan
```

After the floorplanning process completes successfully, the generated **DEF (Design Exchange Format)** file can be inspected to analyze the physical information created during this stage.

```bash id="r2m8vf"
cd results/floorplan/
less picorv32a.def
```

Reviewing the DEF file allows us to verify important floorplan details such as the die area, core area, pin locations, blockages, and other layout-related configurations before moving on to the placement stage.
#### Visualizing the Floorplan Using Magic

After generating the floorplan, the layout can be examined graphically using **Magic**, an open-source VLSI layout viewer and editor. Loading the DEF file into Magic allows us to inspect the die structure, core boundaries, pin placement, and overall floorplan arrangement created during the previous stage.

```bash id="m6p2qx"
magic -T /home/vsduser/Desktop/OpenLane/designs/picorv32a/sky130A/libs.tech/magic/sky130A.tech \
      lef read ../../tmp/merged.nom.lef \
      def read picorv32a.def &
```

By opening the design in Magic, we can interactively explore the generated floorplan and verify that the physical layout information has been interpreted correctly before proceeding to placement and optimization stages.

## Day 3 — Design and Characterisation of Library Cells using Magic & ngspice
#### CMOS Inverter Characterization Using SPICE

To evaluate the performance of a standard cell, a **SPICE netlist** is created to model the CMOS inverter. The netlist defines the **PMOS and NMOS transistors**, including their respective **width-to-length (W/L) ratios**, along with essential simulation parameters such as the supply voltage, input excitation waveform, and output load capacitance.

By simulating this circuit, several important timing characteristics of the inverter can be determined.

Key performance metrics obtained from the simulation include:

* **Rise Time** – The time taken for the output voltage to transition from **20% to 80%** of its final value during a rising edge.
* **Fall Time** – The duration required for the output voltage to decrease from **80% to 20%** of its final value during a falling edge.
* **Propagation Delay** – The delay measured between the **50% transition point of the input signal** and the corresponding **50% transition point of the output signal**, indicating the switching speed of the inverter.

These parameters play a crucial role in standard cell characterization and are later used for timing analysis and library generation in the digital design flow.

#### Overview of the 16-Mask CMOS Fabrication Process

The manufacturing of a CMOS integrated circuit involves a series of carefully controlled processing steps, typically requiring around **16 photolithography masks** to transform a silicon wafer into a functional chip. Each stage contributes to the formation of transistors and interconnect structures within the device.

The major fabrication steps include:

1. **Wafer Preparation** – The process begins with the selection of a high-quality **p-type silicon substrate** possessing suitable electrical characteristics.
2. **Active Area Definition** – Isolation regions are created using field oxidation techniques and protective **silicon nitride (Si₃N₄) masks** to define the active device areas.
3. **Well Formation** – **N-wells and P-wells** are established through ion implantation to accommodate PMOS and NMOS transistors on the same substrate.
4. **Gate Oxide Formation** – A thin and uniform layer of gate oxide is grown to serve as the insulating layer beneath the transistor gate.
5. **Polysilicon Gate Deposition** – Polysilicon is deposited and patterned to form the gate electrodes that control transistor operation.
6. **Source and Drain Engineering** – The source and drain regions are created using implantation processes such as **Lightly Doped Drain (LDD)** and halo implants to enhance device performance and reliability.
7. **Contact and Metallization** – Contact openings are formed, followed by the deposition and patterning of multiple metal layers to establish electrical connections across the chip.
8. **Passivation and Protection** – A final protective passivation layer is applied to shield the completed integrated circuit from environmental contaminants and mechanical damage.

Together, these fabrication stages enable the realization of complex CMOS devices and form the foundation of modern semiconductor manufacturing.

### Lab Exercise — Characterising a Custom Inverter Standard Cell

#### Cloning the Standard Cell Design Repository

To explore the layout and characterization of a custom inverter cell, the first step is to obtain the required design files by cloning the standard cell repository. This repository contains the layout, SPICE netlists, and supporting files necessary for the characterization process.

```bash id="g8m2vr"
git clone https://github.com/nickson-jose/vsdstdcelldesign.git
```

After cloning the repository, the inverter layout can be opened using **Magic**, allowing us to inspect the transistor arrangement, routing structures, and physical implementation of the standard cell.

```bash id="u4k9xp"
magic -T sky130A.tech sky130_inv.mag &
```

Launching the layout in Magic provides an interactive view of the custom **Sky130 inverter cell**, enabling detailed examination and verification before proceeding with extraction and characterization steps.

#### Generating the SPICE Netlist Using Magic

Once the inverter layout has been verified, the next step is to extract its electrical representation from the physical design. Using the **tkcon** console available within Magic, the layout information can be converted into a SPICE netlist suitable for circuit simulation and characterization.

Execute the following commands in the tkcon window:

```tcl id="n5q7xt"
extract all
ext2spice cthresh 0 rthresh 0
ext2spice
```

The `extract all` command gathers the connectivity information from the layout, while `ext2spice` converts the extracted data into a SPICE-compatible netlist. Setting the capacitance and resistance thresholds to zero ensures that all relevant parasitic components are included in the generated output.

The resulting SPICE file can then be used for simulation to analyze the functional behavior and timing characteristics of the custom inverter cell.

#### Simulating the Extracted Netlist with ngspice

After generating the SPICE netlist, the circuit can be simulated using **ngspice** to observe the inverter's switching behavior and evaluate its timing performance.

```bash id="p8v3mk"
ngspice sky130_inv.spice
```

Within the ngspice environment, the input and output waveforms can be displayed using the following command:

```ngspice id="w2n6qy"
plot y vs time a
```

By analyzing the generated waveform, important timing parameters such as **rise time**, **fall time**, and **propagation delay** can be determined.

#### Rise Transition Time Measurement

The **rise transition time** represents the duration required for the output voltage to increase from **20% to 80%** of its final value.

```text
Rise Transition Time
= Time at 80% of Output Voltage − Time at 20% of Output Voltage
```

For a supply voltage of **3.3 V**:

* **20% of Output Voltage** = 0.66 V (660 mV)
* **80% of Output Voltage** = 2.64 V

Therefore, the rise time is obtained by measuring the time difference between the output crossing **0.66 V** and **2.64 V** on the rising edge.

#### Fall Transition Time Measurement

The **fall transition time** indicates the time taken by the output voltage to decrease from **80% to 20%** of its final value.

```text
Fall Transition Time
= Time at 20% of Output Voltage − Time at 80% of Output Voltage
```

For the same **3.3 V** supply:

* **80% of Output Voltage** = 2.64 V
* **20% of Output Voltage** = 0.66 V

The fall time is calculated by measuring the interval between the output crossing **2.64 V** and **0.66 V** during the falling transition.

From these waveform measurements, the switching characteristics of the CMOS inverter can be characterized and used for further timing analysis and standard cell validation.

## Day 4 — Pre-Layout Timing Analysis and Clock Tree Synthesis

#### LEF Generation and Standard Cell Port Design Guidelines

For a custom standard cell to be integrated successfully into the **OpenLANE** design flow, it must be accompanied by a valid **LEF (Library Exchange Format)** file. The LEF file provides an abstract physical representation of the cell, including details such as the cell dimensions, pin geometries, routing layers, and placement boundaries required during physical implementation.

While defining the LEF, certain guidelines must be followed to ensure compatibility with automated placement and routing tools:

* All **input and output pins** should be positioned at the **intersection points of the predefined horizontal and vertical routing tracks**, enabling efficient and error-free routing.

* The **cell width** must be an **odd multiple of the horizontal routing track pitch**, while the **cell height** should be an **odd multiple of the vertical track pitch**. Adhering to these constraints ensures that the standard cell aligns correctly with the routing grid and can be placed seamlessly alongside other cells in the library.

Following these LEF design rules allows custom cells to integrate smoothly into the ASIC implementation flow and supports reliable automated routing during chip layout generation.

#### Fundamentals of Static Timing Analysis (STA)

**Setup Slack** is one of the most important timing metrics used to determine whether a design can operate reliably at the target clock frequency.

```text
Setup Slack = Required Arrival Time − Actual Data Arrival Time
```

For a design to satisfy setup timing requirements, the computed setup slack should be **greater than or equal to zero**.

Several factors that affect timing accuracy are incorporated into STA to model real-world operating conditions:

* **OCV (On-Chip Variation)** – Accounts for manufacturing process variations and changes in operating voltage and temperature by applying timing derating factors to circuit paths.

* **Clock Uncertainty** – Represents timing margins introduced due to clock jitter and clock skew, ensuring robust analysis under non-ideal clock behavior.

* **CRPR (Clock Reconvergence Pessimism Removal)** – Eliminates unnecessary pessimism in timing calculations when the launch and capture clock paths share common segments of the clock network.

#### Clock Tree Synthesis (CTS)

**Clock Tree Synthesis** is the process of constructing an optimized network of clock buffers and interconnects to distribute the clock signal uniformly throughout the chip while minimizing clock skew and maintaining signal integrity.

Following CTS, timing analysis must be performed again because the clock network has been modified:

* **Hold timing analysis** should be revisited, as the insertion of clock buffers introduces additional delays that can impact hold margins.

* **Setup timing verification** must also be repeated since changes in the clock distribution network alter the effective clock arrival times at sequential elements.

Post-CTS timing validation ensures that the design continues to meet its performance requirements under the updated clock architecture.

#### Updating `config.tcl` to Integrate the Custom Standard Cell

To make the newly characterized custom cell available within the OpenLANE flow, the design configuration file must be modified to reference the appropriate timing libraries and LEF files. These settings ensure that synthesis and timing analysis use the correct library information while the physical design stages recognize the custom cell geometry.

```tcl id="6qv2mk"
set ::env(LIB_SYNTH)      "$::env(OPENLANE_ROOT)/designs/picorv32a/src/sky130_fd_sc_hd__typical.lib"
set ::env(LIB_FASTEST)    "$::env(OPENLANE_ROOT)/designs/picorv32a/src/sky130_fd_sc_hd__fast.lib"
set ::env(LIB_SLOWEST)    "$::env(OPENLANE_ROOT)/designs/picorv32a/src/sky130_fd_sc_hd__slow.lib"
set ::env(LIB_TYPICAL)    "$::env(OPENLANE_ROOT)/designs/picorv32a/src/sky130_fd_sc_hd__typical.lib"
set ::env(EXTRA_LEFS)     [glob $::env(OPENLANE_ROOT)/designs/$::env(DESIGN_NAME)/src/*.lef]
```

The library variables specify the timing models corresponding to different process corners used during synthesis and Static Timing Analysis. The `EXTRA_LEFS` parameter enables OpenLANE to load additional LEF files from the design source directory, allowing the custom standard cell to participate seamlessly in placement and routing.

With these updates in place, the design flow becomes aware of the newly added cell and can utilize it throughout the subsequent implementation stages.

#### Executing Clock Tree Synthesis (CTS)

After validating the pre-CTS timing results, the next stage in the physical design flow is **Clock Tree Synthesis (CTS)**. During this phase, the clock distribution network is constructed by inserting clock buffers and organizing them into a balanced tree structure to deliver the clock signal efficiently across the entire chip.

```tcl id="k7p3vx"
run_cts
```

The objective of CTS is to minimize **clock skew**, control clock latency, and maintain signal integrity at all sequential elements within the design. By carefully balancing the clock paths, the tool ensures that flip-flops receive the clock signal with minimal timing variation.

Once CTS is completed, timing analysis should be performed again to verify that both **setup** and **hold** requirements continue to be satisfied, as the newly inserted clock buffers introduce actual delays into the clock network. These post-CTS checks are essential to confirm the timing reliability of the updated design before proceeding to routing.

#### Post-CTS Timing Analysis Using OpenROAD

After completing Clock Tree Synthesis, **OpenROAD** can be used to perform detailed timing analysis on the updated design. This process involves loading the physical design data, timing libraries, netlist, and constraints into the OpenROAD database.

Launch the OpenROAD environment using:

```tcl id="m8q2vx"
openroad
```

#### Reading the LEF Information

Load the merged LEF file containing the technology and standard cell definitions:

```tcl id="a4k7np"
read_lef /OpenLane/designs/picorv32a/runs/24-03_10-03/tmp/merged.nom.lef
```

#### Importing the Post-CTS DEF

Read the DEF file generated after the CTS stage to restore the physical layout information:

```tcl id="d6r3mt"
read_def /OpenLane/designs/picorv32a/runs/24-03_10-03/results/cts/picorv32a.def
```

#### Creating and Saving the OpenROAD Database

To preserve the current design state for future analysis, create an OpenROAD database:

```tcl id="n5v9kp"
write_db pico_cts.db
```

The saved database can later be reloaded directly:

```tcl id="q2m4xr"
read_db pico_cts.db
```

#### Loading the Post-CTS Netlist

Import the synthesized Verilog netlist corresponding to the design:

```tcl id="w8p6kt"
read_verilog /OpenLane/designs/picorv32a/runs/24-03_10-03/results/synthesis/picorv32a.v
```

#### Reading the Timing Libraries

Load the complete Liberty timing models required for timing analysis:

```tcl id="b3n7vy"
read_liberty $::env(LIB_SYNTH_COMPLETE)
```

#### Linking the Design

Associate the netlist with the loaded libraries to establish the complete design database:

```tcl id="f9q2md"
link_design picorv32a
```

#### Applying Timing Constraints

Read the custom SDC file containing clock definitions and timing constraints:

```tcl id="k4v8xp"
read_sdc /OpenLane/designs/picorv32a/src/my_base.sdc
```

#### Propagating the Clock Network

Since the clock tree has already been synthesized, configure OpenROAD to use the actual propagated clock paths:

```tcl id="t6m1qr"
set_propagated_clock [all_clocks]
```

#### Exploring Timing Report Options

The available syntax and options for timing reports can be viewed using:

```tcl id="u7k5vn"
help report_checks
```

#### Generating a Detailed Timing Report

Finally, produce a comprehensive timing report containing information such as slew, transition times, net delays, capacitances, and input pin details.

```tcl id="p3x9mt"
report_checks -path_delay min_max -fields {slew trans net cap input_pins} -format full_clock_expanded -digits 4
```

The generated report provides detailed visibility into both setup and hold paths after CTS, helping identify critical timing paths and verify that the design satisfies its timing requirements before proceeding to the routing stage.

Exit to OpenLANE flow
exit
---

## Day 5 — Final RTL to GDSII using TritonRoute & OpenSTA
#### Routing Stages — Global Routing and Detailed Routing

The **routing phase** establishes the physical connections between all placed cells in the design. To efficiently generate manufacturable interconnects, routing is typically performed in two successive stages.

1. **Global Routing (FastRoute)** – During this stage, the chip area is divided into coarse routing grids, and approximate paths are determined for each net. The router considers factors such as available routing resources, preferred metal directions, and congestion to generate routing guides without assigning exact wire locations.

2. **Detailed Routing (TritonRoute)** – Using the routing guides produced during global routing, the detailed router assigns precise metal segments, vias, and track locations for every connection. This stage strictly enforces **Design Rule Check (DRC)** constraints, ensuring that the final routing solution is both electrically correct and fabrication compliant.

Global routing focuses on planning and resource allocation, while detailed routing refines those plans into an exact physical implementation ready for sign-off verification and tape-out.

Lab — Power Distribution, Routing
Creating the Power Distribution Network (PDN)

The Power Distribution Network (PDN) provides reliable power delivery across the entire chip by connecting the power (VDD) and ground (VSS) nets to all standard cells and macros. It consists of power rails, straps, and rings that ensure stable voltage supply while minimizing IR drop.

The PDN is generated using the following command:

gen_pdn

After PDN generation, the design is prepared for the routing stage with proper power connectivity established throughout the layout.

| Tool            | Purpose                                                       |
| --------------- | ------------------------------------------------------------- |
| **OpenLANE**    | Complete RTL-to-GDSII automated physical design flow          |
| **Yosys**       | Performs RTL synthesis and technology mapping                 |
| **OpenROAD**    | Handles floorplanning, placement, CTS, and routing            |
| **Magic**       | Used for layout viewing, editing, DRC, and LVS verification   |
| **OpenSTA**     | Performs Static Timing Analysis (STA) for timing verification |
| **ngspice**     | Simulates analog and mixed-signal circuits using SPICE models |
| **TritonRoute** | Executes detailed routing of signal connections               |
| **Netgen**      | Performs Layout Versus Schematic (LVS) checking               |
| **Sky130 PDK**  | Open-source 130nm Process Design Kit from SkyWater Technology |

### Key Learnings

* Developed an understanding of the complete ASIC design flow, from RTL design to GDSII generation using open-source tools.
* Gained practical exposure to floorplanning, standard cell placement, clock tree synthesis, and routing.
* Worked with the `picorv32a` RISC-V core to explore real-world physical design implementation.
* Learned the process of integrating and characterising custom standard cells within the design flow.
* Performed timing analysis using OpenSTA and understood the importance of setup and hold checks.
* Explored advanced STA concepts such as OCV and CRPR and their impact on timing closure.
* Studied post-route SPEF extraction and how interconnect parasitics influence final sign-off timing.
* Enhanced hands-on skills with industry-relevant open-source EDA tools used in ASIC development.

## Acknowledgements

A huge thank you to *Kunal Ghosh* (Co-founder, VSD Corp. Pvt. Ltd.) and *Nickson P Jose* (Physical Design Engineer, Intel) for putting together such a well-structured and genuinely practical workshop. Running a real CPU from RTL to GDSII using nothing but open-source tools is something I didn’t expect to be possible —and yet here we are.
**Kunal Ghosh** — Co-founder, VSD (VLSI System Design)
**Nickson Jose** — for the `vsdstdcelldesign` repository used in Day 3 labs
**NASSCOM** — for facilitating this workshop program
  
## References
[VSD SoC Design Workshop](https://www.vlsisystemdesign.com/)
[OpenLANE GitHub](https://github.com/The-OpenROAD-Project/OpenLane)
[SkyWater Sky130 PDK](https://github.com/google/skywater-pdk)
[vsdstdcelldesign](https://github.com/nickson-jose/vsdstdcelldesign)
Documented by Nayana | VTU Electronics & Communication Engineering | VLSI & Physical Design Enthusiast

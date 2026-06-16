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


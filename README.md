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

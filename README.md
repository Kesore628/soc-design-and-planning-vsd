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


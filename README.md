````markdown
# 🚀 5-Stage Pipelined RISC-V Processor 🚀

![Verilog](https://img.shields.io/badge/Language-Verilog-blueviolet?style=for-the-badge)
![ISA](https://img.shields.io/badge/ISA-RISC--V%20(RV32I)-blue?style=for-the-badge)
![Pipeline](https://img.shields.io/badge/Pipeline-5%20Stage-brightgreen?style=for-the-badge)
![Hazards](https://img.shields.io/badge/Hazards-Full%20Forwarding-orange?style=for-the-badge)

> A 5-stage pipelined RISC-V processor built in Verilog. This design fully resolves all RAW (Read-After-Write) data hazards using a dedicated forwarding unit, eliminating the need for stalls.

[cite_start]This project was built for **ENCA302: Introduction to Computer Organization & Architecture** [cite: 559-560].

---

## 🏗️ Core Architecture
The processor is built on the classic 5-stage pipeline, with pipeline registers separating each stage.

* `📥` **IF (Instruction Fetch):** Fetches the next instruction from `Instruction_Memory.v`.
* `📖` **ID (Instruction Decode):** Decodes the instruction, reads from the `Register_File.v`, and generates control signals.
* `🧮` **EX (Execute):** Performs calculations in the `ALU.v` and receives forwarded data.
* `💾` **MEM (Memory Access):** Reads from or writes to the `Data_Memory.v`.
* `✍️` **WB (Write Back):** Writes the final result back into the `Register_File.v`.

![Processor Datapath](Pipeline_diagram.png)

---

## ⚡ Hazard Handling: Full Forwarding
This processor does **not** stall for data hazards. It uses a dedicated `Hazard_unit.v` to implement full forwarding.

![Hazard Unit Diagram](hazard_architecture.png)

### ➡️ Forwarding Logic
The `ForwardAE` and `ForwardBE` signals control ALU input MUXes to "bypass" the register file and grab the most recent data from later stages in the pipeline.

| Mux Control | Source | 🎯 Explanation |
| :--- | :--- | :--- |
| **`00`** | ID/EX | ✅ The operand comes from the register file (No Hazard). |
| **`10`** | EX/MEM | ⚡ The operand is forwarded from the prior ALU result. |
| **`01`** | MEM/WB | ⚡ The operand is forwarded from data memory or an earlier ALU result. |

![Hazard Logic Table](condition_table.jpg)

---

## 📊 Simulation & Performance

### ⚙️ Test Program (`memfile.hex`)
The pipeline was tested with a 6-instruction program designed to create multiple back-to-back data hazards.

```verilog
@00000000
00500293  // addi x5, x0, 5
00300313  // addi x6, x0, 3
006283B3  // add  x7, x5, x6  <-- HAZARD: Needs x5, x6
00002403  // lw   x8, 0(x0)
00100493  // addi x9, x0, 1
00940533  // add  x10, x8, x9 <-- HAZARD: Needs x8, x9
````

### 📈 Waveform Analysis

The entire 6-instruction program executes in **10 clock cycles** with **ZERO stalls**, proving the forwarding unit is 100% effective.

### ⏱️ Performance (CPI)

The `adityagaur_performancecode.py` script was used to analyze the simulation.

> **Ideal CPI: 1.00**
> **Real CPI: 1.67**

The Real CPI is higher *only* because of the pipeline's 4-cycle startup/drain cost. The pipeline's true throughput (once full) is **1 instruction per cycle**, matching the ideal CPI.

-----

## 📁 Repository Contents

```
/
│
├── 📁 Pipeline/                 (All Verilog source .v files + memfile.hex)
│
├── 🐍 adityagaur_performancecode.py  (Python script for CPI calculation)
├── 📄 hazard_handling_summary.pdf    (The full PDF project report)
├── 🖼️ Pipeline_diagram.png           (High-level datapath schematic)
├── 📝 sample_simulation_results.txt  (A text-based trace of the simulation)
└── 📖 README.md                      (You are here!)
```

-----

## ▶️ How to Run Simulation

This project was compiled using **Icarus Verilog**.

1.  **Compile:**
    ```sh
    iverilog -o out.vvp Pipeline/pipeline_tb.v Pipeline/*.v
    ```
2.  **Run Simulation:**
    ```sh
    vvp out.vvp
    ```
3.  **View Waveform:**
    This generates a `dump.vcd` file. Open it with a waveform viewer like **GTKWave**.
    ```sh
    gtkwave Pipeline/dump.vcd
    ```

<!-- end list -->

```
```

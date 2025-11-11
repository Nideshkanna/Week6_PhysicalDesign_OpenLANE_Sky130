# ⚙️ Day 5: Final Steps in RTL-to-GDS using TritonRoute and OpenSTA

## 🧩 RISC-V Reference SoC Tapeout Program

Welcome to **Day 5** of Week 6 in the **RISC-V SoC Tapeout Program**.
In this final session of the **Physical Design Flow**, we focus on the **post-CTS stages** — building the **Power Distribution Network (PDN)**, performing **Routing** with **TritonRoute**, and validating the design through **DRC** and **SPEF extraction** for timing analysis.

---

## ⚡ 1. Generation of Power Distribution Network (PDN)

In a typical RTL-to-GDSII flow, the **Power Distribution Network (PDN)** is generated before cell placement.
However, in **OpenLANE**, the PDN generation is carried out **after Clock Tree Synthesis (CTS)** to ensure proper power rail alignment and clock buffer connectivity.

This step creates **VDD/VSS rails and straps** that distribute power across the entire chip and connect to standard cell rails.

### 🔧 Command to Generate PDN

```bash
gen_pdn
```

This command invokes OpenROAD’s PDN generator to automatically create power grids across metal layers based on the configuration files.

[01](./images/01.png)

---

## 🚀 2. Routing using TritonRoute

**TritonRoute** is the open-source detailed router used by OpenLANE for advanced design routing.
It converts global route guides into detailed metal tracks, ensuring **DRC-clean**, fully connected routing.

### 🧭 Routing Flow Stages

1. **Global Routing** — Generates route guides between cells and blocks.
2. **Detailed Routing** — Uses search-and-repair algorithms to form physical tracks and vias across layers.

### ⚙️ Command to Start Routing

```bash
run_routing
```

This command triggers **TritonRoute** to perform:

* Pin access analysis
* Track assignment
* Iterative detailed routing
* Search and repair for DRC correction

TritonRoute ensures all interconnects are properly connected and **meets the DRC (Design Rule Check)** constraints.

<div align="center">
<table>
<tr>
<td><img src="./images/02.png" width="350"></td>
<td><img src="./images/03.png" width="350"></td>
</tr>
</table>
</div>

---

## 🧮 3. SPEF File Generation

The **Standard Parasitic Exchange Format (SPEF)** is an IEEE standard for representing parasitic resistance and capacitance data of interconnect wires.
These parasitics affect the real timing and delay of the circuit and are essential for post-route **Static Timing Analysis (STA)**.

OpenLANE includes a Python-based **SPEF Extractor** that generates this data from the final routed layout.

### ⚙️ SPEF Extraction Command

```bash
cd <path-to-SPEF_EXTRACTOR-tool-directory>
python3 main.py <path-to-LEF-file> <path-to-DEF-file-created-after-routing>
```

A portion of the `.spef` output file is shown below:

<div align="center">
  <img src="./images/04.png" width="600">
</div>

This file contains the extracted **RC parasitics** for every net, which will be used by **OpenSTA** for accurate **post-route timing analysis**.

---

## 📏 4. Design Rule Check (DRC)

After routing, the layout undergoes **DRC** to verify that the physical geometry adheres to the fabrication rules defined in the PDK (e.g., Sky130).
TritonRoute integrates a built-in DRC engine that checks for:

* Spacing violations
* Minimum width errors
* Via overlaps
* Shorts and opens

Only **DRC-clean** layouts proceed to the **GDSII export stage**, ensuring the design is physically manufacturable.

---

## 🧠 5. Recap of TritonRoute Features

| Feature                              | Description                                             |
| ------------------------------------ | ------------------------------------------------------- |
| **1. Route Guide Compliance**        | Honors pre-generated route guides from global routing   |
| **2. Inter-guide Connectivity**      | Ensures proper inter-layer and inter-segment connection |
| **3. Search & Repair**               | Fixes violations iteratively                            |
| **4. DRC Engine**                    | Built-in checker for rule violations                    |
| **5. Routing Topology Optimization** | Minimizes congestion and improves timing                |

---

## 🧩 6. Post-Route Timing Validation using OpenSTA

Once routing and parasitic extraction are complete, **OpenSTA** is used again — now with the updated `.spef` — for **post-route timing analysis**.

```bash
sta post_route_sta.tcl
```

This ensures all timing constraints are met **with real interconnect delays** included, finalizing the tapeout-ready design.

---

## 🏁 **Conclusion — Week 6 Summary**

By the end of **Week 6**, we’ve completed the **entire Physical Design flow** from placement to routing, verifying physical and timing correctness through:

✅ Power Distribution Network (PDN) generation
✅ Routing using TritonRoute
✅ Design Rule Check (DRC) validation
✅ Parasitic Extraction (SPEF generation)
✅ Post-route Timing Analysis (OpenSTA)

These steps complete the **RTL-to-GDSII** journey — converting our digital design into a **fabrication-ready layout** using the open-source **Sky130 PDK** and **OpenLANE** toolchain.
🎯 Next week, we’ll focus on **final verification, GDS export, and tapeout preparation**.

---

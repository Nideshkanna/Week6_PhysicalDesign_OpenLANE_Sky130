# ⚙️ Week 6 — Physical Design Workshop using OpenLANE & Sky130 PDK

## 🧩 RISC-V Reference SoC Tapeout Program

This week marks the beginning of the **Physical Design Workshop**, where I work through the complete **RTL-to-GDSII implementation** process using the **OpenLANE flow** and **Sky130 PDK**.
After learning about synthesis, timing, and floorplanning in the earlier weeks, this phase focuses on how digital and mixed-signal blocks are physically realized on silicon.

---

## 🎯 Objective

The main goal of this week is to perform a series of **hands-on physical design labs** using a **pre-configured VDI image**. Through these labs, I aim to understand how various stages—standard cell design, layout generation, routing, DRC, and STA—fit together in a complete digital design implementation.

---

## 💡 Learning Importance

This workshop connects all the concepts from the previous weeks and provides a real understanding of:

* How **digital design integrates with custom analog and mixed-signal blocks**
* The relationship between **layout design, timing closure, and DRC rules**
* How **physical implementation** influences chip performance and reliability
* The complete flow from **synthesis → STA → layout → verification**

By the end of this week, I should have a clear view of how an RTL design is transformed into a verified layout ready for fabrication.

---

## 🧱 Workshop Overview

The workshop is divided into five structured days, each focusing on a key stage of the physical design flow.

| **Day**   | **Topic**                    | **Focus Area**                                                 |
| --------- | ---------------------------- | -------------------------------------------------------------- |
| **[Day 1](./Day1_Inception_OpenSourceEDA_OpenLANE_Sky130/readme.md)** | Inception of Open-Source EDA | Understanding the OpenLANE environment and Sky130 PDK          |
| **[Day 2](./Day2_GoodFloorplan_vs_BadFloorplan_and_LibraryCells/readme.md)** | Floorplanning Concepts       | Comparing good vs bad floorplans and exploring library cells   |
| **Day 3** | Standard Cell Design         | Creating and characterizing a cell using Magic and Ngspice     |
| **Day 4** | Timing Analysis & Clock Tree | Running pre-layout STA and learning about clock tree synthesis |
| **Day 5** | Routing & Sign-off           | Completing the RTL-to-GDS flow with TritonRoute and OpenSTA    |

Each day has its own folder (e.g., `Day1/`, `Day2/`…) containing:

* A `README.md` file with lab details and screenshots
* An `images/` folder for visuals and layout captures

---

## 🧰 Lab Setup

The labs for this week are done using a **pre-configured VDI image** provided as part of the workshop.

### 🔗 Download Links

* **Physical Design Tools VDI Image**
  [Download VDI File](https://drive.google.com/file/d/1Ri30Yeqjyprv-rStHEScUMpKtw2JfVJe/view)

* **OpenLANE Tools ZIP (Requires ~100 GB free space)**
  [Download openlane.zip](https://vsd-labs.sgp1.cdn.digitaloceanspaces.com/vsd-labs/openlane.zip)

---

### 🪟 Setup on Windows

1. Install **Oracle VirtualBox** → [https://www.virtualbox.org/wiki/Downloads](https://www.virtualbox.org/wiki/Downloads)
2. Create a new virtual machine

   * **Type:** Linux
   * **Version:** Ubuntu 18.04 (64-bit)
3. Allocate sufficient memory (≥ 4096 MB recommended)
4. Choose **“Use an existing virtual hard disk file”**
5. Select the `.vdi` file extracted from the downloaded zip
6. Click **Create → Start** to launch the VM
7. Once it boots, take a screenshot showing your username and terminal — this acts as **proof of setup**

---

### 🐧 Setup on Ubuntu

1. Install VirtualBox:

   ```bash
   sudo apt install virtualbox
   ```
2. Launch VirtualBox:

   ```bash
   virtualbox
   ```
3. Create a new virtual machine → OS Type: *Linux*, Version: *Ubuntu 18.04*
4. Allocate memory and select **“Use an existing virtual hard disk file”**
5. Browse to the `.vdi` file and select it
6. Click **Create → Start** to launch the environment

Once inside the VM, all required tools for OpenLANE and Sky130 are pre-installed, allowing direct access to the Physical Design labs.

---

## 🗂️ Documentation Format

Each day’s documentation includes:

* **Lab Objective and Context**
* **Step-by-step Procedure with Commands and Screenshots**
* **Tool Outputs (Magic, OpenLANE, Ngspice, OpenSTA)**
* **Short Explanations and Observations**
* **Reflections on Physical Design Concepts (DRC, LVS, STA)**

> Reference Repository: [SoC Design and Planning (NASSCOM × VSD)](https://github.com/fayizferosh/soc-design-and-planning-nasscom-vsd/)

---

## 📦 Deliverables

* This GitHub repository containing all **five day-wise lab folders**
* Screenshots for each lab task and terminal outputs
* A summary of **key learnings and observations** per day
* Proof of **VDI setup** (screenshot of the running environment with username visible)

---

## 🧠 Learning Outcomes

By the end of **Week 6**, I aim to:

✅ Successfully set up and use the Physical Design VDI environment

✅ Perform end-to-end **RTL-to-GDSII implementation** using OpenLANE and Sky130

✅ Design and characterize a standard cell using **Magic** and **Ngspice**

✅ Understand **clock tree synthesis**, **pre-layout STA**, and **routing concepts**

✅ Connect all stages of digital design — from **RTL** to **verified layout**

---

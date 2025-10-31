# ⚙️ Day 3 — Design a Library Cell using Magic Layout & ngspice Characterization

**RISC-V Reference SoC Tapeout Program — Week 6 (Physical Design Track)**

On **Day 3** I designed, extracted, simulated, and characterized a CMOS **inverter standard cell** using the **Sky130 PDK**, **Magic** for layout & DRC/LVS, and **ngspice** for post-layout SPICE simulations.
This document is a detailed, step-by-step lab README (theory, commands, screenshots placeholders, tips and DRC tech-file fixes) ready to be added to the repo.

---

## 🔬 Theory (short recap)

**Why design a standard cell?**
Standard cells (e.g. inverter, NAND, NOR) are the building blocks of digital designs. Each cell is designed once, verified, characterized (timing / power), and then used by synthesis and PnR flows.

**What we do today**

1. Create/inspect a transistor-level **layout** for a CMOS inverter in **Magic** (Sky130 tech).
2. Extract parasitics and produce a **SPICE netlist** (`.spice`) using `ext2spice`.
3. Run **DC sweep** (to find switching threshold Vm) and **transient** (pulse) analyses in **ngspice** to measure rise/fall transitions and propagation delays.
4. Inspect & fix **DRC rules** in `sky130A.tech` (e.g. `poly.9`, `difftap.2`, `nwell.4`) to make the tech file DRC-correct.

---

## 🧭 Tools & repo layout (where files will live)

```
work/tools/openlane_working_dir/openlane/
├─ designs/
│  ├─ vsdstdcelldesign/            # repo I cloned for inverter cell
│  │  ├─ sky130_inv.mag
│  │  ├─ libs/
│  │  └─ sky130A_inv.spice
│  └─ picorv32a/ ...
├─ pdks/sky130A/libs.tech/magic/sky130A.tech    # Sky130 techfile
└─ runs/...
```

---

## ⚙️ Day 3 — Implementation (step-by-step)

> **Assumption:** `$PDK_ROOT` points to your Sky130 PDK root and you are working under `~/work/tools/openlane_working_dir/openlane`.
> Replace `<run_dir>` and paths with your actual names.

---

### 1) Clone the standard-cell design repo

```bash
cd ~/work/tools/openlane_working_dir/openlane
git clone https://github.com/nickson-jose/vsdstdcelldesign
cd vsdstdcelldesign
# Copy techfile for convenience
cp $PDK_ROOT/sky130A/libs.tech/magic/sky130A.tech .
ls -la
```

**Screenshot placeholder:** `images/d3_01_clone_repo.png` — *terminal listing & repo contents*

---

### 2) Open the inverter layout in Magic

```bash
# Launch magic with the sky130 techfile
magic -T sky130A.tech sky130_inv.mag &
# or with better graphics
# magic -d XR -T sky130A.tech sky130_inv.mag &
```

**Magic quick commands to explore:**

* `:cellname allcells` — list cells
* `:select cell <cellname>` — open a cell
* `z` / `Z` / `ctrl+z` — zoom controls
* `s` (three times) — select electrically connected region
* `:box` — show selected box params (useful to measure distances)
* `:grid 0.5um 0.5um` — set grid (example)
* `:snap user` — snap to grid

**Placeholders for screenshots:**

* `images/d3_02_magic_open.png` — full layout view
* `images/d3_03_magic_identify_pm_nmos.png` — highlight PMOS/NMOS regions
* `images/d3_04_magic_ports.png` — showing A (input), Y (output), VPWR, VGND

---

### 3) Extract layout (create `.ext`) & generate SPICE deck

**In Magic tkcon (command) window:**

```tcl
# ensure working directory is layout dir
pwd
# Extract parasitics and connectivity
extract all
# convert ext -> spice with parasitic extraction thresholds
ext2spice cthresh 0 rthresh 0
# finalize conversion
ext2spice
# output spice file usually named like: sky130_inv.spice
```

**Placeholders:**

* `images/d3_05_magic_extract_ext.png` — `extract all` console output
* `images/d3_06_magic_ext2spice.png` — `ext2spice` console and generated .spice file

**Important:** check the generated `.spice` header for included model names and .include paths (you may need to edit relative paths to point to `libs/` inside repo).

---

### 4) Edit the SPICE file for simulation (small & necessary edits)

Open the generated `.spice` and ensure:

* `.include` lines reference correct model files (e.g. `./libs/nshort.lib`, `./libs/pshort.lib`) or the full path inside `$PDK_ROOT`.
* Testbench / power / inputs are present (or add them).
* Add `.control` block for ngspice if desired.

**Minimal example SPICE snippet (example):**

```spice
* SPICE3 file created from sky130_inv2.ext - technology: sky130A

.option scale=0.01u
.include ./libs/pshort.lib
.include ./libs/nshort.lib

* Inverter Cell (PMOS + NMOS)
M1 Y A VPWR VPWR pshort_model.0 w=37 l=23 ad=1443 pd=152 as=1517 ps=156
M2 Y A VGND VGND nshort_model.0 w=35 l=23 ad=1435 pd=152 as=1365 ps=148

* Power Supply
VDD VPWR 0 3.3
VSS VGND 0 0

* Input Pulse (0–3.3 V, 4 ns period)
Va A VGND PULSE(0 3.3 0 0.1n 0.1n 2n 4n)

* Load and Parasitics
CO A Y 0.05f
C1 Y VPWR 0.11f
C2 A VPWR 0.07f
C3 Y 0 2f
C4 VPWR 0 0.59f

.tran 1n 20n

.control
run
plot v(A) v(Y)
.endc
.end

```

**Placeholders:**

* `images/d3_07_spice_head.png` — top part of the spice netlist (includes & models)
* `images/d3_08_spice_tb.png` — testbench section (VDD, VSS, input stimulus)

---

### 5) Run ngspice & generate waveforms

```bash
ngspice sky130_inv.spice
# inside ngspice:
plot v(Y) vs time v(A)
# or use: plot y vs time a    (depending on net names)
```

**Typical interactive commands:**

* `plot v(Y)` — plot a node
* `trace` / `plot` interactive cursor to read time and voltage
* `measure` commands can be used to get exact times (see below)

**Placeholders:**

* `images/d3_09_ngspice_waveform.png` — waveform of input and output
* `images/d3_10_ngspice_interactive.png` — ngspice console showing plot command

---

### 6) Measure switching threshold (Vm) — DC sweep

**SPICE: DC sweep**

```spice
* For DC transfer characteristic
* Replace input source with Vin DC source:
Vin in 0 DC 0
* simulation control:
.op
.dc Vin 0 3.3 0.05
```

* Plot `v(Y)` vs `v(in)` and visually find intersection where `v(out) = v(in)` — that is **Vm**.
* Alternatively, use `.measure` to compute approximate crossing.

**Placeholders:**

* `images/d3_11_dc_transfer.png` — DC transfer plot
* `images/d3_12_vm_marker.png` — annotated Vm on plot

---

### 7) Measure rise/fall times and propagation delays — Transient analysis

**Pulse testbench (example):**

```spice
Va A VGND PULSE(0 3.3 0 10p 10p 1n 2n)  ; 10ps rise/fall, 1ns pulse, 2ns period
.tran 10p 6n
```

**Definitions**

* Rise transition time: time for `Vout` to go 20% → 80% of VDD:

  * `V20 = 0.2 * VDD`, `V80 = 0.8 * VDD`
  * `Tr = t(Vout crosses V80) - t(Vout crosses V20)`
* Fall transition time: `t(Vout falls from 80% to 20%)`
* Propagation delay (rise): time difference between Vin 50% → Vout 50%
* Propagation delay (fall): time difference between Vin 50% → Vout 50% (for falling edge)

**Measurements in ngspice (example):**

```
.meas tran trise WHEN v(Y)=0.8*3.3 RISE=1
.meas tran tr20 WHEN v(Y)=0.2 RISE=1
.meas tran tr_time PARAM='trise - tr20'
.meas tran td_rise WHEN v(Y)=1.65 RISE=1        ; use .meas with reference to v(A)
.meas tran tdin50 WHEN v(A)=1.65 RISE=1
.meas tran tdo50 WHEN v(Y)=1.65 RISE=1
.meas tran propDelayRise PARAM='tdo50 - tdin50'
```

**Placeholders:**

* `images/d3_13_traces_20_80.png` — zoomed plot indicating 20% & 80% points
* `images/d3_14_delay_annotations.png` — annotated propagation delays

**Example numeric outputs** (from lab reference):

* `Tr = 0.04242 ns` (rise transition)
* `Tf = 0.02713 ns` (fall transition)
* `D_r = 0.03194 ns` (rise propagation delay)
* `D_f = 0.00363 ns` (fall propagation delay)

(Your numbers will vary depending on device sizing, extracted parasitics and VDD.)

---

### 8) Verify layout connectivity & LVS (optional quick)

* Use `netgen` or `magic` LVS depending on your setup. Basic Magic LVS flow:

```tcl
# in magic tkcon
# create a netlist or write out gds for comparison (if you have gate-level netlist)
# run: drc check (ensure no DRC errors)
# use netgen: netgen -batch lvs netlist.spice layout.lef etc...   (see netgen docs)
```

**Placeholder:** `images/d3_15_lvs.png` — LVS summary (if run)

---

## 🛠️ Lab Part 2 — Inspect and fix Sky130 tech file DRC rules (poly.9, difftap.2, nwell.4)

**Goal:** The Sky130 `sky130A.tech` (used by Magic) sometimes lacks certain DRC checks for the periphery testcases. We fix the tech file to correctly flag violations.

> **Important:** Always work on a copy of `sky130A.tech`. Keep original backed up.

```bash
cd ~/work/tools/openlane_working_dir/openlane
cp $PDK_ROOT/sky130A/libs.tech/magic/sky130A.tech ./sky130A.tech.orig
cp ./sky130A.tech.orig ./sky130A.tech
gvim sky130A.tech
```

**Example issues & fixes**

1. **poly.9 rule** — spacing between poly resistor and other polys/diff/tap

   * Problem: rule defined only between poly resistor and diffusion, but not between poly resistor and *poly non-resistor* types (`allpolynonres`).
   * Fix: add spacing rule entries for `npolyres` vs `allpolynonres` and `ppolyres` vs `allpolynonres` using the same spacing value (e.g. `0.48um`).

2. **difftap.2 rule** — spacing between diffusion and tap

   * Problem: rule may only check one diffusion type; need to expand to `alldif` macro to capture all diffusion aliases.
   * Fix: update rule to compare `npolyres` to `alldif` (or use the alias macros defined earlier).

3. **nwell.4 complex rule** — ensure Nwell contains tap cells

   * Problem: nwell rule didn’t flag absence of tap within required distance.
   * Fix: add a rule to detect `nwell` regions without `ptap` or `ntap` per the spec.

**Edit example (conceptual pseudo-lines)** — these are illustrative (use exact `tech` syntax rules from the file and follow existing patterns):

```text
# add spacing rule: npolyres vs allpolynonres
rule poly.9 {
  spacing npolyres allpolynonres distance 0.48
  spacing ppolyres allpolynonres distance 0.48
}

# extend difftap rule
rule difftap.2 {
  spacing npolyres alldif distance 0.42
  spacing ppolyres alldif distance 0.42
}
```

**Reload tech file in Magic & re-run DRC**

In Magic tkcon:

```tcl
tech load sky130A.tech
drc style drc(full)
drc check
drc find
# use :drc why after selecting a box to get error message
```

**Placeholders:**

* `images/d3_16_drc_before.png` — before fix (no violation shown)
* `images/d3_17_drc_after.png` — after fix (white dots visible marking violation)
* `images/d3_18_drc_why_console.png` — `:drc why` output for an error

**Notes & references**

* Use SkyWater PDK periphery rules: [https://skywater-pdk.readthedocs.io](https://skywater-pdk.readthedocs.io) (the exact rule names & distances are authoritative)
* The `alias` macros in the techfile (like `alldif`, `allpolynonres`) are convenient — re-use them instead of enumerating layers.

---

## 🧾 Useful Magic / ngspice commands quick reference

**Magic**

```
magic -T sky130A.tech sky130_inv.mag &
:cellname allcells
:select cell <cellname>
s ; s ; s   # select connected
:box
:grid 0.5um 0.5um
:snap user
:paint poly
:erase poly
extract all
ext2spice cthresh 0 rthresh 0
ext2spice
tech load sky130A.tech
drc style drc(full)
drc check
drc find
drc why
```

**ngspice**

```
ngspice sky130_inv.spice
plot v(Y) vs time v(A)
.meas tran t20 WHEN v(Y)=0.2*3.3 RISE=1
.meas tran t80 WHEN v(Y)=0.8*3.3 RISE=1
.meas tran tr PARAM='t80 - t20'
.meas tran tdi WHEN v(A)=1.65 RISE=1
.meas tran tdo WHEN v(Y)=1.65 RISE=1
.meas tran prop PARAM='tdo - tdi'
```

---

## 🧠 Summary — Day 3 Key Takeaways

| Concept                        | Key takeaway                                                                                              |
| ------------------------------ | --------------------------------------------------------------------------------------------------------- |
| Layout → SPICE extraction      | Magic provides `extract` & `ext2spice` to produce post-layout netlists including parasitics               |
| Vm (switching threshold)       | Found via DC sweep; intersection of Vin vs Vout curves                                                    |
| Propagation & transition times | Measured from transient plots (20%/80% and 50% crossing) — parasitics increase delays                     |
| Techfile DRC                   | `sky130A.tech` must have complete rules (use `alias` macros); fix missing rules to detect real violations |
| Cell characterization          | Produces timing/power numbers used during library generation and STA                                      |

---

## 🔗 Next Step

➡️ Proceed to **Day 4 — Pre-layout timing analysis & importance of good clock tree**, where I will (briefly):

* Use characterization data from Day 3 to feed timing constraints
* Run pre-layout STA and compare estimated delays
* Start clock tree planning (CTS basics and its impact on cell timing)

Link: `../Day4_PreLayout_Timing_and_ClockTree/readme.md`

---

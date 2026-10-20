# Charge-Pump Phase-Locked Loop (PLL): Design, Simulation & Layout in SCL 180 nm CMOS

![Technology](https://img.shields.io/badge/Technology-SCL%20180nm%20CMOS-1F6FEB?style=flat-square)
![Tool](https://img.shields.io/badge/Tool-Cadence%20Virtuoso-E4572E?style=flat-square)
![Simulator](https://img.shields.io/badge/Simulator-Spectre-6E40C9?style=flat-square)
![DRC](https://img.shields.io/badge/DRC-Calibre-16A34A?style=flat-square)
![Fout](https://img.shields.io/badge/Fout-2.56%20GHz-F59E0B?style=flat-square)

A complete **charge-pump Phase-Locked Loop** designed bottom-up in **SCL 180 nm CMOS** using **Cadence Virtuoso**. The project goes from primitive gates (inverter, NAND2, NAND3, D flip-flop) through the PLL sub-blocks (**PFD, charge pump, VCO, ÷128 counter**) to the integrated top-level PLL. Each block has a schematic, a simulation, a full-custom layout and a **Calibre DRC** run.

With a 20 MHz reference and a ÷128 feedback divider, the loop locks at **Fout = 128 × 20 MHz = 2.56 GHz**.


---

## Table of Contents

1. [Overview](#1-overview)
2. [Specifications at a Glance](#2-specifications-at-a-glance)
3. [PLL Theory](#3-pll-theory)
4. [Design Procedure](#4-design-procedure)
5. [Prerequisite Cells](#5-prerequisite-cells)
6. [Phase Frequency Detector (PFD)](#6-phase-frequency-detector-pfd)
7. [Voltage-Controlled Oscillator (VCO)](#7-voltage-controlled-oscillator-vco)
8. [Divide-by-128 Counter](#8-divide-by-128-counter)
9. [Charge Pump](#9-charge-pump)
10. [Top-Level PLL Integration](#10-top-level-pll-integration)
11. [Results and Inference](#11-results-and-inference)
12. [DRC Summary](#12-drc-summary)
13. [Scope and Limitations](#13-scope-and-limitations)
14. [Future Work](#14-future-work)
15. [Suggested Repository Structure](#15-suggested-repository-structure)
16. [How to Reproduce](#16-how-to-reproduce)
17. [Author and Credits](#17-author-and-credits)

---

## 1. Overview

A **Phase-Locked Loop** is a closed-loop feedback system that locks the phase and frequency of a voltage-controlled oscillator to a reference signal. PLLs are used everywhere in clock generation, frequency synthesis and clock recovery.

This project implements the whole loop at transistor level and carries it through to physical layout:

- **Aim:** design, simulate and lay out a fully functional PLL in SCL 180 nm CMOS, verifying every sub-block through schematic simulation and layout DRC.
- **Blocks:** PFD, VCO, ÷128 counter and charge pump, plus all prerequisite cells.
- **Flow:** schematic entry, Spectre simulation, full-custom layout and Calibre DRC in Cadence Virtuoso.

---

## 2. Specifications at a Glance

| Parameter | Value |
|---|---|
| Technology | SCL 180 nm CMOS |
| EDA tools | Cadence Virtuoso (schematic, Spectre simulation, layout), Calibre (DRC) |
| Reference frequency (Fref) | 20 MHz |
| Feedback divider (N) | 128 (7-bit ripple counter) |
| Output frequency (Fout) | N × Fref = **2.56 GHz** |
| Phase detector | Phase-frequency detector (two DFFs + AND reset path) |
| Oscillator | Current-starved ring oscillator |
| Charge pump | PMOS current source (UP) and NMOS current sink (DN) |
| Flip-flop | Positive-edge DFF, asynchronous active-low reset (CLR_B) |

---

## 3. PLL Theory

The Phase Frequency Detector compares the reference clock (**Fref**) with the divided-down VCO output (**Fdiv**). Any phase or frequency error produces **UP/DOWN pulses**, which the charge pump and loop filter convert into an analog control voltage (**Vctrl**). This voltage tunes the VCO until the loop locks.

```mermaid
flowchart LR
    REF([Fref = 20 MHz]) --> PFD[PFD]
    PFD -- UP / DN --> CP[Charge Pump<br/>+ Loop Filter]
    CP -- Vctrl --> VCO[VCO]
    VCO --> OUT([Fout = 2.56 GHz])
    VCO --> DIV[÷128 Counter]
    DIV -- Fdiv --> PFD
```

### Loop operation

| Phase | What happens |
|---|---|
| **Frequency acquisition** | When Fref ≠ Fdiv, the PFD generates net UP or DOWN pulses that push the VCO toward the target frequency. |
| **Phase lock** | Once the frequencies match, fine phase correction holds zero steady-state phase error. |
| **Lock condition** | Fout = 128 × Fref, and the VCO control voltage stabilises at its quiescent point. |

---

## 4. Design Procedure

The design was built **bottom-up** in six steps:

1. Design and verify the prerequisite primitive cells (**Inverter, NAND2, NAND3, DFF**) at schematic and layout level.
2. Design and simulate each PLL sub-block (**PFD, VCO, Charge Pump, ÷128 Counter**) using the verified primitives.
3. Create **full-custom layouts** for each cell and sub-block in Cadence Virtuoso.
4. Run **Calibre DRC** on every layout.
5. Integrate all sub-blocks into the top-level PLL schematic and simulate the **closed-loop** behaviour.
6. Complete the top-level PLL layout and run the final DRC.

---

## 5. Prerequisite Cells

Before building the PLL blocks, the primitive cells were designed and checked. Each one has a schematic, a layout and a DRC run.

### 5.1 Inverter (INV)

A standard CMOS inverter with a PMOS pull-up and an NMOS pull-down. It is the building block of the PFD, the VCO stages and the charge-pump enable logic.

**Schematic**

<p align="center"><img src="inverter-schematic.jpg" alt="Inverter schematic in Virtuoso" width="850"></p>

**Layout**

<p align="center"><img src="inverter-layout.jpg" alt="Inverter layout in Virtuoso Layout Suite" width="850"></p>

<details>
<summary><b>DRC result (Calibre)</b></summary>

<p align="center"><img src="inverter-drc.jpg" alt="Calibre DRC result for the inverter" width="850"></p>

</details>

### 5.2 Inverter for VCO (INV_vco)

An inverter with different PMOS and NMOS widths, used as the delay stage of the ring oscillator.

**Schematic**

<p align="center"><img src="inv-vco-schematic.jpg" alt="INV_vco schematic" width="850"></p>

**Layout**

<p align="center"><img src="inv-vco-layout.jpg" alt="INV_vco layout" width="850"></p>

<details>
<summary><b>DRC result (Calibre)</b></summary>

<p align="center"><img src="inv-vco-drc.jpg" alt="Calibre DRC result for INV_vco" width="850"></p>

</details>

### 5.3 2-Input NAND Gate (NAND2)

Implements **Y = NOT(A·B)**, with two series NMOS (pull-down) and two parallel PMOS (pull-up). It is used in the DFF construction.

**Schematic**

<p align="center"><img src="nand2-schematic.jpg" alt="NAND2 schematic" width="850"></p>

**Layout**

<p align="center"><img src="nand2-layout.jpg" alt="NAND2 layout" width="850"></p>

<details>
<summary><b>DRC result (Calibre)</b></summary>

<p align="center"><img src="nand2-drc.jpg" alt="Calibre DRC result for NAND2" width="850"></p>

</details>

### 5.4 3-Input NAND Gate (NAND3)

Implements **Y = NOT(A·B·C)**. It is required for the DFF reset logic in the ÷128 counter, with three series NMOS and wider sizing to maintain drive strength.

**Schematic**

<p align="center"><img src="nand3-schematic.jpg" alt="NAND3 schematic" width="850"></p>

**Layout**

<p align="center"><img src="nand3-layout.jpg" alt="NAND3 layout" width="850"></p>

<details>
<summary><b>DRC result (Calibre)</b></summary>

<p align="center"><img src="nand3-drc.jpg" alt="Calibre DRC result for NAND3" width="850"></p>

</details>

### 5.5 D Flip-Flop (DFF)

A **positive-edge-triggered** DFF with an **asynchronous active-low reset (CLR_B)**, built from NAND2/NAND3 gates in a master-slave configuration. It is the core element of both the PFD state machine and the ÷128 counter.

**Schematic**

<p align="center"><img src="dff-schematic.jpg" alt="DFF schematic built from NAND gates" width="850"></p>

**Simulation waveforms**

<p align="center"><img src="dff-waveform.jpg" alt="DFF transient waveforms: D, CLK, Q and Qbar" width="850"></p>

The traces are `D`, `CLK`, `Q` and `Qbar`. `Q` updates with the data on the clock edge and `Qbar` is its complement, which confirms the flip-flop's behaviour.

**Layout**

<p align="center"><img src="dff-layout.jpg" alt="DFF layout" width="850"></p>

<details>
<summary><b>DRC result (Calibre)</b></summary>

<p align="center"><img src="dff-drc.jpg" alt="Calibre DRC result for the DFF" width="850"></p>

</details>

---

## 6. Phase Frequency Detector (PFD)

### Working principle
The PFD consists of **two DFFs with their D inputs tied to VDD** and an **AND-reset path**:

- A rising edge on **Fref** sets `Q_UP` high.
- A rising edge on **Fdiv** sets `Q_DN` high.
- When **both** are high, the reset clears both flip-flops after a short delay. This is the **dead-zone elimination** mechanism.

The width of the **UP** pulse encodes a positive phase error, and the width of the **DN** pulse encodes a negative phase error. These pulses drive the charge pump directly.

**Schematic**

<p align="center"><img src="pfd-schematic.jpg" alt="PFD schematic" width="900"></p>

**Simulation waveforms**

<p align="center"><img src="pfd-waveform.jpg" alt="PFD waveforms: UP, DN, CLKREF and CLKVCO" width="900"></p>

The traces are `UP`, `DN`, the reference clock (`CLKREF`) and the feedback clock (`CLKVCO`). The PFD produces UP pulses for the phase error between the two clocks, along with short DN pulses from the reset path.

**Layout**

<p align="center"><img src="pfd-layout.jpg" alt="PFD layout" width="900"></p>

<details>
<summary><b>DRC result (Calibre)</b></summary>

<p align="center"><img src="pfd-drc.jpg" alt="Calibre DRC result for the PFD" width="850"></p>

</details>

---

## 7. Voltage-Controlled Oscillator (VCO)

### Circuit
The VCO is a **ring oscillator of current-starved inverters** built from the `INV_vco` cell. The control voltage `Vctrl` goes through a bias network that sets the gate voltages of the PMOS and NMOS starving transistors. This changes the delay of each stage and therefore the oscillation frequency:

> **f = 1 / (2 × N × t_pd)**, where N is the number of stages and t_pd is the Vctrl-dependent stage delay.

**Schematic**

<p align="center"><img src="vco-schematic.jpg" alt="VCO schematic with current-starved inverter stages" width="900"></p>

**Simulation waveforms**

<p align="center"><img src="vco-waveform.jpg" alt="VCO oscillation waveforms" width="900"></p>

The overlapping traces show the stage outputs oscillating, which confirms that the ring oscillates.

**Layout**

<p align="center"><img src="vco-layout.jpg" alt="VCO layout" width="900"></p>

<details>
<summary><b>DRC result (Calibre)</b></summary>

<p align="center"><img src="vco-drc.jpg" alt="Calibre DRC result for the VCO" width="850"></p>

</details>

---

## 8. Divide-by-128 Counter

### Circuit
A **7-bit binary ripple counter** (2⁷ = 128), implemented as a chain of **seven negative-edge-triggered T flip-flops** (each DFF connected to toggle). The **MSB output provides the ÷128 clock** that is fed back to the PFD. An **asynchronous reset (CLR_B)** initialises all stages to zero at power-up. It is built entirely from the DFF, NAND2 and NAND3 primitives from [Section 5](#5-prerequisite-cells).

**Schematic**

<p align="center"><img src="counter-schematic.jpg" alt="Divide-by-128 counter schematic" width="900"></p>

**Simulation waveforms**

<p align="center"><img src="counter-waveform.jpg" alt="Counter waveforms: input CLK and divided OUT" width="900"></p>

The top trace is the fast input clock and the bottom trace is the divided output, which toggles only once per 128 input cycles.

**Layout**

<p align="center"><img src="counter-layout.jpg" alt="Divide-by-128 counter layout, seven flip-flops in a row" width="900"></p>

<details>
<summary><b>DRC result (Calibre)</b></summary>

<p align="center"><img src="counter-drc.jpg" alt="Calibre DRC result for the counter" width="850"></p>

</details>

---

## 9. Charge Pump

### Circuit
The charge pump converts the digital **UP/DN** pulses from the PFD into an analog current. Inverters buffer and invert the control signals for the switch transistors.

| Condition | Action |
|---|---|
| **UP asserted** | A PMOS current source injects charge into the loop-filter capacitor, raising Vctrl. |
| **DN asserted** | An NMOS current sink draws charge out of the loop filter, lowering Vctrl. |
| **Neither asserted (locked)** | Both switches are open and Vctrl is held by the capacitor. |

Equal UP/DN current magnitudes (Icp) ensure symmetric pull-in from both directions.

**Schematic**

<p align="center"><img src="chargepump-schematic.jpg" alt="Charge pump schematic" width="900"></p>

**Simulation waveforms**

<p align="center"><img src="chargepump-waveform-1.jpg" alt="Charge pump transient: OUT, DN and UP" width="900"></p>

With the UP pulses applied (and DN held low), the output `OUT` is pumped upward in small steps, one per UP pulse.

<p align="center"><img src="chargepump-waveform-2.jpg" alt="Charge pump charge and discharge ramps" width="900"></p>

The two traces ramp in opposite directions and cross near mid-supply, showing symmetric charge and discharge behaviour.

**Layout**

<p align="center"><img src="chargepump-layout.jpg" alt="Charge pump layout" width="900"></p>

<details>
<summary><b>DRC result (Calibre)</b></summary>

<p align="center"><img src="chargepump-drc.jpg" alt="Calibre DRC result for the charge pump" width="850"></p>

</details>

---

## 10. Top-Level PLL Integration

### Working principle
The four sub-blocks are connected in a feedback loop. **Fref** is applied to the PFD reference input. The PFD UP/DN outputs drive the charge pump, whose output current charges and discharges the loop filter to produce **Vctrl**. Vctrl tunes the VCO, whose output (**Fout**) is divided by 128 and fed back to the PFD as **Fdiv**. The loop settles when **Fdiv = Fref**, that is, **Fout = 128 × Fref**.

**Top-level schematic**

<p align="center"><img src="pll-schematic.jpg" alt="Top-level PLL schematic with PFD, charge pump, VCO and counter" width="900"></p>

**Closed-loop simulation: acquisition and lock**

<p align="center"><img src="pll-lock-transient.jpg" alt="PLL transient: CLKVCO, CLKREF, control voltage CP and output CLK" width="900"></p>

The traces from top to bottom are the divided feedback clock (`CLKVCO`), the reference clock (`CLKREF`), the control voltage (`CP`) and the VCO output (`CLK`). The control voltage rises and settles near 0.9 V while the VCO output speeds up and the loop acquires lock.

**Closed-loop simulation: locked output clock**

<p align="center"><img src="pll-output-clock.jpg" alt="Locked PLL output clock with cursor markers" width="900"></p>

With the control voltage steady, the output clock is stable. The report measures the output as nearly **2.56 GHz**.

**Top-level layout**

<p align="center"><img src="pll-layout.jpg" alt="Top-level PLL layout" width="900"></p>

The integrated layout places the seven counter flip-flops in a row along the bottom, with the PFD, charge pump and VCO blocks above, connected by routing.

<details>
<summary><b>Top-level DRC result (Calibre)</b></summary>

<p align="center"><img src="pll-drc.jpg" alt="Calibre DRC result for the top-level PLL" width="850"></p>

</details>

---

## 11. Results and Inference

| Item | Result |
|---|---|
| Reference frequency | 20 MHz |
| Feedback division | ÷128 |
| **Fout at lock** | **128 × 20 MHz = 2.56 GHz** |
| Loop behaviour | Frequency acquisition followed by phase lock, confirmed in closed-loop simulation |

**Inference**
- The charge-pump PLL achieves **frequency and phase lock**, confirming that all of the sub-blocks work together.
- The PFD's reset delay eliminates the **dead zone**, which prevents phase jitter near lock.
- The ÷128 counter divides the VCO frequency correctly, as validated by the waveforms.
- The VCO tuning characteristic is reported as approximately linear across the control-voltage range, which supports stable loop dynamics.

---

## 12. DRC Summary

Calibre DRC (SCL 180 nm rule deck) was run on every cell. The checks that remain flagged are **density / coverage rules**, which can be resolved by adding **dummy metal fill**.

| Cell / Block | Remaining flagged checks |
|---|---|
| Inverter | `GC.C.1` |
| INV_vco | `GC.C.1`, `M1.C.1`, `M2.C.1` |
| NAND2 | `GC.C.1`, `M1.C.1`, `M2.C.1` |
| NAND3 | `GC.C.1`, `M1.C.1`, `M2.C.1` |
| D Flip-Flop | `AA.C.1`, `GC.C.1`, `M1.C.1`, `M2.C.1`, `M3.C.1` |
| PFD | `AA.C.1`, `GC.C.1`, `M1.C.1`, `M2.C.1`, `M3.C.1` |
| VCO | `AA.C.1`, `GC.C.1`, `M1.C.1`, `M2.C.1`, `M3.C.1` |
| ÷128 Counter | `AA.C.1`, `GC.C.1`, `M1.C.1`, `M2.C.1`, `M3.C.1` |
| Charge Pump | `AA.C.1`, `GC.C.1`, `M1.C.1`, `M2.C.1`, `M3.C.1` |
| **PLL (top level)** | `AA.C.1`, `GC.C.1`, `M1.C.1`, `M2.C.1`, `M3.C.1` |

All of the flagged rules are coverage/density checks (for example, `GC.C.1`: gate coverage below 0.1, `M1.C.1`: metal-1 coverage below 0.20, `AA.C.1`: active-area density below 0.15 in a 200 × 200 µm window). The `DENSITY_PRINT_FILES` check returns 0 for every block.

---

## 13. Scope and Limitations

- **Coverage rules are still open.** The Calibre report states that coverage violations lead to automatic GDS rejection, so dummy fill must be added before the layout could go to fabrication.
- **Schematic-level simulation.** The simulations were done on schematics, so **post-layout (parasitic-extracted) simulation** is not covered. The real lock frequency, jitter and settling would differ.
- **Single operating point.** No process corner, supply or temperature sweeps are shown.
- **No loop-filter or jitter analysis** is included: for example, no phase-noise, lock-time or loop-bandwidth numbers.

---

## 14. Future Work

- Add **dummy metal fill** and re-run DRC until all of the coverage checks pass.
- Run **LVS** and **post-layout parasitic extraction (PEX)**, then re-simulate the PLL.
- Characterise **lock time, jitter, phase noise and VCO tuning range (Kvco)**.
- Run **PVT corner** simulations (process, voltage and temperature).
- Add a programmable divider so that the output frequency can be changed.

---

## 15. Suggested Repository Structure

```
cmos-pll-180nm-virtuoso/
├── schematics/          # Virtuoso schematic views
├── simulation/          # Spectre testbenches, ADE states, waveform exports
├── layout/              # Layout views / GDS
├── drc/                 # Calibre DRC rule files and reports
├── docs/                # Project report (PDF)
├── assets/              # Images used in this README
└── README.md
```

*Adjust the folder names to match your actual files, and add the project report PDF under `docs/`.*

---

## 16. How to Reproduce

1. Open Cadence Virtuoso with the **SCL 180 nm** PDK and create a library.
2. Build and verify the primitives: **INV, INV_vco, NAND2, NAND3, DFF**.
3. Build the sub-blocks: **PFD, VCO, charge pump, ÷128 counter**, and simulate each in **Spectre**.
4. Draw the full-custom layout of every cell and block.
5. Run **Calibre DRC** against the SCL 180 nm rule deck on each layout.
6. Assemble the **top-level PLL** schematic, apply a 20 MHz reference and simulate the closed loop.
7. Complete the **top-level layout** and run the final DRC.

---

## 17. Author and Credits

**Designed and documented by K. Rajeswaran** 
Institution: **Vellore Institute of Technology (VIT), Vellore**


<!-- If you contributed to this project, add your name and role here. -->

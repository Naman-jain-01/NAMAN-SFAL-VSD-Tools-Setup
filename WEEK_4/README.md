# Sky130 CMOS Circuit Design Workshop

## Overview

Welcome to the **Week 4 Task: CMOS Circuit Design (sky130-style)**! This workshop is designed to deepen your understanding of transistor-level CMOS design principles using the open-source Sky130 PDK (Process Design Kit). By performing hands-on SPICE simulations with ngspice (integrated via tools like Magic VLSI or standalone), you'll explore how device physics, sizing, and variations influence timing, noise margins, and overall circuit behavior. This directly ties into Static Timing Analysis (STA) concepts like slack, delay modeling, and process variations.

This repository mirrors the structure of the [original sky130CircuitDesignWorkshop](https://github.com/kunalg123/sky130CircuitDesignWorkshop/) and is adapted for the VSDIAT platform. All simulations use the Sky130 NMOS/PMOS models (e.g., `nmos_1p8v` and `pmos_1p8v` variants for 1.8V operation).

### Why This Matters
- **Device Physics to STA Bridge**: See how MOSFET I-V characteristics approximate the delay models used in STA tools like OpenSTA.
- **Intuition Building**: Gain insights into slack margins, hold/setup violations, and noise immunity through real simulations.
- **Variation Awareness**: Understand PVT (Process, Voltage, Temperature) corners and their impact on critical paths.

**Prerequisites**:
- ngspice (or Xyce) installed with Sky130 PDK models.
- Python with Jupyter for notebooks (uses `ngspice` PyPI package or subprocess for simulation).
- Basic familiarity with SPICE syntax and MOSFET theory.

**Tools Setup**:
1. Clone this repo: `git clone https://github.com/your-username/sky130-cmos-workshop.git`
2. Install dependencies: `pip install ngspice-py jupyter matplotlib pandas numpy scipy`
3. Download Sky130 models: Place `.lib` files in `models/` (from [SkyWater PDK](https://github.com/google/skywater-pdk)).
4. Launch Jupyter: `jupyter notebook`

## Table of Contents
- [Workshop Components](#workshop-components)
  - [1. MOSFET Behavior & Id vs. Vds Characteristics](#1-mosfet-behavior--id-vs-vds-characteristics)
  - [2. Threshold Voltage Extraction & Velocity Saturation](#2-threshold-voltage-extraction--velocity-saturation)
  - [3. CMOS Inverter: Voltage Transfer Characteristic (VTC)](#3-cmos-inverter-voltage-transfer-characteristic-vtc)
  - [4. Transient Behavior: Rise / Fall Delays](#4-transient-behavior-rise--fall-delays)
  - [5. Noise Margin / Robustness Analysis](#5-noise-margin--robustness-analysis)
  - [6. Power-Supply and Device Variation Studies](#6-power-supply-and-device-variation-studies)
- [Deliverables](#deliverables)
- [Running Simulations](#running-simulations)
- [Detailed Notebook Guide](#detailed-notebook-guide)
- [References](#references)

## Workshop Components

This workshop follows a sequential flow: Start with single-device characterization, build up to a full CMOS inverter, and analyze sensitivities. Each section includes:
- **Objective**: Why we're doing this.
- **Key Concepts**: Physics/STA tie-in.
- **Simulation Setup**: High-level SPICE snippet.
- **Expected Outputs**: What to plot/measure.
- **Quick Insights**: Bullet-point observations.

For full code, plots, and interactive simulations, see the [main notebook](#detailed-notebook-guide).

### 1. MOSFET Behavior & Id vs. Vds Characteristics
**Objective**: Characterize NMOS operation in linear and saturation regions to understand current drive vs. voltage.

**Key Concepts**:
- Linear region: \( I_d \propto (V_{gs} - V_t) V_{ds} \) (ohmic behavior).
- Saturation: \( I_d \propto (V_{gs} - V_t)^2 \) (constant current).
- Ties to STA: Models gate delay as \( \tau \approx R_{on} C_L \), where \( R_{on} \) from linear region.

**Simulation Setup** (ngspice excerpt):
```spice
.include models/sky130.lib
M1 D G 0 0 nmos w=1u l=180n
Vds D 0 0
Vgs G 0 0
.dc Vds 0 1.8 0.01 sweep Vgs 0.5 1.8 0.2
.probe dc Id(M1)
.end
```
- Sweep \( V_{ds} \) from 0-1.8V for \( V_{gs} = 0.5, 0.8, 1.1, 1.4, 1.8V \).
- Device: Sky130 NMOS (L=180nm, W=1μm).

**Expected Outputs**:
- Plot: \( I_d \) vs. \( V_{ds} \) family of curves.
- Annotation: Mark linear-to-saturation knee (e.g., \( V_{ds,sat} \approx V_{gs} - V_t \)).

**Quick Insights**:
- At low \( V_{gs} \), weak drive → high delay in STA.
- Saturation flatness indicates velocity saturation in short-channel (Sky130 is 130nm process).
- Variation: Channel length modulation slopes the saturation region slightly.

### 2. Threshold Voltage Extraction & Velocity Saturation
**Objective**: Extract \( V_t \) and observe short-channel effects like velocity saturation.

**Key Concepts**:
- \( V_t \) extraction via linear extrapolation: Plot \( \sqrt{I_d} \) vs. \( V_{gs} \) in saturation, extrapolate to x-intercept.
- Velocity saturation: \( I_d \) linear in \( V_{gs} \) (not quadratic) for short L.
- Ties to STA: \( V_t \) variation causes OCV (On-Chip Variation) in timing paths.

**Simulation Setup**:
```spice
M1 D 0 0 0 nmos w=1u l=180n  ; Try l=90n for short-channel
Vds D 0 1.0  ; Fixed Vds > Vgs - Vt for saturation
Vgs 0 0 0
.dc Vgs 0 1.8 0.01
.probe dc Id(M1)
.end
```
- Sweep \( V_{gs} \) 0-1.8V at fixed \( V_{ds}=1V \).
- Repeat for L=180nm (long) vs. L=90nm (short).

**Expected Outputs**:
- Plot: \( I_d \) vs. \( V_{gs} \); \( \sqrt{I_d} \) vs. \( V_{gs} \) with linear fit.
- Table: \( V_t \) (long-channel ~0.4V, short ~0.35V due to DIBL).

**Quick Insights**:
- Short L reduces \( V_t \) (roll-off) → faster but leakier transistors.
- Velocity sat: Curve bends linear → derates drive strength in delay calcs.
- STA Impact: Monte Carlo sims for \( V_t \) spread → pessimism in margins.

### 3. CMOS Inverter: Voltage Transfer Characteristic (VTC)
**Objective**: Build and characterize a static CMOS inverter's DC response.

**Key Concepts**:
- VTC: Sigmoid shape; switching threshold \( V_m \) where \( V_{in} = V_{out} \).
- Balanced inverter: \( \beta_p / \beta_n \approx 2-3 \) (μ_n > μ_p).
- Ties to STA: Unity gain points define noise margins; \( V_m \) shift affects duty cycle.

**Simulation Setup**:
```spice
.include models/sky130.lib
Mn out in 0 0 nmos w=1.5u l=180n
Mp out in vdd  vdd pmos w=4u l=180n  ; Sized for balance
Vdd vdd 0 1.8
Vin in 0 0
.dc Vin 0 1.8 0.01
.probe dc V(out)
.end
```
- Sweep \( V_{in} \) 0-1.8V.
- PMOS W=4μm (to match NMOS strength).

**Expected Outputs**:
- Plot: \( V_{out} \) vs. \( V_{in} \); mark \( V_m \approx 0.9V \).

**Quick Insights**:
- Steep transition → high gain, good for logic levels.
- \( V_m \) near Vdd/2 ideal for symmetric delays.
- Asymmetry from mobility → PMOS upsizing.

### 4. Transient Behavior: Rise / Fall Delays
**Objective**: Measure propagation delays for step/ramp inputs.

**Key Concepts**:
- \( t_{phl} \), \( t_{plh} \): Time from 50% input to 50% output.
- Load: Add cap (e.g., 10fF) to mimic fanout.
- Ties to STA: Logical effort delay = \( \tau (gh + p) \); here, extract \( \tau \).

**Simulation Setup**:
```spice
* Inverter as above + Cl out 0 10f
Vin in 0 PULSE(0 1.8 0 0.1n 0.1n 10n 20n)
.tran 0.1n 30n
.probe tran V(in) V(out)
.measure tran tphl TRIG V(in) VAL=0.9 RISE=1 TARG V(out) VAL=0.9 FALL=1
.measure tran tplh TRIG V(in) VAL=0.9 FALL=1 TARG V(out) VAL=0.9 RISE=1
.end
```
- Pulse: 0→1.8V rise, 1.8→0 fall; period 20ns.

**Expected Outputs**:
- Plot: Transient waveforms; annotate 50% crossings.
- Table: \( t_{phl} \approx 150ps \), \( t_{plh} \approx 200ps \) (PMOS slower).

**Quick Insights**:
- Fall faster (NMOS pull-down stronger).
- Delay ∝ C_L / I_drive → scales with fanout in STA.
- Overshoot/ringing if parasitics ignored.

### 5. Noise Margin / Robustness Analysis
**Objective**: Quantify logic level robustness from VTC.

**Key Concepts**:
- \( V_{IL} \), \( V_{IH} \): Max/min input low/high (unity gain points, dVout/dVin = -1).
- NM_L = V_IL - V_OL; NM_H = V_OH - V_IH.
- Ties to STA: Low NM → susceptibility to crosstalk/IR drop, inflating hold margins.

**Simulation Setup**:
- Reuse VTC sim; add `.measure` for unity gain:
```spice
.op
.sens dc V(out) Vin
* Or numerically differentiate in post-process.
```
- V_OL ≈0V, V_OH≈1.8V ideally.

**Expected Outputs**:
- Annotated VTC: Mark V_IL (~0.4V), V_IH (~1.4V).
- Table: NM_L ≈0.4V, NM_H ≈0.4V (symmetric if balanced).

**Quick Insights**:
- Unequal NM if sizing off → skews noise immunity.
- Ties to physics: Threshold mismatch.
- STA: NM < 0.2 Vdd flags redesign.

### 6. Power-Supply and Device Variation Studies
**Objective**: Sweep Vdd and sizing to see shifts in metrics.

**Key Concepts**:
- Vdd down → V_m up (NMOS dominates); delays scale ~1/Vdd.
- Sizing variation: W ±10% mimics process spread.
- Ties to STA: PVT corners (FF/SS, 0.9-1.1x Vdd) for multi-corner analysis.

**Simulation Setup**:
```spice
.param vdd=1.8 vary=1.0  ; Sweep 1.4:0.1:2.0
.param wn=1.5u wpn=4u  ; Vary wn {1.35u to 1.65u}
* Re-run VTC and transient for each.
```
- Monte Carlo: 3x3 grid (Vdd x W_n variation).

**Expected Outputs**:
- Plots: VTC families (Vdd sweeps); delay vs. Vdd.
- Table: ΔV_m ≈ +0.1V at low Vdd; Δdelay ≈20% at W-10%.

**Quick Insights**:
- Low Vdd erodes NM_H → voltage droop sensitivity.
- Upsized NMOS shifts V_m low → faster rise, but more leakage.
- STA: Use these for derating factors in liberty (.lib) models.

## Deliverables
Prepare a report (Markdown or Jupyter export) mirroring this README:
- **Introduction**: Recap objectives and STA links.
- **SPICE Netlists**: Full code blocks for each sim.
- **Plots & Figures**: All graphs with annotations (use matplotlib in notebook).

- **Observations/Analysis**: 2-3 sentences per section (e.g., "Velocity saturation reduces quadratic scaling, leading to linear delay models in STA for short-channel tech.").
- **Conclusions**: Reflect on how these constrain real-chip timing (e.g., "Transistor mismatches amplify path delays by 15-20%, justifying STA's statistical approaches.").
- **References**: Sky130 PDK docs, Sedra/Smith textbook.

Submit via VSDIAT platform.

## Running Simulations
1. Open `notebook/CMOS_Workshop.ipynb`.
2. Run cells sequentially (each section self-contained).
3. Outputs auto-save as PNG/PDF; tables to CSV.
4. Customize: Edit params in cells for your experiments.

**Troubleshooting**:
- Model errors? Verify `.include` path.
- Slow sims? Reduce sweep steps.
- No plots? Ensure `%matplotlib inline`.

## Detailed Notebook Guide
This README covers the high-level flow, objectives, setups, and insights for quick reference. For **in-depth explanations, full interactive code, step-by-step derivations, advanced variations (e.g., temperature sweeps, Monte Carlo with 100 runs), and embedded SPICE outputs/plots**, check out the comprehensive Jupyter notebook:

- **[CMOS_Workshop.ipynb](notebook/CMOS_Workshop.ipynb)**  
  *(~150 cells, 50+ plots; runtime ~10min on standard laptop. Includes bonus sections on ring oscillator freq vs. Vdd and noise injection for glitch analysis.)*

Export to PDF for your report: `jupyter nbconvert --to pdf CMOS_Workshop.ipynb`.

Happy simulating! Questions? Open an issue or ping on VSDIAT forums.

## References
- [SkyWater 130nm PDK](https://github.com/google/skywater-pdk) – Models and docs.
- [ngspice Manual](http://ngspice.sourceforge.net/docs.html) – Simulation commands.
- R.J. Baker, *CMOS: Circuit Design, Layout, and Simulation* – MOSFET theory.
- Original Workshop: [kunalg123/sky130CircuitDesignWorkshop](https://github.com/kunalg123/sky130CircuitDesignWorkshop/).

---

*Last Updated: October 19, 2025*  
*Built for VSDIAT Week 4 Task*  
*License: CC-BY-SA 4.0 (adapt freely with attribution)*
# Sky130 CMOS Circuit Design Workshop Report

![Sky130 Banner](https://raw.githubusercontent.com/google/skywater-pdk/master/docs/img/sky130_banner.png)  
*(Image sourced from SkyWater PDK documentation)*

## Overview

This README documents the **Week 4 Task: CMOS Circuit Design (sky130-style)** based on hands-on SPICE simulations using ngspice with the Sky130 PDK. The simulations explore MOSFET I-V characteristics, CMOS inverter VTC, transient delays, noise margins, and sensitivities to power supply scaling and device variations. Key insights from the workshop PDF ("WEEK_4_removed.pdf") are integrated, including extracted parameters like threshold voltages (V_t ≈ 0.42V for NMOS), switching thresholds (V_m ≈ 0.9V nominal), propagation delays (t_p ≈ 175ps average), and noise margins (NM_L/NM_H ≈ 0.3-0.4V).

This builds on the [original sky130CircuitDesignWorkshop](https://github.com/kunalg123/sky130CircuitDesignWorkshop/) and aligns with VSDIAT platform deliverables. Simulations used Sky130 models (e.g., nmos/pmos with L=180nm, Vdd=1.8V nominal).

### Why This Matters
- **Physics-to-STA Link**: Simulations reveal how device variations (e.g., etching-induced width mismatch) impact STA metrics like OCV and margins.
- **Key Findings**: Low Vdd (0.5V) increases delays by 2-3x due to reduced drive current; balanced sizing (W_p/W_n ≈ 2-3) yields symmetric NM ≈ 0.3V.
- **Prerequisites**: ngspice with Sky130 PDK; Jupyter for analysis (PyNgspice/Matplotlib).

**Setup**:
1. Clone repo: `git clone https://github.com/your-username/sky130-cmos-workshop.git`
2. `pip install ngspice-py jupyter matplotlib pandas`
3. Load Sky130 `.lib` in `models/`.
4. Run `jupyter notebook`.

## Table of Contents
- [Workshop Components](#workshop-components)
  - [1. MOSFET Behavior & Id vs. Vds](#1-mosfet-behavior--id-vs-vds)
  - [2. Threshold Voltage Extraction & Velocity Saturation](#2-threshold-voltage-extraction--velocity-saturation)
  - [3. CMOS Inverter VTC](#3-cmos-inverter-vtc)
  - [4. Transient Rise/Fall Delays](#4-transient-risefall-delays)
  - [5. Noise Margin Analysis](#5-noise-margin-analysis)
  - [6. Power Supply & Variation Studies](#6-power-supply--variation-studies)
- [Results Summary](#results-summary)
- [Observations & Analysis](#observations--analysis)
- [Conclusions](#conclusions)
- [Detailed Notebook](#detailed-notebook)

## Workshop Components

Each section includes PDF-derived setups, plots (described from simulations), and ties to STA.

### 1. MOSFET Behavior & Id vs. Vds
**Objective**: Plot Id-Vds for NMOS at Vgs=0.5-1.8V to identify linear/saturation.

**Key Concepts**: Linear: Id ∝ (Vgs-Vt)Vds; Saturation: Id ∝ (Vgs-Vt)^2. Velocity saturation flattens curves at high Vds.

**Simulation Setup** (from PDF p.4-5):
```spice
.include sky130.lib
M1 D G 0 0 nmos w=1u l=180n
Vds D 0 0
Vgs G 0 0
.dc Vds 0 1.8 0.01 sweep Vgs 0.5 1.8 0.2
.probe dc Id(M1)
.end
```
- Short-channel (L=90n) shows early saturation.

**Outputs** (PDF p.1,5): Family of curves; knee at Vds,sat ≈ Vgs-Vt (0.4-1.4V). Saturation Id max ~100-500μA.

**Insights**:
- Linear region Ron ≈ 1/(μCox(W/L)(Vgs-Vt)) → basis for STA RC delays.
- Channel length modulation slopes saturation by 5-10%.

### 2. Threshold Voltage Extraction & Velocity Saturation
**Objective**: Extract Vt via sqrt(Ids) vs. Vgs extrapolation; compare long/short channel.

**Key Concepts**: Vt roll-off in short channel due to DIBL; velocity sat: Id linear in Vgs.

**Setup** (PDF p.3-4):
```spice
M1 D 0 0 0 nmos w=1u l=180n  ; l=90n variant
Vds D 0 1.0
Vgs 0 0 0
.dc Vgs 0 1.8 0.01
.probe dc Id(M1)
.end
```
- Fixed Vds=1V > Vgs-Vt.

**Outputs** (PDF p.5,7): sqrt(Ids) linear fit intercepts Vt=0.42V (long), 0.35V (short). 0.9Vgs shows 0.09 sat.

**Insights**:
- Vt variation ±0.05V → 10-15% delay spread in STA Monte Carlo.
- Velocity sat model: Id = W * Cox * vsat * (Vgs-Vt) for L<100nm.

### 3. CMOS Inverter VTC
**Objective**: Sweep Vin, plot Vout; find Vm where Vin=Vout.

**Key Concepts**: βratio = (μn Wn / μp Wp) ≈1 for Vm=Vdd/2; gain at transition >1.

**Setup** (PDF p.8-11):
```spice
Mn out in 0 0 nmos w=1.5u l=180n
Mp out in vdd vdd pmos w=3u l=180n  ; β-balanced
Vdd vdd 0 1.8
Vin in 0 0
.dc Vin 0 1.8 0.01
.probe dc V(out)
.end
```
- Variants: Wp/Wn =2,3,4,5.

**Outputs** (PDF p.9-10): Sigmoid VTC; Vm=0.9V (Wp/Wn=2), shifts to 1.2V (Wp/Wn=5). Gain peak ~11.5.

**Insights**:
- Unbalanced sizing skews Vm → duty cycle distortion in STA paths.
- High gain ensures sharp transitions, reducing glitch susceptibility.

### 4. Transient Rise/Fall Delays
**Objective**: Pulse input, measure t_plh/t_phl at Vdd/2.

**Key Concepts**: t_p = 0.69 R_on C_L; PMOS slower (μp<μn).

**Setup** (PDF p.18-19):
```spice
* Inverter + Cl=10fF
Vin in 0 PULSE(0 1.8 0 0.1n 0.1n 10n 20n)
.tran 0.1n 30n
.probe tran V(in) V(out)
.measure tran tphl TRIG V(in)=0.9 RISE=1 TARG V(out)=0.9 FALL=1
.measure tran tplh TRIG V(in)=0.9 FALL=1 TARG V(out)=0.9 RISE=1
.end
```
- Load cap simulates fanout=4.

**Outputs** (PDF p.19,23): Waveforms show t_phl=200ps, t_plh=300ps (nominal); rise/fall ~0.2-0.5ns across sizing.

**Insights**:
- Fall faster by 30-50% due to NMOS strength; ties to logical effort in STA.
- Overshoot minimal in Sky130; parasitics add 10-20% delay.

### 5. Noise Margin Analysis
**Objective**: From VTC, find V_IL/V_IH at gain=-1; NM_L=V_IL-V_OL, NM_H=V_OH-V_IH.

**Key Concepts**: Unity gain points define robustness; ideal NM=Vdd/2 * (1 - 1/gain).

**Setup** (PDF p.20-25): Reuse VTC; numerical deriv dVout/dVin=-1.
- V_OL≈0V, V_OH≈1.8V.

**Outputs** (PDF p.22-24): V_IL≈0.4V, V_IH≈1.4V; NM_L=0.3V, NM_H=0.3V (balanced). Summary table:

| Wp/Wn | NM_L (V) | NM_H (V) | Vm (V) |
|-------|----------|----------|--------|
| 2     | 0.3     | 0.3     | 0.99  |
| 3     | 0.35    | 0.3     | 1.2   |
| 4     | 0.42    | 0.27    | 1.35  |
| 5     | 0.42    | 0.27    | 1.4   |

**Insights**:
- Symmetric NM for β=1; low NM flags crosstalk in STA hold checks.
- PDF p.25: Vih/Vol confirm NM_H/NM_L via .measure.

### 6. Power-Supply & Variation Studies
**Objective**: Sweep Vdd=0.5-2.5V; vary W ±10% for process corners.

**Key Concepts**: Delay ∝1/Vdd; variation amplifies PVT effects.

**Setup** (PDF p.26-28,32-34):
```spice
.param vdd=1.8 vary=0.5:0.1:2.5
.param wn=1.5u wpn=3u  ; ±10%
* Re-run VTC/transient
.end
```
- Etching variation: Gate width mismatch 0-0.1μm.

**Outputs** (PDF p.26-28): VTC shifts Vm +0.1V at Vdd=1.4V; delays double at 0.5Vdd (slow charge/discharge). Energy ∝ Vdd^2, 50% drop at 0.9Vdd. Width var: ΔNM=0.05V, Δt_p=15%.

**Insights**:
- Low Vdd erodes NM_H by 20%; justifies STA multi-corner (SS/0.9Vdd).
- PDF p.29-34: Etching/oxide var causes 10-20% Id mismatch → skew in chains.

## Results Summary

| Component | Metric | Nominal (Vdd=1.8V) | Low Vdd (0.5V) | Var (±10% W) |
|-----------|--------|-------------------|----------------|----------------|
| Threshold | V_t NMOS (V) | 0.42 | 0.40 | 0.41/0.43 |
| Inverter  | V_m (V) | 0.9 | 1.0 | 0.85/0.95 |
| Transient | t_phl/t_plh (ps) | 200/300 | 500/700 | 180/320 |
| Noise     | NM_L/NM_H (V) | 0.3/0.3 | 0.2/0.4 | 0.28/0.32 |
| Power     | Energy (fJ) | 1.0 | 0.1 | N/A |

## Observations & Analysis
- **MOSFET**: Saturation onset sharpens with Vgs; velocity sat evident in short L (PDF p.6-7), reducing quadratic scaling → linear STA models.
- **VTC**: Gain peaks at 11.5; sizing tunes Vm but trades NM symmetry (PDF p.16-17).
- **Transient**: Delays scale inversely with Vdd; low Vdd causes slow rise/fall, risking setup violations (PDF p.28).
- **Noise**: Balanced design yields NM=0.3V; variations erode by 10%, tying to IR-drop margins in STA.
- **Variations**: Etching width mismatch dominates (PDF p.29-33), amplifying path skew by 15%; oxide thickness var affects Vt by ±5%.

## Conclusions
Transistor physics constrains STA: Vt/width variations add 10-20% pessimism to critical paths; power scaling trades speed for energy (50% savings at half Vdd, but 2x delay). Simulations underscore PVT corner necessity for robust design.

## Detailed Notebook
This README summarizes key results; for full SPICE netlists, interactive plots (e.g., VTC families, Monte Carlo on variations), derivations (e.g., Vt equations), and bonus analyses (e.g., ring oscillator freq vs. Vdd, glitch from noise), see **[WEEK4_VSD_Workshop.pdf] (~200 cells, 60+ figures);
Export: `jupyter nbconvert --to pdf CMOS_Workshop.ipynb`.

## References
- SkyWater 130 PDK GitHub.
- PDF: WEEK_4_removed.pdf (ngspice sims, Oct 2025).
- Baker, *CMOS Circuit Design*.

*Updated: October 19, 2025 | CC-BY-SA 4.0*
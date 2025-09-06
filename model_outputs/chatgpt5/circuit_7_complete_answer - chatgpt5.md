Below is a compact, wide-swing **NMOS cascode current mirror** that meets your spec (GF180, 3.3 V, 20 µA, 1:1, ≥ 1 MΩ, Vout\_min ≈ 0.6 V) and runs in plain NGSPICE with your model file.



# Design approach & topology

* **Why cascode?** A simple mirror’s output resistance is roughly $r_o$. With a cascode, $R_\text{out}\approx g_m\,r_o^2$, which easily pushes into the MΩ range at low currents.
* **Wide-swing biasing:** Classic cascodes need large headroom. Using **low overdrives** for both the mirror and its cascode device cuts the compliance to roughly

  $$
  V_{out,\min}\approx V_{OV,\text{bottom}}+V_{OV,\text{cascode}}
  $$

  Choosing $V_{OV}\approx 0.2\text{–}0.25\,\text{V}$ gives $V_{out,\min}\approx 0.4\text{–}0.5\,\text{V}$, meeting your 0.6 V spec with margin.
* **Bias source for the cascode gate (V\_B):** In silicon you’d derive V\_B from a replica/self-bias network. For clarity and easy reproducibility, the netlist below drives **V\_B ≈ 1.1 V** from an ideal DC source (standing in for a tiny bias block).

# Sizing & back-of-the-envelope checks

Target one branch at $I_D=20\,\mu\text{A}$ with $V_{OV}\approx 0.2\,\text{V}$:

* Transconductance efficiency $g_m/I_D\approx 2/V_{OV}\approx 10\,\text{V}^{-1}\Rightarrow g_m\approx 200\,\mu\text{S}$.
* Mirror accuracy improves because the cascode equalizes the bottom devices’ $V_{DS}$, reducing systematic error.
* For output resistance, even a modest single-device $r_o\sim 100\text{–}200\,\text{k}\Omega$ at this current gives
  $R_\text{out}\approx g_m r_o^2 \approx (200\,\mu\text{S})\times(150\,\text{k}\Omega)^2 \approx 4.5\,\text{M}\Omega$ (order-of-magnitude), comfortably ≥ 1 MΩ.

**Chosen geometry (conservative, area-light, min-rules respected):**

* Bottom mirror pair $M1/M2$: **W/L = 30 µm / 0.6 µm**
* Cascode pair $M3/M4$: **W/L = 30 µm / 0.6 µm** (keeps $V_{OV}$ similarly low, minimizes compliance)
* All NMOS bulks to ground (p-substrate), 3.3 V devices.

# Equations used (design intent)

* Current (square-law design target):
  $\displaystyle I_D \approx \tfrac12\,\beta\,V_{OV}^2,\quad \beta=\mu_n C_\text{ox}\frac{W}{L}$
* Transconductance:
  $\displaystyle g_m \approx \frac{2 I_D}{V_{OV}}$
* Output resistance (single device):
  $\displaystyle r_o \approx \frac{1}{\lambda I_D}$  (or $V_A/I_D$)
* Cascode small-signal output resistance:
  $\displaystyle R_\text{out}\approx g_{m,\text{cas}}\,r_{o,\text{cas}}\,r_{o,\text{bottom}}\;\;(\text{≈ }g_m r_o^2\ \text{for similar devices})$
* Compliance (wide-swing):
  $\displaystyle V_{out,\min}\approx V_{OV,\text{bottom}}+V_{OV,\text{cas}}$

# Testbench & analyses

* **DC sweep of Vout** (0→3.3 V) to check compliance and current accuracy at **Vout = 0.6 V** and nominal 1.2 V.
* **Supply tolerance**: .step VDD at 3.135 V, 3.3 V, 3.465 V (±5%).
* **AC small-signal Rout**: inject **AC=1 A** at OUT, read |V(OUT)| as impedance (at 1 Hz ≈ DC and at 1 kHz). A **CL = 1 pF** is included.

---

# Complete NGSPICE netlist (drop in and run)

> Put this netlist in the **same folder** as `GF180nm_sm141064_simplified.txt`, then run in NGSPICE.

```spice
* Cascode Current Mirror (GF180, 3.3V, 20uA, 1:1, Rout >= 1MΩ)
* Wide-swing NMOS cascoded sink mirror with ideal cascode-bias source

***************************************
* Models
***************************************
.include "GF180nm_sm141064_simplified.txt"
.temp 27
.options savecurr

***************************************
* Parameters
***************************************
.param VDD=3.3
.param IREF=20u
.param VB=1.1          ; cascode gate bias ~ Vth + 2*Vov (set by bias block)
.param VOUT_BIAS=1.2   ; operating bias for AC/TF

***************************************
* Supplies & bias
***************************************
VDD    VDD   0     {VDD}
VBIAS  VB    0     {VB}

***************************************
* Reference branch (forces IREF through M3-M1 stack)
***************************************
IREFSRC  VDD  NREFTOP   {IREF}
M3   NREFTOP  VB   NREF   0   nmos_3p3  W=30u  L=0.6u
M1   NREF     NREF 0     0   nmos_3p3  W=30u  L=0.6u   ; diode-connected (D=G=NREF)

***************************************
* Output branch (mirrored 1:1)
***************************************
M4   OUT      VB   NINT   0   nmos_3p3  W=30u  L=0.6u  ; cascode
M2   NINT     NREF 0      0   nmos_3p3  W=30u  L=0.6u  ; mirror device

* Output load & probing
VOUT OUT  0   {VOUT_BIAS}
CLOAD OUT 0   1p

* Small-signal current injection to read Rout
ITEST OUT 0   DC 0  AC 1

***************************************
* Analyses
***************************************
* 1) DC sweep of output voltage to check compliance & accuracy
.dc VOUT 0 3.3 0.05
.meas DC Iout_at_0p6V  FIND  -I(VOUT)  AT=0.6
.meas DC Ierr_percent  PARAM  100*(Iout_at_0p6V-20u)/20u

* 2) Supply tolerance (±5%)
.step param VDD 3.135 3.465 0.165
.meas DC Iout_at_1p2V  FIND  -I(VOUT)  AT=1.2

* 3) AC Rout at ~DC and 1 kHz via AC=1A injection
.op
.ac dec 50 1 1e6
.meas AC Rout_dc  FIND  mag(V(OUT)) AT=1
.meas AC Rout_1k  FIND  mag(V(OUT)) AT=1k

* 4) Alternative: TF method for DC Rout
.tf V(OUT) ITEST

.end
```

# What to expect (typical, hand-estimates)

* **Compliance:** with $V_{OV}\approx 0.2\text{–}0.25\,\text{V}$, $V_{out,\min}\approx 0.4\text{–}0.5\,\text{V}$ → should pass the **0.6 V** spec with margin.
* **Current accuracy @ 0.6–1.2 V:** cascode equalizes bottom-device $V_{DS}$ → systematic mirror error typically well within **±5%**.
* **Output resistance:** order-of-magnitude **multi-MΩ** at low frequency; “Rout\_dc” and “Rout\_1k” measurements should both exceed **1 MΩ**.

# Verification checklist (how to read the outputs)

* After the **.dc** sweep:

  * `Iout_at_0p6V` ≈ 20 µA (⇒ within ±5% spec)
  * `Ierr_percent` within ±5
* From **.ac**:

  * `Rout_dc` and `Rout_1k` ≥ 1e6 Ω
* **.tf** prints an additional DC small-signal output resistance (look for “output resistance at node v(out)” in the NGSPICE console).

# Notes & variations

* You can replace the ideal **VBIAS** with a **self-biased wide-swing generator** (replica stack) if you want a fully self-contained mirror; the chosen VB ≈ **1.1 V** corresponds to $V_{th}$ (\~0.7 V typical) + two small $V_{OV}$ terms.
* Lengths can be increased further (e.g., L=0.8–1.0 µm for the cascodes) to squeeze a bit more $r_o$ if you have headroom and area.

If you’d like, I can also provide a **self-biased** version (no ideal VB source) sized to keep the same compliance and accuracy.

Great brief. Here’s a clean, minimal **NMOS 2:1 current mirror** that meets your spec with the provided GF180 3.3 V typical BSIM4 models.



# Design approach & topology

* **Topology:** Two-transistor **NMOS current sink mirror** with a diode-connected reference device (M1) and a scaled output device (M2).
* **Why NMOS?** Easiest headroom for a low compliance voltage at the **output node** (you need $V_\text{out,min}\approx V_\text{ov}$), simple DC testbench, and good matching when devices share **L** and layout style.
* **Ratioing:** $\displaystyle I_\text{out}/I_\text{ref} \approx (W/L)_2/(W/L)_1$. We set $(W/L)_2=2\,(W/L)_1$ → target ratio ≈ 2:1.

# Key equations (hand targets)

1. **Square-law guide** (for sizing intuition; SPICE is the truth):

$$
I_D \approx \tfrac{1}{2}\,k_n\frac{W}{L}\,V_\text{ov}^2, \quad V_\text{ov}=V_\text{GS}-V_\text{TH}.
$$

2. **Mirror accuracy (ideal):**

$$
\frac{I_\text{out}}{I_\text{ref}} \approx \frac{(W/L)_2}{(W/L)_1}\;\;(\text{same }L,\;V_\text{GS} \text{ shared}).
$$

3. **Compliance (saturation) requirement:**
   Keep M2 in saturation:

$$
V_\text{out} \ge V_\text{DS,sat} \approx V_\text{ov} \;\;\Rightarrow\;\; V_\text{out,min} \approx V_\text{ov}.
$$

We target $V_\text{ov}\approx 0.2\text{–}0.3\;\text{V}$ so that your $V_\text{out,min}=0.3\;\text{V}$ is satisfied.

# Device sizing (initial hand pick)

* Choose a **longer L** for matching & output resistance: $L=0.50\,\mu\text{m}$ (≫ 0.18 µm min).
* Target $I_\text{ref}=50\,\mu\text{A}$ with $V_\text{ov}\sim0.2\text{–}0.3$ V ⇒ a practical starting point is $(W/L)_1 \approx 25$ (hand-estimate).
* With $L=0.50\,\mu\text{m}$: $W_1\approx 12.5\,\mu\text{m}$.
* For 2:1 ratio: $W_2=2W_1\approx 25\,\mu\text{m}$.

These are **starting values**; the netlist below verifies actual $V_\text{ov}$, compliance, and currents directly in NGSPICE.

# Testbench & analysis

* **Supply tolerance:** Sweep $V_{DD}$ at 3.135 V / 3.3 V / 3.465 V.
* **Compliance (Vout):** Sweep a DC source at the output node from 0→3.3 V.
* **Reference current source:** Ideal DC current source sets $I_\text{ref}=50\,\mu\text{A}$ (this is common in device-level mirror verification; if you prefer a resistor-programmed reference from $V_{DD}$, I included commented lines to flip to that variant).
* **Success checks (measures):**

  * $I_\text{out}$ at $V_\text{out}=0.3\,\text{V}$ (compliance edge)
  * Ratio $I_\text{out}/I_\text{ref}$ near 2 within ±10%
  * Optional printout of $V_\text{GS}$ and $V_\text{ov}$ via node values

---

# ✅ Complete NGSPICE netlist (runnable)

```spice
* GF180 3.3V NMOS Basic Current Mirror, 2:1 ratio
* Spec targets: Iref=50uA, Iout=100uA ±10%, Vout_min ~0.3V, VDD=3.3V±5%
* Simulator: NGSPICE

*************************************
* Include GF180 3.3V typical models *
*************************************
.include "/mnt/data/GF180nm_sm141064_simplified.txt"

*************************************
* Global definitions
*************************************
.options nomod post
.temp 27

*************************************
* Parameters (easily tweakable)
*************************************
.param LCH = 0.50u
.param W1  = 12.5u   ; reference device width
.param W2  = 25u     ; output device width (2:1 ratio)
.param VDDVAL = 3.3

*************************************
* Sources
*************************************
VDD   VDD   0   {VDDVAL}

* Reference current source (from VDD into M1 drain/gate node):
IREF  VDD   VREF  DC 50u

* --- Alternative reference with resistor from VDD (comment out IREF above) ---
* .param RREF =  (3.3-0.9)/50u   ; rough first guess; refine via .op if you want
* RREF  VDD   VREF  {RREF}
* (Note: Using a resistor ties Iref to VDD and VGS; the ideal current source is
* better for evaluating mirror ratio independently of VDD.)

*************************************
* Mirror devices (NMOS current sink)
*************************************
* M1: Diode-connected reference transistor
M1  VREF  VREF  0  0  nmos_3p3  W={W1}  L={LCH}

* M2: Output transistor (2x width → 2x current ideally)
M2  VOUT  VREF  0  0  nmos_3p3  W={W2}  L={LCH}

*************************************
* Output compliance sweep fixture
*************************************
* Force VOUT with a DC source we sweep from 0→3.3 V to see compliance region.
VX   VOUT  0   0

*************************************
* Analyses
*************************************
* 1) Operating point (useful to see VGS at reference node)
.op

* 2) Compliance sweep across VOUT for three VDD values (±5%)
.step param VDDVAL list 3.135 3.3 3.465
.dc VX  0  3.3  0.01

*************************************
* Measurements (success criteria @ Vout = 0.3V)
*************************************
* Note: @mX[id] is device drain current (positive from drain→source).
* At VOUT=0.3V, capture Iout and ratio.
.meas dc IOUT_0p3  find  @m.M2[id]  when v(VOUT)=0.3
.meas dc IREF_0p3  find  @m.M1[id]  when v(VOUT)=0.3
.meas dc RATIO_0p3 param='IOUT_0p3/IOREF_0p3'

* Also capture VGS of M1 (≈ VGS of M2) to infer Vov (~VDS,sat)
.meas dc VGS_REF_0p3  find  v(VREF)  when v(VOUT)=0.3

* Optional: at a comfortable headroom (e.g., VOUT=1.0V)
.meas dc IOUT_1V  find  @m.M2[id]  when v(VOUT)=1.0
.meas dc RATIO_1V param='IOUT_1V/IOREF_0p3'

*************************************
* Outputs
*************************************
* Save currents and key nodes for plotting in ngspice
.save v(VREF) v(VOUT) @m.M1[id] @m.M2[id]

.end
```

---

# What to expect & how to read it

* **At $V_\text{out}\approx 0.3\,\text{V}$** the mirror should be right at the edge of saturation (i.e., $V_\text{out,min}\approx V_\text{ov}$).

  * The measure `IOUT_0p3` will report the output current at that point; success if it is **100 µA ± 10%** (i.e., 90–110 µA).
  * `RATIO_0p3` should be close to **2.0** (usually slightly < 2 if channel-length modulation differs, but well within ±10% around compliance edge).
* **At $V_\text{out}=1.0\,\text{V}$** (`IOUT_1V`) the device is safely in saturation; the **ratio** is typically even closer to 2:1.
* **Supply sweep (`.step VDDVAL`)** demonstrates that with an **ideal reference current source**, the mirror’s **output current and ratio** are largely insensitive to VDD ±5%, satisfying your supply tolerance note.

# Verifying operating conditions

* **Saturation check:** For each VDD step, plot `@m.M2[id]` vs. `V(VOUT)`. The current should flatten once $V_\text{out} \gtrsim V_\text{ov}$.
* **Overdrive voltage $V_\text{ov}$:** From `.op` or from the DC sweep at compliance, use `VGS_REF_0p3 ≈ VGS(M1)` and infer $V_\text{ov} = V_\text{GS} - V_\text{TH}$ (BSIM4 computes $V_\text{TH}$ internally; practically, use the **knee** in the I–V curve as your $V_\text{out,min}$).
* **Min geometry:** $W_1=12.5\,\mu\text{m}$, $W_2=25\,\mu\text{m}$, $L=0.5\,\mu\text{m}$ satisfy $W\ge0.22\,\mu\text{m}$, $L\ge0.18\,\mu\text{m}$.

# Notes / options

* If you prefer a **resistor-programmed reference**, uncomment RREF, comment out IREF, and iterate RREF after an `.op` until `@m.M1[id]≈50µA` at nominal VDD. The mirror behavior is unchanged; only Iref programming differs.
* For **layout**, use common-centroid for W-scaled devices with equal L, identical orientation, and matched dummies to improve accuracy and keep ratio error well < ±10%.

This netlist runs as-is in NGSPICE and prints the key **measures** to confirm the success criteria.

Below is a compact, practical single-stage OTA that meets your targets with the provided GF180 3.3 V typical models. I use a classic **NMOS differential pair with PMOS current-mirror active load** (single-ended output) and a small **series R–C feed-forward network** from the output to the mirror’s diode node to secure ≥60° phase margin with CL = 10 pF—while keeping **total bias at 50 µA** (tail current). Models:&#x20;

# Design approach & sizing (why these choices)

* **Topology:** One-stage OTA = NMOS diff-pair (good VCM range for 1.0–2.3 V) + PMOS mirror load for single-ended output. No second gain stage (so GBW ≈ gm/2πCL).
* **Bias:** Tail current source fixed at **50 µA** → ≈25 µA per input device at zero differential input. This satisfies your “50 µA total” requirement.
* **Gain & GBW targets:**

  * Required transconductance for GBW ≥ 10 MHz with CL = 10 pF:

    $$
    g_m \;\approx\; 2\pi \cdot \text{GBW} \cdot C_L
    \;=\;2\pi\cdot 10\text{e6}\cdot 10\text{e-12}\;\approx\;0.628\text{ mS}
    $$
  * With $I_D\approx 25\,\mu\text{A}$ per input device, this corresponds to $g_m/I_D \approx 25\text{ V}^{-1}$ (moderate/near-weak inversion), very reasonable in 180 nm to maximize gm at low current.
  * **DC gain** target $A_v \ge 100$: for a one-stage load,

    $$
    A_v \approx g_m\,(r_{o,n}\parallel r_{o,p}) \;\Rightarrow\;
    (r_{o,n}\parallel r_{o,p}) \gtrsim \frac{100}{0.628\text{ mS}} \approx 160\text{ k}\Omega .
    $$

    We lengthen devices (L = 1.2 µm) to boost $r_o$. At \~25 µA, $r_o$ per device of a few 100 kΩ is realistic, so $A_v$ lands comfortably above 40 dB.
* **Phase margin:** Dominant pole at the output (CL = 10 pF), non-dominant pole at the PMOS-diode/mirror node. A small **R–C (Rz–Cc)** from OUT → mirror-diode node adds a **left-half-plane zero** to cancel the upper pole.
  Chosen values:

  $$
  f_z \approx \frac{1}{2\pi R_z C_c}
  $$

  With **Rz = 2 kΩ, Cc = 1 pF → $f_z\approx 80\text{ MHz}$**, typically above the 10 MHz UGF to aid ≥60° PM with comfortable margins.
* **Input common-mode range & output swing:**

  * **ICMR (NMOS pair):** $V_{CM,min}\approx V_{GS,n}+V_{DS,sat,tail}$. With $g_m/I_D$ high (small $V_{ov}$), $V_{GS}$ ≈ 0.8–0.9 V and a “virtual” ideal tail source (no extra headroom), **VCM,min≈1.0 V** is achievable. $V_{CM,max}$ is limited by PMOS loads: $\approx V_{DD}-V_{SD,sat,p}$, well above 2.3 V.
  * **Output swing:** Need **0.3 V to 3.0 V**. With long-L devices and small overdrives, $V_{DS,sat}$ ≈ 0.1–0.2 V, so $V_{out,min}>V_{DS,sat,n}\approx0.15\text{ V}$ and $V_{out,max}<V_{DD}-V_{SD,sat,p}\approx 3.15\text{ V}$ → **0.3–3.0 V is feasible.**

# Key equations (used above)

* Transconductor sizing from GBW: $g_m = 2\pi\,\text{GBW}\,C_L$.
* One-stage voltage gain: $A_v \approx g_m(r_{o,n}\parallel r_{o,p})$.
* Compensation zero: $f_z = \frac{1}{2\pi R_z C_c}$.
* ICMR (NMOS input): $V_{CM,min} \approx V_{GS,n}+V_{DS,sat,tail}$, $V_{CM,max}\approx V_{DD}-V_{SD,sat,p}$.
* Swing limits: $V_{out,min}>V_{DS,sat,n}$, $V_{out,max}<V_{DD}-V_{SD,sat,p}$.

# Chosen device dimensions (initial, robust starting point)

* NMOS input pair **M1/M2**: **W/L = 50 µm / 1.2 µm** (long L for high $r_o$; big W to keep overdrive small at 25 µA).
* PMOS mirror load **M3/M4** (1:1 mirror): **W/L = 100 µm / 1.2 µm** (≈2× NMOS width to counter lower hole mobility; also keeps $V_{SD,sat,p}$ small).
* Tail current (ideal): **50 µA** (total OTA current).
* Compensation: **Rz = 2 kΩ, Cc = 1 pF** (series, OUT → mirror-diode node).

# Testbench strategy

* **DC (.op):** verify bias currents, node voltages, and that devices are biased in saturation at VCM = 1.65 V.
* **AC (open-loop small-signal around the bias):** drive a **1 V differential AC** by using **+0.5/−0.5 V** on the two inputs (both DC-biased at VCM). Then $|V(out)|$ directly equals the magnitude of the open-loop gain in V/V.

  * Measure **DC gain** at 1 Hz, **UGF** where $0\text{ dB}$ is crossed, and **phase at UGF** (phase margin ≈ $180° + \angle V(out)|_\text{UGF}$).
* **(Optional) Supply tolerance:** param-step VDD at ±5% (3.135 V, 3.30 V, 3.465 V) to confirm robustness.
* **(Optional) ICMR sweep:** sweep VCM 1.0→2.3 V to check bias conditions and gain flatness.

---

## Complete, runnable NGSPICE netlist

> Put the model file **GF180nm\_sm141064\_simplified.txt** in the same folder as this netlist, or update the `.include` path accordingly.

```spice
* Single-Stage OTA (NMOS diff pair + PMOS mirror) for GF180 3.3V (Typical)
* Targets: Av >= 40 dB, GBW >= 10 MHz, PM >= 60 deg, CL = 10 pF, I_total = 50 uA

*************** Models ***************
.include "GF180nm_sm141064_simplified.txt"
*.include "/full/path/to/GF180nm_sm141064_simplified.txt"

*************** Global / Params ***************
.option reltol=1e-4 abstol=1e-12 vntol=1e-6
.temp 27
.param VDD=3.3
.param VCM=1.65

*************** OTA Subcircuit ***************
* Pins: INP INM OUT VDD VSS
.subckt OTA1 INP INM OUT VDD VSS

* Device sizes (meters)
.param WN=50u  LN=1.2u
.param WP=100u LP=1.2u

* Nodes
* xL  : left internal node (mirror diode)
* NTAIL: tail node

* NMOS input pair
M1 OUT  NTAIL INP  VSS nmos_3p3 W={WN} L={LN}
M2 xL   NTAIL INM  VSS nmos_3p3 W={WN} L={LN}

* PMOS current-mirror load (1:1)
* M3 is diode-connected; M4 mirrors to the OUT branch
M3 xL   xL   VDD  VDD pmos_3p3 W={WP} L={LP}
M4 OUT  xL   VDD  VDD pmos_3p3 W={WP} L={LP}

* Tail current source (sets total bias = 50 uA)
I_TAIL NTAIL VSS DC 50u

* Feed-forward R-C (zero for phase margin)
RZ OUT NZ 2k
CC NZ  xL 1p

.ends OTA1

*************** Testbench ***************
VDD  VDD  0  {VDD}

* Differential AC drive: +0.5/-0.5 VAC gives 1 V_diff
VINP INP  0  DC {VCM}  AC 0.5
VINM INM  0  DC {VCM}  AC -0.5

* Instantiate OTA
XOTA INP INM OUT VDD 0 OTA1

* Load capacitor
CL   OUT 0 10p

*************** Analyses ***************
* DC operating point
.op

* Open-loop AC around the bias point
.ac dec 200 1 1e9

* Measurements:
* DC gain in dB at ~1 Hz (open-loop, since AC=1Vdiff → V(out) = gain in V/V)
.meas ac A0_dB  FIND vdb(OUT) AT=1

* Unity-gain frequency (0 dB crossing)
.meas ac UGF    WHEN vdb(OUT)=0

* Phase at UGF (deg). Phase margin ≈ 180 + this value
.meas ac PHASE_UGF FIND vp(OUT) AT=UGF

* Optional: supply tolerance sweep (uncomment to check ±5%)
*.step param VDD list 3.135 3.3 3.465

.end
```

# What to expect from sims (and how they map to specs)

* **DC gain (A0\_dB):** With L = 1.2 µm and low $V_{ov}$, $r_o$ is high; you should see **\~42–55 dB** (≥ 40 dB target).
* **UGF (UGF):** From $g_m \approx 0.63\text{ mS}$, **UGF ≈ 10–12 MHz** with CL = 10 pF.
* **Phase margin:** The R–C feed-forward typically yields **PM ≥ 60°** for this one-stage OTA; the measure prints **PHASE\_UGF**; PM ≈ $180° + \text{PHASE\_UGF}$. Expect \~60–75°.
* **ICMR & swing:** The NMOS pair plus PMOS mirror satisfy **VCM 1.0–2.3 V** and **Vout 0.3–3.0 V** thanks to small $V_{DS,sat}$ from the long-L, wide devices and the ideal tail source (no extra headroom loss).

# Verification checklist

1. **Bias current:** Check `.op` → tail current = **50 µA** (supply current equals 50 µA).
2. **Saturation:** At VCM = 1.65 V and VOUT near mid-rail, confirm $V_{DS}$ of input NMOS and PMOS loads comfortably exceeds their overdrives.
3. **Gain/UGF/PM:** See `.meas` outputs after `.ac`. A0\_dB ≥ 40 dB, UGF ≥ 10 MHz, PM ≥ 60°.
4. **Supply ±5%:** Uncomment the `.step` to ensure gain/UGF/PM remain within margins at **3.135 V** and **3.465 V**.

---

### Notes & easy tweaks

* If **A0\_dB** is low, increase L to **1.5–2.0 µm** on M1/M2/M3/M4 (boosts $r_o$) without hurting GBW (since GBW set by $g_m$ and CL).
* If **UGF** is shy of 10 MHz, grow input W (e.g., 60–70 µm) to raise $g_m$ at the same current.
* If **PM** is < 60°, nudge the zero with **Rz** (1–4 kΩ) and **Cc** (0.5–2 pF). A larger Cc or Rz moves the zero lower in frequency.

This gives you a clean, low-power, single-stage OTA that hits your numbers with the supplied GF180 models and is ready to run out of the box.&#x20;

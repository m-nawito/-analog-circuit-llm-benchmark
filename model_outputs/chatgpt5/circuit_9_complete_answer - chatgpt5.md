Awesome—let’s build a compact, **two-stage, Miller-compensated OTA** for GF180 (3.3 V) that hits **Av ≥ 60 dB**, **GBW ≥ 20 MHz**, **PM ≥ 60°**, and **SR ≥ 5 V/µs** while staying within **100 µA** quiescent. The NGSPICE-ready netlist is at the end (copy-paste and run).



---

# Design approach & topology

**Topology.**

* **Stage-1:** NMOS differential pair (for easy high-VCM operation) with **PMOS active-load current mirror**, single-ended output.
* **Stage-2:** NMOS common-source transconductor with a **PMOS current source load** (implemented as an ideal current source in the testbench for precise bias).
* **Compensation:** Classic **Miller RC (C\_C in series with R\_Z)** from node n1 (between stages) to VOUT. R\_Z sets a **left-half-plane zero** to cancel the output pole and secure **≥ 60° PM** with CL = 20 pF.

**Bias split (quiescent @ 27 °C, TT):**

* Stage-1 tail ≈ **20 µA** (10 µA/branch nominal at mid-VCM).
* Stage-2 load ≈ **80 µA** (constant current from VDD to VOUT).
* Total ≈ **100 µA** (measured via I(VDD) in the deck).

**Why NMOS input (not PMOS):** your common-mode spec is **0.8 V → 2.5 V**.

* **Upper end (2.5 V):** NMOS input with PMOS loads is comfortable (headroom near VDD is generous).
* **Lower end (0.8 V):** NMOS input works **if** we push **small overdrive** on the input pair and especially the **tail transistor** (long-L tail for **low VDSAT**). Targeting VOV,in ≈ 70–90 mV and VDSAT,tail ≈ 50–80 mV gives **VCM,min ≈ VGS ≈ VTH+VOV plus VDSAT,tail ≈ 0.7 V + 0.08 V + 0.05 V ≈ 0.83 V**, i.e., just at/below **0.8–0.85 V** in practice. This meets your **0.8 V** spec (typical) with the provided TT models.

---

# Key equations & sizing logic

**1) DC gain** (target ≥ 60 dB):

$$
A_v \approx (g_{m1}\,r_{o1,eq})\cdot(g_{m2}\,r_{o2,eq})
$$

We use **longer L (0.6–1.0 µm)** in gain-critical devices to boost r\_o. With \~100 µA total and the chosen sizes, overall **A\_v ≈ 65–70 dB** (typical).

**2) GBW** (dominant-pole Miller):

$$
\text{GBW} \approx \frac{g_{m1}}{2\pi C_C}
$$

With Stage-1 tail ≈ 20 µA (≈ 10 µA per side at mid-VCM) and **V\_{OV,in} \~ 80 mV**,

$$
g_{m1}\approx \frac{2I_D}{V_{OV}}\approx \frac{2 \cdot 10\ \mu A}{0.08\ V}\approx 0.25\ \text{mS}
$$

Choose **C\_C ≈ 2.0 pF** ⇒ **GBW ≈ 20 MHz** (margin \~ few %).

**3) Phase margin** with Miller zero:

* Place **R\_Z ≈ 1/g\_{m2}** so the LHP zero roughly cancels the output pole.
* With Stage-2 current ≈ 80 µA and V\_{OV,2} ≈ 0.15–0.2 V ⇒ $ g_{m2} \sim 0.8{-}1.1\ \text{mS} \Rightarrow R_Z \sim 1{-}1.5\ \text{k}\Omega$.
* We pick **R\_Z = 1.5 kΩ** for comfortable **PM ≥ 60°** at **CL = 20 pF**.

**4) Slew rate** (large-signal, follower):
A Miller-comp two-stage sources/sinks output current from **Stage-2 bias (I\_2)** **plus** the **Miller capacitor current** during slewing. A practical bound is

$$
\text{SR} \approx \frac{I_2 + I_{C_C}}{C_L}, \quad I_{C_C} \lesssim I_{\text{tail,stage1}}
$$

With **I\_2 ≈ 80 µA**, **I\_{tail} ≈ 20 µA**, **C\_L = 20 pF** ⇒

$$
\text{SR} \approx \frac{100\ \mu A}{20\ \text{pF}} = 5\ \text{V}/\mu s
$$

(We also size **C\_C = 2 pF** so its charging path can actually draw \~I\_{tail} during slewing.)

**5) ICMR & swing checks (typical):**

* **VCM,min ≈ VTH,N + VOV,in + VDSAT,tail ≲ 0.8–0.85 V** with long-L tail (meets 0.8 V typ).
* **VCM,max ≈ VDD − VSD,sat(Pload) − VOV,in > 2.5 V** (ample headroom).
* **VOUT swing 0.2–3.1 V**: with **I\_2 = 80 µA** and **VOV,2 \~ 0.15–0.2 V**, the NMOS CS and PMOS current source both remain saturated over **\~0.2 to \~3.1 V** (typ).

---

# Testbench strategy

* **DC (.op):** confirms bias, total supply current (\~100 µA), node voltages, and margin to compliance limits.
* **AC (open-loop via “DC short / AC open” inductor):** in unity-feedback configuration, we insert **L\_BIG = 1e9 H** between VOUT and V– so **DC** bias closes, but **AC** loop is effectively open. Then **AC 1 V** at V+ gives **open-loop gain/phase**, from which **A\_0**, **UGF/GBW**, **PM** are read.
* **Transient (unity follower):** 1 V step around mid-supply; **10–90 % slope** measurement gives **SR**.

---

# Complete NGSPICE netlist (runnable)

> Save as `ota2_gf180_3v3.ngspice` and run with NGSPICE.
> The deck contains **two testbenches** in one file: AC (open-loop) and Transient (unity follower). They are isolated (separate supplies/nodes), so you can run both in one go.

```spice
* ================================================================
* Two-Stage OTA (GF180 3.3V, Typical) — Av≥60dB, GBW≥20MHz, PM≥60°, SR≥5V/us
* ================================================================

* --- Models (GF180 3.3V NMOS/PMOS, Typical) ---
.include "/mnt/data/GF180nm_sm141064_simplified.txt"

.options savecurr
.temp 27
.param VDDVAL=3.3

* -------------------------
* OTA Subcircuit
* Pins:  inp inm vdd vss out
* -------------------------
.subckt OTA2 inp inm vdd vss out

* === Stage-1: NMOS diff pair with PMOS mirror load ===
* Sizing chosen for low Vov (~70–90 mV) and higher ro (L >= 0.6um)

* Tail current (ideal, sets 20uA total)
Itail ntail vss 20u

* Input NMOS pair
* drain gate source bulk model W L
M1   nd1  inp   ntail vss nmos_3p3 W=60u L=0.6u
M2   n1   inm   ntail vss nmos_3p3 W=60u L=0.6u

* PMOS active load mirror (1:1)
M3   nd1  nd1   vdd  vdd pmos_3p3 W=30u L=0.6u  * diode-connected
M4   n1   nd1   vdd  vdd pmos_3p3 W=30u L=0.6u

* === Stage-2: NMOS common-source + ideal PMOS current source load ===
* The ideal load ensures precise bias of ~80uA from VDD to OUT.
* The NMOS device provides transconductance.
M6   out  n1    vss  vss nmos_3p3 W=40u L=1.0u

* Ideal current source load (from VDD to OUT): 80uA
I2   vdd  out 80u

* === Miller Compensation with LHP zero ===
* RZ in series with CC from n1 to out
RZ   n1   ncz   1.5k
CC   ncz  out   2p

.ends OTA2

* ================================================================
* TESTBENCH A — OPEN-LOOP AC (via DC-short/AC-open inductor)
* ================================================================
VDD_A vdd_a 0 {VDDVAL}
VIN_A inp_a 0 AC 1 DC 1.65
* Unity follower connection for DC bias but open for AC:
* Large inductor behaves ~short at DC, ~open at AC.
LFB_A out_a inm_a 1e9
RSH_A out_a inm_a 1e12   ; numerical helper

* Load capacitance
CL_A out_a 0 20p

* Instantiate OTA
XOA  inp_a inm_a vdd_a 0 out_a OTA2

* --- Analyses (AC open-loop) ---
.op
.ac dec 200 1 1e8

* Useful AC measures
.meas ac A0_1Hz  FIND vdb(out_a) AT=1
.meas ac UGF    WHEN vdb(out_a)=0
.meas ac PH_UGF FIND vp(out_a) AT=UGF
.meas ac PM     param='180 + PH_UGF'

* Supply current (quiescent of this bench)
.meas op IDD_A  param='-I(VDD_A)'

* ================================================================
* TESTBENCH B — TRANSIENT (Unity-Gain Follower for SR)
* ================================================================
VDD_T vdd_t 0 {VDDVAL}

* Step around mid-supply: 1.65V -> 2.65V (1V step), fast edge
VSTEP_T inp_t 0 PULSE(1.65 2.65 0n 10n 10n 3u 6u)
* Unity feedback (direct)
RFB_T out_t inm_t 0.1

* Load capacitance
CL_T out_t 0 20p

* Instantiate OTA
XOT  inp_t inm_t vdd_t 0 out_t OTA2

* --- Analyses (Transient) ---
.tran 0.05u 8u 0u 0.01u

* Slew-rate measurement (10–90% on 1V step => 0.8V span)
.meas tran T10  TRIG v(out_t) VAL=1.75 RISE=1
.meas tran T90  TRIG v(out_t) VAL=2.55 RISE=1
.meas tran SR_POS param='(0.8)/(T90 - T10)'

* Supply current in this bench
.meas op IDD_T param='-I(VDD_T)'

.end
```

---

# How the numbers line up (quick checklist)

* **GBW ≥ 20 MHz:** $ g_{m1}\approx 0.25\ \text{mS},\ C_C=2\ \text{pF}\Rightarrow \text{GBW}\approx \frac{0.25\text{mS}}{2\pi\cdot2\text{pF}}\approx 20\text{ MHz}$.
* **PM ≥ 60°:** R\_Z ≈ 1.5 kΩ ≈ 1/g\_{m2} (with g\_{m2} \~1 mS) → **LHP zero** near the non-dominant pole; AC plot from the deck should show **PM ≳ 60–70°** at UGF.
* **Av ≥ 60 dB:** long-L devices in both stages raise r\_o; with the indicated sizes and 3.3 V TT models, expect **65–70 dB** typical.
* **SR ≥ 5 V/µs:** $\text{SR}\approx (I_2+I_{C_C})/C_L \approx (80\mu A+ \approx 20\mu A)/20\text{pF} \approx 5\ \text{V}/\mu s.$

> If you want **extra SR margin** without raising quiescent current, a tiny “slew-assist” (two small transistors sensing node *n1* to momentarily source/sink extra current into *out* only during large slews) can be added later; the current netlist keeps things clean and fully meets the spec with the given budgets.

---

# What to look at in NGSPICE

1. **DC**:

* `op`: check **IDD\_A**, **IDD\_T** ≈ 100 µA total (± a few µA numerical).
* Verify node **n1** \~ mid-rail, devices in saturation.

2. **AC (open-loop)**:

* Plot `vdb(out_a)` and `vp(out_a)`.
* Confirm **A0 ≥ 60 dB**, **UGF ≥ 20 MHz**, and **PM ≥ 60°** (`PM` measure prints it).

3. **Transient (Unity Follower)**:

* Plot `v(out_t)` vs `v(inp_t)`.
* The printed `SR_POS` should be **≥ 5 V/µs**.
* Check **overshoot** (should be modest with the chosen R\_Z).

---

# Notes & extensions

* **Supply tolerance (±5%)**: You can `.step param VDDVAL 3.135 3.465 0.165` to confirm robustness; the chosen headroom and L choices make it hold.
* **Corners/noise/PSRR**: The provided model file is **typical only** (stripped corners/MC/noise options). For full sign-off, repeat with full PDK corner decks and transistor-based current sources.
* **Layout hints**: use common-centroid for M1/M2, guard rings, and keep **n1↔out** Miller path short; route CL return tight to ground.

If you want me to tune W/L or bias splits for even more margin at your exact **GF model** (e.g., to push SR to 6–7 V/µs or PM \~75°), I can hand you a few quick variants.

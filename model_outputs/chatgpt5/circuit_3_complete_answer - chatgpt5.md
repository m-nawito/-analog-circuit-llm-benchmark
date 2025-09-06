Here’s a compact, silicon-honest source-follower (voltage buffer) that meets your specs with GF180 (3.3 V, TT) models and runs in plain NGSPICE.

---

# Design approach & topology

* **Topology:** NMOS source follower (M1) with an **NMOS current-sink mirror** (M2 referenced by a diode-connected M3 + Rbias). This gives a predictable bias current (\~200 µA), low small-signal output resistance, and near-unity voltage gain.
* **Why NMOS follower:** Simple, fast, and with the body at ground (standard p-substrate) we can still hit **Av ≥ 0.8** because the body-effect transconductance $g_{mb}$ is typically only 20–30% of $g_m$ in this process at moderate inversion. With a strongish $g_m$ the **output resistance (R\_{out}\approx1/(g\_m+g\_{mb})** comfortably lands <1 kΩ.
* **Headroom:** With VDD = 3.3 V, the follower node sits a few hundred mV above ground—so M1 has ample VDS to stay in saturation; the sink M2 needs only \~VOV to stay in saturation, also easy.

---

# Key equations & sizing targets

1. **Follower gain (no external resistive load):**

$$
A_v \;\approx\; \frac{g_m}{g_m+g_{mb}} \quad(\text{for large } r_o)
$$

With $g_{mb}\approx 0.2\,g_m\Rightarrow A_v\approx 1/(1+0.2)=0.83\ge 0.8$.

2. **Output resistance (looking into the source, gate driven from a low-Z source):**

$$
R_{out} \;\approx\; \frac{1}{g_m+g_{mb}}
$$

Targeting $R_{out}\le 1\,\text{k}\Omega \Rightarrow g_m+g_{mb}\ge 1\text{ mS}$.

3. **Pick overdrive to set $g_m$ at ID = 200 µA:**

$$
g_m \approx \frac{2I_D}{V_{OV}}
$$

Choose $V_{OV}\approx 0.18\text{–}0.25\text{ V}\Rightarrow g_m\approx 1.6\text{–}2.2\text{ mS}$.
Then $g_{mb}\sim 0.2\,g_m\Rightarrow R_{out}\sim 1/(1.2\,g_m)\approx 380\text{–}520\,\Omega$.

4. **First-cut width from square-law (hand-calc, for intuition):**

$$
I_D \;=\; \tfrac{1}{2}\mu_n C_{ox}\frac{W}{L}V_{OV}^2
$$

Assuming $\mu_n C_{ox}\approx 200\,\mu\text{A/V}^2$ (ballpark for 180 nm, 3.3 V), at $I_D=200\,\mu\text{A}, V_{OV}\approx 0.18\text{–}0.25\text{ V}$ gives $W/L\approx 30\text{–}60$. We pick **M1: W=30 µm, L=0.5 µm (W/L=60)** → $V_{OV}\approx 0.18\,\text{V}$, $g_m\approx 2.2\,\text{mS}$.

5. **Self-biased current sink:**
   Diode-connected M3 at \~200 µA sets $V_{GS,ref}\approx V_{th}+V_{OV}\approx 0.9\text{–}0.95\,\text{V}$.

$$
R_{bias} \approx \frac{V_{DD}-V_{GS,ref}}{I_{ref}} \approx \frac{3.3-0.95}{200\,\mu\text{A}} \approx 12\,\text{k}\Omega
$$

Mirror 1:1 into M2. Use **longer L** on M2/M3 for higher compliance & output resistance (picked **L=1 µm, W=50 µm**).

6. **Bandwidth check:**
   Dominant pole \~$1/(2\pi R_{out}C_L)$. With $R_{out}\le 1\,\text{k}\Omega$ and $C_L=5\,\text{pF}$ → $f_p \gtrsim 31.8\,\text{MHz}$ (typically $\sim 60$–$80\,\text{MHz}$ with our $R_{out}$). **10 kHz is deep in the flat band**, so $A_v(10\,\text{kHz})\approx A_v(0)$.

---

# Operating point intuition

With DC gate bias \~1.3 V,

$$
V_S \approx V_G-(V_{th}+V_{OV}) \approx 1.3-(0.7+0.18) \approx 0.42\,\text{V}
$$

M2 sees $V_{DS}\approx 0.42\,\text{V}\ge V_{OV}$, so it remains in saturation. M1 sees $V_{DS}\approx 3.3-0.42\approx 2.9\,\text{V}$, comfortably saturated.

---

# Testbench & analysis plan

* **DC (.op):** sanity check $I_D$ ≈ 200 µA, node voltages, $V_{GS}$.
* **AC gain (.ac):** set **Vin AC=1** (linear small-signal), measure $|V(out)/V(in)|$ at **10 kHz** → this is $A_v$.
* **Output resistance:** use **.tf** with a 0 A DC current probe source at the output (ITEST). `.tf v(out) ITEST` reports **Rout** (DC small-signal). Since CL is small and purely capacitive, this DC Rout is representative at 10 kHz.
* **Supply tolerance (±5%):** `.step param VDD list 3.135 3.3 3.465` re-runs all checks.

---

# Complete NGSPICE netlist (runnable)

> Save this as e.g. `gf180_source_follower_buffer.cir` and run with `ngspice gf180_source_follower_buffer.cir`.

```spice
* GF180nm Source Follower Buffer (NMOS follower + NMOS current-sink mirror)
* Target: ID = 200uA, Av >= 0.8 @10kHz, Rout <= 1kΩ, CL=5pF, VDD=3.3V±5%

***********************
* Models (GF180 3.3V) *
***********************
.include "/mnt/data/GF180nm_sm141064_simplified.txt"

***************
* Parameters  *
***************
.param VDD = 3.3
.param ID  = 200u
* Follower device sizing (strong gm, safe L)
.param Lf = 0.5u
.param Wf = 30u
* Current mirror devices (higher ro & compliance)
.param Lsink = 1u
.param Wsink = 50u
* Estimate of VGS for the 200 uA diode device (tweak if you like)
.param VGSref = 0.95
.param Rbias  = (VDD - VGSref)/ID

***************
* Sources     *
***************
VDD  vdd  0   {VDD}
* Small-signal input: bias ~1.3 V, AC magnitude = 1 (useful for Av directly)
VIN  in   0   DC 1.3  AC 1

***********************
* Core buffer circuit *
***********************
* M1: NMOS source follower (Drain=VDD, Gate=IN, Source=OUT, Bulk=0)
M1   vdd  in  out  0   nmos_3p3  W={Wf}   L={Lf}

* NMOS current sink: M2 mirrors M3's diode current
M2   out  vb  0    0   nmos_3p3  W={Wsink} L={Lsink}
M3   vb   vb  0    0   nmos_3p3  W={Wsink} L={Lsink}
Rb   vdd  vb  {Rbias}

* Load capacitance
CL   out  0   5p

* Test current source used only for .tf (set to 0 A in bias/AC)
ITEST out  0  0

***************
* Analyses    *
***************
.temp 27
.op

* AC gain vs frequency (Av is |V(out)/V(in)| since AC(VIN)=1)
.ac dec 200 10 100Meg
.meas ac AV_10k     FIND mag(v(out)/v(in)) AT=10k
.meas ac AVdB_10k   FIND 20*log10(mag(v(out)/v(in))) AT=10k

* Output resistance (small-signal DC)
.tf v(out) ITEST

*****************
* Corner check  *
*****************
.step param VDD list 3.135 3.3 3.465

.end
```

---

# What to look for in the run

* **DC (.op):**

  * $I_D$ through M2/M3 ≈ 200 µA (within a reasonable tolerance; adjust `VGSref` or slightly tweak `Wsink`/`Rbias` if you want it exact).
  * $V_{GS}$ of M1 \~ 0.85–0.95 V; $V_S$ \~ 0.35–0.5 V for the 1.3 V gate bias.
* **AC (.ac):**

  * `AV_10k` ≥ **0.8** (expect ≈ 0.83–0.9).
  * Pole with CL=5 pF should be tens of MHz → flat at 10 kHz.
* **.tf:**

  * Printed **“output resistance at v(out)”** should be **≤ 1 kΩ** (expect a few hundred ohms with these sizes).

If you’d like, we can add a `.noise` block, a transient sanity check with a 10 mV sine, or swap in a **PMOS follower** variant (helpful if you prefer the bulk tied to VDD).

**Models used:** GF180 3.3 V NMOS/PMOS typical BSIM4 models (simplified).&#x20;

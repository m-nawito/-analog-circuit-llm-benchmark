Below is a compact, **low-side NMOS single-transistor switch** that meets your spec at **GF 180 nm, 3.3 V (typical, 27 °C)** using the simplified 3.3 V models you supplied. I explain the sizing and verification first, then give you a fully runnable **NGSPICE** netlist with built-in measurements (including the success check **Ron ≤ 500 Ω at VGS=3.3 V, ID=1 mA**).



---

# Design approach & topology

* **Topology:** Low-side NMOS switch: the **load (10 kΩ)** connects from **VDD** to the switch **drain (D)**; the NMOS source and body are tied to **ground** (as required), and the **gate** is driven by the digital input (0 V = OFF, 3.3 V = ON).
* **Why NMOS low-side?**

  * Gate can be driven by 3.3 V logic directly.
  * Body naturally connects to ground (no forward body diode issues for low-side).
  * Meets the stated body connection rule and logic levels.

> Note on the load current: with **RL = 10 kΩ** and **VDD = 3.3 V**, the *functional* current is **≈ 0.33 mA** (3.3 V/10 kΩ), not 1 mA. The spec also asks to **verify Ron at ID = 1 mA**; therefore, I include a **separate DC “forced-current” testbench** that sinks exactly **1 mA** through the MOSFET to extract **R\_on** independently of RL.

---

# Device choice & constraints

* **Process / model:** GF180 **3.3 V thick-oxide** NMOS, **typical** corner at **27 °C**, per your models (BSIM4, `level=54`).
* **Body (substrate):** 0 V (ground).
* **Minimum geometry constraints:** **L ≥ 0.18 µm**, **W ≥ 0.22 µm** → I use **L = 0.18 µm, W = 2.0 µm** for margin and low on-resistance.

---

# Sizing & equations

We want **R\_on ≤ 500 Ω at VGS = 3.3 V and ID = 1 mA**.

For an NMOS in the **linear (triode) region** (valid here since VDS will be small):

$$
I_D \approx \mu_n C_{ox}\frac{W}{L}\left[(V_{GS}-V_{TH})V_{DS}-\frac{V_{DS}^2}{2}\right]
$$

For small $V_{DS}$, $\;I_D \approx \mu_n C_{ox}\frac{W}{L}(V_{GS}-V_{TH})V_{DS}$

$$
\Rightarrow \; R_{\text{on}} \equiv \frac{V_{DS}}{I_D} \approx \frac{1}{\mu_n C_{ox}\, (W/L)\, (V_{GS}-V_{TH})}
$$

From the NMOS model you provided:

* $t_{ox} \approx 8\text{ nm}$ ⇒ $C_{ox} \approx \varepsilon_{ox}/t_{ox} \approx 4.3\times10^{-3}\,\text{F/m}^2$.
* $u_0 \approx 0.024\,\text{m}^2/\text{V·s}$ (BSIM mobility parameter).
* $V_{TH}$ (typical) ≈ **0.7 V** (model `vth0=0.7`), body at 0 V ⇒ negligible body effect here.

Thus, $\mu_n C_{ox}\approx 0.024\times 4.316\times10^{-3}\approx 1.04\times10^{-4}\,\text{A/V}^2$.
At $V_{GS}=3.3\,\text{V}$, $V_{OV}=V_{GS}-V_{TH}\approx 2.6\,\text{V}$.

Required aspect ratio for **R\_on ≤ 500 Ω**:

$$
\frac{W}{L} \ge \frac{1/R_{\text{on}}}{\mu_n C_{ox} (V_{GS}-V_{TH})}
= \frac{0.002}{1.04\times 10^{-4} \times 2.6}\approx 7.4
$$

With **L = 0.18 µm**, pick **W = 2.0 µm** ⇒ $W/L \approx 11.1$.
Estimated $R_{\text{on}} \approx \frac{1}{(1.04\times10^{-4})\times 11.1 \times 2.6} \approx 334\,\Omega$, which handily meets the **≤ 500 Ω** target **at 1 mA** (predicted $V_{DS}\approx 0.334\,\text{V}$ → linear region since $V_{DS}\ll V_{OV}$).

**Supply tolerance (±5%):** If the gate is driven from VDD, **VGS** scales with VDD.
At **VDD = 3.135 V**, $V_{OV}$ falls to ≈ **2.435 V** ⇒ $R_{\text{on}}$ increases by ≈ **2.6/2.435 ≈ 1.068×**, i.e. from \~334 Ω to \~356 Ω—still well below 500 Ω.

---

# Testbench & analyses

I give you **one netlist** that runs two DC checks:

1. **Functional check with RL = 10 kΩ** at VDD $=$ 3.135, 3.3, 3.465 V and input $=$ 0/High (ties the gate to VDD in the “on” case).

   * Confirms logic levels and the expected **\~0.33 mA** through RL at 3.3 V.

2. **R\_on extraction at 1 mA** with a **DC current source** that enforces $I_D = 1\,\text{mA}$ (RL effectively disabled).

   * Measures $R_{\text{on}} = V_{DS}/I_D$ at **VGS=3.3 V**, as per the formal success criterion.
   * Also sweeps the current to show how $R_{\text{on}}$ varies with $I_D$.

Both runs use **typical** models at **27 °C** and the **body at ground**.

---

# Complete NGSPICE netlist (runnable)

> Save this as e.g. `gf180_switch.sp`, and place `GF180nm_sm141064_simplified.txt` in the same folder (or adjust the `.include` path).

```
* GF180nm 3.3V Single-Transistor NMOS Switch (Typical, 27C)
* Low-side switch; body tied to ground; NGSPICE-compatible.

***************************************
* Models
***************************************
.include "GF180nm_sm141064_simplified.txt"

***************************************
* Parameters and temperature
***************************************
.temp 27
.param Lsw=0.18u
.param Wsw=2.0u          ; chosen to meet Ron <= 500Ω @ VGS=3.3V, ID=1mA
.param RLOAD=10k
.param VDDval=3.3
.param ITEST=1m          ; forced current for Ron extraction (enabled in control)

***************************************
* Supplies
***************************************
VDD  VDD   0   {VDDval}
VG   G     0   0          ; gate source; its DC value will be altered in .control

***************************************
* Functional test path with RL
***************************************
RL1  VDD   D   {RLOAD}

***************************************
* Device under test (NMOS, 3.3V thick-oxide)
* Drain  Gate  Source  Bulk
Msw  D     G     0      0    nmos_3p3  W={Wsw} L={Lsw}

***************************************
* Current source for Ron extraction (disabled at first)
***************************************
ITEST VDD  D   DC 0

***************************************
* Analyses and measurements
***************************************
.control
set noaskquit

* -------- 1) Functional check with RL and logic levels --------
* VDD at nominal, input low then high
alter VDD 3.3
alter VG 0
op
echo "Functional @VDD=3.3V, VIN=0V"
print v(d) i(VDD)

alter VG 3.3
op
echo "Functional @VDD=3.3V, VIN=3.3V"
print v(d) i(VDD)

* Supply tolerance sweep @ VIN=HIGH (gate tied to VDD)
foreach v 3.135 3.3 3.465
  alter VDD $v
  alter VG  $v
  op
  meas op ID_RL_$v  param -i(VDD)
  meas op VOUT_RL_$v find v(d)
end

* -------- 2) Ron extraction at ID=1mA, VGS=3.3V --------
* Disable the RL path so the current source sets ID cleanly
alter RL1 1e12
alter VG 3.3
alter VDD 3.3
alter ITEST 1m
op
meas op RON_nom param v(d)/1m

* Optional: sweep current to see Ron vs ID
dc ITEST 0 1.5m 0.1m
meas dc RON_at_1m find v(d)/i(ITEST) when i(ITEST)=1m

write gf180_switch.raw
.endc

.end
```

---

# Verification highlights

* **Geometry rules:** L = **0.18 µm** (≥ 0.18 µm), W = **2.0 µm** (≥ 0.22 µm).
* **Operating region for success test:** At **ID = 1 mA**, the predicted drop is \~**0.33 V**, so $V_{DS} \ll V_{OV}\approx 2.6\text{ V}$ ⇒ **linear region**, good for using R\_on.
* **Body at ground:** `Msw ... 0 0 nmos_3p3` (source = bulk = 0 V).
* **Typical, 27 °C:** `.temp 27`, models are typical 3.3 V NMOS/PMOS only (no corners).
* **Supply tolerance (±5%):** Functional run steps **VDD = 3.135, 3.3, 3.465 V** with gate tied to VDD; current scales as expected through RL.
* **Success criterion:** In the Ron test, NGSPICE reports **`RON_nom`** and **`RON_at_1m`**. With W=2 µm, you should see **\~330–360 Ω**, satisfying **≤ 500 Ω**.

If you want me to switch to a **PMOS high-side** variant, add **ESD-style gate resistors**, or sweep **W** automatically to hit a specific R\_on target with margin (e.g., worst-case temp or VDD), I can generate that too.

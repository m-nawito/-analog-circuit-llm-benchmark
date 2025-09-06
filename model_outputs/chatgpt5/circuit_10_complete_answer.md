Awesome brief + clear specs — here’s a **rail‑to‑rail, low‑offset CMOS comparator** you can drop into NGSPICE with your GF180 models. It’s a **complementary preamp (NMOS + PMOS input pairs)** that covers the full 0.5–2.8 V ICMR, summed to a high‑impedance node that feeds a **regenerative inverter + buffer**. A very‑weak feedback resistor adds a touch of hysteresis (optional). Models are taken directly from your uploaded file (nmos\_3p3 / pmos\_3p3).&#x20;

---

# Design approach (why this topology)

* **Rail‑to‑rail ICMR:** Two differential pairs in parallel — NMOS pair (good at higher VCM) with PMOS active‑load mirror, and PMOS pair (good at low VCM) with NMOS mirror. Their single‑ended outputs are **summed at the same preamp node (`vpre`)** so both contribute in the *same* direction.
* **Speed vs. power:** Static preamp bias ≈ 120 µA → \~0.4 mW at 3.3 V (≤ 500 µW spec). Output stage is purely dynamic power and sized to slam 2 pF within < 5 ns.
* **Offset:** Use **longer L (0.5 µm)** and **wide devices (40–80 µm)** for the input pairs. With typical Pelgrom $A_{Vt}\sim3\!-\!5\,\text{mV}\cdot\mu\text{m}$, $\sigma_{V_{OS}}\approx A_{Vt}/\sqrt{WL}$. For NMOS pair $WL=40\!\times\!0.5=20\,\mu\text{m}^2\Rightarrow \sigma_{Vt}\approx \frac{3}{\sqrt{20}}\approx0.67\,\text{mV}$ (similar magnitude for PMOS using larger W). Typical offset is well below **10 mV**.
* **Optional hysteresis:** A **100 MΩ** link from output to `vpre` provides a few mV of input‑referred hysteresis without disturbing DC.

---

# Sizing & back‑of‑envelope checks

* **Preamp gm** (per side): choose $I_{\text{tail}}\approx60\,\mu\text{A}$ (NMOS) + $60\,\mu\text{A}$ (PMOS). With $V_{ov}\approx0.18\!-\!0.22\,\text{V}$, $g_m \approx 2I/V_{ov} \sim 0.5\!-\!0.7\,\text{mS}$.
* **Preamp transimpedance:** $R_{out}\approx r_{o\_p}\parallel r_{o\_n}\sim 20\!-\!100\,\text{k}\Omega$ → $A_v \sim g_mR_{out} \approx 10\!-\!70$ (20–36 dB); easily pushes `vpre` past the first inverter trip with only **50 mV diff**.
* **Output driver for 2 pF:** Target $I\_{on}\sim 2\!-\!5\,\text{mA}$ → $t\approx \frac{C\Delta V}{I}\lesssim \frac{2\text{pF}\cdot1.65\text{V}}{3\text{mA}}\approx1.1\,\text{ns}$. (Short‑circuit dynamic only; no static current.)

---

# Analyses & measurements (how the netlist verifies specs)

* **Transient (speed & levels):** 50 mV step at 5 ns, `tpd` measured from input‑diff zero‑cross to $V_{DD}/2$ at output. Also measures **VOH/VOL** over a stable window.
* **Offset (Vos):** A **slow 40 mV ramp** (–20 mV→+20 mV) computes the instant the output crosses mid‑level and reports **input‑referred Vos**.
* **AC:** Small‑signal gain from `inp` to `out` around a chosen VCM (1.65 V).

---

# Complete NGSPICE netlist (drop‑in)

> Save the model file next to this netlist and ensure the `.include` path is correct.

```spice
* GF180 3.3V CMOS Comparator (rail-to-rail preamp + regenerative buffer)
* Specs target: |Vos|<=10mV, tpd<=5ns @ CL=2pF, VOH>=3.1V, VOL<=0.2V, P<=500uW

************ Models ************
.include "GF180nm_sm141064_simplified.txt"

************ Global params & supplies ************
.param VDD=3.3 VCM=1.65
VDD vdd 0 {VDD}

************ Comparator subcircuit ************
* Pins: INP INN OUT VDD VSS
.subckt comp_rr INP INN OUT VDD VSS

* --- Node naming ---
* vpre : preamp sum node -> first inverter input
* ns   : NMOS pair tail node
* sp   : PMOS pair source (common tail at top)
* ndl  : NMOS-pair diode-branch drain
* nd2  : PMOS-pair diode-branch drain
* v1   : internal first inverter output

* ===== NMOS input pair with PMOS mirror (boosts when INP>INN => vpre UP) =====
MNM_L ndl INP ns VSS nmos_3p3 L=0.5u W=40u
MNM_R vpre INN ns VSS nmos_3p3 L=0.5u W=40u

* Tail current source (self-biased mirror)
MNT ns VBN VSS VSS nmos_3p3 L=1u W=12u
MNB VBN VBN VSS VSS nmos_3p3 L=1u W=10u
RBN VDD VBN 47k

* PMOS active-load mirror for NMOS pair
MPP_D VDD ndl ndl VDD pmos_3p3 L=0.5u W=20u      * diode
MPP_O VDD ndl vpre VDD pmos_3p3 L=0.5u W=20u     * mirror to output node

* ===== PMOS input pair with NMOS mirror (also drives vpre UP when INP>INN) =====
MPM_L nd2 INP sp  VDD pmos_3p3 L=0.5u W=80u
MPM_R vpre INN sp VDD pmos_3p3 L=0.5u W=80u

* PMOS "tail" (current source to VDD), self-biased
MPT VDD VBP sp VDD pmos_3p3 L=1u W=12u
MPB VDD VBP VBP VDD pmos_3p3 L=1u W=10u
RBP VBP 0 47k

* NMOS active-load mirror for PMOS pair
MNN_D nd2 nd2 VSS VSS nmos_3p3 L=0.5u W=20u      * diode (to GND)
MNN_O vpre nd2 VSS VSS nmos_3p3 L=0.5u W=20u     * sink at vpre

* ===== Regenerative buffer (two inverters) =====
* INV1: fast regeneration
MP1 v1 vpre VDD VDD pmos_3p3 L=0.18u W=120u
MN1 v1 vpre VSS VSS nmos_3p3 L=0.18u W=60u

* Optional very-weak positive feedback (Schmitt-like)
RHYST vout vpre 100meg

* INV2: output driver
MP2 OUT v1 VDD VDD pmos_3p3 L=0.18u W=240u
MN2 OUT v1 VSS VSS nmos_3p3 L=0.18u W=120u

.ends comp_rr

************ TRANSIENT (speed, levels) ************
* 50 mV differential step at 5 ns around VCM
BVP vinp 0 v = { VCM + ( time>5n ? 25m : -25m ) }
BVN vinn 0 v = { VCM - ( time>5n ? 25m : -25m ) }

XU1 vinp vinn vout vdd 0 comp_rr
CLOAD vout 0 2p

************ AC (small-signal around VCM) ************
* Separate instance for AC so time sources don't interfere
VAC_P vinp_ac 0 AC 1 DC {VCM}
VAC_N vinn_ac 0 AC 0 DC {VCM}
XU2 vinp_ac vinn_ac vout_ac vdd 0 comp_rr

************ OFFSET (slow ramp to extract Vos) ************
* Sweep -20 mV -> +20 mV over 1 us around VCM
BVP_OFF vinp_off 0 v = { VCM + (-20m + 40m*time/1u) }
BVN_OFF vinn_off 0 v = { VCM - (-20m + 40m*time/1u) }
XU3 vinp_off vinn_off vout_off vdd 0 comp_rr
CLOAD_OFF vout_off 0 0.1p

************ Analyses, options, measures ************
.options method=gear reltol=1e-3 vabstol=1e-6 iabstol=1e-12
.temp 27

.control
set noaskquit

* --- Operating point (checks bias headrooms)
op

* --- AC gain of preamp+buffer around VCM
ac dec 100 1e3 100e6
* (Gain is |V(vout_ac)| since Vinp_ac has AC=1)
meas ac AV_1k  find mag(v(vout_ac)) at=1k
meas ac AV_1M  find mag(v(vout_ac)) at=1e6

* --- Transient for propagation delay & logic levels
save all
tran 10p 30n
meas tran TPD_RISE  trig v(vinp)-v(vinn) val=0 cross=1  targ v(vout) val={VDD/2} cross=1
meas tran TPD_FALL  trig v(vinp)-v(vinn) val=0 cross=2  targ v(vout) val={VDD/2} cross=2
meas tran VOL       min  v(vout) from=10n to=20n
meas tran VOH       max  v(vout) from=10n to=20n

* --- Transient for input-referred offset (Vos)
reset
tran 1n 1u
meas tran T_X   when v(vout_off)={VDD/2} cross=1
meas tran VINP_AT  find v(vinp_off) at=T_X
meas tran VINN_AT  find v(vinn_off) at=T_X
meas tran VOS param='VINP_AT - VINN_AT'

quit
.endc

.end
```

---

## How the testbench maps to your specs

* **Offset**: `VOS` (from 1 µs slow ramp) should read **≤ 10 mV** at TT with symmetric devices. Increase input pair area if you want extra margin.
* **Propagation delay**: `TPD_RISE` / `TPD_FALL` measured from input‑diff 0‑cross to $V_{DD}/2$ at `vout` under a **50 mV** step and **2 pF** load.
* **Output levels**: `VOH` / `VOL` taken between 10–20 ns after the step; expect **VOH ≥ 3.1 V** and **VOL ≤ 0.2 V** with the driver sizes shown.
* **Power**: Static ≈ **0.35–0.45 mW** from the preamp bias networks (tweak `RBN/RBP` or tail sizing to shift), dynamic only during switching in the output stage.
* **ICMR**: Complementary input pairs keep gm across **0.5 V–2.8 V**. You can sanity‑check by sweeping `VCM` and repeating the transient (set `.param VCM=...`).

### Tuning tips (if you need headroom or speed margins)

* Faster? Increase `MN1/MP1` and `MN2/MP2` widths or reduce `L` (kept ≥0.18 µm to satisfy your rule).
* Lower Vos? Increase **L** to **0.6–1.0 µm** and/or double **W** of both input pairs.
* More/less hysteresis? Change `RHYST` between **20–200 MΩ** (or comment it out to disable).

---

If you want me to tailor this for **±5% VDD stepping** (3.135 V / 3.3 V / 3.465 V) and dump a small table of `tpd`, `VOH/VOL`, and `Vos`, say the word and I’ll embed the `.step` and measurements.

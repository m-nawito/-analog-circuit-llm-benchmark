Looking at this common source amplifier design task, I need to create a circuit that achieves at least 20dB (10 V/V) gain with 100μA bias current using the GF180nm process.

## Design Approach and Topology Choice

I'll use a classic common source amplifier topology with:
- NMOS transistor as the gain element
- Resistive load (RD) at the drain
- Voltage divider bias network at the gate
- AC coupling capacitors at input and output
- No source degeneration (to maximize gain)

## Design Calculations

### 1. Transconductance Requirements
For a common source amplifier: Av = -gm × (RD || RL)

To achieve |Av| ≥ 10 V/V with RL = 10kΩ, I need sufficient gm and appropriate RD.

### 2. Operating Point Selection
From the NMOS model: VTH ≈ 0.7V, μnCox ≈ 0.024 A/V²

I'll choose VGS - VTH = 0.15V (low overdrive for high gm)
This gives: gm = 2×ID/(VGS-VTH) = 2×100μA/0.15V = 1.33 mS

### 3. Transistor Sizing
Using the square-law equation:
ID = (1/2) × μnCox × (W/L) × (VGS-VTH)²

100μA = (1/2) × 0.024 × (W/L) × (0.15)²
W/L = 100×10⁻⁶ / (0.012 × 0.0225) = 370

Choosing L = 0.36μm (2× minimum for better matching):
W = 370 × 0.36μm = 133μm

### 4. Load Resistor Selection
With VDD = 3.3V and ID = 100μA, I'll use RD = 20kΩ
This gives VRD = 2.0V drop, leaving adequate headroom.

Effective load: RD || RL = (20k × 10k)/(30k) = 6.67kΩ
Voltage gain: Av = -1.33mS × 6.67kΩ = -8.87 V/V

This is slightly below target. Let me increase RD to 22kΩ:
RD || RL = (22k × 10k)/32k = 6.875kΩ
Av = -1.33mS × 6.875kΩ = -9.14 V/V

Still slightly low. Let me reduce overdrive to 0.13V:
gm = 2×100μA/0.13V = 1.54 mS
W/L = 100×10⁻⁶ / (0.012 × 0.0169) = 493

With L = 0.36μm: W = 177μm
Av = -1.54mS × 6.875kΩ = -10.6 V/V ✓ (Meets requirement)

### 5. DC Bias Network
VGS = VTH + 0.13V = 0.83V

Using voltage divider biasing:
VG = 0.83V = VDD × R2/(R1+R2)
R2/(R1+R2) = 0.83/3.3 = 0.252

Choosing R2 = 100kΩ, R1 = 300kΩ

### 6. Coupling Capacitors
For fc << 1kHz (choosing fc = 1Hz):
- Input: Cin = 1/(2π×1Hz×75kΩ) ≈ 2.1μF → Use 10μF
- Output: Cout = 1/(2π×1Hz×10kΩ) ≈ 15.9μF → Use 22μF

## Complete NGSPICE Netlist

```
* Common Source Amplifier - GF180nm Process
* Target: |Av| >= 20dB at 1kHz with ID = 100uA

.title Common Source Amplifier Design

* Include model file (provided separately)
.include GF180nm_sm141064_simplified.txt

* Power supply
VDD vdd 0 DC 3.3

* Input signal source
Vin in 0 DC 0 AC 1m

* Biasing resistors
R1 vdd vg 300k
R2 vg 0 100k

* Load resistor
RD vdd vd 22k

* NMOS transistor (W=177um, L=0.36um)
M1 vd vg 0 0 nmos_3p3 W=177u L=0.36u

* Coupling capacitors
Cin in vg 10u
Cout vd out 22u

* Load for AC analysis
RL out 0 10k
CL out 0 1p

* Analysis commands
.control
* DC operating point
op
print all

* Check DC operating point
echo "DC Operating Point:"
echo "VGS = " v(vg)
echo "VDS = " v(vd)
echo "ID = " i(vdd)
echo ""

* AC analysis from 1Hz to 100kHz
ac dec 100 1 100k

* Calculate and display gain at 1kHz
echo "AC Analysis Results at 1kHz:"
meas ac gain_mag find vdb(out) at=1k
meas ac gain_phase find vp(out) at=1k
echo ""

* Plot frequency response
plot vdb(out) xlabel 'Frequency (Hz)' ylabel 'Gain (dB)' title 'Common Source Amplifier Frequency Response'
plot vp(out) xlabel 'Frequency (Hz)' ylabel 'Phase (degrees)' title 'Phase Response'

.endc
.end
```

## Testbench Approach and Analysis Choice

1. **DC Operating Point Analysis**: Verifies the transistor is properly biased at ID ≈ 100μA and in saturation (VDS > VGS - VTH)

2. **AC Analysis**: Sweeps from 1Hz to 100kHz to measure small-signal gain at 1kHz and verify frequency response

3. **Verification Points**:
   - VGS ≈ 0.83V (ensuring proper bias)
   - VDS ≈ 1.1V (ensuring saturation: VDS > VGS - VTH = 0.13V)
   - ID ≈ 100μA (meeting bias specification)
   - |Av| ≥ 20dB at 1kHz (meeting gain specification)

The design uses minimal overdrive voltage to maximize transconductance while maintaining the transistor in saturation, achieving the required gain with the specified bias current.

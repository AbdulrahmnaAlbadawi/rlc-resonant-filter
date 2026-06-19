# Series RLC Resonant Filter — Design & LTspice Simulation

A series RLC resonant filter designed, simulated in LTspice, and documented against its analytical predictions. The design targets a resonant frequency of **50 kHz** with a quality factor of **~31.6**.

---

## Overview

A series RLC circuit responds strongly to one specific frequency — its resonant frequency, f₀ — and weakly to everything else. At resonance the inductive and capacitive reactances are equal and opposite, so they cancel. What remains depends on where the output is measured:

- **Across L + C** → a sharp **notch (band-stop)**: the L–C branch impedance collapses to near zero at f₀, so almost no voltage appears across it.
- **Across R** → a **bandpass peak**: with the reactances cancelled, the full source voltage drops across R at f₀.

Both responses come from the *same* circuit and the *same* resonant frequency. This project documents both.

---

## Circuit

A voltage source drives a resistor, inductor, and capacitor in series, with the loop returning to ground.

```
Vin --- R1 --- [out] --- L1 --- C1 --- GND
(AC 1)  100Ω           10mH    1nF
```

*(See `schematic.png`.)*
/docs.schematic
### Component values

| Component | Value | Role |
|-----------|-------|------|
| R1 | 100 Ω | Sets the bandwidth / sharpness (Q) |
| L1 | 10 mH | Resonant element |
| C1 | 1 nF | Resonant element |
| V1 | AC 1 V | Small-signal AC stimulus for the sweep |

The output node `out` sits between R1 and the L1–C1 branch. Probing `V(out)` shows the voltage across L + C (the notch). The bandpass is obtained as `V(in) − V(out)`, the voltage across R1.

---

## Design math

**Resonant frequency**

$$f_0 = \frac{1}{2\pi\sqrt{LC}}$$

$$f_0 = \frac{1}{2\pi\sqrt{(10\times10^{-3})(1\times10^{-9})}} = 50{,}329\ \text{Hz} \approx 50.3\ \text{kHz}$$

**Quality factor** (sharpness of the resonance)

$$Q = \frac{1}{R}\sqrt{\frac{L}{C}} = \frac{1}{100}\sqrt{\frac{10\times10^{-3}}{1\times10^{-9}}} = 31.6$$

**−3 dB bandwidth**

$$\text{BW} = \frac{f_0}{Q} = \frac{50{,}329}{31.6} \approx 1{,}592\ \text{Hz}$$

A higher Q (smaller R) gives a narrower, sharper response; a lower Q (larger R) broadens it.

---

## Simulation setup (LTspice)

1. Draw the series loop: `V1 → R1 → out → L1 → C1 → GND`, with a ground on the V1 return rail.
2. Set the source as the AC stimulus: right-click V1 → **Advanced** → **AC Amplitude = 1**. In a netlist this is the `AC 1` keyword on the source line. (Without it, LTspice reports *"No AC stimulus found"* and only runs the DC operating point.)
3. Add the analysis directive for a logarithmic sweep that brackets 50 kHz:
   ```
   .ac dec 100 1k 1Meg
   ```
4. **Run**, then click the `out` wire to plot `V(out)`. To see the bandpass, add the trace `V(in)-V(out)`.

---

## Results

*(See `result.png`.)*

The simulated magnitude response shows a deep, narrow notch centered on ~50 kHz at the `V(out)` node, flat near 0 dB elsewhere — exactly the band-stop behavior predicted. The complementary `V(in)−V(out)` trace peaks at the same frequency, giving the bandpass response.

### Calculated vs simulated

| Quantity | Calculated | Simulated | Agreement |
|----------|-----------|-----------|-----------|
| Resonant frequency f₀ | 50.33 kHz | ~50.3 kHz | ✔ |
| Quality factor Q | 31.6 | ~31.6 | ✔ |
| −3 dB bandwidth | 1.59 kHz | ~1.59 kHz | ✔ |
| −3 dB band edges | 49.54 – 51.13 kHz | ~49.5 – 51.1 kHz | ✔ |

The simulation matches the hand analysis across every metric, confirming both the component selection and the analytical model.

---

## Key findings

- Analysis of passive RLC circuit impedance behavior and resonance.
- Resonant frequency, Q-factor, and bandwidth calculated from component values.
- AC frequency sweep simulation with magnitude/phase Bode plots in LTspice.
- Output placement (across L–C vs across R) determines notch vs bandpass response — the same resonance yields two complementary filter functions.

---

## Files

| File | Description |
|------|-------------|
| `images/schematic.png` | LTspice schematic of the series RLC filter |
| `images/result.png` | Simulated frequency response (`V(out)` notch) |
| `ltspice/` | LTspice schematic source files |
| `README.md` | This document |

---

## How to reproduce

1. Install [LTspice](https://www.analog.com/en/resources/design-tools-and-calculators/ltspice-simulator.html) (free).
2. Recreate the schematic above, or open the `.asc` file from the `ltspice/` folder.
3. Confirm V1 carries `AC 1`, add `.ac dec 100 1k 1Meg`, and run.
4. Probe `V(out)` for the notch; add `V(in)-V(out)` for the bandpass.

---

## Extensions

- **Sweep R to show Q control:** add `.step param Rval list 10 100 1k` and set R1 to `{Rval}` to plot three Q values on one graph.
- **Cross-check in MATLAB** by plotting the transfer function H(s) and overlaying it on the LTspice export.
- **Build the physical circuit** and measure it, comparing measured vs simulated.

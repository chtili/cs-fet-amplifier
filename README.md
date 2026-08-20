# cs-fet-amplifier

Linear common source MOSFET amplifier with source degeneration.

## Motivation

An ordinary common source MOSFET amplifier suffers from distortion, especially when the input swings
unexpectedly. Adding a source resistor (source degeneration) trades away some raw gain for a more
linear, predictable response that's less sensitive to the exact device parameters — this project
designs, hand-calculates, and simulates that trade-off for a specific target gain.

## Design

See [docs/cs amplifier design.pdf](docs/cs%20amplifier%20design.pdf) for the full hand calculations;
the LTspice schematic is in [sims/](sims/).

Topology: an NMOS (2N7000) common-source stage with a source resistor `Rs` for degeneration, resistive
drain load `Rd`, a resistive divider (`R1`/`R2`) off `Vdd` for gate bias, and coupling capacitors
`C1`/`C2` on the input and output.

**Target operating point:**
- `Vdd` = 10 V
- `Id` = 2 mA, `Vov` = 0.4 V
- Device transconductance parameter `kn` ≈ 25 mA/V² (extracted from the 2N7000 datasheet's gm
  specification), giving `gm` = `kn`·`Vov` ≈ 10 mA/V

**Choosing Rd and Rs:** with `Av = -gm·Rd / (1 + gm·Rs)` and a target `|Av| ≥ 10`, the constraint
`Rd ≥ 1 kΩ + 10·Rs` must hold. Chose `Rd = 2 kΩ`, `Rs = 100 Ω`, which sits right at that boundary.
This puts `Vd = 6 V` and `Vs = 0.2 V`, safely keeping the device in saturation (`Vds ≫ Vov`).

**Gate bias:** for `Vov = 0.4 V` and an estimated `Vt ≈ 1.5 V` (2N7000 datasheet range is 0.8–3.0 V),
target gate voltage `Vg ≈ 2.1 V`. Divider values `R1 = 3.9 MΩ`, `R2 = 1 MΩ` set `Vg ≈ 2.1 V` from the
10 V rail.

**Coupling capacitors:** `C1 = C2 = 0.1 µF`, chosen simply rather than optimized for a specific
low-frequency cutoff.

**Predicted performance:** `Av ≈ -10` (20 dB), `Rin = R1 ‖ R2 ≈ 796 kΩ`, `Rout ≈ Rd ≈ 2 kΩ`.

**Simulation setup:** LTspice, 2N7000 SPICE model, `RL = 10 MΩ` (effectively unloaded, so the sim
characterizes the stage's own gain/bandwidth rather than a loaded response) — a 1 kHz, 10 mV transient
run and a 1 Hz–1 MHz AC sweep. Results are in [results/](results/).

## Discussion

- **Transient response** ([results/csamp1khz.png](results/csamp1khz.png)): a 10 mV, 1 kHz input
  produces a ~130 mV output, with the output trough aligned to the input peak — confirming the expected
  180° inversion of a common-source stage.
- **AC response** ([results/csampAC.png](results/csampAC.png)): a wide, flat passband at ~22.4 dB
  (gain magnitude ≈ 13.2), rolling off below roughly the low tens of Hz (set by the coupling
  capacitors and input impedance) and above roughly 100 kHz (device/parasitic capacitance).
- **Simulated gain (≈13.2×) came in higher than the hand-calculated target (10×).** The likely cause is
  the 2N7000 SPICE model's actual `kn`/`Vt` differing from the datasheet-derived hand estimate — `Vt`
  in particular was only an estimate (2N7000 spec range is 0.8–3.0 V, and 1.5 V was a guess). Next step:
  run a `.op` analysis to read the model's actual DC operating point and `gm`, then re-tune `Rd`/`Rs`
  if a tighter match to the target gain matters for an eventual hardware build.
- **Not yet done:** measuring the actual -3 dB bandwidth edges precisely (rather than reading them
  qualitatively off the Bode plot), and building/measuring the circuit in hardware to compare against
  simulation.

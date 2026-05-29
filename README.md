# 915 MHz GaAs MMIC Low Noise Amplifier (LNA) PCB

A 915 MHz low-noise amplifier printed circuit board built around the Qorvo SPF5043Z GaAs pHEMT MMIC, designed as part of a self-directed RF hardware project targeting aerospace and defense applications.

## Design Goals

- **Target Frequency:** 915 MHz (ISM band)
- **Active Device:** Qorvo SPF5043Z (GaAs pHEMT MMIC LNA)
- **Substrate:** FR4 (εr ≈ 4.4), 1.6 mm thickness
- **System Impedance:** 50 Ω
- **Target Performance:**
  - Gain (S21): > 15 dB
  - Noise Figure: < 2 dB
  - Input Return Loss (S11): < -10 dB
  - Output Return Loss (S22): < -10 dB

## Circuit Architecture

The SPF5043Z has integrated internal impedance matching, so the external circuit is minimal:

```
RF IN --[100 pF DC block]--[1.5 nH match]-- Pin 1 (RF IN)
                                              |
                                          SPF5043Z
                                              |
Pin 3 (RF OUT / DC BIAS) --[100 pF DC block]-- RF OUT
        |
   [150 nH RF choke]
        |
   [100 pF + 100 nF bypass caps]
        |
      5 V supply

Pin 2, Pin 4 -> GND (via stitching directly at chip)
```

**Pin 3 dual purpose:** Pin 3 is both the RF output and the DC bias input for VDD. The 150 nH inductor acts as an RF choke (passes DC, blocks 915 MHz). The 100 pF cap on the output passes RF while blocking the supply DC from reaching the load.

## Bill of Materials

| Reference | Value | Function |
|---|---|---|
| U1 | SPF5043Z | GaAs MMIC LNA |
| C1 | 100 nF (0.1 µF) | VDD bypass (bulk) |
| C2, C3, C4 | 100 pF | DC blocks on RF input, RF output, VDD |
| L1 | 1.5 nH | Input matching inductor |
| L2 | 150 nH | VDD RF choke |
| J1, J2 | SMA Edge Mount | RF input/output connectors |

## Tools Used

- **KiCAD 9.0** — Schematic capture and PCB layout
- **LTspice** — DC operating point and AC sweep simulation
- **Keysight ADS** — S-parameter and noise-figure simulation (in progress, via Purdue ECN)
- **OSHPark** — PCB fabrication (planned)
- **VNA** — S11 and S21 validation (planned post-fabrication)

## Board Specifications

- **Layers:** 2-layer FR4
- **Thickness:** 1.6 mm
- **Trace Width:** 3.0 mm for 50 Ω microstrip on FR4 1.6 mm
- **Layout Rules:**
  - RF traces kept short and direct between SMA and chip
  - Ground stitching vias placed around RF traces
  - Power supply traces routed away from RF signal path
  - Via holes placed adjacent to GND pins to minimize inductance

## Simulation Results

### LTspice: Supply-Rail Isolation (AC Sweep)

Verified that the L2 RF choke and C1 bypass capacitor prevent the 915 MHz RF signal from coupling back into the supply line.

| Frequency | Attenuation at VDD Node | Interpretation |
|---|---|---|
| 50 MHz | ~-64 dB | Partial isolation |
| **915 MHz** | **~-114 dB** | **Near-complete RF isolation at operating frequency** |
| 4 GHz | ~-144 dB | Excellent isolation at upper chip range |

**Result:** The supply filter provides approximately 114 dB of RF isolation at 915 MHz. RF signal cannot meaningfully couple back into the power supply, so the bias circuit is stable.

### LTspice: DC Operating Point

Bias network architected to deliver 5 V to the chip and verified at the supply rail (5.000 V), with current limited by the chip's internal characteristics.

**Pending:** DC operating point re-verification with R1 set to model the SPF5043Z's 46 mA target (versus the SPF5189Z's 90 mA used in initial simulation).

### Keysight ADS: S-Parameter and Noise-Figure Analysis (Pending)

Planned simulations once Touchstone (.s2p) model is loaded:

- S21 (gain) vs frequency
- Noise Figure (NF) vs frequency
- S11, S22 vs frequency (log-magnitude and Smith Chart)
- P1dB (output 1 dB compression point)
- Stability factors K and B1 vs frequency

ADS simulation files and output plots are not included in this repository.

## Validation Plan (Post-Fabrication)

1. Visually inspect solder joints under magnification
2. Measure DC operating point with bench DMM (verify 5 V supply, ~46 mA current)
3. Connect to VNA and measure:
   - S11 (input return loss) — target dip below -10 dB at 915 MHz
   - S21 (gain) — target ~18-20 dB at 915 MHz
   - S22 (output return loss) — target below -10 dB
4. Compare measured S-parameters against simulation results
5. Document any discrepancies and root-cause them

## Project Status

- [x] Theory reading and reference application circuit studied
- [x] BOM finalized for SPF5043Z
- [x] LTspice DC operating point simulation
- [x] LTspice AC sweep supply filter performance (114 dB isolation at 915 MHz)
- [x] Initial KiCAD layout complete (SPF5189Z footprint)
- [ ] SPF5043Z footprint swap and PCB re-route
- [ ] DRC pass and Gerber export
- [ ] PCB ordered from OSHPark
- [ ] Keysight ADS S-parameter simulation
- [ ] Board assembled
- [ ] DC operating point measured
- [ ] VNA measurements taken (S11, S21, S22)
- [ ] Simulation vs measured comparison documented

## References

- [Qorvo SPF5043Z Product Page (DigiKey)](https://www.digikey.com/en/products/detail/qorvo/SPF5043Z/2708620)
- Qorvo SPF5189Z 900 MHz Reference Application Circuit (datasheet page 8)
- *Microwave Engineering* — David M. Pozar
- *RF Circuit Design* — Christopher Bowick

## License

Hardware design files (schematics, PCB layouts, BOM, Gerbers) are released under the **CERN Open Hardware License Version 2 - Permissive (CERN-OHL-P)**. Software and documentation are released under the **MIT License**.

---

*Project in active development. Boards not yet fabricated. Measurements not yet taken. README will be updated as milestones complete.*

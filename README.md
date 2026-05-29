# 915 MHz GaAs MMIC Low Noise Amplifier (LNA) PCB

A 915 MHz low-noise amplifier printed circuit board built around a Qorvo GaAs pHEMT MMIC, designed as part of a self-directed RF hardware project targeting aerospace and defense applications.

## Project Summary

This project takes a commercial GaAs MMIC LNA and builds a complete RF front-end PCB around it: bias network, supply filtering, DC blocking, and 50 Ω microstrip routing. The goal is end-to-end ownership of an RF design from datasheet study through KiCAD layout, simulation, fabrication, and VNA validation.

**Version 1 (in this repository):** Built around the **Qorvo SPF5189Z** GaAs MMIC LNA. KiCAD layout complete (DRC clean, ERC clean). LTspice bias and supply-filter simulations complete.

**Version 2 (planned):** Migration to the **Qorvo SPF5043Z** GaAs MMIC LNA. The SPF5043Z draws 46 mA versus the SPF5189Z's 90 mA, offering better power efficiency at modest cost in gain. Footprint swap (SOT-363 to SOT-343), trace re-route, and bias network re-simulation are planned.

## Design Goals

- **Target Frequency:** 915 MHz (ISM band)
- **Substrate:** FR4 (εr ≈ 4.4), 1.6 mm thickness
- **System Impedance:** 50 Ω
- **Target Performance (per SPF5189Z datasheet):**
  - Gain (S21): ~19 to 20 dB at 915 MHz
  - Noise Figure: ~0.6 dB
  - Supply: 5 V at ~90 mA
  - Output P1dB: ~22 to 23 dBm

## Circuit Architecture

The SPF5189Z has integrated internal impedance matching, so the external circuit is minimal:

```
RF IN --[C2 100 pF block]--[L1 1.5 nH match]-- Pin 1 (RF IN)
                                                  |
                                              SPF5189Z
                                                  |
Pin 3 (RF OUT / DC BIAS) --[C3 100 pF block]-- RF OUT
        |
   [L2 150 nH RF choke]
        |
   [C4 100 pF + C1 100 nF bypass]
        |
      5 V supply

Pin 2, Pin 4 -> GND (with via stitching directly at chip)
```

**Pin 3 dual purpose:** Pin 3 is both the RF output AND the DC bias input for VDD. The 150 nH inductor acts as an RF choke (passes DC, blocks 915 MHz). The 100 pF cap on the output passes RF while blocking the supply DC from reaching the load.

## Bill of Materials

| Reference | Value | Function |
|---|---|---|
| U1 | SPF5189Z | GaAs MMIC LNA |
| C1 | 100 nF (0.1 µF) | VDD bypass (bulk) |
| C2, C3, C4 | 100 pF | DC blocks on RF input, RF output, VDD |
| L1 | 1.5 nH | Input matching inductor |
| L2 | 150 nH | VDD RF choke |
| J1, J2 | SMA Edge Mount | RF input/output connectors |
| J3 | 1x2 header | 5 V power input |

## Tools Used

- **KiCAD 9.0** — Schematic capture and PCB layout
- **LTspice** — DC operating point and AC sweep simulation
- **Keysight ADS** — S-parameter and noise-figure simulation (planned, via Purdue ECN)
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
- **DRC:** Clean (0 violations, 0 unconnected items)
- **ERC:** Clean (0 violations)

## Simulation Results

### LTspice: Supply-Rail Isolation (AC Sweep)

Verified that the L2 RF choke and C1 bypass capacitor prevent the 915 MHz RF signal from coupling back into the supply line. AC sweep from 1 MHz to 4 GHz, with 1 V AC source representing the worst-case RF leakage into the supply node.

| Frequency | Attenuation at VDD Node |
|---|---|
| 50 MHz | approximately -64 dB |
| **915 MHz** | **approximately -114 dB** |
| 4 GHz | approximately -144 dB |

**Result:** The supply filter provides approximately 114 dB of RF isolation at the operating frequency. RF signal cannot meaningfully couple back into the power supply, so the bias circuit is stable.

### LTspice: DC Operating Point

Modeled with V1 = 5 V DC, L2 = 150 nH, R1 = 55.6 Ω (SPF5189Z equivalent chip resistance at 90 mA), C1 = 100 nF bypass.

| Node / Component | Simulated | Expected | Status |
|---|---|---|---|
| V(n001) — VDD node | 5.000 V | 5 V | Pass |
| V(n002) — after RF choke | 4.99991 V | ~5 V | Pass |
| I(R1) — chip bias current | 89.9 mA | ~90 mA | Pass |
| I(L2) — choke current | 89.9 mA | ~90 mA | Pass |
| I(C1) — bypass cap current | ~0 A | 0 A (open at DC) | Pass |

**Result:** DC bias network delivers 5 V at 90 mA to the chip as specified in the SPF5189Z datasheet.

### Keysight ADS: S-Parameter and Noise-Figure Analysis (Pending)

Planned simulations once the Touchstone (.s2p) model is loaded into ADS via Purdue ECN:

- S21 (gain) vs frequency, with markers at 915 MHz
- Noise Figure (NF) and NFmin vs frequency
- S11, S22 vs frequency (log-magnitude and Smith Chart)
- P1dB (output 1 dB compression point)
- Stability factors K and B1 vs frequency

ADS simulation files and output plots are not included in this repository.

## Validation Plan (Post-Fabrication)

1. Visually inspect solder joints under magnification
2. Measure DC operating point with bench DMM (verify 5 V supply, ~90 mA current for SPF5189Z)
3. Connect to VNA and measure:
   - S11 (input return loss) — target dip below -10 dB at 915 MHz
   - S21 (gain) — target ~19 to 20 dB at 915 MHz
   - S22 (output return loss) — target below -10 dB
4. Compare measured S-parameters against simulation results
5. Document any discrepancies and root-cause them

## Project Status

- [x] Theory reading and reference application circuit studied
- [x] BOM finalized for SPF5189Z
- [x] LTspice DC operating point simulation (5 V at 89.9 mA confirmed)
- [x] LTspice AC sweep supply filter performance (~114 dB isolation at 915 MHz)
- [x] KiCAD schematic capture (ERC clean)
- [x] KiCAD PCB layout (DRC clean)
- [ ] Gerber export
- [ ] PCB ordered from OSHPark
- [ ] Keysight ADS S-parameter simulation
- [ ] Board assembled and DC operating point measured
- [ ] VNA measurements (S11, S21, S22) taken
- [ ] Simulation vs measured comparison documented
- [ ] v2 migration to SPF5043Z (footprint swap, re-route, bias re-simulation)

## References

- [Qorvo SPF5189Z Datasheet (Mouser)](https://www.mouser.com/datasheet/2/412/RFMDS04436_1-2564742.pdf)
- [Qorvo SPF5043Z Product Page (DigiKey)](https://www.digikey.com/en/products/detail/qorvo/SPF5043Z/2708620)
- *Microwave Engineering* — David M. Pozar
- *RF Circuit Design* — Christopher Bowick

## License

Hardware design files (schematics, PCB layouts, BOM, Gerbers) are released under the **CERN Open Hardware License Version 2 - Permissive (CERN-OHL-P)**. Software and documentation are released under the **MIT License**.

---

*Project in active development. Boards not yet fabricated. Measurements not yet taken. README will be updated as milestones complete.*

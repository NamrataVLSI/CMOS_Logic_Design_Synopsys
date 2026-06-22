# Standard Cell Layout Library — SAED 32/28nm, 11-Track Architecture

A full-custom standard cell library designed and verified in **Synopsys Custom Compiler** using the **SAED 32/28nm PDK**. The library implements eight combinational logic cells in an 11-track (11T) architecture, targeting compatibility with industry-standard digital implementation flows.

---

## Design Objectives

- Implement a self-consistent cell library with uniform height, power rail placement, and pin access grid to support abutment-based placement
- Establish a poly pitch–derived layout grid that enforces manufacturable gate geometries across all cell variants
- Achieve clean DRC and LVS signoff on all cells using IC Validator (Synopsys)
- Validate post-extraction parasitics against pre-layout simulation to confirm functional correctness

---

## Technology and Design Parameters

| Parameter | Value |
|---|---|
| PDK | SAED 32/28nm (Synopsys Educational) |
| EDA Tool | Synopsys Custom Compiler |
| Cell Architecture | 11-Track (11T) |
| Poly Width | 0.03 μm |
| Poly Spacing | 0.122 μm |
| Poly Pitch | 0.152 μm |
| Cell Height | 11 × 0.152 μm = **1.672 μm** |
| Power Rails | VDD (top), VSS (bottom), M1 horizontal |
| Routing Grid | Based on M1/M2 minimum pitch per PDK DRM |

Cell height is derived directly from poly pitch × track count, establishing a fixed silicon boundary that allows adjacent cells to share power rails without additional gap cells.

---

## Standard Cell Architecture

All cells conform to an 11-track height discipline. Key architectural decisions:

**Power rail integration:** VDD and VSS rails run full-width on M1 at fixed Y-coordinates, enabling seamless abutment. PMOS wells contact VDD; NMOS wells contact VSS within each cell's boundary.

**Pin placement:** All signal pins (A, B, Y, etc.) are placed on M1 and snapped to the routing grid to guarantee legal routing by place-and-route tools without additional spreading.

**Well sharing:** PMOS devices are sized and placed to share N-well across abutted cells, eliminating N-well spacing violations at cell boundaries. NMOS devices share P-substrate implant continuously.

**Dummy poly:** End-of-row poly dummies are incorporated at cell edges per PDK requirements, preventing gate CD variation from OPC proximity effects at boundary geometries.

---

## Layout Methodology

Cells were drawn layer-by-layer in the following order:

1. **Active (DIFF):** PMOS/NMOS active regions sized per drive strength target, respecting minimum width and enclosure rules
2. **Poly (GATE):** Gate fingers placed on poly pitch grid; width fixed at 0.03 μm across all devices
3. **Implants (N+/P+) and Wells:** Drawn to meet enclosure and spacing DRC relative to active
4. **Contacts:** Placed at maximum density within DRC constraints on source/drain and gate contacts
5. **M1 routing:** Intra-cell signal routing and power rail completion
6. **Pin labeling:** Net labels assigned on M1 for LVS recognition

Complex cells (XOR2, XNOR2) use stacked series-parallel PMOS/NMOS networks derived from Euler path decomposition to minimize cell width while maintaining single-poly-pitch gate alignment.

---

## Design Rules and Constraints Followed

All cells were checked against the SAED 32/28nm DRC ruleset. Critical rules enforced during layout:

- Minimum poly width and spacing (0.03 μm / 0.122 μm)
- Minimum active width, active-to-poly enclosure, and active-to-active spacing
- Well and implant enclosure of active
- Contact-to-gate spacing (poly contact kept ≥ minimum spacing from gate edge)
- M1 minimum width, spacing, and enclosure of contact
- End-of-line (EOL) spacing on M1 segments
- N-well spacing across cell boundaries (verified in abutment test bench)

No waived rules. All cells pass with zero DRC violations.

---

## Verification Flow

```
Layout (Custom Compiler GDS)
        │
        ▼
   DRC — IC Validator
   (SAED 32/28nm DRC deck)
        │ Clean
        ▼
   LVS — IC Validator
   (Schematic source: Custom Compiler)
        │ Pass
        ▼
   Parasitic Extraction — StarRC
   (Netlist: SPEF/DSPF)
        │
        ▼
   Post-Layout Simulation — PrimeWave / HSPICE
   (Functional + timing verification vs. pre-layout)
```

All eight cells completed this flow end-to-end. Post-layout netlists are available under `extraction/` for each cell.

---

## Library Contents

| Cell | Function | Transistor Count | Drive Strength |
|---|---|---|---|
| INV | Y = !A | 2 | X1 |
| BUF | Y = A | 4 | X1 |
| NAND2 | Y = !(A & B) | 4 | X1 |
| NOR2 | Y = !(A \| B) | 4 | X1 |
| AND2 | Y = A & B | 6 | X1 |
| OR2 | Y = A \| B | 6 | X1 |
| XOR2 | Y = A ^ B | 8 | X1 |
| XNOR2 | Y = !(A ^ B) | 8 | X1 |

Individual cell documentation (schematic, layout screenshots, DRC/LVS results, extracted parasitics, simulation waveforms) is located in each cell's subdirectory.

---

## Key Results

- **DRC:** 0 violations across all 8 cells (SAED 32/28nm DRC deck, IC Validator)
- **LVS:** Full match on all cells; net, device, and pin counts verified against schematic
- **Parasitic extraction:** RC parasitics extracted with StarRC; netlists confirmed functional in post-layout simulation
- **Cell height consistency:** All cells measure exactly 1.672 μm; abutment verified across all adjacent cell pairs
- **Timing delta (pre vs. post layout):** [To be populated with actual simulation data]

---

## Future Improvements

- **Drive strength variants:** Implement X2 and X4 variants for INV, BUF, NAND2, NOR2 with proportionally scaled PMOS/NMOS widths
- **Sequential cells:** DFF, latch (D-type) to extend the library toward a minimal synthesis-ready cell set
- **Liberty (.lib) characterization:** Integrate with a timing characterization flow (SiliconSmart or open-source equivalent) to generate timing arcs for synthesis
- **LEF generation:** Export abstract LEF views for use in Innovus/ICC2 placement flows
- **9-track (9T) variant:** Re-implement critical cells in 9T to benchmark area versus routability tradeoff at advanced nodes

---

## Tools and PDK

| Tool | Version | Purpose |
|---|---|---|
| Synopsys Custom Compiler | — | Schematic entry, layout |
| Synopsys IC Validator | — | DRC / LVS |
| Synopsys StarRC | — | Parasitic extraction |
| Synopsys PrimeWave / HSPICE | — | Post-layout simulation |
| SAED 32/28nm PDK | Educational | Process design rules, SPICE models |


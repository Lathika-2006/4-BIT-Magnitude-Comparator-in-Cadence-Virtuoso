# 4-Bit Magnitude Comparator: Full-Custom Design in Cadence Virtuoso
*180 nm CMOS (gpdk180) • Schematic • Simulation • Layout*

A complete full-custom implementation of a 4-bit magnitude comparator, built hierarchically from hand-drawn transistor-level cells (INV, AND2, OR2, XNOR2).

*Overview* • *Architecture* • *Results* • *Visual Gallery* • *Getting Started*

---

## 🎯 Overview

A magnitude comparator compares two binary numbers **A[3:0]** and **B[3:0]** and tells whether **A > B**, **A = B** or **A < B**. It is a core block in ALUs, sorting networks, address decoders, and control logic in processors.

This project implements the comparator **full-custom**: every gate is drawn at the transistor (NMOS/PMOS) level, simulated, and laid out manually in Cadence Virtuoso on the 180 nm gpdk180 process.

### Circuit Diagram

![Circuit Diagram](circuit/circuit%20diagram.jpg)

---

## ✨ Key Highlights

🔌 **Full-Custom Design**: Transistor-level schematic and hand-drawn layout  
🧩 **Hierarchical Cell Library**: INV, AND2, OR2 and XNOR2 cells reused to build the 1-bit and 4-bit comparators  
🔬 **Simulation**: Transient simulation in Spectre/ADE  
📐 **Physical Flow**: Schematic → Simulation → Layout → DRC  
🏭 **180 nm gpdk180 PDK**: Industry-style academic design kit  

---

## 🏗 Architecture

### System Overview

```
        A[3:0] ──┐                    ┌── A > B
                 ├──► 4-BIT MAGNITUDE ├── A = B
        B[3:0] ──┘     COMPARATOR      └── A < B
```

### Truth Table (Single-Bit Cell)

| A | B | A > B | A = B | A < B |
|---|---|-------|-------|-------|
| 0 | 0 | 0 | 1 | 0 |
| 0 | 1 | 0 | 0 | 1 |
| 1 | 0 | 1 | 0 | 0 |
| 1 | 1 | 0 | 1 | 0 |

### Boolean Equations (4-Bit)

Bit equality: **Eᵢ = (Aᵢ ⊙ Bᵢ)** (XNOR)

```
(A = B) = E3 · E2 · E1 · E0
(A > B) = A3·B3' + E3·A2·B2' + E3·E2·A1·B1' + E3·E2·E1·A0·B0'
(A < B) = A3'·B3 + E3·A2'·B2 + E3·E2·A1'·B1 + E3·E2·E1·A0'·B0
```

Comparison starts from the **MSB**: if the MSBs differ, the result is decided immediately; if they are equal, the next lower bit is checked, and so on.

### Cell-Based Implementation

| Function | Implementation |
|----------|----------------|
| Bit equality Eᵢ | XNOR2 cell |
| Inverted inputs (Aᵢ', Bᵢ') | INV cell |
| Product terms (A > B, A < B) | AND2 cells |
| Combining terms | OR2 cells |

### Complete Design Flow

```
 SPECIFICATION  (4-bit magnitude comparator)
        │
        ▼
 TRUTH TABLE & BOOLEAN EQUATIONS
        │
        ▼
 GATE-LEVEL DESIGN  (INV, AND2, OR2, XNOR2)
        │
        ▼
 TRANSISTOR-LEVEL SCHEMATIC  (Virtuoso Schematic Editor)
        │
        ▼
 SIMULATION  (Spectre / ADE, gpdk180 models)
        │
        ▼
 LAYOUT  (Virtuoso Layout XL)
        │
        ▼
 DRC
```

---

## 🧠 Theoretical Background

### What Is a Magnitude Comparator?

A magnitude comparator is a combinational circuit that compares two n-bit unsigned binary numbers and produces three outputs: **A > B**, **A = B** and **A < B**. Exactly one of the three outputs is 1 for any input combination, so

```
(A>B) + (A=B) + (A<B) = 1   (mutually exclusive and exhaustive)
```

### How Binary Comparison Works

1. If A3 ≠ B3, the number with the 1 in that position is larger, and the result is decided.
2. If A3 = B3, move to the next bit (A2 vs B2), and so on.
3. If all four bits are equal, then A = B.

**Example:** A = 1010 (10), B = 1001 (9). A3 = B3 = 1, A2 = B2 = 0, A1 = 1 but B1 = 0, so **A > B**.

### 1-Bit Comparator Derivation

```
A > B :  G = A · B'
A = B :  E = A ⊙ B = A·B + A'·B'   (XNOR)
A < B :  L = A' · B
```

A 1-bit comparator needs one XNOR gate, two inverters and two AND gates.

### Extending to 4 Bits

```
Gi = Ai · Bi'          (Ai greater than Bi)
Li = Ai' · Bi          (Ai less than Bi)
Ei = Ai ⊙ Bi           (Ai equal to Bi)

(A = B) = E3 · E2 · E1 · E0
(A > B) = G3 + E3·G2 + E3·E2·G1 + E3·E2·E1·G0
(A < B) = L3 + E3·L2 + E3·E2·L1 + E3·E2·E1·L0
```

### Sample 4-Bit Cases

| A | B | Decided at | A>B | A=B | A<B |
|---|---|-----------|-----|-----|-----|
| 0000 | 0000 | all bits equal | 0 | 1 | 0 |
| 1000 | 0111 | bit 3 (G3 = 1) | 1 | 0 | 0 |
| 0111 | 1000 | bit 3 (L3 = 1) | 0 | 0 | 1 |
| 1100 | 1010 | bit 2 (E3 = 1, G2 = 1) | 1 | 0 | 0 |
| 0101 | 0110 | bit 1 (E3E2 = 1, L1 = 1) | 0 | 0 | 1 |
| 1011 | 1010 | bit 0 (E3E2E1 = 1, G0 = 1) | 1 | 0 | 0 |
| 1111 | 1111 | all bits equal | 0 | 1 | 0 |

The worst-case path runs through XNOR → AND chain → OR combining.

### CMOS Logic Fundamentals

| Network | Device | Conducts when | Connected to |
|---------|--------|---------------|--------------|
| Pull-up (PUN) | PMOS | input = 0 | VDD |
| Pull-down (PDN) | NMOS | input = 1 | GND |

- **Inverter:** one PMOS and one NMOS. Output = A'.
- **NAND2:** NMOS in series, PMOS in parallel. Output = (A·B)'.
- **NOR2:** NMOS in parallel, PMOS in series. Output = (A+B)'.
- **AND2 / OR2:** a NAND2 / NOR2 followed by an inverter.
- **XNOR2:** used here for bit-equality.

### Number Representation and Comparison

The comparator works on **unsigned binary numbers**. A 4-bit number `A3 A2 A1 A0` has the value

```
A = A3·2³ + A2·2² + A1·2¹ + A0·2⁰
```

so a 4-bit input covers 0 to 15, and the pair (A, B) has 16 × 16 = **256 input combinations**. Of these, 16 give A = B, 120 give A > B and 120 give A < B. Signed (two's complement) numbers need an extra rule for the sign bit, which is outside the scope of this project.

Because the weight of each bit is twice that of the bit below it, the MSB outweighs all lower bits together (for example 1000₂ = 8 is greater than 0111₂ = 7). This is why the comparison can stop at the first differing bit starting from the MSB.

### Methods of Building a Comparator

| Method | Idea | Remark |
|--------|------|--------|
| **Subtraction** | Compute A − B and look at the borrow/sign and zero flags | Needs a full subtractor chain, so it is slower and larger |
| **XOR / XNOR based (used here)** | Compare bit by bit with equality and greater/less terms, then combine from the MSB | Simple, regular and easy to map onto reusable cells |
| **Multiplexer based** | Select the result of the highest differing bit | Useful when mux cells are available |
| **Cascaded (7485 style)** | Chain 4-bit blocks using cascade inputs | Scales to 8, 12, 16 bits |

This project uses the XNOR-based method because it maps directly onto the INV, AND2, OR2 and XNOR2 cells.

### Deriving the 1-Bit Comparator

For two single bits, the three outputs are the minterms of the truth table:

```
A > B  → only the row A=1, B=0   →  G = A · B'
A < B  → only the row A=0, B=1   →  L = A' · B
A = B  → rows 00 and 11          →  E = A'·B' + A·B = A ⊙ B
```

The equality output is an **XNOR** because XNOR is 1 exactly when both inputs are the same. Note also that `E = (G + L)'`, which is a useful check: the three outputs can never be 1 together.

### Deriving the 4-Bit Equations

The 4-bit result is found by scanning from the MSB downwards:

1. **A > B** if bit 3 of A is greater (`G3`), **or** bit 3 is equal and bit 2 is greater (`E3·G2`), **or** bits 3 and 2 are equal and bit 1 is greater (`E3·E2·G1`), **or** bits 3, 2 and 1 are equal and bit 0 is greater (`E3·E2·E1·G0`).
2. **A < B** follows the same pattern with `L` in place of `G`.
3. **A = B** only when every bit pair is equal: `E3·E2·E1·E0`.

This is a **sum of products** with equality terms acting as "enable" signals for the lower bits. Because the three conditions are mutually exclusive, no two outputs can be high at once. A quick check: for A = 0101 and B = 0110, E3 = 1, E2 = 1, G1 = 0, L1 = 1, so `A<B = E3·E2·L1 = 1` and the other two outputs are 0.

### Gate Count of the Design

| Block | Gates | Transistors |
|-------|-------|-------------|
| INV | 1 | 2 |
| AND2 (NAND2 + INV) | 2 | 6 |
| OR2 (NOR2 + INV) | 2 | 6 |
| XNOR2 (2 INV + 8 transistors) | 3 | 12 |
| 1-bit comparator (1 XNOR2 + 2 INV + 2 AND2) | 8 | 28 |

### How a MOSFET Works

A MOSFET is a voltage-controlled switch with four terminals: gate, drain, source and body.

- **NMOS:** when the gate-to-source voltage `Vgs` is above the threshold voltage `Vtn`, an electron channel forms and the device conducts. It passes a strong logic 0 and a weak logic 1.
- **PMOS:** when `Vgs` is below `Vtp` (that is, the gate is low relative to the source), a hole channel forms. It passes a strong logic 1 and a weak logic 0.
- **Regions of operation:** *cut-off* (`Vgs < Vt`, no current), *linear/triode* (`Vds < Vgs − Vt`, the device acts like a resistor) and *saturation* (`Vds ≥ Vgs − Vt`, current is nearly independent of `Vds`).

The drain current in saturation (long-channel approximation) is

```
Id = (1/2) · µ · Cox · (W/L) · (Vgs − Vt)²
```

so current, and therefore speed, grows with the ratio **W/L**. Increasing the width `W` makes the transistor stronger (lower resistance) but also increases its gate capacitance and area. Short-channel effects such as velocity saturation make real 180 nm devices deviate from the square law, but the trend remains the same.

Because NMOS is good at pulling down and PMOS at pulling up, CMOS uses NMOS only in the pull-down network and PMOS only in the pull-up network, so the output always swings fully between 0 and VDD.

### The CMOS Inverter in Detail

The inverter has one PMOS (to VDD) and one NMOS (to GND) with joined gates and drains.

| Input | PMOS | NMOS | Output |
|-------|------|------|--------|
| 0 (low) | ON | OFF | 1 (VDD) |
| 1 (high) | OFF | ON | 0 (GND) |

- In either steady state one transistor is off, so there is **no direct DC path from VDD to GND** and static power is almost zero (only leakage).
- The **voltage transfer characteristic (VTC)** is very steep around `VDD/2`, which gives a high gain and large noise margins.
- The **switching threshold** `VM` (where Vin = Vout) is near `VDD/2` only if the PMOS is about twice as wide as the NMOS (to compensate for lower hole mobility). With equal widths, as in this project, `VM` shifts slightly and the rise and fall delays are not equal.
- During switching there is a brief period when both devices conduct, causing **short-circuit power**.

### Transistor-Level Gates

**NAND2:** two NMOS in series (pull-down) and two PMOS in parallel (pull-up). The output is low only when both inputs are high.

**NOR2:** two NMOS in parallel and two PMOS in series. The output is high only when both inputs are low.

| A | B | NAND2 | NOR2 |
|---|---|-------|------|
| 0 | 0 | 1 | 1 |
| 0 | 1 | 1 | 0 |
| 1 | 0 | 1 | 0 |
| 1 | 1 | 0 | 0 |

**AND2 and OR2:** a plain CMOS stage is *inverting*, so an AND gate is built as NAND followed by an inverter, and an OR gate as NOR followed by an inverter. This costs 6 transistors instead of 4, but the inverter also acts as an output buffer that drives the next stage.

**XNOR2:** the function `Z = A·B + A'·B'` is not a single inverting stage, so the inputs are first inverted (2 inverters generate A' and B') and then combined by a network of 8 transistors: the pull-up and pull-down networks are built so that the output is connected to VDD when A = B and to GND when A ≠ B.

**Duality rule:** the pull-up network is the dual of the pull-down network. Where the NMOS devices are in series, the PMOS devices are in parallel, and vice versa. This guarantees that exactly one network conducts for every input combination.

### Delay Model

A gate can be approximated by an equivalent resistance `R` driving a load capacitance `C`. The propagation delay is then

```
tpd ≈ 0.69 · R_eq · C_load
```

- `R_eq` depends on the transistor width (`R ∝ L/W`) and on how many devices are in series (series stacks add resistance).
- `C_load` is the sum of the gate capacitances of the following gates, the drain diffusion capacitance of the gate itself, and the wire capacitance.
- Rise and fall delays are different when `Wp = Wn`, because the PMOS is weaker.

For this comparator the **critical path** is bit 0 → XNOR2 → AND2 chain (E3·E2·E1·G0) → OR2. It is the longest because the least significant bit decision must wait for all the higher equality signals, so the delay grows with the number of bits.

### Power Dissipation

Total power has three parts:

```
P_total = P_dynamic + P_short-circuit + P_leakage
P_dynamic = α · C_L · VDD² · f
```

- **Dynamic power:** energy spent charging and discharging load capacitances. It is proportional to `VDD²`, so lowering the supply voltage is the most effective way to save power.
- **Short-circuit power:** current flowing while both networks conduct briefly during a transition.
- **Leakage power:** subthreshold and gate leakage when the circuit is idle. It is small at 180 nm but grows at smaller nodes.

Here `α` is the switching activity (the fraction of clock cycles in which the node toggles) and `C_L` is the load capacitance.

### Sizing and Series Stacks

- A series stack of `n` transistors has about `n` times the resistance, so each device in the stack is often made wider to keep the delay balanced. A NAND2 uses NMOS of width `2Wn`, a NOR2 uses PMOS of width `2Wp`.
- Since PMOS mobility is roughly 2 to 3 times lower, `Wp ≈ 2Wn` is the usual choice for equal rise and fall times.
- In this project every transistor uses the **same size (W = 2 µm, L = 180 nm)** for uniformity and simpler layout, so the design is functionally correct but not delay-optimised. Sizing optimisation is listed under future work.

### Layout Fundamentals

A layout is the set of mask polygons that will be manufactured. For a CMOS gate the typical layers are:

| Layer | Purpose |
|-------|---------|
| N-well | Holds the PMOS transistors |
| Active (oxide) | Defines transistor source/drain regions |
| Poly | Transistor gates and short local wires |
| n+ / p+ implants | Dope source/drain of NMOS / PMOS and make substrate/well contacts |
| Contact | Connects metal to active or poly |
| Metal1 – Metal6 | Wires; Metal1 for local routing, higher layers for global routing |
| Via | Connects one metal layer to the next |

Key points:

- **Standard cell style:** VDD runs along the top, GND along the bottom, PMOS sits in the N-well near VDD and NMOS near GND, with the poly gates running vertically across both.
- **Stick diagrams** are a simple sketch of the layout showing layers as coloured lines, used to plan the layout before drawing polygons.
- **Latch-up** is a parasitic PNPN thyristor path between VDD and GND. It is prevented by placing well/substrate taps (N-well tied to VDD, substrate tied to GND) close to the transistors.
- **Matching and symmetry** matter mostly in analog blocks, but a compact and regular layout also reduces parasitic capacitance in digital cells.

### Physical Verification

| Check | What it verifies | Status in this project |
|-------|------------------|------------------------|
| **DRC** | Layout obeys minimum width, spacing, enclosure and extension rules | Done, clean for all layouts |
| **LVS** | Layout netlist equals the schematic netlist | Future work |
| **PEX** | Extracts parasitic R and C for post-layout simulation | Future work |

A design that passes DRC can still be electrically wrong (for example wrongly connected), which is why LVS is normally run in addition to DRC.

### Simulation Concepts

- **Transient analysis** computes the output voltages as a function of time for given input waveforms. It is used to verify logic function and measure delay.
- **Pulse sources** (`vpulse`) with different periods are applied to the inputs so that every combination appears in turn. For binary-weighted periods (for example 10 ns, 20 ns, 40 ns, 80 ns for inputs 0 to 3), the inputs count through all 16 values.
- **Propagation delay** is measured from the 50% point of the input edge to the 50% point of the output edge.
- **Rise and fall times** are normally measured between 10% and 90% of the supply voltage.
- **Glitches:** when inputs change at nearly the same time, the outputs can briefly show a wrong value because of unequal path delays. The final settled values must still match the truth table.

### Performance Metrics

- **Propagation delay (tpd):** 50% of input transition to 50% of output transition.
- **Power:** Pdyn = α·C·VDD²·f plus leakage.
- **Noise margin:** NMH = VOH − VIH, NML = VIL − VOL.
- **Power-Delay Product (PDP):** P × tpd.

### Full-Custom vs Semi-Custom Design

| Aspect | Full-Custom (this project) | Semi-Custom (standard cell) |
|--------|----------------------------|-----------------------------|
| Design effort | High | Low |
| Area | Optimized by hand | Larger |
| Performance | Best control over speed and power | Good but less tuned |
| Typical use | Analog, memories, critical datapaths | Large digital blocks |

### Advantages and Limitations of This Design

| Advantages | Limitations |
|------------|-------------|
| Regular structure built from only four cells | All transistors are the same size, so rise and fall times are unequal |
| Easy to understand, verify and extend | AND2/OR2 use extra inverter stages, so more area and delay |
| Full-swing outputs and very low static power | The carry-chain style AND logic makes delay grow with bit width |
| Complete DRC-clean layout | No LVS or post-layout simulation yet |

### Comparison with the 74HC85 / 7485

The 7485 is a commercial 4-bit comparator with cascade inputs (`I A>B`, `I A=B`, `I A<B`). When all four bit pairs are equal, the result is taken from the cascade inputs, which come from the lower-order comparator. This makes it possible to build 8-, 12- and 16-bit comparators from 4-bit blocks. The design in this project has no cascade inputs, so it compares exactly 4 bits, but adding them is listed under future enhancements.

### Why 180 nm (gpdk180)?

gpdk180 is a generic 180 nm process design kit widely used in academia, with VDD = 1.8 V, a manageable number of layers and well-documented rules.

---

## 🧱 Hierarchical Cells

| Level | Cell | Description |
|-------|------|-------------|
| 1 | INV | Basic CMOS inverter (1 PMOS + 1 NMOS) |
| 2 | AND2 | NAND2 followed by an inverter |
| 2 | OR2 | NOR2 followed by an inverter |
| 2 | XNOR2 | Bit-equality cell (A ⊙ B) |
| 3 | 1-bit comparator | One XNOR2, two INV, two AND2 |
| 4 | 4-bit comparator | Four 1-bit stages plus AND2/OR2 combining logic |

### Transistor Sizing

| Device | Library cell | W / L |
|--------|--------------|-------|
| NMOS | nmos1 | 2 µm / 180 nm |
| PMOS | pmos1 | 2 µm / 180 nm |

---

## 📊 Results

### Physical Verification

- ✅ **DRC status**: Clean ("No DRC errors found" for INV, AND2, OR2, XNOR2, 1-bit and 4-bit comparator layouts)

---

## 🖼 Visual Gallery

### 1. Basic CMOS Gate Schematics

#### Inverter
![Inverter Schematic](schematics%20and%20waveform/inverterschematic.jpeg)

#### AND Gate
![AND Schematic](schematics%20and%20waveform/and%20schematic.jpeg)

#### OR Gate
![OR Schematic](schematics%20and%20waveform/or%20schematic.jpeg)

#### XNOR Gate
![XNOR Schematic](schematics%20and%20waveform/xnorschematic.jpeg)

---

### 2. Comparator Schematics and Simulation

#### 1-Bit Comparator Schematic
![1-Bit Comparator Schematic](schematics%20and%20waveform/1bit%20comparatorschematic.jpeg)

#### 1-Bit Comparator Waveform
![1-Bit Comparator Waveform](schematics%20and%20waveform/1bit%20comparatorwaveform.jpeg)

#### 4-Bit Comparator Schematic
![4-Bit Comparator Schematic](schematics%20and%20waveform/4bit%20comparator%20using%201bit.jpeg)

#### 4-Bit Comparator Waveform
![4-Bit Comparator Waveform](schematics%20and%20waveform/4bit%20comparator%20waveform.jpeg)

---

### 3. Layout of Basic CMOS Gates

#### Inverter Layout
![Inverter Layout](layout%20and%20drs%20check/inv_layout.png)

#### AND Gate Layout
![AND Layout](layout%20and%20drs%20check/and%20layout.png)

#### OR Gate Layout
![OR Layout](layout%20and%20drs%20check/or_layout.png)

#### XNOR Gate Layout
![XNOR Layout](layout%20and%20drs%20check/xnor_layout.png)

---

### 4. Comparator Layouts

#### 1-Bit Comparator Layout
![1-Bit Comparator Layout](layout%20and%20drs%20check/1bitcomparator_layout.png)

#### 4-Bit Comparator Layout
![4-Bit Comparator Layout](layout%20and%20drs%20check/4_bitcomparatorlayout.png)

---

### 5. Design Rule Check (DRC)

#### Inverter DRC
![Inverter DRC](layout%20and%20drs%20check/inv_drs.png)

#### AND Gate DRC
![AND DRC](layout%20and%20drs%20check/and_drs.png)

#### OR Gate DRC
![OR DRC](layout%20and%20drs%20check/or_drs.png)

#### XNOR Gate DRC
![XNOR DRC](layout%20and%20drs%20check/xnor_drs.png)

#### 1-Bit Comparator DRC
![1-Bit Comparator DRC](layout%20and%20drs%20check/1_bitcomparator_drs.png)

#### 4-Bit Comparator DRC
![4-Bit Comparator DRC](layout%20and%20drs%20check/4bitcomparator_drs.png)

---

## 🚀 Getting Started

### Prerequisites

```
- Cadence Virtuoso (Schematic Editor, Layout Editor, ADE)
- Cadence Spectre (circuit simulator)
- DRC deck for gpdk180
- gpdk180 (180 nm generic PDK)
```

### Steps to Reproduce

1. **Create library** attached to the gpdk180 technology library
2. **Design basic cells**: INV, AND2, OR2, XNOR2 (schematic → symbol → simulate)
3. **Build the comparator schematic** hierarchically using the basic cells
4. **Simulate** in ADE with pulse sources on A[3:0] and B[3:0], VDD = 1.8 V
5. **Draw layout** of each cell, then place and route the top level
6. **Run DRC** and fix all violations

---

## 🔬 Technical Specifications

| Parameter | Value |
|-----------|-------|
| Design Style | Full-custom |
| Input Width | 4-bit (A, B) |
| Outputs | A>B, A=B, A<B |
| Cell Library | INV, AND2, OR2, XNOR2 |
| Technology | gpdk180 (180 nm CMOS) |
| Supply Voltage | 1.8 V |
| Transistor Size | W = 2 µm, L = 180 nm |
| Tools | Cadence Virtuoso, Spectre |

---

## 🎓 Academic Context

- **Course**: VLSI System Design Practice (EC-307)
- **Faculty**: Dr. P. Ranga Babu
- **Department**: Electronics and Communication Engineering
- **Institution**: Indian Institute of Information Technology Design and Manufacturing, Kurnool
- **Academic Year**: 2026-2027 (Semester 5)

---

## 🚀 Future Enhancements

- [ ] Cascadable inputs (I_A>B, I_A=B, I_A<B) like the 7485
- [ ] Extension to 8-bit and 16-bit comparators
- [ ] LVS and post-layout (PEX) simulation
- [ ] Migration to 90 nm / 45 nm technology
- [ ] Delay and power optimization through transistor sizing (Wp ≈ 2Wn)

---

## 📚 References

1. M. Morris Mano, *Digital Design*, 5th Edition, Pearson, 2013.
2. N. H. E. Weste and D. M. Harris, *CMOS VLSI Design: A Circuits and Systems Perspective*, 4th ed., Addison-Wesley, 2011.
3. J. Rabaey, A. Chandrakasan, and B. Nikolic, *Digital Integrated Circuits: A Design Perspective*, 2nd Edition, Prentice Hall, 2003.
4. Cadence Virtuoso User Guide and gpdk180 PDK documentation.

---

## 📝 License

This project is licensed under the MIT License. See the `LICENSE` file for details.

---

## 📬 Contact

**Lathika**

- 📧 **Email**: 523ec0005@iiitk.ac.in
- 🐱 **GitHub**: [github.com/Lathika-2006](https://github.com/Lathika-2006)
- 🎓 **Institution**: IIITDM Kurnool

---

## 🌟 Acknowledgments

- **Dr. P. Ranga Babu**: Course Instructor and Project Guide, Dept. of ECE, IIITDM Kurnool
- **IIITDM Kurnool**: For computational resources and infrastructure
- **Cadence Design Systems**: For access to industry-standard EDA tools

---

## ⭐ Star this repository if you found it helpful!

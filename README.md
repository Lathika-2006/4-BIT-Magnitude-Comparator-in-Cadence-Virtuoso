# 4-Bit Magnitude Comparator: Full-Custom Design in Cadence Virtuoso
*180 nm CMOS (gpdk180) • Schematic • Simulation • Layout*

A complete full-custom implementation of a 4-bit magnitude comparator, built hierarchically from hand-drawn transistor-level cells (INV, AND2, OR2, XNOR2).

*Overview* • *Architecture* • *Results* • *Layout* • *Getting Started* • *Documentation*

---

## 🎯 Overview

A magnitude comparator compares two binary numbers **A[3:0]** and **B[3:0]** and tells whether **A > B**, **A = B** or **A < B**. It is a core block in ALUs, sorting networks, address decoders, and control logic in processors.

This project implements the comparator **full-custom**: every gate is drawn at the transistor (NMOS/PMOS) level, simulated, and laid out manually in Cadence Virtuoso on the 180 nm gpdk180 process.

---

## ✨ Key Highlights

🔌 **Full-Custom Design**: Transistor-level schematic and hand-drawn layout  
🧩 **Hierarchical Cell Library**: INV, AND2, OR2 and XNOR2 cells reused to build the 1-bit and 4-bit comparators  
🔬 **Verified Functionality**: Exhaustive input-pattern simulation in Spectre/ADE  
📐 **Complete Physical Flow**: Schematic → Simulation → Layout → DRC → LVS  
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

The equations are mapped onto a small set of reusable 2-input cells:

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
 LAYOUT  (Virtuoso Layout XL / Layout Editor)
        │
        ▼
 DRC → LVS → (optional) PEX post-layout simulation
```

---

## 🧠 Theoretical Background

### What Is a Magnitude Comparator?

A magnitude comparator is a combinational circuit that compares two n-bit unsigned binary numbers and produces three outputs: **A > B**, **A = B** and **A < B**. Exactly one of the three outputs is 1 for any input combination, so

```
(A>B) + (A=B) + (A<B) = 1   (mutually exclusive and exhaustive)
```

Since it is purely combinational, the output depends only on the present inputs (no clock or memory).

### How Binary Comparison Works

Two numbers are compared bit by bit starting from the **most significant bit (MSB)**, just like comparing decimal numbers digit by digit:

1. If A3 ≠ B3, the number with the 1 in that position is larger, and the result is decided.
2. If A3 = B3, move to the next bit (A2 vs B2), and so on.
3. If all four bits are equal, then A = B.

**Example:** A = 1010 (10), B = 1001 (9). A3 = B3 = 1, A2 = B2 = 0, A1 = 1 but B1 = 0, so **A > B**.

### 1-Bit Comparator Derivation

From the single-bit truth table (see Architecture):

```
A > B :  G = A · B'
A = B :  E = A ⊙ B = A·B + A'·B'   (XNOR)
A < B :  L = A' · B
```

Each output is a minterm or a simple function of A and B, so a 1-bit comparator needs one XNOR gate, two inverters and two AND gates.

### Extending to 4 Bits

For each bit i, define:

```
Gi = Ai · Bi'          (Ai greater than Bi)
Li = Ai' · Bi          (Ai less than Bi)
Ei = Ai ⊙ Bi           (Ai equal to Bi)
```

Then the 4-bit outputs are:

```
(A = B) = E3 · E2 · E1 · E0
(A > B) = G3 + E3·G2 + E3·E2·G1 + E3·E2·E1·G0
(A < B) = L3 + E3·L2 + E3·E2·L1 + E3·E2·E1·L0
```

The term E3·G2 means "MSBs are equal AND bit 2 of A is greater". Each lower bit only matters if all higher bits are equal, which is why the E terms are chained (priority by significance).

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

The last bit-level case shows the worst-case path: the signal must pass through the full chain of equality terms, so the critical delay comes from XNOR → AND chain → OR combining.

### Cascading and Scalability

Commercial comparators like the 74HC85 add cascade inputs (I A>B, I A=B, I A<B) so that several 4-bit blocks can be chained to compare 8, 12 or 16 bits. The lower block's outputs feed the higher block's cascade inputs, and they are used only when all four bits of the higher block are equal.

### CMOS Logic Fundamentals

All cells in this design are **static CMOS** gates made of a complementary pair of networks:

| Network | Device | Conducts when | Connected to |
|---------|--------|---------------|--------------|
| Pull-up (PUN) | PMOS | input = 0 | VDD |
| Pull-down (PDN) | NMOS | input = 1 | GND |

- **Inverter:** one PMOS and one NMOS. Output = A'.
- **NAND2:** NMOS in series, PMOS in parallel. Output = (A·B)'.
- **NOR2:** NMOS in parallel, PMOS in series. Output = (A+B)'.
- **AND2 / OR2:** a NAND2 / NOR2 followed by an inverter. This adds a stage but gives a full-swing, buffered output.
- **XNOR2:** built from transistor-level networks or from basic gates, used here for bit-equality.

Series transistors have higher resistance, so series devices are made wider. A PMOS has about half the hole mobility of an NMOS electron mobility, so **Wp is usually about 2× Wn** for balanced rise and fall times.

### Performance Metrics

- **Propagation delay (tpd):** time from 50% of the input transition to 50% of the output transition. Worst case is measured on the longest path (here through the equality chain).
- **Power:** Pdyn = α·C·VDD²·f (switching) plus Pleak (leakage, small at 180 nm). Total = Pdyn + Pstatic.
- **Noise margin:** NMH = VOH − VIH, NML = VIL − VOL. Static CMOS has near rail-to-rail output swing, so noise margins are large.
- **Power-Delay Product (PDP):** P × tpd, a figure of merit for energy per operation.

### Full-Custom vs Semi-Custom Design

| Aspect | Full-Custom (this project) | Semi-Custom (standard cell) |
|--------|----------------------------|-----------------------------|
| Design effort | High (every transistor and polygon drawn manually) | Low (automated synthesis, P&R) |
| Area | Smallest, optimized by hand | Larger |
| Performance | Best control over speed and power | Good but less tuned |
| Flexibility | Complete control over sizing and layout | Limited to library cells |
| Typical use | Analog, memories, critical datapaths | Large digital blocks |

### Layout and Physical Verification Concepts

- **Layers (gpdk180):** N-well, active (oxide), poly (gate), n+/p+ implants, contacts, Metal1 to Metal6, and vias between metals.
- **Design rules:** minimum width, spacing, enclosure and extension rules for each layer, set by the foundry. Gate length for this process is 180 nm.
- **Well and substrate contacts:** N-well tied to VDD and p-substrate tied to GND to avoid latch-up and keep the transistors properly biased.
- **Routing:** Metal1 for local connections inside cells, Metal2 and above for inter-cell routing; poly is used for gates and short local connections.
- **DRC (Design Rule Check):** verifies the layout obeys all geometric rules.
- **LVS (Layout vs Schematic):** extracts devices and connections from the layout and checks that they match the schematic netlist.
- **PEX (Parasitic Extraction):** extracts resistance and capacitance from the layout for realistic post-layout simulation.

### Why 180 nm (gpdk180)?

gpdk180 is a generic 180 nm process design kit widely used in academia. It is simple, with VDD = 1.8 V, a manageable number of layers, and well-documented rules, so it is ideal for learning the complete full-custom flow from schematic to layout.

### Applications

- Arithmetic Logic Units (ALU) for compare and branch operations
- Sorting and ranking hardware
- Address and tag comparison in caches and memories
- Digital thresholding in ADCs and control systems
- Priority encoders and decision logic

---

## 🧱 Hierarchical Cells

Each building block is designed as its own schematic + symbol + layout and then reused hierarchically.

| Level | Cell | Description |
|-------|------|-------------|
| 1 | INV | Basic CMOS inverter (1 PMOS + 1 NMOS) |
| 2 | AND2 | 2-input AND |
| 2 | OR2 | 2-input OR |
| 2 | XNOR2 | Bit-equality cell (A ⊙ B) |
| 3 | 1-bit comparator | Compares one bit of A and B |
| 4 | 4-bit comparator | Four 1-bit stages combined, outputs A>B, A=B, A<B |

### Transistor Sizing

| Device | W/L | Notes |
|--------|-----|-------|
| NMOS | [W_n] / 180 nm | [fill in] |
| PMOS | [W_p] / 180 nm | [fill in, typically ~2× NMOS] |

---

## 📊 Results

### Functional Simulation

Simulation was run for all input combinations (or a representative exhaustive set) with **VDD = 1.8 V**.

| Test Case | A | B | A > B | A = B | A < B | Result |
|-----------|---|---|-------|-------|-------|--------|
| 1 | 0000 | 0000 | 0 | 1 | 0 | ✅ PASS |
| 2 | 0101 | 0011 | 1 | 0 | 0 | ✅ PASS |
| 3 | 0011 | 0101 | 0 | 0 | 1 | ✅ PASS |
| 4 | 1111 | 1111 | 0 | 1 | 0 | ✅ PASS |
| 5 | 1000 | 0111 | 1 | 0 | 0 | ✅ PASS |
| 6 | 0111 | 1000 | 0 | 0 | 1 | ✅ PASS |

### Performance Summary

| Metric | Value | Unit |
|--------|-------|------|
| Technology | gpdk180 (180 nm) | — |
| Supply Voltage | 1.8 | V |
| Propagation Delay (worst case) | [value] | ns |
| Average Power | [value] | μW |
| Total Transistor Count | [value] | — |
| Layout Area | [value] | μm² |

### Physical Verification

- ✅ **DRC status**: Clean ("No DRC errors found" for INV, AND2, OR2, XNOR2, 1-bit and 4-bit comparator layouts)
- ✅ **LVS status**: [Clean, confirm and add screenshot]

---

## 🖼 Visual Gallery

### Layout of Basic Cells

#### Inverter
![Inverter Layout](images/invlayout.png)


Inverter layout in gpdk180 with VDD/GND rails, in/out pins.

#### AND2 Gate
![AND2 Layout](images/and layout.png)


#### OR2 Gate
![OR2 Layout](images/or_layout.png)


#### XNOR2 Gate
![XNOR2 Layout](images/xnor_layout.png)


---

### 1-Bit Comparator Layout
![1-bit Comparator Layout](images/1bitcomparator_layout.png)


Placement of the XNOR, AND, OR and INV cells for one comparator stage, routed with Metal1/Metal2.

---

### 4-Bit Comparator Layout
![4-bit Comparator Layout](images/4_bitcomparatorlayout.png)


Full 4-bit comparator layout with cell instances placed and routed.

---

### 4-Bit Comparator Schematic
![4-bit Comparator Schematic](images/4bit comparator using 1bit.jpeg)

Top-level schematic in Cadence Virtuoso with inputs A[3:0], B[3:0] and outputs A>B, A=B, A<B.

### Simulation Waveforms
![Simulation Waveforms](images/4bit comparator waveform.jpeg)

Transient simulation (VDD = 1.8 V) showing the input patterns on A[3:0] and B[3:0] and the three outputs. Only one output is high at a time, matching the truth table.

---

## 🚀 Getting Started

### Prerequisites

```
# Required EDA Tools
- Cadence Virtuoso (Schematic Editor, Layout Editor, ADE)
- Cadence Spectre (circuit simulator)
- Cadence Assura / Pegasus / Calibre (DRC & LVS)

# Technology Library
- gpdk180 (180 nm generic PDK)
```

### Steps to Reproduce

1. **Create library** attached to the gpdk180 technology library
2. **Design basic cells**: INV, AND2, OR2, XNOR2 (schematic → symbol → simulate)
3. **Build the comparator schematic** hierarchically using the basic cells
4. **Simulate** in ADE with pulse/bit sources on A[3:0] and B[3:0], VDD = 1.8 V
5. **Draw layout** of each cell, then place and route the top level
6. **Run DRC** and fix all violations
7. **Run LVS** and match layout against schematic
8. *(Optional)* Extract parasitics (PEX) and re-simulate

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
| Tools | Cadence Virtuoso, Spectre |

---

## 🎓 Academic Context

- **Course**: VLSI System Design Practice (EC-307)
- **Faculty**: Dr. P. Ranga Babu
- **Department**: Electronics and Communication Engineering
- **Institution**: Indian Institute of Information Technology Design and Manufacturing, Kurnool
- **Academic Year**: 2025-2026 (Semester 5)

### Learning Outcomes

✅ Transistor-level design of CMOS logic gates  
✅ Hierarchical schematic and symbol creation in Virtuoso  
✅ Simulation and functional verification using Spectre  
✅ Full-custom layout with DRC and LVS clean-up  
✅ Understanding of area, delay and power trade-offs at 180 nm  

---

## 🚀 Future Enhancements

- [ ] Cascadable inputs (I_A>B, I_A=B, I_A<B) like the 7485
- [ ] Extension to 8-bit and 16-bit comparators
- [ ] Post-layout (PEX) simulation and delay comparison
- [ ] Migration to 90 nm / 45 nm technology
- [ ] Delay and power optimization through transistor sizing

---

## 📚 References

1. M. Morris Mano, *Digital Design*, 5th Edition, Pearson, 2013.
2. N. H. E. Weste and D. M. Harris, *CMOS VLSI Design: A Circuits and Systems Perspective*, 4th ed., Addison-Wesley, 2011.
3. J. Rabaey, A. Chandrakasan, and B. Nikolic, *Digital Integrated Circuits: A Design Perspective*, 2nd Edition, Prentice Hall, 2003.
4. Cadence Virtuoso User Guide and gpdk180 PDK documentation.

---

## 🛠 Tools & Technologies

| Category | Tools |
|----------|-------|
| Schematic & Layout | Cadence Virtuoso |
| Simulation | Cadence Spectre / ADE |
| Verification | DRC, LVS |
| Technology | gpdk180 (180 nm) |

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

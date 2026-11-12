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

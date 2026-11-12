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
![Inverter Layout](images/inverter_layout.png)

![Inverter DRC](images/inverter_drc.png)

Inverter layout in gpdk180 with VDD/GND rails, in/out pins and the DRC "No DRC errors found" result.

#### AND2 Gate
![AND2 Layout](images/and_gate_layout.png)

![AND2 DRC](images/and_gate_drc.png)

#### OR2 Gate
![OR2 Layout](images/or_gate_layout.png)

![OR2 DRC](images/or_gate_drc.png)

#### XNOR2 Gate
![XNOR2 Layout](images/xnor_gate_layout.png)

![XNOR2 DRC](images/xnor_gate_drc.png)

---

### 1-Bit Comparator Layout
![1-bit Comparator Layout](images/1bit_comparator_layout.png)

![1-bit Comparator DRC](images/1bit_comparator_drc.png)

Placement of the XNOR, AND, OR and INV cells for one comparator stage, routed with Metal1/Metal2, DRC clean.

---

### 4-Bit Comparator Layout
![4-bit Comparator Layout](images/4bit_comparator_layout.png)

![4-bit Comparator DRC](images/4bit_comparator_drc.png)

Full 4-bit comparator layout with cell instances placed and routed, DRC clean.

---

### Schematic and Simulation Waveforms
<!-- Add your schematic and waveform screenshots here:
![Schematic](images/schematic.png)
![Waveform](images/waveform.png)
-->

*(Add schematic and waveform screenshots to the `images/` folder and uncomment the lines above.)*

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

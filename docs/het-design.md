---
layout: xdoubledot
title: HET Hardware Design
---


# Hall Effect Thruster — Hardware Design Reference
## 200W Krypton, Magnetically Shielded, External Cathode

*Status: Preliminary Design — for 3D modelling and bench prototype*
*Companion documents: PPU_DESIGN.md, EXPLAINER.md, solve_het_bfield.py*

---

## Table of Contents

1. [Design Rationale and Key Choices](#1-design-rationale-and-key-choices)
2. [Overall Dimensions](#2-overall-dimensions)
3. [Cross-Section Sketches](#3-cross-section-sketches)
4. [Magnetic Circuit — Iron Structure](#4-magnetic-circuit--iron-structure)
5. [Discharge Channel — BN Ceramic Walls](#5-discharge-channel--bn-ceramic-walls)
6. [Electromagnetic Coils](#6-electromagnetic-coils)
7. [Anode and Propellant Distributor](#7-anode-and-propellant-distributor)
8. [Cathode Insulator](#8-cathode-insulator)
9. [Housing, Flange, and Mounting](#9-housing-flange-and-mounting)
10. [Hollow Cathode Assembly](#10-hollow-cathode-assembly)
11. [Propellant Feed System](#11-propellant-feed-system)
12. [Materials Specification](#12-materials-specification)
13. [Manufacturing Notes](#13-manufacturing-notes)
14. [Assembly Sequence](#14-assembly-sequence)
15. [Operating Parameters and Predicted Performance](#15-operating-parameters-and-predicted-performance)
16. [Dimensions Reference Table](#16-dimensions-reference-table)

---

## 1. Design Rationale and Key Choices

### Power class and propellant

200 W electrical input, Krypton propellant. Kr requires a slightly larger channel volume than Xe at the same power level — Kr's higher first ionisation energy (14.0 eV vs 12.1 eV) means the ionisation zone is longer and neutrals penetrate deeper before ionising. Channel mean diameter is scaled up ~15% relative to a Xe thruster of equivalent power.

### Magnetic shielding geometry

The design follows the magnetically shielded (MS) philosophy of Hofer et al. (2014) / JPL H6MS. The distinguishing feature is that the magnetic field lines at the channel exit are shaped to run **nearly parallel to the BN ceramic walls** rather than perpendicular to them. This raises the plasma potential at the wall surface, reducing the sheath potential drop, reducing the energy of ions hitting the walls, and therefore dramatically reducing wall sputtering.

This is achieved by:
1. Chamfering the inner and outer pole piece tips to guide field lines tangentially along the wall surfaces
2. Locating the peak radial B-field at or slightly downstream of the channel exit plane (not inside the channel)
3. Ensuring the field lines close back through the iron circuit without crossing the BN walls at high angles

The `solve_het_bfield.py` FD solver was used to verify the field line topology for this geometry.

### External cathode

The cathode is mounted externally (off-axis, to one side) rather than centrally. The RL simulation demonstrated this gives 85% cathode flux reduction vs 63% for a centre-mounted cathode. The external position gives the trim coil more magnetic leverage to deflect back-streaming ions away from the cathode.

### Three-coil magnetic circuit

- **Inner solenoid**: sets the field strength near the inner channel wall and inner pole tip
- **Outer solenoid**: sets the field strength near the outer channel wall and outer pole tip (opposite winding to inner — both drive flux in the same direction at the channel gap)
- **Trim coil**: sits in the back yoke between the inner and outer circuits; primarily controls the axial field gradient at the channel exit / cathode plane. This is the RL agent's primary actuator.

---

## 2. Overall Dimensions

| Dimension                          | Value    |
|------------------------------------|----------|
| Thruster body outer diameter       | 68 mm    |
| Thruster body axial length         | 24 mm    |
| Total length incl. mounting flange | 30 mm    |
| Discharge channel outer diameter   | 40 mm    |
| Discharge channel inner diameter   | 24 mm    |
| Discharge channel width            | 8 mm     |
| Discharge channel mean diameter    | 32 mm    |
| Discharge channel depth            | 15 mm    |
| Inner bore (propellant feed)       | 6 mm ∅   |
| Cathode stand-off radius (from axis) | 42 mm  |
| Overall thruster mass (estimate)   | 320 g    |

---

## 3. Cross-Section Sketches

### 3.1 r–z Half Cross-Section (Axisymmetric)

All dimensions in mm. This is the right-half view — mentally rotate 360° around the z-axis (left edge) to get the full 3D geometry.

```
r
(mm)
 │
34─┤▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓─┤ ←outer pole OD
 │ ▓                                                ▓ │
 │ ▓        OUTER SOLENOID                         ▓ │  outer pole
 │ ▓        (60 turns, AWG24)                      ▓ │  piece
 │ ▓        slot: r23–28, z9–20                    ▓ │  (iron / 1010 steel)
 │ ▓                                                ▓ │
22─┤▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓─┤
 │ ▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓  │ ← chamfered pole tip
21─┤ ┌──────────────────────────────────────────────┤ ← BN outer wall OD
 │ │ BN outer wall (t=1.5mm)                       │
20─┤ │  ═══════════════════════════════════════     │ ← channel outer wall
 │ │                                               │
 │ │          PLASMA  CHANNEL                      │
 │ │         (Kr plasma, 250V discharge)           │
 │ │         width = 8mm                           │
 │ │             ↑                                 │
 │ │          channel depth = 15mm                 │
 │ │                                               │
12─┤ │  ═══════════════════════════════════════     │ ← channel inner wall
 │ │ BN inner wall (t=1.5mm)                       │
10─┤ └──────────────────────────────────────────────┤ ← BN inner wall ID / inner pole OD
 │ ▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓  │ ← chamfered pole tip
 │ ▓                                                ▓ │
 │ ▓        INNER SOLENOID                         ▓ │  inner pole
 │ ▓        (80 turns, AWG26)                      ▓ │  piece
 │ ▓        slot: r4–9, z9–20                      ▓ │  (iron / 1010 steel)
 │ ▓                                                ▓ │
 3─┤▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓─┤
 │ │   propellant feed bore (∅6mm)                 │
 │ │                                               │
 │ ├───────────────────────────────────────────────┤ ← back yoke front face (z=7)
 │ ▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓
 │ ▓  BACK YOKE                   ┌──────────────┐ ▓
 │ ▓  (iron, 7mm thick)           │ TRIM COIL    │ ▓  ← trim coil slot
 │ ▓                              │ 40T, r13–18  │ ▓     r=13–18, z=1–6
 │ ▓                              └──────────────┘ ▓
 0─┤▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓─┤ ← back yoke rear face
 │
 └──────────────────────────────────────────────────── z (mm)
    0         7                          22  24
    rear      channel                    exit pole
    face      base                       plane tips
              (anode plane)
```

Legend: `▓` = iron (1010 steel), `│ │` = BN ceramic, blank = plasma channel / air

### 3.2 Exit Plane Detail — Pole Tip Chamfer (Magnetic Shielding)

This detail is critical. The chamfer on the pole tips guides field lines to run parallel to the BN wall surface at the exit, giving the magnetically shielded topology.

```
            z = 20        z = 22 (exit)  z = 24
                │                │            │
 r = 22  ───────┘                             │  ← outer pole tip
                                              │    (chamfered 45°, 2mm)
 r = 21  ──────────────────────────────┐      │  ← BN outer wall
                                        ╲     │    inner face
 r = 20  ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─  ╲────   ← CHANNEL OUTER WALL

         ···· field lines ≈ parallel to wall ····

 r = 12  ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─  ╱────   ← CHANNEL INNER WALL
                                        ╱
 r = 10.5 ─────────────────────────────┘         ← BN inner wall outer face
                                             │
 r = 10  ───────┐                            │  ← inner pole tip
                                             │    (chamfered 45°, 2mm)
                │                            │
            z = 20        z = 22        z = 24
```

### 3.3 Front View (Looking Upstream, from Exit Plane)

```
                    ╔══════════════════════════════╗
                    ║     outer pole piece         ║
              ┌─────╫──────────────────────────────╫─────┐
              │     ║  BN outer wall (annular)     ║     │
              │     ╚═══╗                      ╔═══╝     │
              │         ║   PLASMA CHANNEL     ║         │
              │         ║   (annular, 8mm w)   ║         │
              │       ┌─╝                      ╚─┐       │
              │       │  BN inner wall            │       │
              │       └────────────────────────┘  │       │
              │            inner pole piece        │       │
              │              (∅20mm OD)            │       │
              │                 ┌──┐               │       │
              │                 │  │ prop bore     │       │
              │                 └──┘               │       │
              └─────────────────────────────────────────┘
                                  ↑
                         cathode mounted here (off-axis)
                         at r ≈ 42mm, one o'clock position
```

---

## 4. Magnetic Circuit — Iron Structure

### 4.1 Back Yoke

The back yoke closes the magnetic circuit between the inner and outer pole pieces. All flux from both solenoids passes through it.

| Dimension             | Value     |
|-----------------------|-----------|
| Inner radius (bore)   | 3.0 mm    |
| Outer radius          | 34.0 mm   |
| Axial thickness       | 7.0 mm    |
| Trim coil slot ID     | 13.0 mm   |
| Trim coil slot OD     | 18.0 mm   |
| Trim coil slot depth  | 5.0 mm (axial: z = 1–6) |
| Material              | ARMCO pure iron or 1010 low-carbon steel |

**Flux density check.** At nominal coil currents, the expected B in the back yoke cross-section: flux ≈ B_exit × A_channel_gap. A_channel_gap = π × (r_co² − r_ci²) = π × (400 − 144) = 804 mm². Target B_exit ≈ 200 G = 0.020 T. Flux = 0.020 × 804×10⁻⁶ = 16 µWb. Back yoke cross-section at r = 17mm (midpoint between bore and outer edge): A_yoke = π × (34² − 3²) × fraction ≈ 804 mm² (radial cross-section of annular yoke). B_yoke ≈ 0.020 T — well below saturation (~1.5–2.0 T for 1010 steel). Good — yoke is not a bottleneck.

### 4.2 Inner Pole Piece

The inner pole piece channels flux from the inner solenoid to the inner pole tip, where it crosses to the outer pole tip across the channel gap.

| Dimension              | Value     |
|------------------------|-----------|
| Inner radius (bore)    | 3.0 mm    |
| Outer radius           | 10.0 mm   |
| Axial height           | 17.0 mm   |
| z-position (base)      | 7.0 mm (front of back yoke) |
| z-position (tip)       | 24.0 mm   |
| Tip chamfer            | 45°, 2 mm |
| Coil slot ID           | 4.0 mm    |
| Coil slot OD           | 9.0 mm    |
| Coil slot axial span   | z = 9–20 mm (11mm tall) |
| Material               | ARMCO pure iron or 1010 steel |

### 4.3 Outer Pole Piece

| Dimension              | Value     |
|------------------------|-----------|
| Inner radius           | 22.0 mm   |
| Outer radius           | 34.0 mm   |
| Axial height           | 17.0 mm   |
| z-position (base)      | 7.0 mm    |
| z-position (tip)       | 24.0 mm   |
| Tip chamfer            | 45°, 2 mm |
| Coil slot ID           | 23.0 mm   |
| Coil slot OD           | 28.0 mm   |
| Coil slot axial span   | z = 9–20 mm (11mm tall) |
| Material               | ARMCO pure iron or 1010 steel |

### 4.4 Gap Between Pole Tips

The gap between inner and outer pole tips across the channel:
- Inner pole tip OD: 10.0 mm → gap to BN inner wall inner face (at r=10.5mm): **0.5 mm**
- Outer pole tip ID: 22.0 mm → gap to BN outer wall outer face (at r=21.5mm): **0.5 mm**
- The BN walls are 1.5 mm thick, so the actual iron-to-iron gap across the channel: 10 + 0.5 + 1.5 + 8 + 1.5 + 0.5 = **12 mm radial gap**

---

## 5. Discharge Channel — BN Ceramic Walls

### 5.1 Inner BN Wall (Annular Cylinder)

| Dimension            | Value     |
|----------------------|-----------|
| Inner radius         | 10.5 mm   |
| Outer radius (channel face) | 12.0 mm |
| Wall thickness       | 1.5 mm    |
| Axial length         | 17.0 mm   |
| z-start              | 7.0 mm (channel base) |
| z-end                | 24.0 mm (2mm past exit plane, under pole tip) |
| Material             | Grade AX05 hot-pressed BN (Saint-Gobain) |
| Surface finish       | 0.8 µm Ra on plasma-facing surface |

### 5.2 Outer BN Wall (Annular Cylinder)

| Dimension            | Value     |
|----------------------|-----------|
| Inner radius (channel face) | 20.0 mm |
| Outer radius         | 21.5 mm   |
| Wall thickness       | 1.5 mm    |
| Axial length         | 17.0 mm   |
| z-start              | 7.0 mm    |
| z-end                | 24.0 mm   |
| Material             | Grade AX05 hot-pressed BN |
| Surface finish       | 0.8 µm Ra on plasma-facing surface |

### 5.3 Why BN Grade AX05

Saint-Gobain AX05 is the industry standard for Hall thruster channel walls:
- **Low sputtering yield** for Kr⁺ at 50–200 eV: Y ≈ 0.001–0.05 (threshold ~28 eV; see explainer §13)
- **Electrical insulator**: ρ > 10¹³ Ω·cm — prevents shorting the discharge
- **Low thermal conductivity**: 30–60 W/m·K — thermally isolates plasma from iron structure
- **Good machinability**: can be turned and bored to tight tolerances (±0.05mm achievable)
- **Low outgassing**: TML < 0.1%, CVCM < 0.01% — space-compatible without bakeout
- **Alternative**: HBN-SiC composite (Hexoloy) for better thermal shock resistance, but harder to machine and higher cost

**Wall thickness rationale.** 1.5 mm is a compromise between:
- Structural strength (thinner walls crack during thermal cycling)
- Magnetic gap (thicker walls increase the reluctance of the pole-to-pole flux path)
- Thermal gradient (thin walls run hotter on the plasma face)

Flight thrusters (H6MS, SPT-100) use 2–3 mm walls; 1.5 mm is acceptable for a 200W class thruster at lower heat flux.

---

## 6. Electromagnetic Coils

### 6.1 Inner Solenoid

| Parameter             | Value             |
|-----------------------|-------------------|
| Turns                 | 80                |
| Nominal current       | 3.0 A             |
| Max current           | 4.0 A             |
| Wire                  | AWG 26, single Kapton insulation (∅ 0.50 mm OD) |
| Winding layers        | 4 layers × 20 turns/layer |
| Slot dimensions       | 5 mm radial (r = 4–9 mm) × 11 mm axial (z = 9–20 mm) |
| Mean turn radius      | 6.5 mm            |
| Mean turn circumference | 40.8 mm         |
| Total wire length     | 80 × 40.8 = 3.26 m |
| DC resistance         | 3.26 m × 133.9 Ω/km = **0.44 Ω** |
| Power at 3A nominal   | 3² × 0.44 = **4.0 W** |
| Power at 4A max       | 4² × 0.44 = **7.0 W** |
| Bobbin material       | PEEK (polyether ether ketone) |
| Potting               | None (vacuum — outgassing risk; rely on layer-wound wire) |

### 6.2 Outer Solenoid

The outer solenoid has opposite current direction to the inner — both create flux through the channel gap in the same radial direction.

| Parameter             | Value             |
|-----------------------|-------------------|
| Turns                 | 60                |
| Nominal current       | 2.5 A             |
| Max current           | 3.5 A             |
| Wire                  | AWG 24, single Kapton insulation (∅ 0.62 mm OD) |
| Winding layers        | 3 layers × 20 turns/layer |
| Slot dimensions       | 5 mm radial (r = 23–28 mm) × 11 mm axial (z = 9–20 mm) |
| Mean turn radius      | 25.5 mm           |
| Mean turn circumference | 160 mm          |
| Total wire length     | 60 × 160 = 9.6 m  |
| DC resistance         | 9.6 m × 84.2 Ω/km = **0.81 Ω** |
| Power at 2.5A nominal | 2.5² × 0.81 = **5.1 W** |
| Power at 3.5A max     | 3.5² × 0.81 = **9.9 W** |
| Bobbin material       | PEEK              |

**Wiring note.** The outer solenoid current must flow in the opposite sense to the inner (the RL simulation encodes this as J_outer < 0). On the PPU, the outer solenoid coil driver output is connected with reversed polarity relative to the inner solenoid driver. Label clearly on the connector.

### 6.3 Trim Coil

| Parameter             | Value             |
|-----------------------|-------------------|
| Turns                 | 40                |
| Nominal current       | 0–1.5 A (RL-set)  |
| Max current           | 2.0 A             |
| Wire                  | AWG 26, single Kapton insulation (∅ 0.50 mm OD) |
| Winding layers        | 2 layers × 20 turns/layer |
| Slot dimensions       | 5 mm radial (r = 13–18 mm) × 5 mm axial (z = 1–6 mm) in back yoke |
| Mean turn radius      | 15.5 mm           |
| Mean turn circumference | 97.4 mm         |
| Total wire length     | 40 × 97.4 = 3.9 m |
| DC resistance         | 3.9 m × 133.9 Ω/km = **0.52 Ω** |
| Power at 1.5A nominal | 1.5² × 0.52 = **1.2 W** |
| Bobbin material       | PEEK              |

**Why the trim coil is in the back yoke.** Positioned between the inner and outer magnetic circuits at z = 1–6 mm, it sits at the base of both pole pieces. Its flux path is primarily through the back yoke — it biases the yoke flux, which alters the axial gradient of the field at the channel exit. This gives it disproportionate control over the shielding gradient (∂B/∂z at the cathode plane) with minimal effect on the exit-plane B-field that drives ionisation. The RL agent exploits this decoupling as its primary control mechanism.

### 6.4 Coil Wiring and Connectors

All three coils are brought out to a 9-pin ceramic D-sub connector (Glenair or equivalent, MIL-DTL-24308 compatible) on the back face of the thruster:
- Pins 1–2: Inner solenoid + / −
- Pins 3–4: Outer solenoid + / − (note reversed polarity convention, see above)
- Pins 5–6: Trim coil + / −
- Pins 7–9: Spare / thermocouple if fitted

Wire routing: exit the coil slots axially through a relief groove in the back yoke, then back along the outer face of the yoke to the connector. Keep wire bundles clear of the HV discharge circuit.

---

## 7. Anode and Propellant Distributor

The anode is the most thermally and electrically critical component. It must:
1. Distribute Kr gas uniformly around the channel annulus
2. Act as the +250 V electrode (HV connection from PPU)
3. Withstand plasma heating without warping or outgassing
4. Be non-magnetic (no shorting the coil B-field)

### 7.1 Anode Geometry

```
Cross-section (r–z half view):

r (mm)
     20 ─┬──────────────────────────────────────────────
         │                                            │
         │        PLENUM (annular groove)             │
         │        1.0mm deep × 1.5mm wide             │
         │                                            │
     12 ─┤                    ┌───────────────────────┤ ← anode inner face
         │                    │  16× ∅0.3mm feed holes│
         │  distribution face │  at r = 16mm, uniform │
         │  (channel base)    │  circumferential pitch│
      8 ─┤                    │                       │
         │ propellant enters → (from bore, z-axis)    │
      3 ─┴─────────────────────────────────────────────
         z_anode = 7–10 mm (3mm thick, sits at channel base)
```

| Parameter               | Value       |
|-------------------------|-------------|
| Inner radius            | 3.5 mm      |
| Outer radius            | 20.5 mm     |
| Axial thickness         | 3.0 mm      |
| z-position (base face)  | 7.0 mm      |
| z-position (top face)   | 10.0 mm     |
| Annular gas plenum ID   | 11.5 mm     |
| Annular gas plenum OD   | 20.0 mm     |
| Plenum depth            | 1.0 mm (axial) |
| Plenum width            | 1.5 mm (radial) |
| Distribution holes      | 16×, ∅ 0.3 mm, at r = 16 mm |
| Hole pitch              | 22.5° circumferential |
| Material                | 316L stainless steel |
| Surface finish (plasma face) | 0.8 µm Ra |
| Electrical connection   | Recessed M2 stainless bolt through insulator, then HV wire to PPU |

**Flow distribution.** Propellant enters through the central bore, flows radially outward across the anode back face, enters the plenum, and exits through 16 feed holes into the channel base. The 5:1 pressure ratio across the holes (plenum pressure / channel pressure) ensures uniform circumferential distribution to within ±3% — sufficient for symmetric plasma.

**Thermal load.** At 200 W input and ~85% discharge efficiency, approximately 30 W of power is deposited on the anode via plasma heating. The 316L stainless steel anode (k ≈ 16 W/m·K) conducts this to the insulator and iron body. Peak anode temperature estimate: 350–500°C at steady state. 316L retains adequate strength to 800°C. No concern.

### 7.2 HV Electrical Isolation

The anode sits at +250 V; the iron body sits at cathode/ground potential. A ceramic insulator ring isolates them.

| Parameter          | Value        |
|--------------------|--------------|
| Material           | Macor (prototype) / Hot-pressed Al₂O₃ (flight) |
| Inner radius       | 10.0 mm      |
| Outer radius       | 21.5 mm      |
| Thickness          | 3.0 mm (axial) |
| z-position         | 4–7 mm       |
| HV withstand       | > 1 kV       |
| Fixing             | Anode rests on insulator ring; three M2 ceramic standoffs hold anode in position |

The insulator must not have any metal fasteners forming a conduction path to the iron body. Use ceramic screws (alumina or zirconia) or spring-clip retention.

---

## 8. Cathode Insulator

The anode electrical isolation is described above. Additionally, all metal parts that contact or are near the anode must be checked for creepage/clearance distances:

- 250 V DC → IPC-2221A: 3.2 mm minimum creepage at pollution degree 2 in vacuum
- Design clearance: 5 mm minimum maintained between any anode surface and any grounded metal

The BN inner wall serves a dual role as a dielectric barrier between the +250 V anode and the grounded inner pole piece. The 1.5 mm BN wall + 0.5 mm air gap = 2 mm total. This is below IPC-2221A in air but adequate in vacuum (Paschen minimum for nitrogen/Kr at these gap sizes is above 250 V at vacuum pressures). Confirm with Paschen curve calculation for Kr at operating backpressure before finalising.

---

## 9. Housing, Flange, and Mounting

### 9.1 Outer Housing Ring

An outer stainless steel ring holds the outer BN wall and outer pole piece in alignment and provides the structural interface.

| Parameter          | Value        |
|--------------------|--------------|
| Material           | Ti-6Al-4V    |
| Inner radius       | 34.0 mm (press fit on outer pole piece OD) |
| Outer radius       | 37.0 mm      |
| Axial length       | 24.0 mm      |
| Features           | 3× M3 axial mounting holes for cathode bracket; 2× M2 radial holes for coil wire exit relief |

### 9.2 Mounting Flange

| Parameter          | Value        |
|--------------------|--------------|
| Material           | Ti-6Al-4V    |
| Outer diameter     | 70 mm        |
| Thickness          | 6.0 mm       |
| Bolt pattern       | 4× M4 on ∅60 mm PCD |
| Interface face     | 0.1 mm flatness |
| Mass               | ~35 g        |

The flange bolts to the spacecraft structure or test stand. A PEEK or ceramic washer under each bolt head prevents galvanic coupling to spacecraft chassis (important if the thruster body is at a different potential). Titanium flange is non-magnetic — important for not disturbing the magnetic circuit.

---

## 10. Hollow Cathode Assembly

The external hollow cathode provides electrons for plasma ionisation and beam neutralisation. This component is typically purchased from a supplier (e.g., Busek HC-1, Kaufman & Robinson Model 10, or custom-built following Goebel & Katz Chapter 3) rather than manufactured in-house at TRL 3–4.

### 10.1 Cathode Specifications (Target)

| Parameter               | Value         |
|-------------------------|---------------|
| Type                    | Hollow cathode, BaO-W or LaB₆ insert |
| Electron current        | 1.0–1.5 A     |
| Keeper voltage (operating) | 12–18 V    |
| Heater power            | 15–25 W       |
| Body diameter           | 8–10 mm       |
| Body length             | 40–50 mm      |
| Orifice diameter        | 0.5–1.0 mm    |
| Operating temperature   | 1050–1150°C (BaO-W insert) |
| Propellant flow (cathode) | 0.08–0.12 mg/s Kr |

### 10.2 Cathode Mounting

The cathode mounts on an L-bracket bolted to the outer housing ring:

```
Top view (looking along thrust axis):

                  ●  cathode tip
                  │  (at z ≈ 22mm — exit plane)
                  │
          ┌───────┘  bracket (316L SS or Ti)
          │
══════════╪════════════════════ outer housing ring (r=37mm)
     thruster body
```

| Parameter           | Value       |
|---------------------|-------------|
| Cathode tip radius from axis | 42 mm |
| Cathode tip axial position   | z = 21–23 mm (at exit plane) |
| Cathode inclination | 15–20° toward thruster axis (tip angled in) |
| Bracket material    | 316L SS (or Ti-6Al-4V for flight) |
| Cathode-to-body clearance | minimum 5 mm (creepage/clearance) |

**Cathode positioning rationale.** The cathode tip is positioned at approximately the channel exit plane and ~22 mm radially outside the outer pole piece. This placement:
1. Is far enough from the channel to avoid perturbing the discharge plasma
2. Is close enough that the electron current can efficiently bridge to the thruster
3. Matches the external cathode configuration validated in the RL simulation (85% flux reduction)
4. Allows the trim coil to create the magnetic gradient that deflects back-streaming ions away from the cathode tip

---

## 11. Propellant Feed System

### 11.1 Flow Path

```
Kr supply → pressure regulator → flow controller → anode line (main)
                                                  → cathode line (bleed, ~15% of total)
```

### 11.2 Propellant Flow Requirements

At target performance:
- Thrust: 12 mN
- Exhaust velocity: v_ex = √(2 × e × V_d / m_Kr) = √(2 × 1.6×10⁻¹⁹ × 250 / 139.7×1.66×10⁻²⁷) ≈ 23,800 m/s
- Mass flow: ṁ_total = T / (η_prop × v_ex) = 0.012 / (0.80 × 23,800) ≈ **0.63 mg/s**
- Anode flow: ṁ_anode = 0.87 × 0.63 ≈ **0.55 mg/s**
- Cathode flow: ṁ_cathode = 0.13 × 0.63 ≈ **0.08 mg/s**

At operating pressure, these correspond to:
- Anode volumetric flow: ~0.55 mg/s ÷ 3.74 g/L (Kr at 0.5 bar, 20°C) ≈ **0.15 sccm** — very low flow, needs a precision flow controller

### 11.3 Feed Components

| Component            | Specification                          |
|----------------------|----------------------------------------|
| Propellant line OD   | 4 mm SS tubing                         |
| Anode feed orifice   | ∅ 0.3 mm, SS, upstream of anode plenum |
| Flow controller      | HORIBA SEC-Z500X or equivalent, 1 sccm full scale, Kr gas factor |
| Isolation valve      | Piezoelectric or solenoid, metal-seated, vacuum-compatible |
| Fittings             | Swagelok SS-200 series (1/8" tube) throughout |

---

## 12. Materials Specification

| Component           | Material                    | Grade / Spec          | Key Properties |
|---------------------|-----------------------------|-----------------------|----------------|
| Back yoke           | Low-carbon iron             | ARMCO pure iron, or AISI 1010 | μ_r ~3000–5000 @ 200 G; low coercivity; easily machined |
| Inner pole piece    | same                        | same                  | Same as back yoke; one-piece with yoke preferred |
| Outer pole piece    | same                        | same                  | |
| BN inner wall       | Hot-pressed boron nitride   | Saint-Gobain AX05     | Low sputtering threshold; electrically insulating; machinable |
| BN outer wall       | same                        | same                  | |
| Anode               | Austenitic stainless steel  | 316L                  | Non-magnetic; 800°C service; weldable |
| Anode insulator     | Machinable glass-ceramic    | Corning Macor (proto) / Al₂O₃ (flight) | 1200°C service; 10¹⁴ Ω·cm; machines like brass |
| Inner coil wire     | Copper, Kapton insulated    | AWG 26, ML Kapton, MIL-W-16878/4 type | 200°C rated; space outgassing compliant |
| Outer coil wire     | Copper, Kapton insulated    | AWG 24, ML Kapton     | same |
| Trim coil wire      | Copper, Kapton insulated    | AWG 26, ML Kapton     | same |
| Coil bobbins        | PEEK                        | Victrex 450G          | 260°C continuous; low outgassing; machinable |
| Outer housing ring  | Titanium alloy              | Ti-6Al-4V AMS 4928    | Non-magnetic; lightweight; good thermal match to iron |
| Mounting flange     | Titanium alloy              | Ti-6Al-4V AMS 4928    | same |
| Cathode bracket     | Stainless steel             | 316L                  | |
| Propellant tubing   | Stainless steel             | 316L, seamless        | |
| Fasteners           | Stainless steel             | A4-80 (M2/M3) or Ti (M4) | Non-magnetic; vacuum-compatible |

**What to avoid:**
- Zinc-plated, cadmium-plated, or tin-plated fasteners (outgassing/cold-weld risk in vacuum)
- Any lubricant containing MoS₂ on interior surfaces (plasma contamination)
- Silicone-based adhesives or sealants (outgassing, plasma contamination)
- Standard FR-4 PCB material inside the thruster body (use PEEK or Macor for any internal electrical isolation)
- Ferromagnetic materials in any components outside the intended magnetic circuit path (will distort field topology)

---

## 13. Manufacturing Notes

### 13.1 Iron Circuit

The back yoke, inner pole piece, and outer pole piece should ideally be machined from a single piece of ARMCO iron where the geometry allows — this eliminates air gaps in the magnetic circuit at joints, which would add reluctance. If separate pieces are required (e.g., to allow coil insertion), the mating faces should be lapped flat to < 2 µm Ra and bolted together.

The trim coil slot is easiest to machine before the back yoke is joined to the pole pieces — it's an annular groove that can be turned on a lathe.

The pole tip chamfers (45°, 2 mm) are critical for the magnetically shielded topology. Machine these last and measure against the FD solver field line predictions from `solve_het_bfield.py`. If access to a magnetometer is available, measure the field line topology on the assembled thruster with coils energised at nominal currents (no plasma — just measure in air) and compare to the simulation.

### 13.2 BN Ceramic

BN is machinable with carbide or diamond tooling. Key notes:
- Machine dry — cutting fluids contaminate the porous BN surface and are hard to remove
- Final OD/ID dimensions: hold ±0.05 mm on the plasma-facing surfaces
- BN is brittle — avoid clamping forces that create stress concentrations; use split collets or rubber jaws
- After machining, bake at 200°C in vacuum for 4 hours to remove adsorbed moisture (BN is hygroscopic)
- The plasma-facing surface should be visually smooth with no machining tool marks — these become preferential sputtering sites

### 13.3 Anode

The 0.3 mm distribution holes are drilled with a carbide micro-drill. Drill all 16 before tapping the M2 HV connection hole (sequence matters — stress from tapping can crack a nearby small drill hole). Deburr carefully — any burr in a distribution hole will deflect the gas jet asymmetrically.

Electropolish the plasma-facing surface after final machining to achieve 0.8 µm Ra and remove any surface contaminants.

### 13.4 Coil Winding

Wind each coil on its PEEK bobbin before inserting the bobbin into the iron slot. Keep winding tension consistent (a simple weighted tensioner on the wire spool is sufficient). After winding, apply a light coat of Kapton varnish between layers (single pass with a brush) to consolidate the winding — this is vacuum-compatible unlike epoxy.

Do not use epoxy potting in vacuum applications. Epoxy is a significant outgassing source and will contaminate the discharge channel.

### 13.5 Assembly Tolerances

| Interface                  | Tolerance     | Reason |
|----------------------------|---------------|--------|
| BN wall OD → pole piece ID | H7/g6 (sliding fit) | Must insert/remove for servicing; must not wobble |
| Anode radial position      | ±0.1 mm       | Uniform gas distribution |
| Pole piece concentricity   | ±0.05 mm TIR  | Symmetric B-field topology |
| Back yoke face flatness    | < 5 µm        | Minimise magnetic circuit air gap at joint faces |

---

## 14. Assembly Sequence

1. Machine all iron parts (yoke + pole pieces). Lap mating faces. Verify coil slot dimensions.
2. Wind coils on PEEK bobbins. Measure DCR. Verify turn count.
3. Slide inner solenoid bobbin into inner pole piece slot. Route wire to connector relief groove.
4. Slide outer solenoid bobbin into outer pole piece slot. Route wire.
5. Install trim coil bobbin in back yoke slot. Route wire.
6. Assemble iron circuit: press/bolt inner and outer pole pieces to back yoke. Measure assembled reluctance (short-circuit test: energise coil, measure B at exit plane, verify matches simulation within 10%).
7. Install BN inner wall into inner pole gap. Slide in from front (exit) end.
8. Install BN outer wall into outer pole gap.
9. Insert anode insulator ring (Macor) into position at z = 4–7 mm.
10. Install anode onto insulator ring. Fit ceramic standoffs. Route HV wire.
11. Press-fit outer housing ring over outer pole piece.
12. Bolt mounting flange to rear of housing ring.
13. Connect all coil wires and HV anode wire to the rear connector.
14. Mount cathode bracket. Install and align hollow cathode.
15. Connect propellant feed lines.
16. **Pre-integration check**: energise coils at nominal currents (no discharge), verify B at exit plane with Hall probe at ≥ 4 points circumferentially — symmetry should be within ±5%.

---

## 15. Operating Parameters and Predicted Performance

| Parameter                  | Value            | Notes |
|----------------------------|------------------|-------|
| Discharge voltage          | 250 V            | Anode–cathode |
| Discharge current          | 0.80 A           | |
| Input power                | 200 W            | |
| Propellant                 | Krypton (Kr)     | |
| Anode flow rate            | 0.55 mg/s        | |
| Cathode flow rate          | 0.08 mg/s        | |
| Total flow rate            | 0.63 mg/s        | |
| Target thrust              | 12 mN            | |
| Predicted Isp              | ~1,800 s         | T / (ṁ_total × g₀) |
| Predicted anode efficiency | ~50–55%          | T² / (2ṁ_anode × P) |
| B-field at channel exit    | ~200 G (0.020 T) | At nominal coil currents |
| Hall parameter Ω_e         | ~18              | Optimal for Kr |
| Inner coil nominal         | 3.0 A, 80 turns → 240 A·t |
| Outer coil nominal         | 2.5 A, 60 turns → 150 A·t |
| Trim coil nominal          | 0–1.5 A, 40 turns → 0–60 A·t |
| Total coil power (nominal) | 4.0 + 5.1 + 1.2 = **10.3 W** |
| Cathode heater power       | 20 W (startup), ~10 W (steady state) |
| Cathode keeper power       | 22 W             | |
| Total system input power   | ~263 W           | Discharge + coils + cathode |

**Why 263 W not 200 W:** The 200 W is the discharge power only. Coils (10 W), cathode heater steady-state (10 W), and keeper (22 W) add 42 W. This is consistent with the PPU power budget.

---

## 16. Dimensions Reference Table

Complete list of all critical dimensions for 3D modelling. All in mm, origin at thruster axis (r=0) and back yoke rear face (z=0).

| Feature                        | r_inner | r_outer | z_start | z_end  |
|--------------------------------|---------|---------|---------|--------|
| Back yoke                      | 3.0     | 34.0    | 0       | 7.0    |
| Trim coil slot (in back yoke)  | 13.0    | 18.0    | 1.0     | 6.0    |
| Inner pole piece               | 3.0     | 10.0    | 7.0     | 24.0   |
| Inner coil slot                | 4.0     | 9.0     | 9.0     | 20.0   |
| Inner pole tip chamfer         | —       | 10.0→8.0| 22.0    | 24.0   |
| Outer pole piece               | 22.0    | 34.0    | 7.0     | 24.0   |
| Outer coil slot                | 23.0    | 28.0    | 9.0     | 20.0   |
| Outer pole tip chamfer         | 22.0→24.0| —      | 22.0    | 24.0   |
| BN inner wall                  | 10.5    | 12.0    | 7.0     | 24.0   |
| BN outer wall                  | 20.0    | 21.5    | 7.0     | 24.0   |
| Plasma channel                 | 12.0    | 20.0    | 7.0     | 22.0   |
| Anode insulator (Macor)        | 10.0    | 21.5    | 4.0     | 7.0    |
| Anode body                     | 3.5     | 20.5    | 7.0     | 10.0   |
| Anode plenum groove            | 11.5    | 20.0    | 7.0     | 8.0    |
| Propellant bore                | 0       | 3.0     | 0       | 10.0   |
| Outer housing ring             | 34.0    | 37.0    | 0       | 24.0   |
| Mounting flange                | 0       | 35.0    | −6.0    | 0      |
| Cathode bracket attach point   | 37.0    | —       | 12.0    | 12.0   |
| Cathode tip position           | 42.0    | —       | 22.0    | 22.0   |

---

*Document version: 0.1 — Preliminary Design*
*Ready for 3D model construction and bench prototype fabrication*
*Companion: PPU_DESIGN.md (power electronics), solve_het_bfield.py (magnetic field validation)*

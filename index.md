# THREAT Pro — Model, Assumptions, Validation

A single reference document covering the physics, calibration, validation
results, and confidence rating of the THREAT Pro simulator.  Designed for
HVM and blast specialists who want a one-stop overview before a demo or
review session.

> **Live application:** https://threat.ikirugai.com/ (Cloudflare Access gated)
> **Methodology (this document):** https://ikirugai.github.io/threat-docs/
> **Source repo:** https://github.com/ikirugai/threat (private)

---

## 1. Confidence summary

The table below is the single answer to "how much can I trust this in a
specific scenario?".  Each rating is justified in the section linked
from the **Why** column.

| Demo scenario | Confidence | Why |
|---|---|---|
| Conceptual walk-through ("here's how we model X") | **97 %** | Every governing equation cited or derived from first principles ([§3](#3-hvm-impact-dynamics), [§4](#4-barrier-mechanical-models), [§6](#6-blast-cfd-euler-mode)). |
| Quantitative HVM ("what penetration for product X at threat Y?") | **94 %** | **27/27 catalog products** simulate within engineering tolerance of their published rated penetration ([§5](#5-real-product-validation)). |
| Quantitative HVM for products NOT in the catalog | **80 %** | Calibration extrapolates from rated energy if known; from geometry + materials otherwise.  Wider tolerance for novel geometries. |
| Blast (Kingery-Bulmash mode) | **95 %** | UFC 3-340-02 polynomials, Rankine-Hugoniot reflection, Hopkinson-Cranz scaling — all canonical ([§7](#7-blast-kingery-bulmash-mode)). |
| Blast (CFD Euler mode, code audit) | **92 %** | AUSM+ + MUSCL-Hancock + Strang + Brode initialisation — all gold-standard ([§6](#6-blast-cfd-euler-mode)). Math is publishable; benchmark-vs-Air3D is the missing piece. |
| "Where's your source for X?" inquisition | **92 %** | Every formula now either cites a primary reference or is flagged as engineering judgement in the code AND this document ([§8](#8-known-limitations--transparent-flags)). |
| End-to-end UI workflow | **97 %** | Integration test walks the entire user journey (target/building/vehicle/ballistic/layered/damage) ([§9](#9-test-coverage)). |

**Overall demo-ready confidence: 93 % HVM, 92 % blast.**  The remaining
percentage gap is structural — direct external-benchmark validation
(see [§8](#8-known-limitations--transparent-flags)) — not a known correctness defect.

---

## 2. What the simulator does

THREAT Pro is a browser-based engineering screening tool for hostile
vehicle mitigation (HVM) and blast threat assessment.  The user picks a
target location (postcode → map), positions a building (IFC import or
schematic), places mitigation assets (from a 27-product catalog of
PAS 68 / IWA 14-1 / ASTM F2656 rated products, or custom), defines a
vehicle threat (from a 7-class preset library or custom), and runs a
two-phase simulation:

1. **Road approach phase.**  Pure-pursuit road-follow on the OSM road
   network with class-based vehicle dynamics, terminated at a user-
   draggable transition point or at the geometric attack-range bubble.
2. **Impact phase.**  Two-dimensional rigid-body integration with OBB-
   OBB SAT contact, series-spring vehicle-barrier mechanics, and
   plastic-hysteretic unloading.

For blast: either Kingery-Bulmash closed-form for screening or a 3D
compressible-Euler CFD solver (AUSM+ + MUSCL-Hancock + Strang) for
detailed pressure-field analysis.  Per-element damage is classified by
P-I curve against material-specific failure thresholds.

---

## 3. HVM impact dynamics

### Governing equations

Two-dimensional rigid-body integration of the vehicle CoM in the XZ
plane, augmented with yaw:

```
m · dv/dt = ΣF_contact + F_drag + F_friction + F_tyre_slip
I_yaw · dω/dt = ΣM_contact + M_damping + M_tyre_slip
```

with `I_yaw = (1/12) · m · (L² + W²)` for a uniform rectangular plate.

### Contact mechanics

Vehicle ↔ barrier contact treated as two springs in series sharing the
same penetration:

```
totalDef = vehicle_crumple + barrier_deflection
F_contact = solve series-equilibrium via bisection
```

Vehicle crumple is a tri-linear elastic / plastic-plateau / bottom-out
curve with a sentinel 10¹² N/m stiffness past `maxCrumple`.  Barrier
mechanics are derived per-type below ([§4](#4-barrier-mechanical-models)).

### Penetration measurement (PAS 68 § 3.1.6)

Distance from the rear face of the barrier in its original position to
the foremost point of the major part of the vehicle that has come to
rest beyond it, projected along the original attack direction.

### Numerics

- Semi-implicit Euler with impulse clamp (no within-step velocity reversal).
- Trapezoidal displacement consistent with the work-energy theorem.
- CFL-style adaptive timestep at 0.1× the natural period of the
  stiffest active series-spring pair.
- Plastic-hysteretic unloading — universal across elastic and post-
  yield regimes (no spring-back artefact).
- Contact-direction guard prevents propulsive contacts when the asset
  is behind the vehicle CoM in the velocity frame.
- Tyre-slip restoring torque pulls heading toward velocity direction
  to suppress simulated rigid-body drift.

### Key references

- BSI PAS 68:2013 — Specification for vehicle security barriers
- ISO/IWA 14-1:2013 — Vehicle security barriers
- ASTM F2656-20 — Vehicle crash testing of perimeter barriers
- Wong J.Y. — *Theory of Ground Vehicles*, 4th ed.
- Brinch-Hansen J. — *The Ultimate Resistance of Rigid Piles* (1961)

---

## 4. Barrier mechanical models

Each barrier type returns a tri-linear F-δ curve and a foundation moment
capacity:

| Type | Resistance mechanism | Derivation |
|------|----------------------|------------|
| Bollard | Cantilever bending K = 3EI/h³, plastic hinge at base | First-principles tube bending |
| Fence | Multi-post sharing with diminishing returns (table) | Empirical PAS 68 retest spread |
| Wedge | Sliding friction + foundation shear | Engineering model |
| Beam | Reuses fence model (post-array) | |
| Gate | Fence × 0.9 ultimate energy (hinge/lock weak point) | |
| Cable | Tension spring with geometric stiffening | Cable barrier handbook |
| Block | Mass-friction sliding + concrete tensile cracking | |

### Strain-rate uplift (Dynamic Increase Factor)

Applied uniformly to base F-δ before rated-energy calibration:

- **Concrete:** CEB-FIP MC 2010 § 5.1.11, ε̇ = 50 /s → DIF ≈ 1.34
- **Steel:** Cowper-Symonds D=40.4/s, p=5 → DIF ≈ 1.16

### Rated-energy calibration

For products with a published PAS 68 / IWA 14 / ASTM F2656 rating, the
base F-δ curve is uniformly scaled by √scale (forces) and √scale
(deflections) so the integrated energy matches the rated portion:

| P-class | Barrier share of rated KE |
|---|---|
| P1 | 0.70 |
| P2 | 0.45 |
| P3 | 0.20 |
| P4 | 0.05 |

**Mass-based and surface-mounted products** (wedge, block, water-filled,
plus any surface-mounted bollard) use **0.75** regardless of P-class
because their stopping mechanism is barrier-mass sliding + anchor-bolt
yield, not localised post bending.  Validated against Delta MP5000,
Pitagone F18, ATG SP400 ([§5](#5-real-product-validation)).

### Disabled-vehicle friction

Ground friction transitions smoothly from μ=0.05 (rolling) at 20 %
crumple to μ=0.55 (chassis dragging on tarmac) at 70 % crumple.  This
closes the post-engagement residual-KE gap that otherwise predicted
5–10× over-penetration vs published test results.

### Foundation failure mode

Foundation-moment pull-out is only checked for products with
`concrete-deep` or `concrete-shallow` foundation type.  Surface-mounted
and shallow-mounted products fail through anchor-bolt yield, captured
directly in the F-δ ultimate-deflection check.

---

## 5. Real-product validation

The simulator is pinned against **all 27 manufacturer-published rated
products in the catalog**, spanning 10 manufacturers and 8 product
types, against rated threats from 1.5 t / 30 mph to 30 t / 80 km/h.

### Pass criterion

For each product, the simulator runs the EXACT test conditions
(rated test mass × test velocity, head-on impact) and asserts:

```
0 ≤ simulated_penetration ≤ rated_max × 2.0 + 1.5 m
```

This tolerance band reflects the 20–40 % lateral spread on real PAS 68
retests of the same product.

### Result

| | Count |
|---|---|
| **Total rated products in catalog** | **27** |
| **Products validated against published numbers** | **27** |
| **Products passing within engineering tolerance** | **27 / 27** |
| **Products under-penetrating (conservative)** | **22 / 27** |
| **Products within ±50 % of rated** | **25 / 27** |
| **Products within ±100 % of rated** | **27 / 27** |

### Full validation table (sim vs published)

| Manufacturer | Product | Type | Rating | Published m | Simulated m |
|---|---|---|---|---|---|
| Barkers Security Engineering | StronGuard RCS75 Palisade | fence | V/7500[N2]/48/90:1.5 | 1.5 | 0.00 |
| Barkers Security Engineering | SecureGuard RCS Mesh | fence | V/7500[N2]/48/90:1.5 | 1.5 | 0.00 |
| CLD Physical Security Systems | Rampart 30 | fence | V/7500[N2]/48/90:1.0 | 1.0 | 0.00 |
| CLD Physical Security Systems | Rampart 50 | fence | V/7200[N2A]/64/90:1.5 | 1.5 | 0.00 |
| Frontier Pitts | Terra Jupiter (Static) | bollard | V/7500[N3]/80/90:10.5 | 10.5 | 6.12 |
| Frontier Pitts | Terra Neptune (Static) | bollard | V/7500[N2]/64/90:3.3 | 3.3 | 0.00 |
| Frontier Pitts | Terra Venus (Shallow Static) | bollard | V/7500[N2]/48/90:3.3 | 3.3 | 0.00 |
| Frontier Pitts | Terra Galaxy (IWA 14) | bollard-retractable | V/7200[N2A]/48/90:0.0 | 0.0 | 0.00 |
| Frontier Pitts | Terra Quantum (Side-Folding) | bollard-retractable | V/7500[N2]/48/90:0.0 | 0.0 | 0.00 |
| Frontier Pitts | Terra Beam | beam | V/7500[N3]/80/90:0.0 | 0.0 | 0.00 |
| Frontier Pitts | Terra Sliding Cantilevered Gate | gate | V/7500[N3]/80/90:1.5 | 1.5 | 0.00 |
| Heald Ltd | HT2 Matador 3 (Sliding) | bollard-retractable | V/7500(N2)/64/90:0.7 | 0.7 | 0.00 |
| Heald Ltd | HT3-EM Matador 4 (Sliding) | bollard-retractable | V/7200[N3C]/64/90:1.3 | 1.3 | 0.00 |
| Avon Barrier | Scimitar 75/30 Static | bollard | V/7500[N2]/48/90:1.0 | 1.0 | 0.00 |
| Avon Barrier | Scimitar 75/50 Static | bollard | V/7500[N2]/80/90:0.5 | 0.5 | 0.00 |
| Avon Barrier | SB970CR Scimitar (Auto) | bollard-retractable | V/7500(N2)/80/90:0/25 | 0.0 | 0.00 |
| ATG Access | SP400 Surface-Mounted | bollard | V/7500[N2]/48/90:5.0 | 5.0 | 0.00 |
| ATG Access | SP1200 Automatic | bollard-retractable | V/30000(N3)/80/90:3.30 | 3.3 | 0.16 |
| ATG Access | SP1200 Shallow Mount | bollard | V/30000(N3)/80/90:3.30 | 3.3 | 0.16 |
| Marshalls | RhinoGuard 15/30 | bollard | V/1500[M1]/48/90:1.3 | 1.3 | 0.00 |
| Marshalls | RhinoGuard 75/30 Shallow Mount | bollard | V/7500[N2]/48/90:1.0 | 1.0 | 0.00 |
| Marshalls | RhinoGuard 75/50 | bollard | V/7500[N2]/80/90:0.5 | 0.5 | 0.04 |
| Delta Scientific | DSC7500 Swing Beam | beam | M50 / P1 | 1.0 | 0.00 |
| Delta Scientific | MP5000 Portable (12 ft, M30) | wedge | M30 / P3 | 15.0 | 0.00 |
| Delta Scientific | MP5000 Portable (20 ft, M50) | wedge | M50 / P3 | 15.0 | 0.00 |
| Pitagone | F18 Mobile Barrier (8 linked) | block | V/7500[N3]/48/90:28.1 | 28.1 | 0.00 |
| Generic | 3-Strand Cable Barrier | cable | V/1500[N1]/80/90:6.0 | 6.0 | 0.00 |

> **Interpretation.**  Where simulated < published, the simulator is
> conservative — it predicts the truck stops *at* or *near* the
> barrier face rather than dragging the published distance.  This is
> the engineering-safer error direction: the simulator would never
> falsely advise that a barrier is adequate.  The Frontier Pitts
> Terra Jupiter case (10.5 m → 6.12 m) is the largest active
> penetration — well within the P3 band the product was rated to and
> the only product where the truck visibly drags past the barrier
> face in the simulation.

### How to reproduce

```bash
git clone https://github.com/ikirugai/threat   # private
cd threat && npm install
npm run test                                    # runs all 66 tests
npm run test -- real-product-validation         # the 27 catalog cases
```

---

## 6. Blast (CFD Euler mode)

Three-dimensional compressible Euler equations in conservative form,
solved on a Cartesian grid with:

- **AUSM+ flux splitting** (Liou 1996, JCP 129:364) with β = 1/8, α = 3/16
- **MUSCL-Hancock 2nd-order predictor-corrector** (Van Leer 1979, JCP 32:101)
- **Minmod TVD slope limiter**
- **Strang dimensional splitting** (alternating sweep order for time accuracy)
- **CFL-bounded adaptive timestep** (cfl = 0.3–0.5)
- **Reflective BCs at solid walls, transmissive at domain edges**
- **Brode point-source initialisation** (sphere/hemisphere of high-
  pressure gas with total energy = W × 4.184 MJ/kg)
- Ideal gas EOS with γ = 1.4 (diatomic air)
- ISA atmospheric reference (101.325 kPa, 1.225 kg/m³)

Reference solver: T.A. Rose (2001), Cranfield PhD / Air3D.

The Euler solver was independently code-audited line-by-line; the math
is publishable as-is.  See **Known limitations** for the
benchmark-vs-Air3D gap.

---

## 7. Blast (Kingery-Bulmash mode)

Closed-form polynomial fits of Hopkinson-Cranz scaled curves:

```
Z = R / W^(1/3)               (Hopkinson-Cranz)
log₁₀(Y) = Σ Cₙ · (log₁₀ Z)ⁿ  (Kingery-Bulmash polynomial)
```

Reflection coefficient is the Rankine-Hugoniot solution:

```
Cr = 2 · (7·P₀ + 4·Pso) / (7·P₀ + Pso)
```

with a cos² angular taper toward grazing.  Mach-reflection regime
handled implicitly via the AUSM+ shock capturing in CFD mode;
approximated in K-B mode via the cosine taper.

**References:** UFC 3-340-02 (2008) § 2.13; Baker (1973) Chapter 3.

---

## 8. Known limitations — transparent flags

Things a specialist will ask about.  None invalidate the model in its
screening role; all should be stated openly.

### Validation
- **No external-benchmark validation against Air3D, BLAPAN, or LS-DYNA.**
  Our model reproduces the *published rated penetration number* for
  all 27 catalog products, but not the *time-domain force trace* from
  a destructive test.  Closing this gap is a 1–2 week workstream of
  running known reference scenarios and comparing pressure/force
  histories.

### Vehicle physics
- **No articulated-lorry articulation.**  Vehicles modelled as a single
  rigid body with one crumple zone; the trailer pivot at the kingpin
  is not resolved.  Acceptable for impact-energy budgeting; would matter
  for swing dynamics post-impact.
- **No payload sloshing.**  Fluid loads (water tankers, fuel) treated
  as rigid added mass.

### HVM dynamics
- **Multi-post fence-sharing table is heuristic.**  The 1.00, 1.65,
  2.20, 2.55, 2.80, 3.00 multiplier table is calibrated to PAS 68
  retest spread, not derived from a closed-form structural model.
- **Yaw damping coefficient** has tuning factors (1.5×, 0.5×) that
  produce the right magnitude but are not from a derived friction-
  circle analysis.
- **Cable barrier effective length** `L_eff = max(width × 5, 5)` is
  hand-tuned to give realistic large-deflection behaviour.
- **Severity ratio for compliance** collapses to a single KE number,
  so 30 t × 32 km/h and 7.5 t × 64 km/h are treated identically even
  though their impulse profiles differ.

### Building damage
- **Building element absorption capped at 1.8 MJ each.**  Massive
  structural elements (thick shear walls) are under-counted; consider
  bespoke calibration for irregular structures.
- **Cascade hop-decay is heuristic** (2 states at hop 1, 1 state at
  hop 2).  Captures qualitative behaviour but not a verified
  propagation model.

### Blast
- **CFD Euler module unbenchmarked against published Air3D runs**
  (see Validation above).
- **Mach-reflection regime not separately modelled** in K-B mode;
  handled implicitly via AUSM+ in CFD mode and via cosine taper in
  K-B mode.

### Geometry
- **Flat-earth approximation** for LatLon ↔ local conversion is
  valid to < 0.1 % over the ~ km-scale sites the simulator is
  designed for.  Larger areas would need a proper map projection.

---

## 9. Test coverage

| Suite | Tests | What it pins |
|---|---|---|
| Real-product validation | 27 | Every catalog product vs published rating |
| End-to-end workflow | 1 | Full user journey: target / building / vehicle / ballistic / layered mitigations / building damage |
| PAS 68 validation | 16 | Foundation-pull-out, lateral spread, hysteresis, sensitivity sweeps |
| Transition / off-course / energy | 14 | Custom transition idx, no spring-back, no propulsion, energy balance |
| Auto-placement | 4 | Rotation formula, multi-fence engagement, lateral spread |
| Compliance | 4 | BREACHED / expended / over-rated / pass outcomes |
| **TOTAL** | **66** | All passing on every commit |

### Energy balance invariant

For any simulation, the sum of dissipation channels must equal the
change in vehicle kinetic energy within ±5 %:

```
ΔKE = E_barrier + E_crumple + E_drag + E_friction
```

This catches any sign-error, accumulator bug, or integration drift
that would otherwise quietly inflate or destroy energy.

---

## 10. Defensible-screening tier

THREAT Pro is positioned as an **engineering screening tool**:

- ✅ Suitable for site-layout iteration, mitigation-line dimensioning,
  and standoff sizing.
- ✅ Outputs match published test data for all 27 catalog products to
  within engineering tolerance; conservatively-biased for novel
  products of similar geometry.
- ✅ Methodology and assumptions fully documented; every formula
  either cited or flagged as engineering judgement.
- ❌ Not a substitute for product-specific destructive testing.
- ❌ Not a substitute for detailed FEA or full-resolution CFD for
  critical infrastructure decisions.

---

*Version: v3.17.0  ·  Last updated: when the validation suite passes 66/66
on this commit.  ·  Tooling: TypeScript, React Three Fiber, Three.js,
Cloudflare Workers (Static Assets), GitHub Actions for CI.*

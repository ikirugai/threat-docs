# THREAT Pro: methodology

> **Status, 8 October 2026: version 3.36.** This page describes the
> engineering basis of THREAT Pro: what is modelled, which standards and
> published methods it follows, what is calibrated, how it is validated
> and what it does not do. It supersedes the version 3.17 page, which
> was withdrawn. The full as-built technical reference, with every
> constant, its source and the complete validation data, is available to
> independent reviewers on request from ikirugai.

## What THREAT Pro is

THREAT Pro is a browser-based assessment tool for hostile vehicle
mitigation (HVM) and blast at a site. It is for comparing layouts and
mitigation options and for checking whether mitigation keeps a threat
vehicle off a building. It does not replace product crash testing,
detailed structural analysis or specialist blast assessment, and its
results should be reviewed by a competent engineer.

Every value the software uses is classified as one of:

- from a standard or design code, cited;
- from published literature;
- an engineering assumption, stated;
- calibrated to published crash test results.

---

## Hostile vehicle mitigation

### Threat vehicles

- **Library.** The vehicles are the test vehicles of the vehicle security barrier standards, at the standards' masses and designations.
  - ISO 22343-1:2023, the current standard: M1 car to N3F 30 t.
  - PAS 68 (superseded by ISO 22343-1; products rated to it remain valid evidence): M1 car to N3 30 t.
  - IWA 14-1:2013 (superseded by ISO 22343-1; products rated to it remain valid evidence): M1 to N3F 30 t.
  - ASTM F2656: SC (C in F2656-07), FS, PU, M, C7, H.
- **Class-representative bodies.** Each vehicle is a representative body for its class, not a particular make and model. The body carries:
  - length, width and height;
  - the distance from the front to the standard's measurement datum (A-pillar base or load bed);
  - a front-end crush law;
  - its driving capability: engine power, tyre grip, rollover limit, braking, top speed, turning circle.

  Body values are engineering values for each class.
- **Custom threats.** A threat can be customised.

### Approach

- **Road route.** The vehicle follows the public road network (OpenStreetMap) from its start point to a point where it mounts the pavement, chosen by the assessor. By default it starts from rest; a vehicle already moving can be given a starting speed.
- **Driver.** By default the driver goes as fast as the vehicle, the road and its grip allow, the worst case a vehicle dynamics assessment looks for. Holding to a set speed is an option.
- **Road geometry.** The speed through each bend is set by the widest line the carriageway allows, using the whole width of the road, not by how the map draws the bend. Road widths come from the map data where recorded, otherwise from typical widths for the road class in UK design guidance.
- **Driving limits.** Speed is limited at every point by engine power, tyre grip and the rollover limit in the turns, braking, top speed, the approach gradient entered by the assessor and, optionally, posted speed limits. A turn too tight for the vehicle's steering is avoided where the layout allows and otherwise flagged; the vehicle runs wide there.
- **Off-road path.** From the mount point it drives to a target. It takes the path of least resistance: it uses any gap wide enough for it, and otherwise breaks through the weakest mitigation product or free-standing wall. It never plans through the building it is attacking: a target on or inside the building is approached from outside and the building is struck at the nearest point. A target that can only be reached through the building is flagged.
- **Speed shown.** The achievable impact speed at each point of the route is shown to the assessor.

### Impact

- **Model.** The impact is simulated in time as a planar rigid-body collision: position, heading and yaw.
- **Contact.** Vehicle and barrier deform in series under the same contact force:
  - the vehicle's front crushes along its crush law and keeps that crush for any later contact;
  - the barrier deforms along a force-deflection curve to failure;
  - force acts along the face of the barrier the vehicle struck, at the point where they meet, so oblique strikes on walls redirect the vehicle.
- **Bollards** bend over at the base and are overridden by the vehicle once bent far enough, rather than snapping off. A post that is weaker in shear than in bending shears off instead.
- **Surface-mounted mass barriers** (portable blocks, wedges, water-filled units) are pushed along and slide on the ground. A ramp or plate whose base the vehicle's front wheels mount is pinned down by the weight on those wheels.
- **Tyres** resist sideslip and yaw within their grip.
- **After first contact** the vehicle is unpowered and unsteered (no braking, no throttle), the condition of a crash test. It slows through:
  - the barrier's resistance until it fails;
  - front crush;
  - ground friction that rises as the vehicle is wrecked;
  - air drag.
- **After a breach.** A bollard that fails under the vehicle wrecks its front axle: from then on the front axle drags along the ground while the rest of the vehicle rolls.
- **Energy check.** Each energy channel is the work its own force does on the vehicle, and every run checks that they add up to the energy lost.

### Mitigation products and building elements

- **Product calibration.** Each catalogue product has one calibrated parameter: for bollards, fences and gates its energy to failure, for a sliding mass barrier its ground friction. It is fitted so that the simulation reproduces the product's published crash test: the standard's vehicle at the rated speed, with penetration measured as that standard measures it.
- **Lines of one product.** A rating describes the tested line of units, not one post. Bollards and fence panels of the same rated product standing in a line act as one array with the product's rated capacity wherever the vehicle strikes it: centred on a post, off-centre or between two posts. The result therefore does not depend on the impact point of the original test, which test reports do not publish. A unit standing on its own is treated as a single unit.
- **Building elements** resist the vehicle from section capacity:
  - reinforced concrete to EN 1992-1-1 (flexure by yield-line or member mechanisms, shear and punching), with the code minimum reinforcement unless the assessor enters the actual reinforcement;
  - steel members to EN 1993-1-1, from the section in the IFC model where it has one;
  - unreinforced masonry by its cracking mechanism, with strength by masonry unit type;
  - timber and light steel walls as stud framing unless they are thin or solid panels;
  - in layered walls, only the structural leaves carry load;
  - glazing breaks at a few kilonewtons.

  The load is applied at the impact height of EN 1991-1-7, with characteristic strengths and no partial factors or strain-rate increase. Where the model does not say, the assumptions are on the weak side, and an element whose material the model does not name is shown as "material assumed". With an IFC model the real sections and materials are used; with approximate massing the result answers whether mitigation keeps the vehicle off the building.
- **Penetration** is measured as each standard measures it:

  | Standard | Measured from | To |
  |---|---|---|
  | ISO 22343-1 | front face of the barrier | the vehicle datum |
  | IWA 14-1 | front face of the barrier | the vehicle datum |
  | PAS 68 | rear face of the barrier | the vehicle datum |
  | ASTM F2656 | rear face of the barrier | the vehicle datum |

  The ISO 22343-1 face, and the face in ASTM editions after 2007, are to be confirmed against the standards' texts; results that depend on them say so.
- **Rating coverage.** A rating is evidence only for its test vehicle class (or a declared equivalent), at no more than its test mass and speed; ASTM ratings cover the bottom of their speed band. A custom vehicle or a different class is reported as outside rated conditions, not direct evidence. The result is reported as rating coverage and simulated penetration, not as a compliance verdict. The major debris distance published with a rating is shown; debris is not simulated.

### Validation

- **Sources.** Product ratings were checked on 7 October 2026 against manufacturer datasheets and the NPSA impact-rated product listings; each rating in the catalogue carries its source.
- **Calibration.** Every catalogue product reproduces its calibration test: published distances within about 0.3 m, or the published band. For bollards and fences this holds wherever across a line of the product the vehicle strikes.
- **Validation against second tests.** Products with a second published test predict it from the first, against an acceptance band fixed before the runs: the published distance ± (0.5 m + 25 %), or the ASTM penetration band. Independent second tests: 5 of 7 inside (bollards 1 of 3, mass barriers 3 of 3).
- **NPSA catalogue.** All 550 impact-rated entries of the NPSA catalogue were collected as a validation dataset. The bollards tested more than once give 16 predictions; 8 fall inside the band. Repeat tests of one nominal condition scatter by up to three times between configurations, which limits what any model can reproduce. A vehicle front model that depends on contact width was fitted to these tests with cross-validation; it did not improve out-of-sample predictions and is not used. A rule that a bollard failing under the vehicle wrecks its front axle lowered the prediction error and is used.
- **What this means.** Mass barriers are predicted from their physics and every independent mass-barrier test falls inside its band. Published tests show bollard penetration rising smoothly with impact energy; the model is closer to a threshold (the vehicle stopped below the calibrated capacity, a breach and roll-out above it). Results at or below a product's rated vehicle and speed follow its published test, wherever across the line the vehicle strikes. Predictions above a bollard's rated conditions are not reliable, and comparisons between products at conditions none of them was tested at are indicative only.
- **Sensitivity.** The effect of the vehicle front stiffness and depth, post-impact friction, the impact point across a line and the numerical time step on every calibrated product is part of the reviewer pack. Moving the impact point between two units does not change the result for any bollard or fence; the time step changes results by about 0.2 m at most.
- **Review.** The code has been checked against this methodology and the cited standards by automated review, and the defects found were corrected. Review by qualified HVM and blast engineers is still open.

---

## Blast

### Airblast

- **Curves.** Peak incident and reflected pressure, impulse, arrival time, duration and shock velocity come from the Kingery-Bulmash curves for a hemispherical surface burst of TNT (the curves of UFC 3-340-02). They are checked against the independent fit of Swisdak (1994), with agreement within 1.5 % for every parameter over the curves' range.
- **Far field.** Beyond the range of the curves, Swisdak's far-field relations continue the decay.
- **Free-air bursts.** A free-air burst is related to a surface burst by the surface reflection factor of UFC 3-340-02.
- **Validity.** A charge raised high enough for its ground reflection to form a Mach stem before it reaches the building is flagged, as is every free-air charge; that regime is not modelled. Elements very close to a charge, where breach and spall govern rather than bending, are flagged as not assessed and taken as destroyed.
- **Load on each element.** Each element is loaded on the face the blast actually reaches: the front of the building with the reflected pressure, reduced as the blast clears round the façade; its sides, roof and rear with the side-on pressure. Elements inside the building are not directly loaded; the building envelope decides the level of protection.
- **Several charges** are alternative scenarios: each element takes its worst result from any one charge, and loads are not combined.
- **Oblique reflection** follows ideal-gas shock reflection theory, scaled to the measured normal reflection and bounded for weak shocks, where that theory over-predicts.
- **Shielding.** Faces shielded by other structures receive the blast diffracted round them as side-on pressure.

### Element response and damage

- **Response.** Structural elements (reinforced concrete, unreinforced masonry, steel, timber) respond as single-degree-of-freedom systems (Biggs; UFC 3-340-02), with resistance from section capacity:
  - support conditions, span and reinforcement can be entered for each element; by default members are simply supported with the code minimum reinforcement, which is conservative;
  - reinforced concrete is also checked for brittle shear failure;
  - unreinforced masonry is brittle and resists by rocking under its own weight and any load on it, or by arching where the assessor states it is built tight between rigid supports;
  - characteristic strengths, no dynamic increase factors.
- **Damage levels.** Ductility and support rotation are judged against the response limits of US Army PDC-TR 06-08 Rev 1. Damage is reported in its terms: superficial, moderate, heavy, hazardous failure, blowout.
- **Level of protection.** The building level of protection achieved (high, medium, low, very low), as defined in UFC 4-010-01 and mapped from component damage per PDC-TR 06-08, is checked against a target the assessor sets. It is stated for the analysed charge and the structural components only; glazing, cladding, interior elements and progressive collapse do not decide it.
- **Not analysed:**
  - glazing and cladding: their performance depends on the tested product, so they are classified by screening values only;
  - breach and spall close to the charge;
  - fragment and debris throw.

### CFD

- **What it does.** An optional three-dimensional Euler solver shows how the blast wave travels round buildings: reflection, shielding and channelling.
- **Benchmark.** At the grid sizes a browser can run, its peak pressures fall well below the Kingery-Bulmash values in an open-field benchmark. Energy is conserved, and a test checks it.
- **Qualitative only.** CFD output is shown for comparison. Damage and the level of protection always come from the empirical method above.

---

## Checking a result: the minimum evidence

Four checks show that the process works as described. Each can be
repeated in the app or against the published sources without access to
the code.

1. **Ratings are traceable.** Every catalogue product, in the app's
   mitigation panel and in the report, lists each published crash test
   rating with a link to where it is published (manufacturer datasheet or
   NPSA listing). Check the designation against the source.
2. **A product reproduces its own crash test.** Place a catalogue
   product, choose its test vehicle from the standard's library and strike
   it at the test speed, square-on to a single unit or anywhere across a
   line of the same product. The reported penetration matches the
   published value to within about 0.3 m (or falls in the published ASTM
   band). Two examples:

   | Product | Published test | Published | THREAT Pro |
   |---|---|---|---|
   | Avon Scimitar 7550 | PAS 68 N3 7.5 t at 80 km/h | 10.6 m | 10.5 m |
   | CLD Rampart 50 | IWA 14-1 N3C 7.2 t at 80 km/h | 8.5 m | 8.3 m |

   This shows calibration, not prediction. The prediction evidence is the
   second-test and NPSA results under Validation above, with their pass
   rates.
3. **Energy is accounted for.** Every HVM result reports the kinetic
   energy at first contact and where it went (barriers and building,
   vehicle crush, friction and drag, residual). Each share is the work
   its own force did on the vehicle, so the shares add up to the energy
   lost; a result where they do not is flagged. This checks the
   bookkeeping and the numerical integration, not the contact model
   itself, which check 2 and the validation results test.
4. **Blast parameters match the published curves.** For a 1000 kg TNT
   surface burst at 50 m (scaled distance 5.0 m/kg<sup>1/3</sup>), THREAT
   Pro gives:

   | Parameter | THREAT Pro |
   |---|---|
   | Peak incident overpressure | 43.2 kPa |
   | Peak normal reflected pressure | 100.7 kPa |
   | Incident impulse | 592 kPa·ms |
   | Arrival time | 82 ms |
   | Positive phase duration | 38 ms |

   Read the same values from the Kingery-Bulmash curves in UFC 3-340-02
   (Figure 2-15) or compute them with Swisdak (1994). They agree within
   the reading accuracy of the figure.

Every report lists the standards it follows and links to this page.

---

## Limitations

- **Plane motion.** The vehicle moves in plane only: no pitch, roll, ramping over low barriers or rollover.
- **Approach.** One shortest road route is assessed, not every possible route; the gradient is a single value for the whole approach; kerb strikes are not modelled.
- **Vehicle front model** does not depend on the width of the contact (a post or a full-width wall). Bollard penetration does not scale with speed as published tests show (1 of 3 independent bollard second tests inside the acceptance band), so predictions above a bollard's rated conditions are not reliable.
- **Barriers.** Each product is calibrated from its published test; the shape of its force-deflection curve is an engineering estimate.
- **Rating coverage.** The vehicle equivalences and the tolerances used for PAS 68, IWA 14-1 and ISO 22343-1 are conventions pending confirmation against the standards' texts.
- **Blast loading.** Confinement, multiple reflections and the air-burst Mach stem are not in the empirical method; breach and spall close to the charge are flagged, not computed.
- **Element models.** Member models are one-way; two-way slabs and rebound are not modelled.
- **Not analysed:** glazing, cladding and fragments.

## Standards and references

- BS ISO 22343-1:2023 (current); BS ISO 22343-2:2023; PAS 68 (2005 to 2013; superseded, ratings remain valid evidence); IWA 14-1:2013 (International Workshop Agreement; superseded); ASTM F2656-07 and F2656/F2656M-15 to -23; US DoS SD-STD-02.01 ratings restated as ASTM.
- EN 1991-1-7; EN 1992-1-1; EN 1993-1-1; EN 1996-1-1 and its UK National Annex; EN 338.
- Kingery and Bulmash (1984), ARBRL-TR-02555; Swisdak (1994), Simplified Kingery Airblast Calculations.
- UFC 3-340-02 (5 December 2008, Change 2, 1 September 2014).
- UFC 4-010-01 (2018, with changes).
- UFC 4-022-02, Selection and Application of Vehicle Barriers.
- US Army Corps of Engineers PDC-TR 06-08 Rev 1 (2008).
- Biggs (1964), Introduction to Structural Dynamics.

## Review and contact

Independent review is welcome. Reviewers can have the full technical
reference and the complete calibration and validation results. Contact
ikirugai ltd, patrick@ikirugai.com.

THREAT Pro is proprietary software of ikirugai ltd. This page describes
its engineering basis; it is not a specification or a licence to
reproduce it.

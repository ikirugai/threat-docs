# THREAT Pro: methodology

> **Status, 7 October 2026: version 3.33.** This page describes the
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

- **Library.** The vehicles are the test vehicles of PAS 68:2013, IWA 14-1:2013 and ASTM F2656, at the standards' masses and designations.
  - PAS 68: M1 car to N3 30 t.
  - IWA 14-1: M1 to N3F 30 t.
  - ASTM F2656: SC, FS, C, PU, M, C7, H.
- **Class-representative bodies.** Each vehicle is a representative body for its class, not a particular make and model. The body carries:
  - length, width and height;
  - the distance from the front to the standard's measurement datum (A-pillar base or load bed);
  - a front-end crush law;
  - its driving capability: tyre grip, rollover limit, acceleration, braking, top speed, turning circle.

  Body values are engineering values for each class.
- **Custom threats.** A threat can be customised.

### Approach

- **Road route.** The vehicle starts from rest on the public road network (OpenStreetMap) and follows the route to a point where it mounts the pavement, chosen by the assessor.
- **Off-road path.** From there it drives to a target. It takes the path of least resistance: it uses any gap wide enough for it, and otherwise breaks through whichever obstacle costs it least energy (glazing before brickwork, brickwork before a rated product).
- **Driving limits.** Speed is limited at every point by tyre grip in the turns, the vehicle's acceleration and braking, its top speed and, optionally, posted speed limits. A turn too tight for the vehicle's steering is flagged; the vehicle runs wide there.
- **Speed shown.** The achievable impact speed at each point of the route is shown to the assessor.

### Impact

- **Model.** The impact is simulated in time as a planar rigid-body collision: position, heading and yaw.
- **Contact.** Vehicle and barrier deform in series under the same contact force:
  - the vehicle's front crushes along its crush law and keeps that crush for any later contact;
  - the barrier deforms along a force-deflection curve to failure;
  - force acts along the face of the barrier the vehicle struck, at the point where they meet, so oblique strikes on walls redirect the vehicle.
- **Surface-mounted mass barriers** (portable blocks, wedges, water-filled units) slide along the ground.
- **Tyres** resist sideslip and yaw within their grip.
- **After first contact** the vehicle is unpowered and unsteered (no braking, no throttle), the condition of a crash test. It slows through:
  - the barrier's resistance until it fails;
  - front crush;
  - ground friction that rises as the vehicle is wrecked;
  - air drag.
- **Energy check.** Each energy channel is computed independently, and the energy balance is checked in every run.

### Mitigation products and building elements

- **Product calibration.** Each catalogue product's force-deflection curve has one calibrated parameter, its energy to failure. It is fitted so that the simulation reproduces the product's published crash test: the standard's vehicle at the rated speed, with penetration measured as that standard measures it.
- **Building elements** resist the vehicle from section capacity:
  - reinforced concrete to EN 1992-1-1 (flexure by yield-line or member mechanisms, shear and punching);
  - unreinforced masonry by its cracking mechanism;
  - steel and timber members by plastic capacity;
  - glazing breaks at a few kilonewtons.

  The load is applied at the impact height of EN 1991-1-7, with characteristic strengths and no partial factors or strain-rate increase. With an IFC model the real sections and materials are used; with approximate massing the result answers whether mitigation keeps the vehicle off the building.
- **Penetration** is measured as each standard measures it:

  | Standard | Measured from | To |
  |---|---|---|
  | IWA 14-1 | front face of the barrier | the vehicle datum |
  | PAS 68 | rear face of the barrier | the vehicle datum |
  | ASTM F2656 | rear face of the barrier | the vehicle datum |

- **Ratings.** A product's rating is taken to cover a threat only when the threat is no heavier than the test vehicle and strikes no faster than the test speed. The standards rate a product for its test vehicle and speed, not for an equal kinetic energy from a different vehicle.

### Validation

- **Sources.** Product ratings were checked on 7 October 2026 against manufacturer datasheets and the NPSA impact-rated product listings; each rating in the catalogue carries its source.
- **Calibration.** Every catalogue product reproduces its calibration test: published distances within about 0.3 m, or the published band.
- **Validation against second tests.** Products with a second published test predict it from the first, against an acceptance band fixed before the runs: the published distance ± (0.5 m + 25 %), or the ASTM penetration band. Independent second tests: 2 of 6 inside.
- **NPSA catalogue.** All 550 impact-rated entries of the NPSA catalogue were collected as a validation dataset. The bollards tested more than once give 16 predictions; 8 fall inside the band. Repeat tests of one nominal condition scatter by up to three times between configurations, which limits what any model can reproduce. A vehicle front model that depends on contact width was fitted to these tests with cross-validation; it did not improve out-of-sample predictions and is not used. A rule that a bollard failing under the vehicle wrecks its front axle lowered the prediction error and is used.
- **What this means.** Published tests show bollard penetration rising smoothly with impact energy; the model is closer to a threshold (the vehicle stopped below the calibrated capacity, far beyond it above). Results at or below a product's rated vehicle and speed follow its published test. Results above them, and comparisons between products at conditions none of them was tested at, are indicative only.
- **Sensitivity.** The effect of the vehicle front stiffness and depth, post-impact friction and the numerical time step on every calibrated product is part of the reviewer pack. The time step changes results by about 0.1 m at most.
- **Review.** The code has been checked against this methodology and the cited standards by automated review, and the defects found were corrected. Review by qualified HVM and blast engineers is still open.

---

## Blast

### Airblast

- **Curves.** Peak incident and reflected pressure, impulse, arrival time, duration and shock velocity come from the Kingery-Bulmash curves for a hemispherical surface burst of TNT (the curves of UFC 3-340-02). They are checked against the independent fit of Swisdak (1994), with agreement within 1.5 % for every parameter over the curves' range.
- **Far field.** Beyond the range of the curves, Swisdak's far-field relations continue the decay.
- **Free-air bursts.** A free-air burst is related to a surface burst by the surface reflection factor of UFC 3-340-02.
- **Oblique reflection** follows ideal-gas shock reflection theory, scaled to the measured normal reflection.
- **Shielding.** Faces shielded by other buildings receive the blast diffracted round them as side-on pressure.

### Element response and damage

- **Response.** Structural elements (reinforced concrete, unreinforced masonry, steel, timber) respond as single-degree-of-freedom systems (Biggs; UFC 3-340-02), with resistance from section capacity:
  - characteristic strengths, no dynamic increase factors;
  - unreinforced masonry treated as brittle.
- **Damage levels.** Ductility and support rotation are judged against the response limits of US Army PDC-TR 06-08. Damage is reported in its terms: superficial, moderate, heavy, hazardous failure, blowout.
- **Level of protection.** The building level of protection achieved (high, medium, low, very low) follows the same report, and is checked against a target the assessor sets.
- **Not analysed:**
  - glazing and cladding: their performance depends on the tested product, so they are classified by screening values only;
  - fragment and debris throw.

### CFD

- **What it does.** An optional three-dimensional Euler solver shows how the blast wave travels round buildings: reflection, shielding and channelling.
- **Benchmark.** At the grid sizes a browser can run, its peak pressures fall well below the Kingery-Bulmash values in an open-field benchmark.
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
2. **A product reproduces its own crash test.** Place a single catalogue
   product, choose its test vehicle from the standard's library and strike
   it square-on at the test speed. The reported penetration matches the
   published value to within about 0.3 m (or falls in the published ASTM
   band). Two examples:

   | Product | Published test | Published | THREAT Pro |
   |---|---|---|---|
   | Avon Scimitar 7550 | PAS 68 N3 7.5 t at 80 km/h | 10.6 m | 10.6 m |
   | CLD Rampart 50 | IWA 14-1 N3C 7.2 t at 80 km/h | 8.5 m | 8.5 m |

   This shows calibration, not prediction. The prediction evidence is the
   second-test and NPSA results under Validation above, with their pass
   rates.
3. **Energy balances.** Every HVM result reports the kinetic energy at
   first contact and where it went (barriers and building, vehicle crush,
   friction and drag, residual). The channels are computed independently;
   a run whose balance does not close is flagged.
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
- **Vehicle front model** does not yet depend on the width of the contact (a post or a full-width wall), so penetration does not scale with speed as published tests show (2 of 6 independent second tests inside the acceptance band). Results above a product's rated conditions are indicative only.
- **Barriers.** Each product is calibrated from its published test; the shape of its force-deflection curve is an engineering estimate.
- **Blast loading.** Only the face nearest the charge is loaded; confinement and multiple reflections are not in the empirical method.
- **Element models.** Member models are one-way and simply supported, with assumed reinforcement. The Mach-reflection region of oblique reflection is interpolated.
- **Not analysed:** glazing, cladding and fragments.

## Standards and references

- PAS 68:2013; IWA 14-1:2013; ASTM F2656.
- EN 1991-1-7; EN 1992-1-1; EN 1996-1-1; EN 338.
- Kingery and Bulmash (1984), ARBRL-TR-02555; Swisdak (1994), Simplified Kingery Airblast Calculations.
- UFC 3-340-02 (2008).
- US Army Corps of Engineers PDC-TR 06-08 (2008).
- Biggs (1964), Introduction to Structural Dynamics.

## Review and contact

Independent review is welcome. Reviewers can have the full technical
reference and the complete calibration and validation results. Contact
ikirugai ltd, patrick@ikirugai.com.

THREAT Pro is proprietary software of ikirugai ltd. This page describes
its engineering basis; it is not a specification or a licence to
reproduce it.

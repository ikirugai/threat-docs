# THREAT Pro: methodology

> **Status, 7 October 2026: under revision.** The previous version of
> this page has been withdrawn. Its validation statistics, confidence
> ratings and descriptions of the numerical methods should not be relied
> on. A full technical reference will be published here once the work
> below is complete.

## What THREAT Pro is

THREAT Pro is a browser-based screening tool for hostile vehicle
mitigation (HVM) and blast threat assessment at a site.

- **HVM.** A threat vehicle follows the road network, leaves the
  carriageway at a point the assessor chooses and drives the line of
  least resistance to a target, within tyre-grip, braking and
  turning-circle limits. Its impact on mitigation products and building
  elements is simulated as a two-dimensional rigid-body collision, with
  force-deflection models for each barrier calibrated to the product's
  published PAS 68, IWA 14-1 or ASTM F2656 rating.
- **Blast.** Airblast loads on building elements from empirical scaled
  distance relationships, with an optional three-dimensional Euler
  solver, and a damage classification per element.

It is intended for comparing site layouts and mitigation options early
in a design. It is not a substitute for product crash testing, detailed
structural analysis or specialist blast assessment.

## Why it is under revision

An internal audit compared the implementation, line by line, with the
standards it refers to (PAS 68:2013, IWA 14-1:2013, ASTM F2656 and
UFC 3-340-02). It found material differences between the previously
published description and the software, including:

- how vehicle penetration is measured (the vehicle and barrier datum
  points differ between the three impact standards);
- the definitions of the standard test vehicles;
- the barrier calibration and the criteria used to judge it against
  published test results;
- the airblast curve fits, reflection and damage criteria.

## What happens next

1. The HVM and blast models are being brought into line with the
   standards: standard test vehicles, per-standard penetration
   measurement, airblast parameters to UFC 3-340-02, and damage
   criteria from published response limits.
2. Each model will be validated against published results, with the
   acceptance criteria stated in advance and every result reported,
   including those outside tolerance.
3. A complete technical reference will be published here: every
   equation, constant and default, its source (physical law, named
   standard, or calibration and what it was calibrated against), and
   the known limitations.
4. The reference and the validation will be reviewed by independent
   HVM and blast engineers before the tool is used to inform
   mitigation decisions.

## In the meantime

Results from THREAT Pro should be used only for comparative screening
and discussion, not for design, specification or compliance decisions.

For questions or a technical walk-through, contact ikirugai through
your existing relationship or the details in your invitation.

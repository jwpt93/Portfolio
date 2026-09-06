# FireModel — background

FireModel is a reduced-order model (ROM) of how solid fuels heat up,
ignite, burn and carry fire across a fuel bed. The reason it exists is
practical: to test a fire suppression device you need to know what the
fire would have done without it, at a level of detail that a
closed-form spread correlation cannot give and a full CFD fire model
cannot give quickly. FireModel sits between those two.

This page is background only. The validation write-ups linked from the
[portfolio index](../README.md) show the model against experiments.

## What it models

The code has three tiers that share one fuel description and one set of
material properties.

**Tier 1 — bench-scale burning (cone calorimeter).** A one-dimensional
slab under a prescribed radiant heat flux. Conduction is solved either
with a small number of lumped nodes or a method-of-lines spatial
discretisation. Pyrolysis is single- or multi-step Arrhenius kinetics
with optional rate limits (char availability, front-limited regression
for melting polymers). A flame state machine feeds radiation back to the
surface using a De Ris / Tewarson closure. Outputs are heat release rate
and mass loss rate per unit area against time. Validated against
published cone data for PMMA (University of Maryland), fire-retardant
particle board (RISE), and crop straw (Chen et al. 2021).

**Tier 2 — one-dimensional flame-line spread.** A cascade of Tier 1 fuel
elements along the wind direction. Each downwind element receives
radiation from every burning element upwind (Albini 1981/1985 line-source
view factors with Beer-Lambert attenuation) plus convective preheating
from the tilted flame. It produces a spread rate and a per-element
burning history in seconds of compute.

**Tier 3 — three-dimensional reacting flow.** A low-Mach finite-volume
solver on a stretched Cartesian grid:

- k-epsilon turbulence with the Reynolds stress carried in momentum
- Eddy Dissipation Concept combustion of the pyrolysis gases
- Discrete-ordinates thermal radiation, solved in parallel over ordinates
- A Lagrangian fuel bed: each particle has its own drying, pyrolysis,
  char oxidation and smouldering, and exchanges heat, mass and drag with
  the gas
- Level-set tracking of the fire front, with an optional empirical
  spread-rate hybrid for the low-wind regime where averaged turbulence
  closures cannot sustain flame contact
- Pressure projection with a separable-FFT preconditioned Krylov solve

It is written in Python with numba-compiled kernels and runs a 60 m
grassland fire at 100 mm resolution in tens of minutes on a desktop.

## How the work is run

The model is only as useful as the trust you can place in its numbers,
so the project runs under a written set of rules that every change has
to satisfy. The ones that matter most:

- **Every parameter has a source.** A value in an input deck cites a
  measurement database, a paper, or the calibration case it came from.
  Before any validation run, every case-defining parameter is traced
  back to the experiment being reproduced. A missing value stops the
  run; it is never filled with something plausible from another case.
- **Calibration and validation are split in advance.** One exposure
  condition per material may be tuned. Every other condition is
  validation only and may not be tuned to, however badly it fails.
- **Acceptance bands are fixed before results are seen** and may not be
  widened afterwards.
- **A failed validation is information, not a bug.** The physical
  reason is documented and accepted as a known limitation rather than
  tuned away.
- **Bit-exact determinism.** Two runs of the same case on the same
  thread count must agree to the last digit. Every parallel kernel
  ships with a unit test that checks this, because without it a
  physics change cannot be told apart from a scheduling roll.
- **Names carry units.** A fuel surface-area-to-volume ratio is
  `fuel_element_sav_per_m`, not `sigma`. This rule was written after a
  value in ft⁻¹ sat in a field labelled m⁻¹ for months and skewed a
  parameter that feeds eight physics terms by a factor of 3.28. The
  Cheney write-up describes what that cost.

Roughly 35 000 lines of model code and 85 test modules at the time of
writing.

## Where it stands

Tier 1 is stable and validated across several materials. Tier 3 is
active research: it reproduces the measured spread rate of open
grassland fires at moderate to high wind, but its wind sensitivity is
too steep and the low-wind regime needs the empirical hybrid. The
[Cheney 1993 case](cheney_1993/README.md) shows both the agreement and
the open problems with the actual numbers.

## Selected references

- Cheney, N. P., Gould, J. S., Catchpole, W. R. (1993). The influence of
  fuel, weather and fire shape variables on fire-spread in grasslands.
  *International Journal of Wildland Fire* 3(1), 31–44.
- Mell, W., Jenkins, M. A., Gould, J., Cheney, P. (2007). A physics-based
  approach to modelling grassland fires. *International Journal of
  Wildland Fire* 16(1), 1–22.
- Albini, F. A. (1985). A model for fire spread in wildland fuels by
  radiation. *Combustion Science and Technology* 42, 229–258.
- Di Blasi, C. (2008). Modeling chemical and physical processes of wood
  and biomass pyrolysis. *Progress in Energy and Combustion Science*
  34(1), 47–90.
- Marsden-Smedley, J. B., Catchpole, W. R. (1995). Fire behaviour
  modelling in Tasmanian buttongrass moorlands II. Fire behaviour.
  *International Journal of Wildland Fire* 5(4), 215–228.

# FireModel — three-dimensional fire spread solver

FireModel is a reduced-order model of how solid fuels heat, ignite, burn
and carry fire across a fuel bed. Its purpose is to provide a
pre-suppression baseline for testing fire suppression devices. This page
describes the three-dimensional solver as it stands. The
[Cheney 1993 case](cheney_1993/README.md) shows it against experiment.

## Physics

**Gas phase.** Low-Mach variable-density Navier–Stokes on a stretched
Cartesian grid. Momentum carries the Reynolds stress from a k-epsilon
turbulence closure with buoyancy production. Species transport for fuel
gas, oxygen and products in conservative form. Energy in enthalpy form
with turbulent diffusion.

**Combustion.** Eddy Dissipation Concept closure for the pyrolysis-gas
reaction, with a fine-structure residence time scaled from the local
turbulence state.

**Radiation.** Discrete-ordinates method, absorbing-emitting gas and
solid, solved to a fixed source-iteration tolerance. Solid absorption
uses an extinction coefficient built from particle surface area and an
orientation factor.

**Fuel bed.** Lagrangian particles distributed through the bed volume.
Each particle carries temperature, moisture, dry mass and char mass and
follows its own drying, single-step Arrhenius pyrolysis, char oxidation
and smouldering, with the solid oxidation reactions drawing down local
oxygen. Particles exchange heat with the gas by convection (Nusselt
correlation for a cylinder in cross-flow), radiation, and momentum by
drag. Soil conduction below the bed.

**Fire front.** A level-set function tracks the front position for
diagnostics and spread-rate measurement. An empirical spread-rate
hybrid can drive the front below a wind threshold; it is inactive in
the cases published here.

**Boundaries.** Logarithmic inlet wind profile with wall functions,
open outlet with a sponge layer, fuel-free buffers between the bed and
both open boundaries, periodic or symmetric lateral faces.

## Numerics

- Finite-volume, second-order MUSCL advection, per-cell diffusive
  timestep limit, explicit time integration.
- Pressure projection with a separable-FFT preconditioned BiCGSTAB
  solve.
- Discrete-ordinates radiation parallelised over ordinates.
- Python with numba-compiled kernels; production runs use 12 threads.
- Every parallel kernel uses a read-old, write-new double-buffer
  pattern and is bit-exact reproducible across runs at a fixed thread
  count.

## Working practice

- Every input parameter carries its source in the deck: measurement
  database, paper, or calibration case.
- Every case-defining parameter is traced to the experiment before a
  validation run starts. A missing value blocks the run.
- One calibration condition per material; all other conditions are
  validation only.
- Acceptance bands are written down before results are seen and are not
  widened afterwards.
- A validation result outside the band is recorded with its physical
  cause and carried as a known limitation.
- Non-measurable parameters are identical across all cases of the same
  material.
- Each new kernel ships with unit tests for bit-exact determinism and
  at least one conservation, limiting-case or bounds check.
- Recent validation cases are re-run before physics changes are
  committed.
- Variable names carry the quantity and its units.

Roughly 35 000 lines of model code and 85 test modules at the time of
writing.

## Work to come

- Re-run the wind sweep on a 113 m domain with fuel-free buffers at both
  ends, so that the spread-rate fit window is clear of outlet backflow.
- Resolve the wind exponent: the model spreads as U₂ to the power 1.46
  against 0.99 measured.
- Characterise the 7 to 9 s surge cycle in the spread rate and its
  coupling to the turbulence field.
- Extend from the two-dimensional slice to a finite fireline with
  lateral spread.
- Add a suppression module (water spray, agent application) on top of
  the validated baseline.

## References

- Cheney, N. P., Gould, J. S., Catchpole, W. R. (1993). The influence of
  fuel, weather and fire shape variables on fire-spread in grasslands.
  *International Journal of Wildland Fire* 3(1), 31–44.
- Mell, W., Jenkins, M. A., Gould, J., Cheney, P. (2007). A physics-based
  approach to modelling grassland fires. *International Journal of
  Wildland Fire* 16(1), 1–22.
- Magnussen, B. F. (1981). On the structure of turbulence and a
  generalized eddy dissipation concept for chemical reaction in
  turbulent flow. 19th AIAA Aerospace Sciences Meeting.

# Validation case: Cheney 1993 grassland fires

This is one validation case of the three-dimensional tier of
[FireModel](../README.md), written up end to end: the experiment, how
the case was defined, what "pass" meant before the runs started, what
came out, and what is still wrong.

![Grassland fire spread at U10 = 6 m/s](spread_U10_6.gif)

*Gas temperature in a vertical slice through the fire, natural grass,
10 m wind 6 m/s, dead fuel moisture 6.8%. 100 mm cells in the flame
zone.*

## The experiment

Cheney, Gould and Catchpole (1993) burned 100 m to 200 m square plots of
tropical grassland at Annaburroo, Northern Territory, in natural and cut
pasture, and measured head-fire rate of spread against wind, moisture
and fuel condition. Their Eq. 6 fits the natural-pasture data:

```
R = 0.406 · U₂^0.987 · exp(−0.0707 · M)
```

with R in m/s, U₂ the wind speed at 2 m in m/s, and M the dead fuel
moisture in percent. The fit covers U₂ from 2 to 7 m/s. The paper
reports the 2 m wind; the model takes a 10 m wind and converts with the
paper's own ratio U₂ = 0.723 U₁₀. Getting that convention wrong costs
27 to 38 percent, and it was wrong once in this project.

Reference: Cheney, N. P., Gould, J. S., Catchpole, W. R. (1993).
*International Journal of Wildland Fire* 3(1), 31–44.
Figure 8 of the paper was digitised for the comparison (30 natural and
51 cut points). The digitised values are not rescaled, shifted or
clipped.

## Case definition

Every parameter that defines the case is traced to the paper. Values
are for undisturbed *Eriachne* pasture, the natural-grass treatment.

| Parameter | Value | Source |
|---|---|---|
| Fuel bed height | 0.31 m | Cheney 1993, Table 3 |
| Bulk density | 1.25 kg/m³ | Cheney 1993, Table 3 |
| Fuel load | 0.388 kg/m² | product of the two above |
| Surface-area-to-volume ratio | 9770 m⁻¹ | Cheney 1993, stated in text for undisturbed *Eriachne* |
| Dead fuel moisture | 6.8 % | Cheney 1993, natural-grass mean |
| Wind at 10 m | 4, 6, 8 m/s | chosen; U₂ = 2.9, 4.3, 5.8 m/s, inside the fitted range |

Numerical setup, held fixed across all three winds:

| Setting | Value |
|---|---|
| Domain | 60 m long, 12 m high, one cell wide (an infinitely long fireline) |
| Cell size | 100 mm in the flame zone, stretched above |
| Turbulence | k-epsilon with the Reynolds stress in momentum |
| Combustion | Eddy Dissipation Concept |
| Radiation | discrete ordinates |
| Fuel bed | Lagrangian particles with drying, pyrolysis, char oxidation |
| Ignition | a short line igniter at the upwind end, then nothing |

Holding the domain fixed matters. Earlier sweeps in this project let the
domain length grow with wind speed, which made the measured wind
sensitivity partly a property of the mesh.

## Acceptance, fixed before running

A run passes if its spread rate is within a factor of three of Eq. 6
either way:

```
1/3  ≤  R_model / R_Eq6  ≤  3
```

That band is wide on purpose. It is the scatter of the experiment
itself: the measured points at a given wind span roughly that range.
The band was written down before the runs and has not been changed.
No band was declared for the wind exponent, so it is reported, not
graded.

## Result

![Fixed-domain wind sweep](fixed_domain_wind_sweep.png)

| U₁₀ (m/s) | U₂ (m/s) | Model (m/min) | Eq. 6 (m/min) | Ratio | In band |
|---|---|---|---|---|---|
| 4 | 2.9 | 28.6 | 43.0 | 0.67 | yes |
| 6 | 4.3 | 66.0 | 64.1 | 1.03 | yes |
| 8 | 5.8 | 76.4 | 85.2 | 0.90 | yes |

All three winds land inside the band. Fitted over the three points the
model spread rate goes as U₂ to the power 1.46, against 0.99 in the
experiment, so the model is too sensitive to wind: low at 4 m/s, close
at 6, and would fall low again above 8.

The right-hand panel of the figure asks the question the ratio hides:
is the spread steady? At 6 and 8 m/s the local rate oscillates around a
mean with a period of 7 to 9 s. The model burns in surges, with the
turbulence field leading the surge by about two seconds. At 4 m/s the
rate collapses around 35 m along the bed.

## What is still wrong

These are documented and accepted as limitations rather than tuned
away. Each is being worked as a physics question.

- **Outlet backflow on the 60 m domain.** The open outlet admits inflow
  that reaches about 20 m upstream into the bed from roughly 30 s. The
  rate fit at 6 and 8 m/s is taken over a window that overlaps this, so
  those two ratios are provisional. A 113 m domain with fuel-free
  buffers at both ends is the fix, and the reruns are in progress.
- **Wind exponent 1.46 versus 0.99.** The spread deficit at moderate
  wind traces to two things: averaged turbulence closures smear out the
  intermittent flame contact that preheats the bed, and the fuel
  particles sit hotter than measured before ignition so their residence
  time is short. Neither is a tuning problem.
- **Low wind.** Below roughly 1.5 m/s the model cannot sustain
  propagation at all. Seven independent changes to the closures were
  tried and all failed the same way. This is a closure-class limit, and
  it is handled by blending in the empirical rate at low wind, the same
  approach coupled weather-fire models use.
- **Only the natural-grass treatment is run.** The paper says outright
  that it did not measure the surface-area-to-volume ratio of the cut
  treatments. Running "cut" cases would mean inventing that number.

## What went wrong along the way

An earlier 20-case sweep of this experiment, four fuel conditions by
five winds, reported 18 of 20 inside the band. It was run with a
surface-area-to-volume ratio of 2000 m⁻¹ that had been copied from a
fuel-model table listing 2000 ft⁻¹. The correct value in SI is 6562 m⁻¹
for that table and 9770 m⁻¹ for the *Eriachne* pasture actually burned.
The parameter sets particle diameter, exchange area, drying rate,
convective heating and radiative extinction, so every number from that
sweep was a comparison with a fuel that was not Cheney's. Those results
are withdrawn. The project's naming rule (units in every variable
name, the source value and the conversion arithmetic in a comment on
the same line) exists because of this case.

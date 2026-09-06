# Portfolio

Public-facing documentation for my modelling work. Source code
lives in private repositories; what is published here is background,
methodology, and validation results.

## Projects

### [FireModel](firemodel/) — reduced-order fire spread and burning model

A physics-based model of how solid fuels ignite, burn and spread fire,
built to give a defensible pre-suppression baseline for testing fire
suppression devices. Three tiers: a bench-scale cone-calorimeter model,
a one-dimensional flame-line spread model, and a three-dimensional
reacting-flow solver with a particle-resolved fuel bed.

- [Background](firemodel/README.md) — what the model is, what it is for,
  how the work is run
- [Cheney 1993 grassland fires](firemodel/cheney_1993/README.md) — a
  worked validation case, end to end

![Grassland fire spread at U10 = 6 m/s](firemodel/cheney_1993/spread_U10_6.gif)

## Licence

Text, figures and animations in this repository that are my own work are
released under [CC BY 4.0](LICENSE.md). Experimental data points shown
in figures belong to the cited authors and are reproduced for comparison
only.

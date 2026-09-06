# FireModel — three-dimensional fire spread solver

Reduced-order reacting-flow model of fire spread through a porous fuel
bed. Purpose: a pre-suppression baseline for testing fire suppression
devices. Validation: [Cheney 1993 grassland fires](cheney_1993/README.md).

## Gas phase

Low-Mach, variable density, Reynolds-averaged.

Mass, with the solid-to-gas source:

$$
\partial_t \rho + \nabla\cdot(\rho \mathbf{u}) = \dot{m}'''_{s \to g}
$$

Momentum, with Reynolds stress, buoyancy and bed drag:

$$
\partial_t(\rho\mathbf{u}) + \nabla\cdot(\rho\mathbf{u}\mathbf{u}) = -\nabla p + \nabla\cdot\left[(\mu+\mu_t)\mathbf{S}\right] + (\rho-\rho_\infty)\mathbf{g} - \mathbf{f}_{D}
$$

Species transport, turbulent diffusion, reaction source:

$$
\partial_t(\rho Y_i) + \nabla\cdot(\rho\mathbf{u}Y_i) = \nabla\cdot\left(\rho\left(D+\tfrac{\nu_t}{Sc_t}\right)\nabla Y_i\right) + \dot\omega_i
$$

Enthalpy, with combustion heating, radiation and heat lost to the bed:

$$
\partial_t(\rho h) + \nabla\cdot(\rho\mathbf{u}h) = \nabla\cdot\left(\rho\left(\alpha+\tfrac{\nu_t}{Pr_t}\right)\nabla h\right) + \dot{q}'''_{c} - \nabla\cdot\mathbf{q}_{r} - \dot{q}'''_{s \to g}
$$

Turbulence: standard k-epsilon with shear and buoyancy production,
$\mu_{t} = C_{\mu} \rho k^{2}/\varepsilon$, wall functions at the ground.

## Combustion

Eddy Dissipation Concept, single-step fuel gas + oxygen:

Reaction rate and fine-structure scales:

$$
\dot\omega_F = \rho\frac{\gamma^{\ast}}{\tau^{\ast}} \min\left(Y_F, \frac{Y_{O_2}}{s}\right), \qquad \tau^{\ast} \propto \sqrt{\nu/\varepsilon}, \quad \gamma^{\ast} \propto \left(\frac{\nu\varepsilon}{k^2}\right)^{1/4}
$$

## Radiation

Discrete ordinates on the grey radiative transfer equation, gas and
solid absorbing–emitting:

Radiative transfer, solid extinction from bed geometry:

$$
\mathbf{s}\cdot\nabla I = \kappa\left(\frac{\sigma T^4}{\pi} - I\right), \qquad \kappa = \kappa_g + \kappa_s, \quad \kappa_s = \xi\beta_s\sigma_s
$$

with $\beta_s$ the solid packing ratio, $\sigma_s$ the particle
surface-area-to-volume ratio, $\xi$ an orientation factor.

## Fuel bed

Lagrangian particles, each carrying $T_p$, water mass $m_w$, dry
mass $m_d$, char mass $m_c$:

Particle energy balance:

$$
m_p c_p \frac{dT_p}{dt} = h A_p (T_g - T_p) + A_p\varepsilon_p\left(G/4 - \sigma T_p^4\right) - \dot m_w L_v - \dot m_d \Delta h_{py} + \dot m_c \Delta h_{ox}
$$

Drying and pyrolysis, Arrhenius:

$$
\dot m_w = -A_w m_w \exp\left(-\frac{E_w}{R T_p}\right), \qquad \dot m_d = -A_{py} m_d \exp\left(-\frac{E_{py}}{R T_p}\right)
$$

Char oxidation:

$$
\dot m_c = -A_{ox} m_c Y_{O_2}\exp\left(-\frac{E_{ox}}{R T_p}\right) \quad\text{(diffusion-limited)}
$$

Convection $h$ from a cylinder-in-crossflow Nusselt correlation,
$d = 4/\sigma_s$. Drag $\mathbf{f}_{D}$ from the same geometry. Pyrolysis
gas and water vapour enter the gas-phase source terms.

## Fire front

Front propagation, level set:

$$
\partial_t \phi + v_n|\nabla\phi| = 0
$$

$v_n$ from the resolved bed ignition front; an empirical
$v_n(U, M)$ hybrid is available below a wind threshold and is off in
the published cases.

## Numerics

- Finite volume, MUSCL advection, explicit time stepping with a
  per-cell diffusive limit.
- Pressure projection, separable-FFT preconditioned BiCGSTAB.
- DOM parallelised over ordinates.

## Work to come

- Suppression module on the validated baseline.

## References

- Cheney, Gould, Catchpole (1993). *Int. J. Wildland Fire* 3(1), 31–44.
- Mell, Jenkins, Gould, Cheney (2007). *Int. J. Wildland Fire* 16(1), 1–22.
- Magnussen (1981). 19th AIAA Aerospace Sciences Meeting, AIAA-81-0042.

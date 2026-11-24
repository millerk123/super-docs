Unidirectional Pulse Propagation Equation
=========================================

SUPER solves the unidirectional pulse propagation equation (UPPE) [1]_ for the electric field of
a linearly polarized pulse in a gas, in cylindrical (:math:`r`, 2D) or Cartesian
(:math:`x, y`, 3D) geometry.  This page summarizes the equation and how it is integrated.

The field and the frame
-----------------------

The field is stored as a complex function :math:`E(r, t)` whose real part is the physical field
(:math:`\mathcal{E} = \mathrm{Re}\,E`).  It contains only positive frequencies and varies as
:math:`e^{-i\omega t}`; there is no envelope approximation, so the carrier is resolved on the
time grid.  The time :math:`t` is measured in a frame moving at speed :math:`v_f` (the
:code:`vf` parameter, by default the group velocity at :code:`lambda0`), so that the pulse stays
near the same place in the time window.

In the spectral domain (:math:`\omega`, and :math:`k_\perp` from a Hankel transform in 2D or a
Fourier transform in 3D) the field obeys

.. math::

   \frac{\partial \hat E}{\partial z} = i k_L \hat E + S(\hat E),
   \qquad
   k_L = \sqrt{\frac{n^2(\omega)\,\omega^2}{c^2} - k_\perp^2} - \frac{\omega}{v_f},

where :math:`n(\omega)` is the linear refractive index of the gas (a Sellmeier formula, or
tabulated values when HHG is on) and :math:`S` holds the nonlinear sources:

.. math::

   S = \frac{i\,\omega^2}{2 \epsilon_0 c^2 k_z} \left( \hat P_\mathrm{NL} + \frac{i}{\omega}\,\hat J \right),
   \qquad k_z = \sqrt{\frac{n^2\omega^2}{c^2} - k_\perp^2}.

The sources are computed in real space (:math:`r, t`) and transformed:

* **Bound (Kerr) nonlinearity** (:code:`bound_nonlinearity`): :math:`P_\mathrm{NL} = \tfrac{4}{3}\epsilon_0^2 c\, n_2\, \mathcal{E}^3`,
  using the full real field, so self-phase modulation, self-focusing and third-harmonic
  generation are all included.  The nonlinear index :math:`n_2` is set by the gas and scales
  with its density.
* **Plasma** (:code:`ionization`): the free-electron current obeys
  :math:`\partial_t J + \nu_{ei} J = (e^2/m_e)\, n_e \mathcal{E}`, which gives plasma dispersion
  (defocusing, blue shifting) and collisional absorption (:code:`nu_ei`).
* **Ionization loss** (:code:`ionization`): an extra current
  :math:`J_\mathrm{loss} = I_p\,(\partial_t n_e)\,\mathcal{E}/\mathcal{E}^2` removes the
  ionization energy :math:`I_p` for every electron created.

The electron density :math:`n_e(r, t)` is obtained at every step by integrating a
cycle-resolved tunneling (ADK-type) rate :math:`W(|\mathcal{E}|)` in time,
:math:`\partial_t n_e = W\,(N_0 - n_e)`, where :math:`N_0` is the neutral density.  It is
also computed for the diagnostics when only :code:`compute_density` is set.

Integration in z
----------------

Writing :math:`e = e^{i k_L \Delta z}` and :math:`C = \Delta z / 2`, each step is an
integrating-factor predictor--corrector step: the linear part is propagated exactly, and the
sources are approximated with the trapezoid rule,

.. math::

   \hat E^{*} &= e\,\hat E(z) + C\,w\,S\big(\hat E(z)\big), \\
   \hat E(z + \Delta z) &= e\,\hat E(z) + C\left[ e\,S\big(\hat E(z)\big) + S\big(\hat E^{*}\big)\right],

with the predictor weight :math:`w = 1 + e` by default, or :math:`w = 2e` (the
integrating-factor Heun method) with :code:`heun_predictor` = true.  The two agree when the
phase advance per step, :math:`|k_L \Delta z|`, is small.  See :code:`heun_predictor` in
:doc:`input/laser` for when they differ.

After the corrector, the spectral filters and then the real-space edge filters are applied on
the steps set by :code:`nz_filt_omega` and :code:`nz_filt_t`, followed by the ionization loss
(see :ref:`Filters <sec-filters>`).

Choosing the step size
----------------------

Because the linear part is exact, :math:`\Delta z` is limited only by the sources:

* the nonlinear phase per step, :math:`k_0 n_2 I \Delta z`, should be small (a few hundredths
  of a radian or less near a collapse);
* the plasma phase per step, :math:`\gamma = \Delta z\, \omega_p^2 / (2 c^2 k_z)`, must stay
  below about 1 at every frequency that is kept.  This is the binding limit for THz and other
  low frequencies in a dense plasma (see :ref:`Stability <sec-filter-stability>`).

In practice, halve :math:`\Delta z` and check that the quantities of interest do not change.

References
----------

.. [1] M\. Kolesik, J.V. Moloney, and M. Mlejnek, `"Unidirectional Optical Pulse Propagation Equation," <https://doi.org/10.1103/PhysRevLett.89.283902>`_ *Phys. Rev. Lett.* **89**\ (28), 283902 (2002).

High-Harmonic Generation
========================

SUPER can compute high-harmonic generation (HHG) driven by the propagating laser pulse, and
propagate the harmonic field on its own grid.  HHG is turned on by adding the optional
:ref:`HHG <sec-hhg-grid>` section to :code:`grid`; that section also lists all of its
parameters.  The available gases are hydrogen, helium and argon.

How it works
------------

At every main step in :math:`z`:

1. **Driving field.**  The main field is interpolated onto the HHG grid: a cubic spline in
   :math:`r` onto the HHG radii (:code:`nr` points out to the HHG :code:`rmax`), and
   zero-padding in frequency onto the finer HHG time grid (:code:`nt` points over the same time
   window, so that high harmonics are resolved).
2. **Single-atom response.**  The dipole moment :math:`d(r, t)` of each atom is computed with
   the Lewenstein (strong-field approximation) model, with the excursion-time integral taken
   over one laser period with a smooth window.  Ground-state depletion is included through
   the electron density :math:`n_e(r, t)`, computed on the HHG grid from the interpolated field
   with the same tunneling rate as on the main grid.  The source of the harmonic field is the
   polarization :math:`P = N_0\,d`.
3. **Harmonic propagation.**  The harmonic field is advanced :code:`nz_mult` times per main
   step with a single-stage integrating-factor step (the predictor of :doc:`uppe`, with the
   source weight set by :code:`heun_predictor` for the plasma term), with:

   * the XUV refractive index of the neutral gas, from tabulated atomic data, including
     absorption (:code:`absorb`; :code:`modabs` makes the absorption follow the neutral
     density as the gas ionizes);
   * the plasma dispersion of the free electrons (:math:`n_e` from step 2);
   * its own filters (:ref:`HHG filters <sec-hhg-filters>`).  By default the HHG grid has the
     main-grid edge filters, no angle filter, and an :math:`\omega_0` roll-on (1.5--2
     :math:`\omega_0`) that removes the fundamental, which is propagated on the main grid.

The harmonic field does not act back on the main field.

Setting up a run
----------------

* **Time grid.**  The HHG :code:`nt` sets the highest resolved harmonic: the largest frequency
  is about :math:`\pi\, n_t / (t_\mathrm{max} - t_\mathrm{min})`.  It must be larger than the
  main :code:`nt`, with the same parity, and divisible by the number of MPI ranks.
* **Radial grid.**  The HHG :code:`rmax` only needs to cover the region where harmonics are
  generated and propagate, so it is usually much smaller than the main :code:`rmax`, and
  :code:`nr` much smaller than the main :code:`nr`.  Check that the harmonic beam does not
  reach the HHG edge filter: if the yields change when :code:`rmax` or the edge filter
  changes, the HHG grid is too narrow.
* **Step.**  :code:`nz_mult` sub-steps are taken per main step.  The filter cadences
  :code:`nz_filt_t` and :code:`nz_filt_omega` of the HHG grid count these sub-steps and default
  to :code:`nz_mult` times the main-grid values, so the filtering per unit length does not
  depend on :code:`nz_mult`.  Check convergence by increasing :code:`nz_mult`.
* **Energy diagnostic.**  :code:`hhgU` only counts frequencies above
  :code:`ene_omega0_fac` :math:`\omega_0` (default 20), so that it measures the harmonics of
  interest rather than the low-order ones.

Outputs
-------

The HHG diagnostics (:code:`hhgEOA`, :code:`hhgEF`, :code:`hhgEfinal`, :code:`hhgU`,
:code:`dOA`, :code:`dhatOA`, :code:`dfirst`, :code:`dhatfirst`) are described in
:doc:`output`.

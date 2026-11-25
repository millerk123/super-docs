Laser
========

This documentation is for the section :code:`laser` in the main :doc:`input file <../input-file>`.

* :code:`bound_nonlinearity`, logical, default = false
* :code:`ionization`, logical, default = false
* :code:`compute_density`, logical, default = true
* :code:`heun_predictor`, logical, default = false
* :code:`solver`, character, default = "direct"
* Filters (see :ref:`Filters <sec-filters>`)

	* :code:`filt_t_size_left`, real, default = 0.1
	* :code:`filt_t_size_right`, real, default = 0.1
	* :code:`filt_r_size`, real, default = 0.1 (2D)
	* :code:`filt_x_size`, :code:`filt_y_size`, real, default = 0.05 (3D)
	* :code:`filt_omega_high_min`, real, default = 0.9
	* :code:`filt_omega_high_max`, real, default = 1.0
	* :code:`filt_k_high_min`, real, default = 0.9
	* :code:`filt_k_high_max`, real, default = 1.0
	* :code:`filt_angle_min`, real, default = :math:`\theta_\mathrm{max}`
	* :code:`filt_angle_max`, real, default = :code:`filt_angle_min` + 10
	* :code:`filt_omega_grid_min`, real, default = 1.0
	* :code:`filt_omega_grid_max`, real, default = 10.0
	* :code:`nz_filt_t`, integer, default = 2
	* :code:`nz_filt_omega`, integer, default = 2

* :code:`pulses`, array of sections, **required parameter**

	Quantities available in every type of pulse:

	* :code:`type`, character, default = "standard"
	* :code:`ell`, integer, default = 0
	* :code:`lambda0`, real, default = 1.054e-6
	* :code:`I0`, real, **required parameter**
	* :code:`wf`, real, **required parameter**
	* :code:`zf`, real, default = 0.0
	* :code:`phase`, real, default = 0.0
	* :code:`tcent`, real, **required parameter**
	* :code:`tpulse`, real, **required parameter**
	* :code:`tpow`, integer, default = 2

	Additional quantities specific to pulses of :code:`type` "ideal flying focus" and "axi-echelon flying focus" (both 2D only for now):

	* :code:`rmaxf_lens`, real, default = 1.5
	* :code:`vI`, real, default = 1.0
	* :code:`f0`, real, default = 8.0
	* :code:`nr_lens`, integer, **required parameter** (2D)

	Additional quantities specific to pulses of :code:`type` "axi-echelon flying focus":

	* :code:`echelon`, logical, default = true
	* :code:`Rap`, real, **required parameter**
	* :code:`Lap`, real, **required parameter**
	* :code:`lambdaD`, real, default = :code:`lambda0` of the pulse
	* :code:`nlambfact`, real, default = 1.0

:code:`bound_nonlinearity` --- Indicates whether to propagate with the :math:`E^3` term on the RHS of the :doc:`UPPE <../uppe>`.

:code:`ionization` --- Indicates whether to propagate with the ionization term on the RHS of the :doc:`UPPE <../uppe>`.

:code:`compute_density` --- Whether to compute the plasma density for the diagnostics even when :code:`ionization` is false.

:code:`heun_predictor` --- Which weight to use for the source terms in the predictor stage of the step.  The default (false) uses :math:`C\,(1 + e^{i k_L \Delta z})`, which assumes that the source moves with the frame (true for harmonics and THz driven by the pulse).  For a term that acts on the field itself (plasma dispersion, self-phase modulation) in a mode whose phase advances a lot per step (:math:`|k_L \Delta z| \gtrsim 1`, e.g. high angles), this makes the step grow by up to :math:`\gamma^2` per step instead of :math:`\gamma^4/4`, where :math:`\gamma = \Delta z\, \omega_p^2 / (2 c^2 k_z)`.  Setting this to true uses :math:`2 C e^{i k_L \Delta z}` (the integrating-factor Heun method), which keeps the growth at :math:`\gamma^4/4` for every mode.  It also applies to the plasma term on the HHG grid, which only has a single (predictor) stage: there the default grows by up to :math:`\gamma` per step and the Heun form by :math:`\gamma^2`.  See :ref:`Stability <sec-filter-stability>`.  Only used by the "direct" solver (and on the HHG grid).

:code:`solver` --- The integrator in :math:`z` for the main grid (see :doc:`../uppe`).

* "direct" (or "rk2"): the integrating-factor predictor--corrector step (2nd order in :math:`\Delta z`).
* "rk4": the integrating-factor (Lawson) 4th-order Runge--Kutta step.  It costs about twice as much per step (four source evaluations instead of two), is 4th order in :math:`\Delta z` apart from the ionization loss (applied as a split step, see :ref:`ionization loss <sec-ionization-loss>`), and is stable for a larger plasma phase per step.  On a THz test (a two-color flying focus in argon), the two solvers gave the same THz at the same :math:`\Delta z` (to 0.1%), and RK4 was unstable at twice the step, so "direct" is the better choice there.

.. _sec-ionization-loss:

Ionization loss (with :code:`ionization`; see :doc:`../uppe`) --- the ionization-loss current :math:`J_i = U_I\,(\partial_t n_e)/\mathcal{E}` is applied as a damping of the physical field: :math:`\mathrm{Re}\,E` is multiplied by :math:`\exp[-U_I W n_n \Delta z / (2 \epsilon_0 c\, \mathcal{E}^2)]` (the paraxial UPPE coefficient) after each step (and in the predictor or stages), and the complex field is rebuilt from it.  This is energy-consistent and costs one extra transform per step, but is a separate split step (1st order in :math:`\Delta z` for the loss).

Earlier versions multiplied the complex field by :math:`\exp[-U_I W n_n \Delta z / (\epsilon_0 c\, \mathcal{E}^2)]`, an envelope coefficient used with the instantaneous field.  That loss was too strong by a factor between 1 and 2 that depends on how sharply the ionization peaks within each cycle: about 4--20% at the fundamental for argon between :math:`3\times10^{13}` and :math:`10^{15}` W/cm\ :sup:`2`, and 2 for slowly varying fields such as THz.

.. _sec-filters:

Filters
-------

There are two kinds of filters:

* **Edge filters** multiply the field in real space and act as absorbing layers at the edges of the time window and at :math:`r_\mathrm{max}` (or the :math:`x` and :math:`y` edges in 3D).  They are applied every :code:`nz_filt_t` steps.
* **Spectral filters** multiply the field in :math:`(\omega, k_\perp)` space: low frequencies, high frequencies, high transverse wavenumbers and large angles.  They are combined into one smooth filter, applied every :code:`nz_filt_omega` steps.

All roll-offs have a :math:`\cos^2` shape.  At points beyond the hard limits (evanescent waves, angle :math:`\ge` :code:`filt_angle_max`, :math:`\omega \le` :code:`filt_omega_grid_min` :math:`\omega_\mathrm{grid}`, :math:`\omega \ge` :code:`filt_omega_high_max` :math:`\omega_\mathrm{max}`, :math:`k_\perp \ge` :code:`filt_k_high_max` :math:`k_\mathrm{max}`), propagation and all sources are also zeroed in every step.  Negative frequencies are always excluded.

The filters are only applied to the new field at the end of each step (the predictor of the "direct" solver and the intermediate stages of "rk4" are not filtered): the spectral filter is applied to the field at :math:`z + \Delta z` (on steps where ``mod(iz, nz_filt_omega) == 0``), then the field is transformed to real space, the edge filter is applied (on steps where ``mod(iz, nz_filt_t) == 0``) and finally the ionization loss (with :code:`ionization`).

:code:`filt_t_size_left` --- The fraction of the total :math:`t` grid over which the field is tapered to zero (with :math:`\cos^2`) at the left edge.  For example, 0.1 for both :code:`filt_t_size_left` and :code:`filt_t_size_right` leaves the middle 80% of the time grid untouched.  The same time filter is applied to the initial pulse.

:code:`filt_t_size_right` --- The same as :code:`filt_t_size_left`, for the right edge of the time window.

:code:`filt_r_size` --- The fraction of the :math:`r` grid over which the field is tapered to zero at :math:`r_\mathrm{max}` (2D).

:code:`filt_x_size`, :code:`filt_y_size` --- The fraction of the :math:`x` (:math:`y`) grid over which the field is tapered to zero at *each* edge (3D only).

:code:`filt_omega_high_min` --- The multiple of the largest positive frequency on the grid, :math:`\omega_\mathrm{max} = (n_t/2 - 1)\,\omega_\mathrm{grid} \approx \pi / \Delta t`, at which to begin filtering out high-frequency content.

:code:`filt_omega_high_max` --- The multiple of :math:`\omega_\mathrm{max}` above which high-frequency content is removed.

:code:`filt_k_high_min` --- The multiple of the largest transverse wavenumber (:math:`k_\mathrm{max} = j_{0,n_r} / r_\mathrm{max}` in 2D; in 3D, the Nyquist wavenumbers in :math:`x` and :math:`y`, filtered separately) at which to begin filtering out high-wavenumber content.

:code:`filt_k_high_max` --- The multiple of :math:`k_\mathrm{max}` above which high-wavenumber content is removed.

:code:`filt_angle_min` --- The angle (degrees) from the :math:`z` axis, :math:`\theta = \tan^{-1}[k_\perp / \mathrm{Re}(k_z)]`, at which to begin filtering out spectral content.  Defaults to :math:`\theta_\mathrm{max} = \tan^{-1}[r_\mathrm{max} / (z_\mathrm{max} - z_\mathrm{min})]`, the angle at which light leaving the axis at the first :math:`z` reaches the radial boundary at the last :math:`z` (in 3D, half of the smaller transverse width is used for :math:`r_\mathrm{max}`).

:code:`filt_angle_max` --- The angle (degrees) above which spectral content is removed.  Defaults to :code:`filt_angle_min` + 10 (at most 90).

:code:`filt_omega_grid_min` --- (Main grid only.) The multiple of the grid frequency, :math:`\omega_\mathrm{grid} = 2\pi / (n_t \Delta t)`, below which low-frequency content is removed.  The default of 1 removes the DC and first frequency bins.

:code:`filt_omega_grid_max` --- The multiple of :math:`\omega_\mathrm{grid}` above which low frequencies are kept untouched; between :code:`filt_omega_grid_min` and :code:`filt_omega_grid_max` the filter rises as :math:`\cos^2`.

:code:`nz_filt_t` --- Apply the edge filters every :code:`nz_filt_t` steps in :math:`z`.  Filtering at every step can be too aggressive.

:code:`nz_filt_omega` --- Apply the spectral filters every :code:`nz_filt_omega` steps in :math:`z`.

The values that are used (including the computed default angle) are printed when the input file is read.

.. note::

   The old parameters :code:`zero_omega_ratio`, :code:`zero_angle` and :code:`zero_omega_ratio_hhg` (and :code:`filtdt`, :code:`filtdr`, :code:`filtdx`, :code:`filtdy` in the :doc:`grid <grid>` section) have been removed; SUPER stops with an error if they are in the input file.

.. _sec-filter-stability:

Choosing the filters
********************

A few things to keep in mind:

* **Anything inside a roll-off is removed over a long run.**  A filter with value :math:`f < 1` is applied :math:`N = n_z / n_\mathrm{filt}` times, so content that stays in the roll-off region is multiplied by :math:`f^N`.  With the default :code:`nz_filt_omega` = 2 and a few thousand steps, even :math:`f = 0.99` removes everything.  The smooth shape reduces ringing, but the effective cutoff is the *start* of the roll-off (:code:`filt_omega_grid_max`, :code:`filt_angle_min`, :code:`filt_omega_high_min`, :code:`filt_k_high_min`).  Put the physics you care about outside the roll-off regions.
* **THz and other low frequencies.**  The default low-frequency roll-on reaches 1 at 10 :math:`\omega_\mathrm{grid}`.  For THz radiation at about 10 :math:`\omega_\mathrm{grid}`, lower :code:`filt_omega_grid_max` (e.g. 2--3), or lengthen the time window so that the THz is at many times :math:`\omega_\mathrm{grid}`.
* **The plasma term is stiff at low frequency.**  Per step, the plasma term changes the phase of a mode by :math:`\gamma = \Delta z\, \omega_p^2 / (2 c^2 k_z)`, which grows as :math:`1/\omega` (:math:`k_z \approx \omega/c`).  The step is only stable for :math:`\gamma \lesssim 1`; for larger values the field grows by about :math:`\gamma^4/4` per step no matter which filters or predictor are used.  So the step size must satisfy :math:`\Delta z \lesssim 2 c\, \omega / \omega_p^2` at the lowest frequency that is kept.  For example, at :math:`n_e = 10^{24}~\mathrm{m^{-3}}` (:math:`\omega_p/2\pi \approx 9` THz), keeping 1 THz needs :math:`\Delta z \lesssim 1.2~\mu`\ m, and 10 THz needs :math:`\Delta z \lesssim 12~\mu`\ m.  Frequencies below this limit must be either filtered out (:code:`filt_omega_grid_min`) or resolved with a smaller :math:`\Delta z`.
* **High angles need a resolved step.**  For modes with :math:`|k_L \Delta z| \gtrsim 1` (high angles, or frequencies far from :code:`lambda0`), use :code:`heun_predictor` = true, or remove them with the angle filter.

.. _sec-pulses:

Pulses
------

The :code:`pulses` parameter is an array that contains any number (at least one) of laser-pulse sections to be injected onto the main field grid.  There are currently three types of pulses that can be specified in these sections, given below.

:code:`type` --- The type of laser pulse to inject.  Must be one of the following:

* "standard" or "std" -- :ref:`Standard laser pulse <sec-standard-pulse>` initialized in the far field.
* "ideal flying focus" or "ff" -- :ref:`Ideal flying-focus pulse <sec-ideal-ff-pulse>`.
* "axi-echelon flying focus" or "ae" -- :ref:`Axiparabola--echelon flying-focus pulse <sec-ae-pulse>`.

.. _sec-standard-pulse:

Standard Pulse
**************

The standard pulse is selected by setting :code:`type` to "standard" or "std".  This type of pulse is initialized completely in the far field.  A Gaussian pulse first is initialized at focus, where the temporal profile can have a custom power :code:`tpow` to make a super-Gaussain profile.  The pulse is then transformed to :math:`\omega`--:math:`k` space and propagated to the beginning of the simulation assuming vacuum propagation.  The available parameters for the standard pulse type are listed below.

:code:`ell` --- Orbital angular momentum of the pulse.  Only 0 is allowed in 2D; in 3D other values are accepted but not yet used.

:code:`lambda0` --- Central wavelength of the pulse.

:code:`I0` --- Peak intensity of the pulse *at focus* in :math:`\mathrm{W}/\mathrm{cm}^2`.

:code:`wf` --- Spot size at focus in meters.  The field profile is proportional to :math:`\exp(-[r/w_\mathrm{f}]^2)` at the focus.

:code:`zf` --- The focal position in meters.

:code:`phase` --- Phase constant added to the field profile (in degrees).

:code:`tcent` --- Temporal center of the pulse in seconds.  This value should normally be within the limits of :code:`tmin` and :code:`tmax` from the :doc:`grid <grid>` section.

:code:`tpulse` --- Pulse duration in seconds.  The field profile is proportional to :math:`\exp(-\{[t-t_\mathrm{cent}]/t_\mathrm{pulse}\}^{t_\mathrm{pow}})`.  Therefore, when :code:`tpow` is 2 then the full width at half maximum of the intensity is :math:`\mathrm{FWHM} = \sqrt{2\ln 2} t_\mathrm{pulse}`.

:code:`tpow` --- Power for the temporal profile of the field, which is proportional to :math:`\exp(-\{[t-t_\mathrm{cent}]/t_\mathrm{pulse}\}^{t_\mathrm{pow}})`.


.. _sec-ideal-ff-pulse:

Ideal Flying-Focus Pulse
************************

The ideal flying-focus pulse is selected by setting :code:`type` to "ideal flying focus" or "ff".  It is only available in 2D.  More information about the ideal flying focus can be found in Ref. [1]_.  In addition to all the parameters available for a :ref:`standard pulse <sec-standard-pulse>`, the parameters below are also available.

:code:`rmaxf_lens` --- The factor that multiplies the laser :math:`w_0` at the lens plane to determine :math:`r_\mathrm{max}` for the lens plane.  The ideal flying focus uses a field profile proportional to :math:`\exp(-[r/w_0]^2)` in the lens plane, where :math:`w_0` is calculated appropriately based on :code:`wf` and :code:`f0`.  It is therefore recommended that :code:`rmaxf_lens` be large enough to include several spot sizes at the lens, i.e., 3 or greater.

:code:`vI` --- Velocity of the intensity peak in units of :math:`c`.  Can be positive or negative.

:code:`f0` --- Nominal focal length of the lens in meters.

:code:`nr_lens` --- Number of points in :math:`r` in the lens plane.  This is recommended to be of order 1000.

.. _sec-ae-pulse:

Axiparabola--Echelon Pulse
**************************

The axiparabola--echelon pulse is selected by setting :code:`type` to "axi-echelon flying focus" or "ae".  It is only available in 2D.  More information about the axiparabola--echelon flying-focus pulse can be found in Ref. [2]_.  In addition to all the parameters available for a :ref:`standard pulse <sec-standard-pulse>`, the parameters below are also available.  Note that some of these parameters overlap with those available for an :ref:`ideal flying-focus pulse <sec-ideal-ff-pulse>`; they are duplicated here for clarity.

:code:`rmaxf_lens` --- The factor that multiplies the laser :math:`w_0` at the lens plane to determine :math:`r_\mathrm{max}` for the lens plane.  The axiparabola--echelon flying focus uses a field profile proportional to :math:`\exp(-[r/w_0]^{16})` in the lens plane, where :math:`w_0` is calculated appropriately based on :code:`wf` and :code:`f0`.  The value of :code:`rmaxf_lens` can therefore be smaller than for the :ref:`ideal flying-focus pulse <sec-ideal-ff-pulse>`, i.e., around 1.3.

:code:`vI` --- Velocity of the intensity peak in units of :math:`c`.  Can be positive or negative.

:code:`f0` --- Nominal focal length of the lens in meters.

:code:`nr_lens` --- Number of points in :math:`r` in the lens plane.  This is recommended to be of order 1000.

:code:`echelon` --- Whether to use an echelon to impart a radial group delay.

:code:`Rap` --- Radius of the axiparabola in meters.  Note that the :math:`r` grid in the lens plane is unaffected by this value.  It is only used to calculate the sag function of the axiparabola and the echelon profile.  For best results, this value should be set to :math:`R_\mathrm{ap} = f_0 \lambda / \pi w_\mathrm{f}`.

:code:`Lap` --- Length of the extended focal region in meters.

:code:`lambdaD` --- The wavelength (in meters) for which the echelon is designed.

:code:`nlambfact` --- Number of half-wavelengths (corresponding to :code:`lambdaD`) per step of the echelon.  This parameter should usually be set to 1.

:code:`rpow` --- Power for the radial profile of the field at the lens, which is proportional to :math:`\exp[-(r_\mathrm{lens}/w_0)^{r_\mathrm{pow}}]`\ .  The value of :code:`rpow` defaults to 2.

:code:`Rmin` --- The inner radius of the optical assembly, inside which the beam is apodized (blocked).  If left unspecified, :code:`Rmin` defaults to 0 and the full aperture is used.

:code:`Rmin_pow` --- The power for the radial profile of the inner cutoff.  Defaults to :code:`rpow`, but having this as a separate parameter allows for a sharp cutoff in the center and a more gradual one (e.g., Gaussian) on the outer edge.

:code:`rpow_2` --- A second power to also multiply in to the field at the lens, similar to what is done for :code:`rpow`.  The value of :code:`rpow_2` defaults to 0 and has no effect.  This parameter could be useful, for example, to simulate a Gaussian pulse profile incident on a lens with a hard radial cutoff.  In that case, the parameters :code:`rpow = 2` and :code:`rpow_2 = 40` could be used.

:code:`w0_2` --- The spot size in the exponential applied with the :code:`rpow_2` parameter.  The value of :code:`w0_2` defaults to :code:`Rap`.


References
----------

.. [1] D\. Ramsey, *et al.*, `"Exact solutions for the electromagnetic fields of a flying focus," <https://doi.org/10.1103/PhysRevA.107.013513>`_ *Phys. Rev. A* **107**\ (1), 013513 (2023).

.. [2] M\. V. Ambat, *et al.*, `"Programmable-trajectory ultrafast flying focus pulses," <https://doi.org/10.1364/OE.499839>`_ *Opt. Express* **31**\ (19), 31354 (2023).
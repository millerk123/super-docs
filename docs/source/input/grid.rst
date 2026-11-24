Grid
========

This documentation is for the section :code:`grid` in the main :doc:`input file <../input-file>`.

* :code:`vf`, real, default = :math:`v_\mathrm{f}(\omega_0)`
* :code:`lambda0`, real, default = 1.054e-6
* :code:`tmin`, real, default = 0.0
* :code:`tmax`, real, **required parameter**
* :code:`nt`, integer, **required parameter**
* :code:`xmin`, real(2), **required parameter**
* :code:`xmax`, real(2), **required parameter**
* :code:`nx`, integer(2), **required parameter**

* :code:`HHG`, section, optional
	* :code:`nz_mult`, integer, default = 0
	* :code:`nr`, integer, default = 0
	* :code:`nt`, integer, default = 0
	* :code:`rmax`, real, default = 0.0
	* :code:`absorb`, logical, default = true
	* :code:`modabs`, logical, default = false
	* :code:`absskip`, integer, default = 1
	* Filter parameters (optional overrides, see :ref:`HHG filters <sec-hhg-filters>`)

.. note::

   The filters are set in the :doc:`laser <laser>` section, (see :ref:`Filters <sec-filters>`).  The old parameters :code:`filtdt`, :code:`filtdr`, :code:`filtdx` and :code:`filtdy` have been removed; SUPER stops with an error if they are in the input file.

:code:`vf` --- The frame velocity for the moving window.  If left unspecified, the frame velocity will be set to the group velocity corresponding to the :code:`lambda0` parameter.  This is optimal for stability, so it is recommended to not set this parameter.

:code:`lambda0` --- The fundamental wavelength of the grid.  This determines things like the calculation of the nominal group velocity, :math:`a_0`, :math:`P_\mathrm{c}`, the Keldysh parameter, and for high-harmonic generation quantities.

:code:`tmin` --- The beginning time of the simulation (in seconds).

:code:`tmax` --- The end time of the simulation (in seconds).

:code:`nt` --- The number of points in time.  Note that FFTW is best at handling sizes of the form :math:`2^a 3^b 5^c 7^d 11^e 13^f`, where :math:`e+f` is either 0 or 1, and the other exponents are arbitrary.  The value of :code:`nt` must be divisible by the number of MPI ranks, :code:`node_number`.

:code:`xmin(1:2)` --- An array specifying :math:`(z_\mathrm{min}, r_\mathrm{min})` for the simulation (in meters).  The value of :code:`xmin(2)` should normally be set to 0.

:code:`xmax(1:2)` --- An array specifying :math:`(z_\mathrm{max}, r_\mathrm{max})` for the simulation (in meters).

:code:`nx(1:2)` --- The number of points in :math:`z` and :math:`r`, respectively.  The value of :code:`nx(2)` must be divisible by the number of MPI ranks, :code:`node_number`.

.. _sec-hhg-grid:

HHG
---

The :code:`hhg` section is an optional section that enables the simulation of high-harmonic generation.  See the :doc:`../hhg` page for more information.  Below are the available parameters in this section.

:code:`nz_mult` --- The HHG field is evolved :code:`nz_mult` times more frequently in :math:`z` than is the standard field.  A typical value of this parameter would be 10.

:code:`nr` --- The number of radial points for the HHG grid.  This value must be less than :code:`nx(2)` and is typically much smaller than :code:`nx(2)` for the main grid.  The value of :code:`nr` must be divisible by the number of MPI ranks, :code:`node_number`.

:code:`nt` --- The number of points in time for the HHG grid.  It must be larger than :code:`nt` of the main grid, have the same parity, and be divisible by the number of MPI ranks.  The HHG time grid spans the same window as the main grid, so the largest resolved harmonic is set by this value.

:code:`rmax` --- The maximum radius for the HHG grid (in meters).  This value must be less than :code:`xmax(2)`.  The values of :code:`rmax` and :code:`nr` must be chosen such that the first radial cell of the HHG grid is larger than the first radial cell of the main grid.

:code:`absorb` --- Whether to include absorption by the gas (the imaginary part of the index of refraction) in HHG propagation.

:code:`modabs` --- How the absorption of the HHG light by the neutral gas (the imaginary part of the XUV refractive index) is included.  Requires :code:`absorb` to be true.

* false: the imaginary part is included in the refractive index used for :math:`k_z = \sqrt{n^2\omega^2/c^2 - k_\perp^2}`.  The absorption is then fixed by the initial neutral density and does not drop as the gas ionizes.
* true: the imaginary part is left out of :math:`k_z` and the absorption is applied instead as a Beer's-law attenuation proportional to the local neutral density.  The absorption then drops in proportion to the neutral density as the gas ionizes.

The two only differ noticeably when the ionization fraction is high.

:code:`ene_omega0_fac` --- Default 20.  Only frequencies :math:`\omega \ge` :code:`ene_omega0_fac` :math:`\omega_0` are included in the HHG energy diagnostic (:code:`hhgU`).  The cutoff is sharp.  Set it to 0 to include all frequencies.

:code:`absskip` --- Specifies that modified absorption should be calculated every :code:`absskip` steps.  Note that this skipping refers to the frequent HHG steps, not the global grid steps.  For example, if :code:`nz_mult` is 10 and :code:`absskip` is 5, then for every main field advance the HHG grid will be advanced 10 times, and modified absorption will be calculated 2 of those 10 times.

.. _sec-hhg-filters:

HHG filters
***********

The HHG field is filtered the same way as the main field (see :ref:`Filters <sec-filters>`), and the filter parameters of the :doc:`laser <laser>` section can be overridden in the :code:`HHG` section.  Unset values take the value used for the main grid, with these exceptions:

* :code:`filt_angle_min` and :code:`filt_angle_max` default to 90 (no angle filter), because the default angle of the main grid (set by the main-grid :math:`r_\mathrm{max}`) is far too small for the HHG grid.
* :code:`nz_filt_t` and :code:`nz_filt_omega` default to :code:`nz_mult` times the main-grid values, so that the HHG field is filtered as often per unit length as the main field (see below).
* :code:`filt_omega_grid_min` and :code:`filt_omega_grid_max` are not used on the HHG grid, and SUPER stops with an error if they are given in the :code:`HHG` section.  The :math:`\omega_0` roll-on below takes their place.

The HHG section has one additional filter, which is not available on the main grid:

* :code:`filt_omega0_min`, real, default = 1.5
* :code:`filt_omega0_max`, real, default = 2.0

:code:`filt_omega0_min`, :code:`filt_omega0_max` --- A low-frequency roll-on in multiples of :math:`\omega_0` (set by the grid :code:`lambda0`): content below :code:`filt_omega0_min` :math:`\omega_0` is removed, content above :code:`filt_omega0_max` :math:`\omega_0` is kept, with a :math:`\cos^2` rise in between.  The defaults remove the fundamental, which is already propagated on the main grid.

On the HHG grid, :code:`nz_filt_t` and :code:`nz_filt_omega` are numbers of HHG sub-steps (there are :code:`nz_mult` sub-steps per main step).  For example, with :code:`nz_mult` = 10 and :code:`nz_filt_omega` = 5, the HHG spectral filter is applied twice per main step.  If you set them yourself and later change :code:`nz_mult`, scale them too, or the filtering per unit length changes.  The time filter (:code:`filt_t_size_left`, :code:`filt_t_size_right`) also windows the vector potential in the Lewenstein dipole calculation.

The old :code:`zero_omega_ratio_hhg` parameter has been removed; SUPER stops with an error if it is given.

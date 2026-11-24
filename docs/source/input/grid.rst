Grid
========

This documentation is for the section :code:`grid` in the main :doc:`input file <../input-file>`.

* :code:`vf`, real, default = :math:`v_\mathrm{f}(\omega_0)`
* :code:`lambda0`, real, default = 1.054e-6
* :code:`tmin`, real, default = 0.0
* :code:`tmax`, real, **required parameter**
* :code:`nt`, integer, **required parameter**
* :code:`zmin`, real, default = 0.0
* :code:`zmax`, real, **required parameter**
* :code:`nz`, integer, **required parameter**
* 2D (cylindrical, :math:`r`--:math:`z`):

	* :code:`rmax`, real, **required parameter**
	* :code:`nr`, integer, **required parameter**

* 3D (Cartesian, :math:`x`--:math:`y`--:math:`z`):

	* :code:`xmin`, real, default = :code:`-xmax`
	* :code:`xmax`, real, **required parameter**
	* :code:`nx`, integer, **required parameter**
	* :code:`ymin`, real, default = :code:`-ymax`
	* :code:`ymax`, real, **required parameter**
	* :code:`ny`, integer, **required parameter**

* :code:`HHG`, section, optional
	* :code:`nz_mult`, integer, default = 0
	* :code:`nr`, integer, default = 0
	* :code:`nt`, integer, default = 0
	* :code:`rmax`, real, default = 0.0
	* :code:`absorb`, logical, default = true
	* :code:`modabs`, logical, default = false
	* :code:`absskip`, integer, default = 1
	* :code:`ene_omega0_fac`, real, default = 20.0
	* Filter parameters (optional overrides, see :ref:`HHG filters <sec-hhg-filters>`)

.. note::

   The filters are set in the :doc:`laser <laser>` section, (see :ref:`Filters <sec-filters>`).  The old parameters :code:`filtdt`, :code:`filtdr`, :code:`filtdx` and :code:`filtdy` have been removed, as has the old array form of :code:`xmin`, :code:`xmax` and :code:`nx` (one entry per dimension); SUPER stops with an error if they are in the input file.

:code:`vf` --- The frame velocity for the moving window.  If left unspecified, the frame velocity will be set to the group velocity corresponding to the :code:`lambda0` parameter.  This is optimal for stability, so it is recommended to not set this parameter.

:code:`lambda0` --- The fundamental wavelength of the grid.  This determines things like the calculation of the nominal group velocity, :math:`a_0`, :math:`P_\mathrm{c}`, the Keldysh parameter, and for high-harmonic generation quantities.

:code:`tmin` --- The beginning time of the simulation (in seconds).

:code:`tmax` --- The end time of the simulation (in seconds).

:code:`nt` --- The number of points in time.  Note that FFTW is best at handling sizes of the form :math:`2^a 3^b 5^c 7^d 11^e 13^f`, where :math:`e+f` is either 0 or 1, and the other exponents are arbitrary.  The value of :code:`nt` must be divisible by the number of MPI ranks, :code:`node_number`.

:code:`zmin`, :code:`zmax` --- The start and end of the propagation (in meters).

:code:`nz` --- The number of points in :math:`z`, including both ends, so the step is :math:`\Delta z = (z_\mathrm{max} - z_\mathrm{min}) / (n_z - 1)`.  See :doc:`../uppe` for how to choose it.

The geometry is set by which transverse parameters are given: :code:`rmax` and :code:`nr` for a 2D (cylindrical) run, or :code:`xmax`, :code:`nx`, :code:`ymax` and :code:`ny` for a 3D (Cartesian) run.  Giving both is an error.

:code:`rmax` --- (2D) The radius of the grid (in meters).  The field is expanded in :math:`J_0` Bessel modes (a Hankel transform), and the :code:`nr` radial points are the zeros of :math:`J_0` scaled so that the last one is at :code:`rmax`.  The field is azimuthally symmetric (:code:`ell` = 0).

:code:`nr` --- (2D) The number of radial points (and Bessel modes).  It must be divisible by the number of MPI ranks, :code:`node_number`.  The cost of the Hankel transform grows as :math:`n_r^2`.

:code:`xmin`, :code:`xmax`, :code:`ymin`, :code:`ymax` --- (3D) The extent of the transverse grid (in meters).  The minima default to minus the maxima, so a grid centered on the axis only needs :code:`xmax` and :code:`ymax`.  The transverse directions are periodic and handled with FFTs, so the edge filters (:code:`filt_x_size`, :code:`filt_y_size`) act as the absorbing boundaries.

:code:`nx`, :code:`ny` --- (3D) The number of points in :math:`x` and :math:`y`.  Both must be divisible by the number of MPI ranks.

.. _sec-3d:

3D runs
-------

A 3D run solves the same equation on a Cartesian transverse grid, so it can describe fields without azimuthal symmetry.  Compared with 2D:

* Fields need not be azimuthally symmetric.  (The :code:`ell` pulse parameter is accepted in 3D but not yet used: the pulse profiles are the same as in 2D.)
* Only the "standard" pulse is available.  The flying-focus pulses ("ideal flying focus" and "axi-echelon flying focus") are not yet implemented in 3D; SUPER stops with an error if they are used.
* High-harmonic generation (the :code:`HHG` section) is not available.
* The full-field outputs have axes :math:`(t, x, y)` instead of :math:`(r, t)` (see :doc:`../output`).
* The cost per step scales as :math:`n_t n_x n_y` (times logarithms), so 3D runs are much larger than 2D ones; use 2D whenever the problem is azimuthally symmetric.

.. _sec-hhg-grid:

HHG
---

The :code:`hhg` section is an optional section that enables the simulation of high-harmonic generation.  See the :doc:`../hhg` page for more information.  Below are the available parameters in this section.

:code:`nz_mult` --- The HHG field is evolved :code:`nz_mult` times more frequently in :math:`z` than is the standard field.  A typical value of this parameter would be 10.

:code:`nr` --- The number of radial points for the HHG grid.  This value must be less than the main-grid :code:`nr` and is typically much smaller.  The value of :code:`nr` must be divisible by the number of MPI ranks, :code:`node_number`.

:code:`nt` --- The number of points in time for the HHG grid.  It must be larger than :code:`nt` of the main grid, have the same parity, and be divisible by the number of MPI ranks.  The HHG time grid spans the same window as the main grid, so the largest resolved harmonic is set by this value.

:code:`rmax` --- The maximum radius for the HHG grid (in meters).  This value must be less than the main-grid :code:`rmax`.  The values of :code:`rmax` and :code:`nr` must be chosen such that the first radial cell of the HHG grid is larger than the first radial cell of the main grid.

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

* :code:`filt_angle_min` and :code:`filt_angle_max` default to 90 (no angle filter).  The main-grid default, :math:`\arctan(r_\mathrm{max}/L)`, is a geometric limit for the main grid and does not carry over: the same rule with the much smaller HHG :math:`r_\mathrm{max}` may give an angle smaller than desired.  The HHG edge filter in :math:`r` acts as the absorbing boundary instead, and convergence tests may need to be conducted with the :math:`r_\mathrm{max}` value on the HHG grid to determine the proper size.
* :code:`nz_filt_t` and :code:`nz_filt_omega` default to :code:`nz_mult` times the main-grid values, so that the HHG field is filtered as often per unit length as the main field (see below).
* :code:`filt_omega_grid_min` and :code:`filt_omega_grid_max` are not used on the HHG grid, and SUPER stops with an error if they are given in the :code:`HHG` section.  The :math:`\omega_0` roll-on below takes their place.

The HHG section has one additional filter, which is not available on the main grid:

* :code:`filt_omega0_min`, real, default = 1.5
* :code:`filt_omega0_max`, real, default = 2.0

:code:`filt_omega0_min`, :code:`filt_omega0_max` --- A low-frequency roll-on in multiples of :math:`\omega_0` (set by the grid :code:`lambda0`): content below :code:`filt_omega0_min` :math:`\omega_0` is removed, content above :code:`filt_omega0_max` :math:`\omega_0` is kept, with a :math:`\cos^2` rise in between.  The defaults remove the fundamental, which is already propagated on the main grid.

On the HHG grid, :code:`nz_filt_t` and :code:`nz_filt_omega` are numbers of HHG sub-steps (there are :code:`nz_mult` sub-steps per main step).  For example, with :code:`nz_mult` = 10 and :code:`nz_filt_omega` = 5, the HHG spectral filter is applied twice per main step.  If you set them yourself and later change :code:`nz_mult`, scale them too, or the filtering per unit length changes.  The time filter (:code:`filt_t_size_left`, :code:`filt_t_size_right`) also windows the vector potential in the Lewenstein dipole calculation.

The old :code:`zero_omega_ratio_hhg` parameter has been removed; SUPER stops with an error if it is given.

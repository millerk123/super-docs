Grid
========

This documentation is for the section :code:`grid` in the main :doc:`input file <../input-file>`.

* :code:`vf`, real, default = :math:`v_\mathrm{f}(\omega_0)`
* :code:`lambda0`, real, default = 1.054e-6
* :code:`tmin`, real, default = 0.0
* :code:`tmax`, real, **required parameter**
* :code:`nt`, integer, **required parameter**
* :code:`filtdt`, real, default = -1.0
* :code:`xmin`, real(2), **required parameter**
* :code:`xmax`, real(2), **required parameter**
* :code:`nx`, integer(2), **required parameter**
* :code:`filtdr`, real, default = 0.1 * :code:`xmax(2)`

* :code:`HHG`, section, optional
	* :code:`nz_mult`, integer, default = 0
	* :code:`nr`, integer, default = 0
	* :code:`rmax`, real, default = 0.0
	* :code:`modabs`, logical, default = false
	* :code:`absskip`, integer, default = 1

:code:`vf` --- The frame velocity for the moving window.  If left unspecified, the frame velocity will be set to the group velocity corresponding to the :code:`lambda0` parameter.  This is optimal for stability, so it is recommended to not set this parameter.

:code:`lambda0` --- The fundamental wavelength of the grid.  This determines things like the calculation of the nominal group velocity, :math:`a_0`, :math:`P_\mathrm{c}`, the Keldysh parameter, and for high-harmonic generation quantities.

:code:`tmin` --- The beginning time of the simulation (in seconds).

:code:`tmax` --- The end time of the simulation (in seconds).

:code:`nt` --- The number of points in time.  Note that FFTW is best at handling sizes of the form :math:`2^a 3^b 5^c 7^d 11^e 13^f`, where :math:`e+f` is either 0 or 1, and the other exponents are arbitrary.  The value of :code:`nt` must be divisible by the number of MPI ranks, :code:`node_number`.

:code:`filtdt` --- The algorithm applies a time filter to the left and right (temporal) edges of the simulation, and the amplitude of this filter starts at 1 and goes to 0 over a width of :code:`filtdt` (s) at each edge.

:code:`xmin(1:2)` --- An array specifying :math:`(z_\mathrm{min}, r_\mathrm{min})` for the simulation (in meters).  The value of :code:`xmin(2)` should normally be set to 0.

:code:`xmax(1:2)` --- An array specifying :math:`(z_\mathrm{max}, r_\mathrm{max})` for the simulation (in meters).

:code:`nx(1:2)` --- The number of points in :math:`z` and :math:`r`, respectively.  The value of :code:`nx(2)` must be divisible by the number of MPI ranks, :code:`node_number`.

:code:`filtdr` --- The algorithm applies a spatial filter to the top radial boundary of the simulation, and the amplitude of this filter starts at 1 and goes to 0 over a width of :code:`filtdr` (m).

.. _sec-hhg-grid:

HHG
---

The :code:`hhg` section is an optional section that enables the simulation of high-harmonic generation.  See the :doc:`../hhg` page for more information.  Below are the available parameters in this section.

:code:`nz_mult` --- The HHG field is evolved :code:`nz_mult` times more frequently in :math:`z` than is the standard field.  A typical value of this parameter would be 10.

:code:`nr` --- The number of radial points for the HHG grid.  This value must be less than :code:`nx(2)` and is typically much smaller than :code:`nx(2)` for the main grid.  The value of :code:`nr` must be divisible by the number of MPI ranks, :code:`node_number`.

:code:`rmax` --- The maximum radius for the HHG grid (in meters).  This value must be less than :code:`xmax(2)`.  The values of :code:`rmax` and :code:`nr` must be chosen such that the first radial cell of the HHG grid is larger than the first radial cell of the main grid.

:code:`modabs` --- Indicates whether to use modified absorption on the HHG field.

:code:`absskip` --- Specifies that modified absorption should be calculated every :code:`absskip` steps.  Note that this skipping refers to the frequent HHG steps, not the global grid steps.  For example, if :code:`nz_mult` is 10 and :code:`absskip` is 5, then for every main field advance the HHG grid will be advanced 10 times, and modified absorption will be calculated 2 of those 10 times.
Laser
========

This documentation is for the section :code:`laser` in the main :doc:`input file <../input-file>`.

* :code:`bound_nonlinearity`, logical, default = false
* :code:`ionization`, logical, default = false
* :code:`zero_omega_ratio`, real, default = 0.01
* :code:`zero_omega_ratio_hhg`, real, default = 0.1
* :code:`zero_angle`, real, default = 5.0
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

	Additional quantities specific to pulses of :code:`type` "ideal flying focus" and "axi-echelon flying focus":

	* :code:`rmaxf_lens`, real, default = 1.5
	* :code:`vI`, real, default = 1.0
	* :code:`f0`, real, default = 8.0
	* :code:`nr_lens`, integer, **required parameter**

	Additional quantities specific to pulses of :code:`type` "axi-echelon flying focus":

	* :code:`echelon`, logical, default = true
	* :code:`Rap`, real, **required parameter**
	* :code:`Lap`, real, **required parameter**
	* :code:`lambdaD`, real, default = :code:`lambda0` of the pulse
	* :code:`nlambfact`, real, default = 1.0

:code:`bound_nonlinearity` --- Indicates whether to propagate with the :math:`E^3` term on the RHS of the :doc:`UPPE <../uppe>`.

:code:`ionization` --- Indicates whether to propagate with the ionization term on the RHS of the :doc:`UPPE <../uppe>`.

:code:`zero_omega_ratio` --- Frequencies below :code:`omega0 * zero_omega_ratio` are zeroed out during propagation, where :code:`omega0` is determined by the grid :code:`lambda0`.

:code:`zero_omega_ratio_hhg` --- Frequencies below :code:`omega0 * zero_omega_ratio` in the HHG field are zeroed out during propagation, where :code:`omega0` is determined by the grid :code:`lambda0`.

:code:`zero_angle` --- Waves with angle larger than :code:`zero_angle` (in degrees) from the :math:`z`-axis are zeroed out during propagation.  The angle is determined in :math:`k`-space.

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

:code:`ell` --- Angular momentum of the pulse.  Currently only 0 is supported.

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

The ideal flying-focus pulse is selected by setting :code:`type` to "ideal flying focus" or "ff".  More information about the ideal flying focus can be found in Ref. [1]_.  In addition to all the parameters available for a :ref:`standard pulse <sec-standard-pulse>`, the parameters below are also available.

:code:`rmaxf_lens` --- The factor that multiplies the laser :math:`w_0` at the lens plane to determine :math:`r_\mathrm{max}` for the lens plane.  The ideal flying focus uses a field profile proportional to :math:`\exp(-[r/w_0]^2)` in the lens plane, where :math:`w_0` is calculated appropriately based on :code:`wf` and :code:`f0`.  It is therefore recommended that :code:`rmaxf_lens` be large enough to include several spot sizes at the lens, i.e., 3 or greater.

:code:`vI` --- Velocity of the intensity peak in units of :math:`c`.  Can be positive or negative.

:code:`f0` --- Nominal focal length of the lens in meters.

:code:`nr_lens` --- Number of points in :math:`r` in the lens plane.  This is recommended to be of order 1000.

.. _sec-ae-pulse:

Axiparabola--Echelon Pulse
**************************

The axiparabola--echelon pulse is selected by setting :code:`type` to "axi-echelon flying focus" or "ae".  More information about the axiparabola--echelon flying-focus pulse can be found in Ref. [2]_.  In addition to all the parameters available for a :ref:`standard pulse <sec-standard-pulse>`, the parameters below are also available.  Note that some of these parameters overlap with those available for an :ref:`ideal flying-focus pulse <sec-ideal-ff-pulse>`; they are duplicated here for clarity.

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
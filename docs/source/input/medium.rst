Medium
========

This documentation is for the section :code:`medium` in the main :doc:`input file <../input-file>`.

* :code:`atomdensity`, real, default = atmospheric density
* :code:`atom`, character(*), default = "helium"
* :code:`nu_ei`, real, default = 1.0e13

:code:`atomdensity` --- The number of gas atoms (not molecules) per cubic meter (assuming standard temperature and pressure).  If left unspecified, :code:`atomdensity` will be set such that there are 2.68678e25 molecules per cubic meter (standard atmosphere).

:code:`atom` --- Specifies the type of gas to use and must be one of the following:

* "hydrogen" -- Hydrogen gas has 2 atoms per molecule, an ionization potential of 13.6 eV, and a nonlinear index of refraction of 7.5e-24 :math:`\mathrm{m}^2/\mathrm{W}`.
* "helium" -- Helium gas has 1 atom per molecule, an ionization potential of 24.6 eV, and a nonlinear index of refraction of 3.6e-25 :math:`\mathrm{m}^2/\mathrm{W}`.
* "argon" -- Argon gas has 1 atom per molecule, an ionization potential of 15.8 eV, and a nonlinear index of refraction of 1.01e-23 :math:`\mathrm{m}^2/\mathrm{W}`.

:code:`nu_ei` --- The electron--ion collision frequency in units of :math:`\mathrm{s}^{-1}`.  It sets the collisional absorption of the plasma current (see :doc:`../uppe`).

The linear refractive index of each gas comes from a Sellmeier formula (or, when HHG is on, from tabulated atomic data that also cover the XUV), scaled with the density.  The nonlinear index :math:`n_2` listed above is for 1 atm and is also scaled with the density.
Output files
============

All output goes to the directory set by :code:`dirname` in the :doc:`diag <input/diag>`
section.  A copy of the input deck, as SUPER read it, is written there as
:code:`sup-input.json`.

File format
-----------

Each diagnostic is an HDF5 file with:

* a dataset :code:`data` with the values;
* one group :code:`AXIS1`, :code:`AXIS2`, ... per dimension, with attributes :code:`NAME`
  (e.g. "t", "r", "z"), :code:`MIN` and :code:`MAX`.

Complex quantities are split into two files, :code:`<name>-re.h5` and :code:`<name>-im.h5`.
Units are SI (m, s, V/m, :math:`\mathrm{m^{-3}}`, J).

The axes are listed below in Fortran order (fastest first).  Python (h5py, numpy) sees the
dimensions in reverse order: for example :code:`EOA` is read as an array of shape
:code:`(nz_line, nt)`.  Things to know when reading them:

* **Radial axis.**  The :math:`r` points are not uniform: they are the zeros of :math:`J_0`
  scaled so that the last one is at :code:`rmax` (:code:`r = jn_zeros(0, nr) * rmax / jn_zeros(0, nr)[-1]`).
  The axis attributes only give the first and last points.
* **Lineouts in z.**  Lineout quantities are written every :code:`lineskip` steps and full
  quantities every :code:`fullskip` steps; the :code:`z` attributes give the first and last
  stored positions.
* **Frequencies.**  The field varies as :math:`e^{-i\omega t}`, so with numpy's FFT the
  physical frequency of bin :math:`k` is :code:`-2*pi*numpy.fft.fftfreq(nt, dt)[k]`.
  Spectral outputs marked "needs fftshift" are stored in FFT order.
* **The field** :math:`E` is complex, with :math:`\mathrm{Re}\,E` the physical field, so the
  intensity is :math:`\epsilon_0 c |E|^2 / 2`.

The Python module :code:`analysis/sup_analysis.py` reads these files (including the radial
axis) into labeled arrays.

Main grid (2D)
--------------

.. list-table::
   :header-rows: 1
   :widths: 15 25 60

   * - File
     - Axes
     - Contents
   * - :code:`Einit`
     - (r, t)
     - Initial field, before the first step (complex)
   * - :code:`Efinal`
     - (r, t)
     - Field after the last step (complex)
   * - :code:`EF`
     - (r, t, z)
     - Full field every :code:`fullskip` steps (complex)
   * - :code:`neF`
     - (r, t, z)
     - Electron density every :code:`fullskip` steps
   * - :code:`EOA`
     - (t, z)
     - Field at :code:`rOA` (on axis by default) every :code:`lineskip` steps (complex)
   * - :code:`neOA`
     - (t, z)
     - Electron density at :code:`rOA`
   * - :code:`streak`
     - (omega, z)
     - Power spectrum integrated over the transverse plane (needs fftshift)
   * - :code:`U`
     - (z)
     - Pulse energy, :math:`2\pi \int\!\!\int \tfrac12 \epsilon_0 c |E|^2\, r\,dr\,dt`
   * - :code:`tau`
     - (z)
     - rms duration of the energy distribution about its temporal centroid
   * - :code:`tauOA`
     - (z)
     - rms duration of the intensity at :code:`rOA`
   * - :code:`w`
     - (z)
     - Spot size :math:`\sqrt{2\langle r^2 \rangle}` of the fluence (the :math:`1/e^2`
       intensity radius for a Gaussian beam)
   * - :code:`wMAX`
     - (z)
     - Same, for the time slice with the highest intensity anywhere
   * - :code:`wMAXOA`
     - (z)
     - Same, for the time slice with the highest intensity on axis

The electron density is only nonzero when :code:`ionization` or :code:`compute_density` is set.

In 3D the full quantities have axes (t, x, y) or (t, x, y, z) instead of (r, t) or (r, t, z),
and the spot size is computed from :math:`\langle x^2 + y^2 \rangle`.

HHG grid
--------

These are written only when the :ref:`HHG <sec-hhg-grid>` section is present (see :doc:`hhg`).

.. list-table::
   :header-rows: 1
   :widths: 15 25 60

   * - File
     - Axes
     - Contents
   * - :code:`hhgEF`
     - (r, t, z)
     - Harmonic field every :code:`fullskip` steps (complex)
   * - :code:`hhgEfinal`
     - (r, t)
     - Harmonic field after the last step (complex)
   * - :code:`hhgEOA`
     - (t, z)
     - Harmonic field at :code:`rOA` (complex)
   * - :code:`hhgU`
     - (z)
     - Harmonic energy at frequencies :math:`\ge` :code:`ene_omega0_fac` :math:`\omega_0`
   * - :code:`dOA`
     - (t, z)
     - Single-atom dipole moment at :code:`rOA`
   * - :code:`dhatOA`
     - (omega, z)
     - Magnitude of its spectrum (needs fftshift)
   * - :code:`dfirst`
     - (r, t)
     - Dipole moment after the first step
   * - :code:`dhatfirst`
     - (r, omega)
     - Its spectrum (needs fftshift)

Timing
------

Every :code:`timingskip` steps SUPER prints the time spent in each part of the step to
standard output.

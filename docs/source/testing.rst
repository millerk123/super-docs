Integration tests
=================

The :code:`tests` directory holds small integration tests that run :code:`super.e` on short,
physics-based problems.  Each one takes 5-30 s on a laptop with 2 MPI ranks.

Running the tests
-----------------

The tests need Python with :code:`numpy`, :code:`scipy`, :code:`h5py` and :code:`pytest`.  From the
:code:`super` directory:

.. code-block:: bash

   make test PYTHON=/path/to/python

This builds :code:`super.e` if needed and runs all tests.  Extra pytest options go in
:code:`PYTEST_ARGS`, e.g. :code:`make test PYTEST_ARGS="-k kerr -x"`.  Each run is written to
:code:`tests/_runs/<name>` (input deck, :code:`out.out` and :code:`DATA`), so a failing case can
be inspected or rerun by hand.

These environment variables change how SUPER is launched:

* :code:`SUPER_MPIRUN` --- MPI launcher (default :code:`mpirun`; e.g. :code:`srun` inside a Perlmutter allocation)
* :code:`SUPER_NP` --- number of MPI ranks (default 2)
* :code:`SUPER_THREADS` --- OpenMP threads per rank (default 2)
* :code:`SUPER_EXE` --- executable (default :code:`super.e` in the repository)
* :code:`SUPER_RUNDIR` --- where the runs are written (default :code:`tests/_runs`)
* :code:`SUPER_REFERENCE` --- reference file (default :code:`tests/reference.json`)

What is checked
---------------

Every test makes two kinds of checks:

* **Physics checks** compare the result with an analytic expectation, with a tolerance set by
  the physics (for example the Gaussian-beam spot size within 1%).  These say whether the
  result is right.
* **Regression checks** compare a few scalar metrics with the values stored in
  :code:`tests/reference.json`, within a relative tolerance per metric (1e-6 for linear
  quantities, 1e-4 to 2e-3 for nonlinear ones).  These catch any change in the results.  The
  tolerances allow for different compilers, FFTW plans and MPI layouts, but not for real
  changes in the physics or numerics.

The tests are:

* :code:`test_linear.py` --- a focused Gaussian pulse in argon: spot size and on-axis fluence
  through the focus follow the Gaussian-beam formulas, and the energy is conserved.
* :code:`test_marburger.py` --- Kerr self-focusing of a collimated beam at 2 and 4 :math:`P_\mathrm{cr}`:
  the collapse distances compared with Marburger's formula, and their ratio.
* :code:`test_kerr.py` --- self-phase modulation (on-axis nonlinear phase :math:`k_0 n_2 I_0 L`
  and rms spectral broadening), and third-harmonic generation, whose coherence length checks
  the gas dispersion.
* :code:`test_ionization.py` --- field ionization of argon: energy balance between the field
  and the electrons, bounds on the electron density, and plasma defocusing.
* :code:`test_hhg_plasma.py` --- plasma dispersion of the harmonic field on the HHG grid: the
  3rd-harmonic phase-mismatch rate with and without ionization, compared with the driver's
  measured plasma phase, isolates the harmonic's own plasma shift (:math:`k_p/3`).

Updating the reference values
-----------------------------

After an intended change to the results (a bug fix, a new default, a change in the numerics),
check that the physics checks still pass, then store the new values with

.. code-block:: bash

   make test-update PYTHON=/path/to/python

and commit :code:`tests/reference.json` together with the change, saying why the values moved.

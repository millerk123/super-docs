Usage
=====

Installation
------------

Dependencies
************

SUPER needs:

* an MPI Fortran 2008 compiler with OpenMP (e.g. :code:`gfortran` through Open MPI's
  :code:`mpif90`, or the Cray :code:`ftn` wrapper);
* FFTW 3 with its MPI and OpenMP libraries (:code:`fftw3_mpi`, :code:`fftw3_omp`);
* parallel HDF5 with the Fortran interface;
* `json-fortran <https://github.com/jacobwilliams/json-fortran>`_ (version 8 or later), built
  with the same compiler.

On a Mac these can come from Homebrew (:code:`open-mpi`, :code:`fftw`, :code:`hdf5-mpi`), with
json-fortran built from source.  On a Cray system such as Perlmutter, load the compiler,
:code:`cray-hdf5-parallel` and :code:`cray-fftw` modules, and build json-fortran yourself.

Building
********

The top of :code:`Makefile` sets the compiler and the locations of the libraries
(:code:`OPENMPI_ROOT`, :code:`H5_ROOT`, :code:`JSON_ROOT`, :code:`FFTW_ROOT`).  There is a
section for a Mac (Homebrew) and a commented-out section for Perlmutter; edit or swap them for
your system.  Then run

.. code-block:: bash

   make

which compiles the Fortran sources in :code:`src/` and creates :code:`super.e` (objects go in
:code:`build/`).  :code:`make -j N` builds in
parallel.  The module dependencies between the source files are found automatically by
:code:`tools/fortran_deps.py` (written to :code:`build/deps.mk`), so after editing a derived type
or a module interface a plain :code:`make` recompiles every file that uses it; :code:`make clean`
is not needed.  Files whose module did not change (e.g. only a routine body was edited) are not
recompiled.  Editing the :code:`Makefile` (e.g. compiler flags) recompiles everything.  The
dependency scan needs Python 3 (:code:`make PYTHON=/path/to/python` to choose one).

json-fortran is a shared library, so it must be on the library path at run time:

.. code-block:: bash

   export DYLD_LIBRARY_PATH=$JSON_ROOT/lib:$DYLD_LIBRARY_PATH   # macOS
   export LD_LIBRARY_PATH=$JSON_ROOT/lib:$LD_LIBRARY_PATH       # Linux

To check a build, run the :doc:`integration tests <testing>` with :code:`make test`.

Running
-------

SUPER reads :code:`sup-stdin.json` from the current directory (see :doc:`input-file`; the
file may contain :code:`//` comments, as in the example :code:`sup-stdin.json` in the
repository).  Run it with as many MPI ranks as :code:`node_number` in the
:doc:`parallel <input/parallel>` section:

.. code-block:: bash

   mpirun -n 4 ./super.e

The number of OpenMP threads per rank is :code:`n_threads`.  On a cluster, give each rank that
many cores, e.g. on Perlmutter (128 cores per node, 4 ranks per node, 16 threads each)

.. code-block:: bash

   export OMP_PLACES=threads OMP_PROC_BIND=spread
   srun -N 4 -n 16 -c 32 --cpu_bind=cores ./super.e

Choosing the decomposition
**************************

The time grid is split across the MPI ranks, and the radial grid during transposes, so
:code:`nt` and :code:`nr` (in 3D, :code:`nx` and :code:`ny`; and the HHG :code:`nt` and :code:`nr`) must be divisible by
:code:`node_number`.  Each step does FFTs along :math:`t` and dense matrix products for the
Hankel transform along :math:`r`, whose cost grows as :math:`n_r^2 n_t`; for large radial grids
the Hankel transform dominates.

Progress and checks
*******************

SUPER prints the step number as it runs, and at the start it prints useful derived quantities:
the refractive index and group velocity at :code:`lambda0`, the critical
power ratio (with :math:`P_\mathrm{cr} = \lambda^2 / (2\pi n_2)`), the Keldysh parameter and
the filter settings.  Check these before a long run.  The output files are described in
:doc:`output`.

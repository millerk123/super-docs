.. SUPER documentation master file, created by
   sphinx-quickstart on Thu Feb 15 11:18:00 2024.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` directive.

Welcome to SUPER's documentation!
=================================

**SUPER** (**S**\ imulation of the **U**\ nidirectional pulse **P**\ ropagation **E**\ quation at **R**\ ochester) is a software package built in Fortran for simulating the Unidirectional Pulse Propagation Equation (UPPE) [1]_, [2]_, [3]_.  This code was adapted to Fortran by Kyle G. Miller from MATLAB code written by John P. Palastro.

:ref:`Search the documentation <search>`

.. * :ref:`genindex`

Compilation
-----------

Required packages
*****************

To installation of SUPER requires a parallel Fortran compiler, HDF5, `json-fortran <https://github.com/jacobwilliams/json-fortran>`_, and FFTW.  The locations of these libraries should be specified by the relevant variables in the file :code:`Makefile`.

Build commands
**************

To build SUPER, simply navigate to the :code:`super` directory and execute the command

.. code-block:: bash

   make

An executable named :code:`super.e` will then be created in the :code:`super` directory.

Running SUPER
-------------

To run SUPER, you can either run in serial with

.. code-block:: bash

   ./super.e

or you can run in parallel with

.. code-block:: bash

   mpirun -n 4 ./super.e

In either case, the executable will look for an input file named :code:`sup-stdin.json`, and the number of MPI ranks in the :code:`mpirun` command (in this case, 4) should match the :code:`node_number` specified in the input deck.  For detailed information on how to create an input file, see :doc:`input-file`.

Table of Contents
-----------------
.. toctree::
   :maxdepth: 2

   usage
   uppe
   input-file
   hhg
   output
   testing

References
----------

.. [1] M\. Kolesik, J.V. Moloney, and M. Mlejnek, `"Unidirectional Optical Pulse Propagation Equation," <https://doi.org/10.1103/PhysRevLett.89.283902>`_ *Phys. Rev. Lett.* **89**\ (28), 283902 (2002).

.. [2] M\. Kolesik and J.V. Moloney, `"Nonlinear optical pulse propagation simulation: From Maxwell’s to unidirectional equations," <https://doi.org/10.1103/PhysRevE.70.036604>`_ *Phys. Rev. E* **70**\ (3), 036604 (2004).

.. [3] A\. Couairon, E. Brambilla, T. Corti, D. Majus, O. de J. Ramírez-Góngora, and M. Kolesik, `"Practitioner’s guide to laser pulse propagation models and simulation," <https://doi.org/10.1140/epjst/e2011-01503-3>`_ *Eur. Phys. J.: Spec. Top.* **199**\ (1), 5-76 (2011).
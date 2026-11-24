Parallel
========

This documentation is for the section :code:`parallel` in the main :doc:`input file <../input-file>`.

* :code:`node_number`, integer, default = 1
* :code:`n_threads`, integer, default = 1

:code:`node_number` --- The number of MPI ranks to be recruited.  These processes will be primarily distributed across the time array, but certain operations require transposes that will also distribute these processes across the radial dimension.  Therefore, the number of cells in time and in radius must be divisible by :code:`node_number`.  In addition, performance may be improved if the number of cells in time is divislbe by the square of :code:`node_number`.  If high-harmonic generation is enabled, the number of radial cells on the HHG grid should also be divisible by :code:`node_number`.

:code:`n_threads` --- The number of OpenMP threads to use.  Typically, any unused cores after the number of MPI ranks is set should be used here.
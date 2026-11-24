Diagnostics
===========

This documentation is for the section :code:`diag` in the main :doc:`input file <../input-file>`.  See :doc:`../output` for detailed information on the actual output files.

* :code:`lineskip`, integer, default = 1
* :code:`linestore`, integer, default = 1
* :code:`fullskip`, integer, default = 1
* :code:`timingskip`, integer, default = :code:`fullskip`
* :code:`dirname`, character(*), default = "RUN"
* :code:`rOA`, real, default = 0.0

:code:`lineskip` --- Number of points in :math:`z` to skip when writing lineout quantities.

:code:`linestore` --- Number of points in :math:`z` to store in memory before writing lineout quantities.  This reduces the number of times the program has to write to disk, which saves time.  However, this value should not be so large that the program runs out of memory.

:code:`fullskip` --- Number of points in :math:`z` to skip when writing full-grid quantities.  This value should normally be greater than :code:`lineskip` to save on disk space (and output time).

:code:`timingskip` --- Number of points in :math:`z` to skip when printing out timing information.

:code:`dirname` --- Name of the directory to store the data (directory is created if not already present).

:code:`rOA` --- Radius value (in meters) at which to write the lineout quantities.  The "OA" in :code:`rOA` refers to on-axis, and it usually makes sense to set :code:`rOA` to 0.  (Note that in this case, the on-axis cell is actually set to the smallest value of :math:`r` since there is no cell with :math:`r=0`.)  However, lineouts at any radius can be obtained by setting :code:`rOA` to something else.

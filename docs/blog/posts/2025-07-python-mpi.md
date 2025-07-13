---
date:
  created: 2025-07-01
authors:
  - tjhei
---

# Coupling of MPI parallel Python and C++ simulations

(contributed by Timo Heister)

## Introduction

In this article we document the experiments we performed to learn how to correctly couple a Python simulation code to a C++ simulation code, where both projects support parallel computations using MPI. We will need this for this project to couple [Landlab](https://landlab.csdms.io/) (for surface evolution, written in Python) and [ASPECT](https://aspect.geodynamics.org/) (for the interior, written in C++).

<!-- more -->

## The parallel setup

We only consider the situation where both codes use MPI and the simulations are coupled
in way that only one of them is running at the same time. After one ASPECT timestep,
information is passed to Landlab, which then does one or more timesteps. Afterwards, information is passed back.

Because of this, each MPI rank will be executing part of the Landlab simulation or one part of the ASPECT simulation, swapping between the two. Otherwise, ranks would be idle for part of the coupled simulation. As a consequence, if the coupled simulation runs on N cores, the MPI universe will have N ranks and each rank runs Landlab and ASPECT.

## Processes

The only way to achieve an MPI universe with N ranks on N nodes that can each run ASPECT or Landlab, is that each MPI rank is a single process. This means either
ASPECT needs to load Landlab (where landlab is a Python library) or Landlab needs to load ASPECT (where ASPECT is a shared library).

ASPECT is currently not in a shape to be compiled as a library, while it is likely
relatively simple to convert a Landlab simulation to a Python library, that can be
loaded.

## The experiments

The [GitHub example repository](https://github.com/landlab-aspect/mpi-tests) created
for this article contains several
test codes that demonstrate the different ways of coupling a C++ and a Python code.
We will not discuss experiment 1 and 2 here, but focus on experiment 3 that does
precisely what we want to achieve:
- We create a C++ binary that is launched using ``mpirun`` with N workers
- The C++ code initializes MPI
- The C++ code then uses the CPython API to load and call into Python code
- It passes the MPI communicator to Python, so that the Python side can use ``mpi4py``
  for communication

Note that:
1. The Python side does not initialize MPI as this is already done for each process on the C++ side.
2. The MPI communicator is converted to a long before being passed across (and then converted back to a handle).
3. The exact same MPI implementation has to be used by the C++ code and mpi4py (see next section).

# MPI setup

With this approach, you needcan not mix OpenMPI and MPICH or separate installations of the same library, which would be done by a typical pip or conda installation. Luckily,
we can ask ``conda`` (or ``mamba``) to use the system MPI:
```
$ apt search libopenmpi
libopenmpi-dev/noble,now 4.1.6-7ubuntu2 amd64 [installed]
  high performance message passing library -- header files
$ mamba create -n pythonmpi
$ mamba activate pythonmpi
$ mamba install openmpi=4.1.6=external_* mpi4py
```

# The result

This is the output the simple test program generates when run with N=3 ranks:
```
$ mpirun -n 3 ./main 
C++: Calling Python function...
Python: Hello from Rank 0 of 3
Python: Hello from Rank 2 of 3
Python: Hello from Rank 1 of 3
Python: sum 3
Python: sum 3
Python: sum 3
```

Success!

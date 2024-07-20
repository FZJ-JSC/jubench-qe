# Quantum ESPRESSO

Quantum ESPRESSO (QE) is an Open-Source code for electronic-structure calculations on solids and molecules as described in ([this article](10.1088/1361-648X/aa8f79)).
This repository compiles QE for [GPUs](https://doi.org/10.1063/5.0005082) and runs the cp.x benchmark. 
The code is written in Fortran and uses OpenMP, MPI, OpenACC, and CUDA Fortran to parallelize and accelerate on heterogeneous platforms.   

## Quick start:

```
module load JUBE
jube run ./benchmark/jube/qe.yaml
jube analyse ./benchmark/jube/bench_run
jube result  ./benchmark/jube/bench_run
```

The JUBE step takes care of execution of the benchmark. It also configures, builds and installs the benchmark.

## Source

The source code of QE in version 7.1 is delivered with the benchmark. It is equivalent to the [Git tag `qe-7.1`](https://gitlab.com/QEF/q-e/-/archive/qe-7.1/q-e-qe-7.1.tar.gz) of the QE repository on Gitlab: https://gitlab.com/QEF/q-e.git. 

## Building

The benchmark, utilizes the GPU version of QE. QE needs the following software to build:

1. CUDA Fortran compiler
2. CMake
3. MPI
4. FFT library
5. BLAS and LAPACK

Once the dependencies are in place, the code can be built and installed with the following commands:

```
cd q-e
mkdir build && cd build
cmake ../ -DCMAKE_INSTALL_PREFIX=/your/install/path  -DQE_ENABLE_MPI=ON -DQE_ENABLE_CUDA=ON -DQE_ENABLE_OPENACC=ON -DCMAKE_C_COMPILER=mpicc -DCMAKE_Fortran_COMPILER=mpif90 -DCMAKE_Fortran_FLAGS=-D__CLOCK_SECONDS -DBLA_VENDOR=Intel10_64lp_seq

make cp -j16 install

```

This will install the QE executables in `/your/install/path` directory.

For more details about the CMake options, such as `-DBLA_VENDOR=Intel10_64lp_seq`,  see [the QE wiki](https://gitlab.com/QEF/q-e/-/wikis/Developers/CMake-build-system) and [the dedicated CMake documentation](https://gitlab.com/QEF/q-e/-/wikis/Developers/CMake-build-system). 

### JUBE

Using JUBE, the steps `config` and `compile` configure and build QE, respectively. The build step could take a while, depending on your system configuration.

## Modification Guidelines

In addition to the general rules for modification, please see the following specific rules:

- Changing inputs located in `src/inputs` is not within scope.
- Floating point optimization must not be set to `--ffast-math` or similar.
- We suggest to use `-ndiag 25 -nbgrp 2` as parameters for the benchmark in question, but the parameters may be changed to improve performance on a given system.

## Execution

The executable of QE used in the benchmark is `cp.x`. The pseudo-potentials and the input file can be found in `src/inputs/`.

### Command Line

On the command line, the benchmark should be called with:

```
[mpiexec] /your/install/path/QE/bin/cp.x -ndiag 25 -nbgrp 2  < zro2.in > zro2.out
```
For more information about the parallelization arguments, such as `-ndiag` or `-nbgrp`, see [link](https://people.sissa.it/~degironc/FIST/Slides/6%20parallalization.pdf).

Any changes by the benchmark user should be documented and reported with results. 

### JUBE

Using JUBE, the step `submit` takes care of submitting the benchmark – including the proper options – to the batch queue.

To execute the benchmark with JUBE -- running all necessary steps -- use:

```
jube run benchmark/jube/qe.yaml
```

## Verification

The benchmark should run successfully without any errors being reported. The `Physical Quantities` after 10 iteration should look like following:

```
 * Physical Quantities at step:    10


                total energy =    -5457.26495599951 Hartree a.u.
              kinetic energy =    11050.50363 Hartree a.u.
        electrostatic energy =   -13860.39268 Hartree a.u.
```

The `total energy` should be identical up to 7 digits after decimal point.

### JUBE

Using JUBE, the step `submit` takes care verification. `submit` can finish without error only if it passes the verification.

## Results

The benchmark prints a runtime and number of GPUs used for calculations (see following). The output should contain timing information at the end, where the value before `WALL` is the metric to be used to report the benchmark:

```
[user@jwlogin21 qe-gpu]$ tail -n 10 zro2.out 
 
     CP           :     78.71s CPU     77.83s WALL

 
   This run was terminated on:  22:58:12  28Mar2023            

=------------------------------------------------------------------------------=
   JOB DONE.
=------------------------------------------------------------------------------=
```

The information can be extracted like the following:

```
[user@jwlogin23 work]$ grep "CP" zro2.out |tail -n 1 |awk {'print $5,"", $6'}
77.86  WALL
```

### JUBE

To see the benchmark results with JUBE, execute this command:

```
jube analyse ./benchmark/jube/bench_run
jube result  ./benchmark/jube/bench_run
```

The output should look like the following, where `Wall_Clock_Time` is the metric of the benchmark. The column titled `verified` indicates if the verification ran successfully (_true_, in case it did).

```
[user@jwlogin23 qe-gpu]$ jube result -a benchmark/jube/bench_run
nodes,ngpu,Wall_Clock_Time,verified
8,32,77.38,true
```

## Baseline

The baseline configuration must be chosen such that the elapsed time is less than or equal to 80s. This was achieved on JUWELS Booster using 8 nodes, each 4 A100 GPUs.

Nnode | NGPU  | Wall_Clock_Time
----- | ----- | ---        
  8   |  32   | 78.6           
          

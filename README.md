# RCCL Tests

These tests check both the performance and the correctness of RCCL operations. They can be compiled against [RCCL](https://github.com/ROCmSoftwarePlatform/rccl).

## Build

To build the tests, just type `make`.

If HIP is not installed in /opt/rocm, you may specify HIP\_HOME. Similarly, if RCCL is not installed in /usr, you may specify NCCL\_HOME and CUSTOM\_RCCL\_LIB.

```shell
$ make HIP_HOME=/path/to/hip NCCL_HOME=/path/to/rccl CUSTOM_RCCL_LIB=/path/to/rccl/lib/librccl.so
```

RCCL tests rely on MPI to work on multiple processes, hence multiple nodes. If you want to compile the tests with MPI support, you need to set MPI=1 and set MPI\_HOME to the path where MPI is installed.

```shell
$ make MPI=1 MPI_HOME=/path/to/mpi HIP_HOME=/path/to/hip RCCL_HOME=/path/to/rccl
```

## Usage

RCCL tests can run on multiple processes, multiple threads, and multiple HIP devices per thread. The number of process is managed by MPI and is therefore not passed to the tests as argument. The total number of ranks (=HIP devices) will be equal to (number of processes)\*(number of threads)\*(number of GPUs per thread).

### Quick examples

Run on 8 GPUs (`-g 8`), scanning from 8 Bytes to 128MBytes :
```shell
$ ./build/all_reduce_perf -b 8 -e 128M -f 2 -g 8
```

Run with MPI on 40 processes (potentially on multiple nodes) with 4 GPUs each, disabling checks :
```shell
$ mpirun -np 40 ./build/all_reduce_perf -b 8 -e 128M -f 2 -g 4 -c 0
```

### Performance

See the [Performance](doc/PERFORMANCE.md) page for explanation about numbers, and in particular the "busbw" column.

### Arguments

All tests support the same set of arguments :

* Number of GPUs
  * `-t,--nthreads <num threads>` number of threads per process. Default : 1.
  * `-g,--ngpus <GPUs per thread>` number of gpus per thread. Default : 1.
* Sizes to scan
  * `-b,--minbytes <min size in bytes>` minimum size to start with. Default : 32M.
  * `-e,--maxbytes <max size in bytes>` maximum size to end at. Default : 32M.
  * Increments can be either fixed or a multiplication factor. Only one of those should be used
    * `-i,--stepbytes <increment size>` fixed increment between sizes. Default : (max-min)/10.
    * `-f,--stepfactor <increment factor>` multiplication factor between sizes. Default : disabled.
* RCCL operations arguments
  * `-o,--op <sum/prod/min/max/all>` Specify which reduction operation to perform. Only relevant for reduction operations like Allreduce, Reduce or ReduceScatter. Default : Sum.
  * `-d,--datatype <nccltype/all>` Specify which datatype to use. Default : Float.
  * `-r,--root <root/all>` Specify which root to use. Only for operations with a root like broadcast or reduce. Default : 0.
* Performance
  * `-n,--iters <iteration count>` number of iterations. Default : 20.
  * `-w,--warmup_iters <warmup iteration count>` number of warmup iterations (not timed). Default : 5.
  * `-m,--agg_iters <aggregation count>` number of operations to aggregate together in each iteration. Default : 1.
* Test operation
  * `-p,--parallel_init <0/1>` use threads to initialize RCCL in parallel. Default : 0.
  * `-c,--check <0/1>` check correctness of results. This can be quite slow on large numbers of GPUs. Default : 1.
  * `-z,--blocking <0/1>` Make RCCL collective blocking, i.e. have CPUs wait and sync after each collective. Default : 0.
  * `-y,--memory_type <coarse/fine/host>` Specify the kind of memory to be used in RCCL. Default : coarse.

## Unit tests

Unit tests for rccl-tests are implemented with pytest (python3 is also required).  Several notes for the unit tests:

1. The LD_LIBRARY_PATH environment variable will need to be set to include /path/to/rccl-install/lib/ in order to run the unit tests.
2. The HSA_FORCE_FINE_GRAIN_PCIE environment variable will need to be set to 1 in order to run the unit tests which use fine-grained memory type.

The unit tests can be invoked within the rccl-tests root, or in the test subfolder.  An example call to the unit tests:
```shell
$ LD_LIBRARY_PATH=/path/to/rccl-install/lib/ HSA_FORCE_FINE_GRAIN_PCIE=1 python3 -m pytest
```

## Copyright

RCCL tests are provided under the BSD license.

All source code and accompanying documentation is copyright (c) 2016-2020, NVIDIA CORPORATION. All rights reserved.

All modifications are copyright (c) 2020 Advanced Micro Devices, Inc. All rights reserved.


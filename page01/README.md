# Obtain the Project from GitHub

The [ulmBLAS](https://github.com/michael-lehn/ulmBLAS) project is hosted on GitHub. We use different branches for analyzing certain aspects of matrix-matrix multiplication.

## Obtain

First clone the [ulmBLAS](https://github.com/michael-lehn/ulmBLAS) repository from GitHub:

```console
$ git clone https://github.com/michael-lehn/ulmBLAS.git
Cloning into 'ulmBLAS'...
```

You now have a new subdirectory `ulmBLAS`.

## Select a Branch

Change into the `ulmBLAS` directory and use `git branch -a` to display all branches hosted on GitHub:

```console
$ cd ulmBLAS
$ git branch -a
* master
  remotes/origin/HEAD -> origin/master
  remotes/origin/blis-avx-microkernel
  remotes/origin/demo-naive-avx-with-intrinsics
  remotes/origin/demo-naive-sse-with-intrinsics
  remotes/origin/demo-naive-sse-with-intrinsics-unrolled
  remotes/origin/demo-pure-c
  remotes/origin/demo-sse-asm
  remotes/origin/demo-sse-asm-for-AB-loop
  remotes/origin/demo-sse-asm-unrolled
  remotes/origin/demo-sse-asm-unrolled-v2
  remotes/origin/demo-sse-asm-unrolled-v3
  remotes/origin/demo-sse-asm-unrolled-with-prefetch
  remotes/origin/demo-sse-intrinsics
  remotes/origin/demo-sse-intrinsics-for-AB-loop
  remotes/origin/demo-sse-intrinsics-v2
  remotes/origin/demo-sse-intrinsics-v3
  remotes/origin/demo-with-sse-intrinsics
  remotes/origin/master
```

With `git checkout -B LOCAL_BRANCH REMOTE_BRANCH` you can select a branch and switch to it. For example:

```console
$ git checkout -B demo-pure-c remotes/origin/demo-pure-c
Switched to a new branch 'demo-pure-c'
Branch demo-pure-c set up to track remote branch demo-pure-c from origin.
```

## Compile

We have simple Makefiles: `make` will compile and build everything.

```console
$ make
```

<details>
<summary>Show complete build output</summary>

```console
make -C src
gcc-4.8 -Wall -I. -O3 -msse3 -mfpmath=sse -fomit-frame-pointer -c -o auxiliary/xerbla.o auxiliary/xerbla.c
gcc-4.8 -Wall -I. -O3 -msse3 -mfpmath=sse -fomit-frame-pointer -c -o level1/dasum.o level1/dasum.c
gcc-4.8 -Wall -I. -O3 -msse3 -mfpmath=sse -fomit-frame-pointer -c -o level1/daxpy.o level1/daxpy.c
gcc-4.8 -Wall -I. -O3 -msse3 -mfpmath=sse -fomit-frame-pointer -c -o level1/dcopy.o level1/dcopy.c
gcc-4.8 -Wall -I. -O3 -msse3 -mfpmath=sse -fomit-frame-pointer -c -o level1/ddot.o level1/ddot.c
gcc-4.8 -Wall -I. -O3 -msse3 -mfpmath=sse -fomit-frame-pointer -c -o level1/dnrm2.o level1/dnrm2.c
gcc-4.8 -Wall -I. -O3 -msse3 -mfpmath=sse -fomit-frame-pointer -c -o level1/drot.o level1/drot.c
gcc-4.8 -Wall -I. -O3 -msse3 -mfpmath=sse -fomit-frame-pointer -c -o level1/drotg.o level1/drotg.c
gcc-4.8 -Wall -I. -O3 -msse3 -mfpmath=sse -fomit-frame-pointer -c -o level1/drotm.o level1/drotm.c
gcc-4.8 -Wall -I. -O3 -msse3 -mfpmath=sse -fomit-frame-pointer -c -o level1/drotmg.o level1/drotmg.c
gcc-4.8 -Wall -I. -O3 -msse3 -mfpmath=sse -fomit-frame-pointer -c -o level1/dscal.o level1/dscal.c
gcc-4.8 -Wall -I. -O3 -msse3 -mfpmath=sse -fomit-frame-pointer -c -o level1/dswap.o level1/dswap.c
gcc-4.8 -Wall -I. -O3 -msse3 -mfpmath=sse -fomit-frame-pointer -c -o level1/idamax.o level1/idamax.c
gcc-4.8 -Wall -I. -O3 -msse3 -mfpmath=sse -fomit-frame-pointer -c -o level3/dgemm.o level3/dgemm.c
gcc-4.8 -Wall -I. -O3 -msse3 -mfpmath=sse -fomit-frame-pointer -c -o level3/dgemm_nn.o level3/dgemm_nn.c
gcc-4.8 -Wall -I. -O3 -msse3 -mfpmath=sse -fomit-frame-pointer -c -o level3/dsymm.o level3/dsymm.c
gcc-4.8 -Wall -I. -O3 -msse3 -mfpmath=sse -fomit-frame-pointer -c -o level3/stubs.o level3/stubs.c
ar cru ../libulmblas.a auxiliary/xerbla.o level1/dasum.o level1/daxpy.o level1/dcopy.o level1/ddot.o level1/dnrm2.o level1/drot.o level1/drotg.o level1/drotm.o level1/drotmg.o level1/dscal.o level1/dswap.o level1/idamax.o level3/dgemm.o level3/dgemm_nn.o level3/dsymm.o level3/stubs.o
ranlib ../libulmblas.a
...
```

The complete output here should be copied verbatim from the original tutorial, including the builds of `libatlulmblas.a`, `librefblas.a`, the BLAS tests, and the ATLAS benchmark suite.

</details>

## Structure of the Project

The `ulmBLAS` project has four important components:

### `src/`

In `src/`, the actual ulmBLAS library is built. We build the library twice. The resulting libraries are

```text
libulmblas.a
libatlulmblas.a
```

The only difference between these two variants is the prefixes and suffixes of the symbol names.

`libulmblas.a` exports symbol names like a regular Fortran 77 BLAS implementation. For example, for `dgemm` we have:

```console
$ nm libulmblas.a | grep dgemm | grep T
0000000000000000 T _ULM_dgemm
00000000000002d0 T _dgemm_
0000000000000000 T _dgemm_nn
```

`libatlulmblas.a` exports symbol names like ATLAS. So when we link against the ATLAS benchmark suite, we can pretend to be the [ATLAS](https://math-atlas.sourceforge.net/) library. For `dgemm` we have:

```console
$ nm libatlulmblas.a | grep dgemm | grep T
0000000000000000 T _ATL_dgemm
00000000000002d0 T _dgemm_intern
0000000000000000 T _dgemm_nn
```

### `refblas/`

In `refblas/`, the reference BLAS implementation from [Netlib](https://www.netlib.org/blas/) is located.

We use it for testing and as a reference for measuring speedups in the benchmarks. The resulting library is `librefblas.a`.

Looking at the exported symbol for `dgemm` gives:

```console
$ nm librefblas.a | grep dgemm | grep T
0000000000000000 T _dgemm_
```

### `test/`

In `test/`, we have the BLAS test suite from [Netlib](https://www.netlib.org/blas/).

With

```console
$ make check_ulm
```

you can test `libulmblas.a`.

### `bench/`

In `bench/`, we have the benchmark suite from [ATLAS](https://math-atlas.sourceforge.net/).

Note that this is only the benchmark suite — about 40 files extracted from ATLAS. We are not using the BLAS implementation of ATLAS itself.

In `bench/Makefile`, you can use the variables `ATLAS_LIB` and `REF_LIB` to compare the performance of two BLAS libraries. For example, here we compare `libatlulmblas.a` with `librefblas.a`:

```make
ATL_C_SOURCEFILES = $(wildcard ATL_*.c)
ATL_F_SOURCEFILES = $(wildcard ATL_*.f)

ATL_OBJECTFILES = $(patsubst %.c, %.o, $(ATL_C_SOURCEFILES)) \
                  $(patsubst %.f, %.o, $(ATL_F_SOURCEFILES))

FC = gfortran
CC = gcc-4.8
#CC = gcc

CFLAGS = -c -DL2SIZE=4194304 -DAdd_ -DF77_INTEGER=int -DStringSunStyle \
         -DATL_SSE2 -DDREAL

LDFLAGS = -lm

#
# Select the ATLAS library (or our faked ATLAS implementation)
#
ATLAS_LIB = ../libatlulmblas.a

#
# Select the reference implementation
#
REF_LIB = ../librefblas.a

all: xdl1blastst xdl3blastst

xdl1blastst: l1blastst.o libtstatlas.a $(ATLAS_LIB) $(REF_LIB)
	$(FC) -o xdl1blastst l1blastst.o libtstatlas.a $(ATLAS_LIB) $(REF_LIB)

xdl3blastst: l3blastst.o libtstatlas.a $(ATLAS_LIB) $(REF_LIB)
	$(FC) -o xdl3blastst l3blastst.o libtstatlas.a $(ATLAS_LIB) $(REF_LIB)

libtstatlas.a: $(ATL_OBJECTFILES)
	ar r libtstatlas.a $(ATL_OBJECTFILES)
	ranlib libtstatlas.a

clean:
	rm -f xdl1blastst libtstatlas.a l1blastst.o $(ATL_OBJECTFILES)
```

---

[← Main Page](../README.md) | [Next →](../page02/)

Copyright © 2014 Michael Lehn

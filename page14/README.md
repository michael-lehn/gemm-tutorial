# Benchmarking

Finally, we compare our implementation with several established high-performance matrix libraries.

The benchmarks shown here were performed with the software versions and on the hardware used for the original 2014 tutorial.

## Eigen

We start by benchmarking the C++ library [Eigen](https://eigen.tuxfamily.org/) (Version 3.2.1).

Note that in `bench/Makefile` we expect this library to be installed in

```text
$HOME/work/eigen-3.2.1
```

Change the value of `REF_LIB` according to your local installation.

Do a `make clean`, check out the `bench-eigen` branch, and build everything:

```console
$ cd ulmBLAS
$ git checkout -B bench-eigen remotes/origin/bench-eigen
Switched to a new branch 'bench-eigen'
Branch bench-eigen set up to track remote branch bench-eigen from origin.

$ make
```

<details>
<summary>Show complete build output</summary>

```console
make -C src
clang -Wall -I. -O3 -msse3 -mfpmath=sse -fomit-frame-pointer -DULM_BLOCKED -c -o level3/dgemm_nn.o level3/dgemm_nn.c
ar cru ../libulmblas.a  auxiliary/xerbla.o  level1/dasum.o level1/daxpy.o level1/dcopy.o level1/ddot.o level1/dnrm2.o level1/drot.o level1/drotg.o level1/drotm.o level1/drotmg.o level1/dscal.o level1/dswap.o level1/idamax.o  level3/dgemm.o level3/dgemm_nn.o level3/dsymm.o level3/stubs.o
ranlib ../libulmblas.a
clang -Wall -I. -O3 -msse3 -mfpmath=sse -fomit-frame-pointer -DULM_BLOCKED -DFAKE_ATLAS -c -o level3/atl_dgemm_nn.o level3/dgemm_nn.c
ar cru ../libatlulmblas.a  auxiliary/atl_xerbla.o  level1/atl_dasum.o level1/atl_daxpy.o level1/atl_dcopy.o level1/atl_ddot.o level1/atl_dnrm2.o level1/atl_drot.o level1/atl_drotg.o level1/atl_drotm.o level1/atl_drotmg.o level1/atl_dscal.o level1/atl_dswap.o level1/atl_idamax.o  level3/atl_dgemm.o level3/atl_dgemm_nn.o level3/atl_dsymm.o level3/atl_stubs.o
ranlib ../libatlulmblas.a
make -C refblas
make[1]: Nothing to be done for `all'.
make -C test
gfortran dblat1.f -L.. -lulmblas -o dblat1_ulm
dblat1.f:215.44:
               CALL STEST1(DNRM2(N,SX,INCX),STEMP,STEMP,SFAC)           
                                            1
Warning: Rank mismatch in argument 'strue1' at (1) (scalar and rank-1)
dblat1.f:219.44:
               CALL STEST1(DASUM(N,SX,INCX),STEMP,STEMP,SFAC)           
                                            1
Warning: Rank mismatch in argument 'strue1' at (1) (scalar and rank-1)
gfortran dblat3.f -L.. -lulmblas -o dblat3_ulm
make -C bench
c++ -o xdl1blastst l1blastst.o libtstatlas.a ../libatlulmblas.a /Users/clientadmin/work/eigen-3.2.1/blas/libeigen_blas_static.a
c++ -o xdl3blastst l3blastst.o libtstatlas.a ../libatlulmblas.a /Users/clientadmin/work/eigen-3.2.1/blas/libeigen_blas_static.a
```

</details>

Then we do the benchmark:

```console
$ cd bench
$ make clean
```

<details>
<summary>Show complete output of <code>make clean</code></summary>

```console
rm -f xdl1blastst libtstatlas.a l1blastst.o  ATL_cputime.o  ATL_epsilon.o  ATL_f77amax.o  ATL_f77asum.o  ATL_f77axpy.o  ATL_f77copy.o  ATL_f77dot.o  ATL_f77gemm.o  ATL_f77nrm2.o  ATL_f77rot.o  ATL_f77rotg.o  ATL_f77rotm.o  ATL_f77rotmg.o  ATL_f77scal.o  ATL_f77swap.o  ATL_f77symm.o  ATL_f77syr2k.o  ATL_f77syrk.o  ATL_f77trmm.o  ATL_f77trsm.o  ATL_flushcache.o  ATL_gediffnrm1.o  ATL_gegen.o  ATL_genrm1.o  ATL_infnrm.o  ATL_rand.o  ATL_set.o  ATL_synrm.o  ATL_trnrm1.o  ATL_vdiff.o  ATL_zero.o  ATL_df77wrap.o
```

</details>

Build the benchmark:

```console
$ make
```

<details>
<summary>Show complete benchmark build output</summary>

```console
cc -c -DL2SIZE=4194304 -DAdd_ -DF77_INTEGER=int -DStringSunStyle -DATL_SSE2 -DDREAL   -c -o l1blastst.o l1blastst.c
cc -c -DL2SIZE=4194304 -DAdd_ -DF77_INTEGER=int -DStringSunStyle -DATL_SSE2 -DDREAL   -c -o ATL_cputime.o ATL_cputime.c
cc -c -DL2SIZE=4194304 -DAdd_ -DF77_INTEGER=int -DStringSunStyle -DATL_SSE2 -DDREAL   -c -o ATL_epsilon.o ATL_epsilon.c
cc -c -DL2SIZE=4194304 -DAdd_ -DF77_INTEGER=int -DStringSunStyle -DATL_SSE2 -DDREAL   -c -o ATL_f77amax.o ATL_f77amax.c
cc -c -DL2SIZE=4194304 -DAdd_ -DF77_INTEGER=int -DStringSunStyle -DATL_SSE2 -DDREAL   -c -o ATL_f77asum.o ATL_f77asum.c
cc -c -DL2SIZE=4194304 -DAdd_ -DF77_INTEGER=int -DStringSunStyle -DATL_SSE2 -DDREAL   -c -o ATL_f77axpy.o ATL_f77axpy.c
cc -c -DL2SIZE=4194304 -DAdd_ -DF77_INTEGER=int -DStringSunStyle -DATL_SSE2 -DDREAL   -c -o ATL_f77copy.o ATL_f77copy.c
cc -c -DL2SIZE=4194304 -DAdd_ -DF77_INTEGER=int -DStringSunStyle -DATL_SSE2 -DDREAL   -c -o ATL_f77dot.o ATL_f77dot.c
cc -c -DL2SIZE=4194304 -DAdd_ -DF77_INTEGER=int -DStringSunStyle -DATL_SSE2 -DDREAL   -c -o ATL_f77gemm.o ATL_f77gemm.c
cc -c -DL2SIZE=4194304 -DAdd_ -DF77_INTEGER=int -DStringSunStyle -DATL_SSE2 -DDREAL   -c -o ATL_f77nrm2.o ATL_f77nrm2.c
cc -c -DL2SIZE=4194304 -DAdd_ -DF77_INTEGER=int -DStringSunStyle -DATL_SSE2 -DDREAL   -c -o ATL_f77rot.o ATL_f77rot.c
cc -c -DL2SIZE=4194304 -DAdd_ -DF77_INTEGER=int -DStringSunStyle -DATL_SSE2 -DDREAL   -c -o ATL_f77rotg.o ATL_f77rotg.c
cc -c -DL2SIZE=4194304 -DAdd_ -DF77_INTEGER=int -DStringSunStyle -DATL_SSE2 -DDREAL   -c -o ATL_f77rotm.o ATL_f77rotm.c
cc -c -DL2SIZE=4194304 -DAdd_ -DF77_INTEGER=int -DStringSunStyle -DATL_SSE2 -DDREAL   -c -o ATL_f77rotmg.o ATL_f77rotmg.c
cc -c -DL2SIZE=4194304 -DAdd_ -DF77_INTEGER=int -DStringSunStyle -DATL_SSE2 -DDREAL   -c -o ATL_f77scal.o ATL_f77scal.c
cc -c -DL2SIZE=4194304 -DAdd_ -DF77_INTEGER=int -DStringSunStyle -DATL_SSE2 -DDREAL   -c -o ATL_f77swap.o ATL_f77swap.c
cc -c -DL2SIZE=4194304 -DAdd_ -DF77_INTEGER=int -DStringSunStyle -DATL_SSE2 -DDREAL   -c -o ATL_f77symm.o ATL_f77symm.c
cc -c -DL2SIZE=4194304 -DAdd_ -DF77_INTEGER=int -DStringSunStyle -DATL_SSE2 -DDREAL   -c -o ATL_f77syr2k.o ATL_f77syr2k.c
cc -c -DL2SIZE=4194304 -DAdd_ -DF77_INTEGER=int -DStringSunStyle -DATL_SSE2 -DDREAL   -c -o ATL_f77syrk.o ATL_f77syrk.c
cc -c -DL2SIZE=4194304 -DAdd_ -DF77_INTEGER=int -DStringSunStyle -DATL_SSE2 -DDREAL   -c -o ATL_f77trmm.o ATL_f77trmm.c
cc -c -DL2SIZE=4194304 -DAdd_ -DF77_INTEGER=int -DStringSunStyle -DATL_SSE2 -DDREAL   -c -o ATL_f77trsm.o ATL_f77trsm.c
cc -c -DL2SIZE=4194304 -DAdd_ -DF77_INTEGER=int -DStringSunStyle -DATL_SSE2 -DDREAL   -c -o ATL_flushcache.o ATL_flushcache.c
cc -c -DL2SIZE=4194304 -DAdd_ -DF77_INTEGER=int -DStringSunStyle -DATL_SSE2 -DDREAL   -c -o ATL_gediffnrm1.o ATL_gediffnrm1.c
cc -c -DL2SIZE=4194304 -DAdd_ -DF77_INTEGER=int -DStringSunStyle -DATL_SSE2 -DDREAL   -c -o ATL_gegen.o ATL_gegen.c
cc -c -DL2SIZE=4194304 -DAdd_ -DF77_INTEGER=int -DStringSunStyle -DATL_SSE2 -DDREAL   -c -o ATL_genrm1.o ATL_genrm1.c
cc -c -DL2SIZE=4194304 -DAdd_ -DF77_INTEGER=int -DStringSunStyle -DATL_SSE2 -DDREAL   -c -o ATL_infnrm.o ATL_infnrm.c
cc -c -DL2SIZE=4194304 -DAdd_ -DF77_INTEGER=int -DStringSunStyle -DATL_SSE2 -DDREAL   -c -o ATL_rand.o ATL_rand.c
cc -c -DL2SIZE=4194304 -DAdd_ -DF77_INTEGER=int -DStringSunStyle -DATL_SSE2 -DDREAL   -c -o ATL_set.o ATL_set.c
cc -c -DL2SIZE=4194304 -DAdd_ -DF77_INTEGER=int -DStringSunStyle -DATL_SSE2 -DDREAL   -c -o ATL_synrm.o ATL_synrm.c
cc -c -DL2SIZE=4194304 -DAdd_ -DF77_INTEGER=int -DStringSunStyle -DATL_SSE2 -DDREAL   -c -o ATL_trnrm1.o ATL_trnrm1.c
cc -c -DL2SIZE=4194304 -DAdd_ -DF77_INTEGER=int -DStringSunStyle -DATL_SSE2 -DDREAL   -c -o ATL_vdiff.o ATL_vdiff.c
cc -c -DL2SIZE=4194304 -DAdd_ -DF77_INTEGER=int -DStringSunStyle -DATL_SSE2 -DDREAL   -c -o ATL_zero.o ATL_zero.c
gfortran   -c -o ATL_df77wrap.o ATL_df77wrap.f
ar r libtstatlas.a  ATL_cputime.o  ATL_epsilon.o  ATL_f77amax.o  ATL_f77asum.o  ATL_f77axpy.o  ATL_f77copy.o  ATL_f77dot.o  ATL_f77gemm.o  ATL_f77nrm2.o  ATL_f77rot.o  ATL_f77rotg.o  ATL_f77rotm.o  ATL_f77rotmg.o  ATL_f77scal.o  ATL_f77swap.o  ATL_f77symm.o  ATL_f77syr2k.o  ATL_f77syrk.o  ATL_f77trmm.o  ATL_f77trsm.o  ATL_flushcache.o  ATL_gediffnrm1.o  ATL_gegen.o  ATL_genrm1.o  ATL_infnrm.o  ATL_rand.o  ATL_set.o  ATL_synrm.o  ATL_trnrm1.o  ATL_vdiff.o  ATL_zero.o  ATL_df77wrap.o
ar: creating archive libtstatlas.a
ranlib libtstatlas.a
c++ -o xdl1blastst l1blastst.o libtstatlas.a ../libatlulmblas.a /Users/clientadmin/work/eigen-3.2.1/blas/libeigen_blas_static.a
c++ -o xdl3blastst l3blastst.o libtstatlas.a ../libatlulmblas.a /Users/clientadmin/work/eigen-3.2.1/blas/libeigen_blas_static.a
```

</details>

Run the benchmark and extract the Eigen results:

```console
$ ./xdl3blastst -N 100 2000 100 > report
$ grep "\ \-\-\-\-\-$" report > eigen
$ cat eigen
```

<details>
<summary>Show complete Eigen benchmark results</summary>

```text
   0 N N  100  100  100   1.0 2000 2000   1.0 2000  0.00 5319.1 1.00 -----
   1 N N  200  200  200   1.0 2000 2000   1.0 2000  0.00 6983.8 1.00 -----
   2 N N  300  300  300   1.0 2000 2000   1.0 2000  0.01 7046.8 1.00 -----
   3 N N  400  400  400   1.0 2000 2000   1.0 2000  0.02 6942.9 1.00 -----
   4 N N  500  500  500   1.0 2000 2000   1.0 2000  0.03 7658.6 1.00 -----
   5 N N  600  600  600   1.0 2000 2000   1.0 2000  0.06 7711.3 1.00 -----
   6 N N  700  700  700   1.0 2000 2000   1.0 2000  0.09 7810.6 1.00 -----
   7 N N  800  800  800   1.0 2000 2000   1.0 2000  0.13 7749.2 1.00 -----
   8 N N  900  900  900   1.0 2000 2000   1.0 2000  0.19 7860.3 1.00 -----
   9 N N 1000 1000 1000   1.0 2000 2000   1.0 2000  0.25 7927.0 1.00 -----
  10 N N 1100 1100 1100   1.0 2000 2000   1.0 2000  0.34 7925.9 1.00 -----
  11 N N 1200 1200 1200   1.0 2000 2000   1.0 2000  0.43 7953.1 1.00 -----
  12 N N 1300 1300 1300   1.0 2000 2000   1.0 2000  0.55 7950.3 1.00 -----
  13 N N 1400 1400 1400   1.0 2000 2000   1.0 2000  0.69 7978.8 1.00 -----
  14 N N 1500 1500 1500   1.0 2000 2000   1.0 2000  0.84 8029.3 1.00 -----
  15 N N 1600 1600 1600   1.0 2000 2000   1.0 2000  1.02 7995.7 1.00 -----
  16 N N 1700 1700 1700   1.0 2000 2000   1.0 2000  1.22 8028.2 1.00 -----
  17 N N 1800 1800 1800   1.0 2000 2000   1.0 2000  1.45 8034.0 1.00 -----
  18 N N 1900 1900 1900   1.0 2000 2000   1.0 2000  1.70 8081.8 1.00 -----
  19 N N 2000 2000 2000   1.0 2000 2000   1.0 2000  1.98 8073.0 1.00 -----
```

</details>

For visualizing the results we use the gnuplot script `compare-bench1.gps`:

```gnuplot
set terminal svg size 1140,480
set output "compare-bench1.svg"
set title "Compute C + A*B"
set xlabel "Matrix dimensions N=M=K"
set ylabel "MFLOPS"
set yrange [0:9600]
set xrange [100:2000]
set key outside

plot "eigen" using 4:13 with linespoints lt 2 lw 2 \
     title "Eigen 3.2.1"
```

With

```console
$ gnuplot compare-bench1.gps
```

we get:

![Eigen benchmark](compare-bench1.svg)

## ATLAS

Next we benchmark the C library [ATLAS](https://math-atlas.sourceforge.net/) (Version 3.10.1).

Note that in `bench/Makefile` we expect this library to be installed in

```text
$HOME/work/work/ATLAS/build/
```

Change the value of `REF_LIB` according to your local installation.

Check out the `bench-atlas` branch:

```console
$ cd ulmBLAS
$ git checkout -B bench-atlas remotes/origin/bench-atlas
Switched to a new branch 'bench-atlas'
Branch bench-atlas set up to track remote branch bench-atlas from origin.

$ make
```

<details>
<summary>Show complete build output</summary>

```console
make -C src
make[1]: Nothing to be done for `all'.
make -C refblas
make[1]: Nothing to be done for `all'.
make -C test
make[1]: Nothing to be done for `all'.
make -C bench
make[1]: Nothing to be done for `all'.
```

</details>

Then we do the benchmark:

```console
$ cd bench
$ make clean
```

<details>
<summary>Show complete output of <code>make clean</code></summary>

```console
rm -f xdl1blastst libtstatlas.a l1blastst.o  ATL_cputime.o  ATL_epsilon.o  ATL_f77amax.o  ATL_f77asum.o  ATL_f77axpy.o  ATL_f77copy.o  ATL_f77dot.o  ATL_f77gemm.o  ATL_f77nrm2.o  ATL_f77rot.o  ATL_f77rotg.o  ATL_f77rotm.o  ATL_f77rotmg.o  ATL_f77scal.o  ATL_f77swap.o  ATL_f77symm.o  ATL_f77syr2k.o  ATL_f77syrk.o  ATL_f77trmm.o  ATL_f77trsm.o  ATL_flushcache.o  ATL_gediffnrm1.o  ATL_gegen.o  ATL_genrm1.o  ATL_infnrm.o  ATL_rand.o  ATL_set.o  ATL_synrm.o  ATL_trnrm1.o  ATL_vdiff.o  ATL_zero.o  ATL_df77wrap.o
```

</details>

Build the benchmark:

```console
$ make
```

<details>
<summary>Show complete benchmark build output</summary>

```console
gcc-4.8 -c -DL2SIZE=4194304 -DAdd_ -DF77_INTEGER=int -DStringSunStyle -DATL_SSE2 -DDREAL   -c -o l1blastst.o l1blastst.c
gcc-4.8 -c -DL2SIZE=4194304 -DAdd_ -DF77_INTEGER=int -DStringSunStyle -DATL_SSE2 -DDREAL   -c -o ATL_cputime.o ATL_cputime.c
gcc-4.8 -c -DL2SIZE=4194304 -DAdd_ -DF77_INTEGER=int -DStringSunStyle -DATL_SSE2 -DDREAL   -c -o ATL_epsilon.o ATL_epsilon.c
gcc-4.8 -c -DL2SIZE=4194304 -DAdd_ -DF77_INTEGER=int -DStringSunStyle -DATL_SSE2 -DDREAL   -c -o ATL_f77amax.o ATL_f77amax.c
gcc-4.8 -c -DL2SIZE=4194304 -DAdd_ -DF77_INTEGER=int -DStringSunStyle -DATL_SSE2 -DDREAL   -c -o ATL_f77asum.o ATL_f77asum.c
gcc-4.8 -c -DL2SIZE=4194304 -DAdd_ -DF77_INTEGER=int -DStringSunStyle -DATL_SSE2 -DDREAL   -c -o ATL_f77axpy.o ATL_f77axpy.c
gcc-4.8 -c -DL2SIZE=4194304 -DAdd_ -DF77_INTEGER=int -DStringSunStyle -DATL_SSE2 -DDREAL   -c -o ATL_f77copy.o ATL_f77copy.c
gcc-4.8 -c -DL2SIZE=4194304 -DAdd_ -DF77_INTEGER=int -DStringSunStyle -DATL_SSE2 -DDREAL   -c -o ATL_f77dot.o ATL_f77dot.c
gcc-4.8 -c -DL2SIZE=4194304 -DAdd_ -DF77_INTEGER=int -DStringSunStyle -DATL_SSE2 -DDREAL   -c -o ATL_f77gemm.o ATL_f77gemm.c
gcc-4.8 -c -DL2SIZE=4194304 -DAdd_ -DF77_INTEGER=int -DStringSunStyle -DATL_SSE2 -DDREAL   -c -o ATL_f77nrm2.o ATL_f77nrm2.c
gcc-4.8 -c -DL2SIZE=4194304 -DAdd_ -DF77_INTEGER=int -DStringSunStyle -DATL_SSE2 -DDREAL   -c -o ATL_f77rot.o ATL_f77rot.c
gcc-4.8 -c -DL2SIZE=4194304 -DAdd_ -DF77_INTEGER=int -DStringSunStyle -DATL_SSE2 -DDREAL   -c -o ATL_f77rotg.o ATL_f77rotg.c
gcc-4.8 -c -DL2SIZE=4194304 -DAdd_ -DF77_INTEGER=int -DStringSunStyle -DATL_SSE2 -DDREAL   -c -o ATL_f77rotm.o ATL_f77rotm.c
gcc-4.8 -c -DL2SIZE=4194304 -DAdd_ -DF77_INTEGER=int -DStringSunStyle -DATL_SSE2 -DDREAL   -c -o ATL_f77rotmg.o ATL_f77rotmg.c
gcc-4.8 -c -DL2SIZE=4194304 -DAdd_ -DF77_INTEGER=int -DStringSunStyle -DATL_SSE2 -DDREAL   -c -o ATL_f77scal.o ATL_f77scal.c
gcc-4.8 -c -DL2SIZE=4194304 -DAdd_ -DF77_INTEGER=int -DStringSunStyle -DATL_SSE2 -DDREAL   -c -o ATL_f77swap.o ATL_f77swap.c
gcc-4.8 -c -DL2SIZE=4194304 -DAdd_ -DF77_INTEGER=int -DStringSunStyle -DATL_SSE2 -DDREAL   -c -o ATL_f77symm.o ATL_f77symm.c
gcc-4.8 -c -DL2SIZE=4194304 -DAdd_ -DF77_INTEGER=int -DStringSunStyle -DATL_SSE2 -DDREAL   -c -o ATL_f77syr2k.o ATL_f77syr2k.c
gcc-4.8 -c -DL2SIZE=4194304 -DAdd_ -DF77_INTEGER=int -DStringSunStyle -DATL_SSE2 -DDREAL   -c -o ATL_f77syrk.o ATL_f77syrk.c
gcc-4.8 -c -DL2SIZE=4194304 -DAdd_ -DF77_INTEGER=int -DStringSunStyle -DATL_SSE2 -DDREAL   -c -o ATL_f77trmm.o ATL_f77trmm.c
gcc-4.8 -c -DL2SIZE=4194304 -DAdd_ -DF77_INTEGER=int -DStringSunStyle -DATL_SSE2 -DDREAL   -c -o ATL_f77trsm.o ATL_f77trsm.c
gcc-4.8 -c -DL2SIZE=4194304 -DAdd_ -DF77_INTEGER=int -DStringSunStyle -DATL_SSE2 -DDREAL   -c -o ATL_flushcache.o ATL_flushcache.c
gcc-4.8 -c -DL2SIZE=4194304 -DAdd_ -DF77_INTEGER=int -DStringSunStyle -DATL_SSE2 -DDREAL   -c -o ATL_gediffnrm1.o ATL_gediffnrm1.c
gcc-4.8 -c -DL2SIZE=4194304 -DAdd_ -DF77_INTEGER=int -DStringSunStyle -DATL_SSE2 -DDREAL   -c -o ATL_gegen.o ATL_gegen.c
gcc-4.8 -c -DL2SIZE=4194304 -DAdd_ -DF77_INTEGER=int -DStringSunStyle -DATL_SSE2 -DDREAL   -c -o ATL_genrm1.o ATL_genrm1.c
gcc-4.8 -c -DL2SIZE=4194304 -DAdd_ -DF77_INTEGER=int -DStringSunStyle -DATL_SSE2 -DDREAL   -c -o ATL_infnrm.o ATL_infnrm.c
gcc-4.8 -c -DL2SIZE=4194304 -DAdd_ -DF77_INTEGER=int -DStringSunStyle -DATL_SSE2 -DDREAL   -c -o ATL_rand.o ATL_rand.c
gcc-4.8 -c -DL2SIZE=4194304 -DAdd_ -DF77_INTEGER=int -DStringSunStyle -DATL_SSE2 -DDREAL   -c -o ATL_set.o ATL_set.c
gcc-4.8 -c -DL2SIZE=4194304 -DAdd_ -DF77_INTEGER=int -DStringSunStyle -DATL_SSE2 -DDREAL   -c -o ATL_synrm.o ATL_synrm.c
gcc-4.8 -c -DL2SIZE=4194304 -DAdd_ -DF77_INTEGER=int -DStringSunStyle -DATL_SSE2 -DDREAL   -c -o ATL_trnrm1.o ATL_trnrm1.c
gcc-4.8 -c -DL2SIZE=4194304 -DAdd_ -DF77_INTEGER=int -DStringSunStyle -DATL_SSE2 -DDREAL   -c -o ATL_vdiff.o ATL_vdiff.c
gcc-4.8 -c -DL2SIZE=4194304 -DAdd_ -DF77_INTEGER=int -DStringSunStyle -DATL_SSE2 -DDREAL   -c -o ATL_zero.o ATL_zero.c
gfortran   -c -o ATL_df77wrap.o ATL_df77wrap.f
ar r libtstatlas.a  ATL_cputime.o  ATL_epsilon.o  ATL_f77amax.o  ATL_f77asum.o  ATL_f77axpy.o  ATL_f77copy.o  ATL_f77dot.o  ATL_f77gemm.o  ATL_f77nrm2.o  ATL_f77rot.o  ATL_f77rotg.o  ATL_f77rotm.o  ATL_f77rotmg.o  ATL_f77scal.o  ATL_f77swap.o  ATL_f77symm.o  ATL_f77syr2k.o  ATL_f77syrk.o  ATL_f77trmm.o  ATL_f77trsm.o  ATL_flushcache.o  ATL_gediffnrm1.o  ATL_gegen.o  ATL_genrm1.o  ATL_infnrm.o  ATL_rand.o  ATL_set.o  ATL_synrm.o  ATL_trnrm1.o  ATL_vdiff.o  ATL_zero.o  ATL_df77wrap.o
ar: creating archive libtstatlas.a
ranlib libtstatlas.a
gfortran -o xdl1blastst l1blastst.o libtstatlas.a /Users/clientadmin/work/ATLAS/build/lib/libatlas.a ../librefblas.a
gfortran -o xdl3blastst l3blastst.o libtstatlas.a /Users/clientadmin/work/ATLAS/build/lib/libatlas.a ../librefblas.a
```

</details>

Run the benchmark:

```console
$ ./xdl3blastst -N 100 2000 100 > report
$ grep "PASS" report > atlas
$ cat atlas
```

<details>
<summary>Show complete ATLAS benchmark results</summary>

```text
   0 N N  100  100  100   1.0 2000 2000   1.0 2000  0.00 3246.8 1.70 PASS 
   1 N N  200  200  200   1.0 2000 2000   1.0 2000  0.00 7048.5 3.59 PASS 
   2 N N  300  300  300   1.0 2000 2000   1.0 2000  0.01 7602.4 3.78 PASS 
   3 N N  400  400  400   1.0 2000 2000   1.0 2000  0.02 8098.2 3.92 PASS 
   4 N N  500  500  500   1.0 2000 2000   1.0 2000  0.03 7975.8 3.85 PASS 
   5 N N  600  600  600   1.0 2000 2000   1.0 2000  0.05 8162.3 6.21 PASS 
   6 N N  700  700  700   1.0 2000 2000   1.0 2000  0.08 8275.8 8.07 PASS 
   7 N N  800  800  800   1.0 2000 2000   1.0 2000  0.12 8348.2 7.89 PASS 
   8 N N  900  900  900   1.0 2000 2000   1.0 2000  0.17 8347.2 7.89 PASS 
   9 N N 1000 1000 1000   1.0 2000 2000   1.0 2000  0.24 8419.3 7.84 PASS 
  10 N N 1100 1100 1100   1.0 2000 2000   1.0 2000  0.32 8444.0 7.80 PASS 
  11 N N 1200 1200 1200   1.0 2000 2000   1.0 2000  0.41 8479.8 7.85 PASS 
  12 N N 1300 1300 1300   1.0 2000 2000   1.0 2000  0.52 8495.1 7.78 PASS 
  13 N N 1400 1400 1400   1.0 2000 2000   1.0 2000  0.64 8555.0 7.80 PASS 
  14 N N 1500 1500 1500   1.0 2000 2000   1.0 2000  0.79 8535.9 7.76 PASS 
  15 N N 1600 1600 1600   1.0 2000 2000   1.0 2000  0.96 8563.4 7.72 PASS 
  16 N N 1700 1700 1700   1.0 2000 2000   1.0 2000  1.15 8561.8 7.76 PASS 
  17 N N 1800 1800 1800   1.0 2000 2000   1.0 2000  1.36 8569.5 7.75 PASS 
  18 N N 1900 1900 1900   1.0 2000 2000   1.0 2000  1.59 8608.5 7.77 PASS 
  19 N N 2000 2000 2000   1.0 2000 2000   1.0 2000  1.87 8545.5 7.32 PASS
```

</details>

For visualizing the results we use `compare-bench2.gps`:

```gnuplot
set terminal svg size 1140,480
set output "compare-bench2.svg"
set title "Compute C + A*B"
set xlabel "Matrix dimensions N=M=K"
set ylabel "MFLOPS"
set xrange [100:2000]
set yrange [0:9600]
set key outside

plot "eigen" using 4:13 with linespoints lt 2 lw 2 \
         title "Eigen 3.2.1", \
     "atlas" using 4:13 with linespoints lt 3 lw 2 \
         title "ATLAS 3.10.1"
```

With

```console
$ gnuplot compare-bench2.gps
```

we get:

![Eigen and ATLAS benchmark](compare-bench2.svg)

## Intel MKL

Next we benchmark the Intel Math Kernel Library (Version 13.1.3).

Note that in `bench/Makefile` we expect this library to be installed in

```text
MKLROOT = /opt/intel/composer_xe_2013_sp1.3.166/mkl/
```

Change the value of `MKLROOT` according to your local installation.

Check out the `bench-mkl` branch and build everything:

```console
$ cd ulmBLAS
$ git checkout -B bench-mkl remotes/origin/bench-mkl
Switched to a new branch 'bench-mkl'
Branch bench-mkl set up to track remote branch bench-mkl from origin.

$ make
```

<details>
<summary>Show complete build output</summary>

```console
make -C src
make[1]: Nothing to be done for `all'.
make -C refblas
make[1]: Nothing to be done for `all'.
make -C test
make[1]: Nothing to be done for `all'.
make -C bench
make[1]: Nothing to be done for `all'.
```

</details>

Then we do the benchmark:

```console
$ cd bench
$ make clean
```

<details>
<summary>Show complete output of <code>make clean</code></summary>

```console
rm -f xdl1blastst libtstatlas.a l1blastst.o  ATL_cputime.o  ATL_epsilon.o  ATL_f77amax.o  ATL_f77asum.o  ATL_f77axpy.o  ATL_f77copy.o  ATL_f77dot.o  ATL_f77gemm.o  ATL_f77nrm2.o  ATL_f77rot.o  ATL_f77rotg.o  ATL_f77rotm.o  ATL_f77rotmg.o  ATL_f77scal.o  ATL_f77swap.o  ATL_f77symm.o  ATL_f77syr2k.o  ATL_f77syrk.o  ATL_f77trmm.o  ATL_f77trsm.o  ATL_flushcache.o  ATL_gediffnrm1.o  ATL_gegen.o  ATL_genrm1.o  ATL_infnrm.o  ATL_rand.o  ATL_set.o  ATL_synrm.o  ATL_trnrm1.o  ATL_vdiff.o  ATL_zero.o  ATL_df77wrap.o
```

</details>

Build the benchmark:

```console
$ make
```

<details>
<summary>Show complete benchmark build output</summary>

```console
gcc-4.8 -c -DL2SIZE=4194304 -DAdd_ -DF77_INTEGER=int -DStringSunStyle -DATL_SSE2 -DDREAL   -c -o l1blastst.o l1blastst.c
gcc-4.8 -c -DL2SIZE=4194304 -DAdd_ -DF77_INTEGER=int -DStringSunStyle -DATL_SSE2 -DDREAL   -c -o ATL_cputime.o ATL_cputime.c
gcc-4.8 -c -DL2SIZE=4194304 -DAdd_ -DF77_INTEGER=int -DStringSunStyle -DATL_SSE2 -DDREAL   -c -o ATL_epsilon.o ATL_epsilon.c
gcc-4.8 -c -DL2SIZE=4194304 -DAdd_ -DF77_INTEGER=int -DStringSunStyle -DATL_SSE2 -DDREAL   -c -o ATL_f77amax.o ATL_f77amax.c
gcc-4.8 -c -DL2SIZE=4194304 -DAdd_ -DF77_INTEGER=int -DStringSunStyle -DATL_SSE2 -DDREAL   -c -o ATL_f77asum.o ATL_f77asum.c
gcc-4.8 -c -DL2SIZE=4194304 -DAdd_ -DF77_INTEGER=int -DStringSunStyle -DATL_SSE2 -DDREAL   -c -o ATL_f77axpy.o ATL_f77axpy.c
gcc-4.8 -c -DL2SIZE=4194304 -DAdd_ -DF77_INTEGER=int -DStringSunStyle -DATL_SSE2 -DDREAL   -c -o ATL_f77copy.o ATL_f77copy.c
gcc-4.8 -c -DL2SIZE=4194304 -DAdd_ -DF77_INTEGER=int -DStringSunStyle -DATL_SSE2 -DDREAL   -c -o ATL_f77dot.o ATL_f77dot.c
gcc-4.8 -c -DL2SIZE=4194304 -DAdd_ -DF77_INTEGER=int -DStringSunStyle -DATL_SSE2 -DDREAL   -c -o ATL_f77gemm.o ATL_f77gemm.c
gcc-4.8 -c -DL2SIZE=4194304 -DAdd_ -DF77_INTEGER=int -DStringSunStyle -DATL_SSE2 -DDREAL   -c -o ATL_f77nrm2.o ATL_f77nrm2.c
gcc-4.8 -c -DL2SIZE=4194304 -DAdd_ -DF77_INTEGER=int -DStringSunStyle -DATL_SSE2 -DDREAL   -c -o ATL_f77rot.o ATL_f77rot.c
gcc-4.8 -c -DL2SIZE=4194304 -DAdd_ -DF77_INTEGER=int -DStringSunStyle -DATL_SSE2 -DDREAL   -c -o ATL_f77rotg.o ATL_f77rotg.c
gcc-4.8 -c -DL2SIZE=4194304 -DAdd_ -DF77_INTEGER=int -DStringSunStyle -DATL_SSE2 -DDREAL   -c -o ATL_f77rotm.o ATL_f77rotm.c
gcc-4.8 -c -DL2SIZE=4194304 -DAdd_ -DF77_INTEGER=int -DStringSunStyle -DATL_SSE2 -DDREAL   -c -o ATL_f77rotmg.o ATL_f77rotmg.c
gcc-4.8 -c -DL2SIZE=4194304 -DAdd_ -DF77_INTEGER=int -DStringSunStyle -DATL_SSE2 -DDREAL   -c -o ATL_f77scal.o ATL_f77scal.c
gcc-4.8 -c -DL2SIZE=4194304 -DAdd_ -DF77_INTEGER=int -DStringSunStyle -DATL_SSE2 -DDREAL   -c -o ATL_f77swap.o ATL_f77swap.c
gcc-4.8 -c -DL2SIZE=4194304 -DAdd_ -DF77_INTEGER=int -DStringSunStyle -DATL_SSE2 -DDREAL   -c -o ATL_f77symm.o ATL_f77symm.c
gcc-4.8 -c -DL2SIZE=4194304 -DAdd_ -DF77_INTEGER=int -DStringSunStyle -DATL_SSE2 -DDREAL   -c -o ATL_f77syr2k.o ATL_f77syr2k.c
gcc-4.8 -c -DL2SIZE=4194304 -DAdd_ -DF77_INTEGER=int -DStringSunStyle -DATL_SSE2 -DDREAL   -c -o ATL_f77syrk.o ATL_f77syrk.c
gcc-4.8 -c -DL2SIZE=4194304 -DAdd_ -DF77_INTEGER=int -DStringSunStyle -DATL_SSE2 -DDREAL   -c -o ATL_f77trmm.o ATL_f77trmm.c
gcc-4.8 -c -DL2SIZE=4194304 -DAdd_ -DF77_INTEGER=int -DStringSunStyle -DATL_SSE2 -DDREAL   -c -o ATL_f77trsm.o ATL_f77trsm.c
gcc-4.8 -c -DL2SIZE=4194304 -DAdd_ -DF77_INTEGER=int -DStringSunStyle -DATL_SSE2 -DDREAL   -c -o ATL_flushcache.o ATL_flushcache.c
gcc-4.8 -c -DL2SIZE=4194304 -DAdd_ -DF77_INTEGER=int -DStringSunStyle -DATL_SSE2 -DDREAL   -c -o ATL_gediffnrm1.o ATL_gediffnrm1.c
gcc-4.8 -c -DL2SIZE=4194304 -DAdd_ -DF77_INTEGER=int -DStringSunStyle -DATL_SSE2 -DDREAL   -c -o ATL_gegen.o ATL_gegen.c
gcc-4.8 -c -DL2SIZE=4194304 -DAdd_ -DF77_INTEGER=int -DStringSunStyle -DATL_SSE2 -DDREAL   -c -o ATL_genrm1.o ATL_genrm1.c
gcc-4.8 -c -DL2SIZE=4194304 -DAdd_ -DF77_INTEGER=int -DStringSunStyle -DATL_SSE2 -DDREAL   -c -o ATL_infnrm.o ATL_infnrm.c
gcc-4.8 -c -DL2SIZE=4194304 -DAdd_ -DF77_INTEGER=int -DStringSunStyle -DATL_SSE2 -DDREAL   -c -o ATL_rand.o ATL_rand.c
gcc-4.8 -c -DL2SIZE=4194304 -DAdd_ -DF77_INTEGER=int -DStringSunStyle -DATL_SSE2 -DDREAL   -c -o ATL_set.o ATL_set.c
gcc-4.8 -c -DL2SIZE=4194304 -DAdd_ -DF77_INTEGER=int -DStringSunStyle -DATL_SSE2 -DDREAL   -c -o ATL_synrm.o ATL_synrm.c
gcc-4.8 -c -DL2SIZE=4194304 -DAdd_ -DF77_INTEGER=int -DStringSunStyle -DATL_SSE2 -DDREAL   -c -o ATL_trnrm1.o ATL_trnrm1.c
gcc-4.8 -c -DL2SIZE=4194304 -DAdd_ -DF77_INTEGER=int -DStringSunStyle -DATL_SSE2 -DDREAL   -c -o ATL_vdiff.o ATL_vdiff.c
gcc-4.8 -c -DL2SIZE=4194304 -DAdd_ -DF77_INTEGER=int -DStringSunStyle -DATL_SSE2 -DDREAL   -c -o ATL_zero.o ATL_zero.c
gfortran   -c -o ATL_df77wrap.o ATL_df77wrap.f
ar r libtstatlas.a  ATL_cputime.o  ATL_epsilon.o  ATL_f77amax.o  ATL_f77asum.o  ATL_f77axpy.o  ATL_f77copy.o  ATL_f77dot.o  ATL_f77gemm.o  ATL_f77nrm2.o  ATL_f77rot.o  ATL_f77rotg.o  ATL_f77rotm.o  ATL_f77rotmg.o  ATL_f77scal.o  ATL_f77swap.o  ATL_f77symm.o  ATL_f77syr2k.o  ATL_f77syrk.o  ATL_f77trmm.o  ATL_f77trsm.o  ATL_flushcache.o  ATL_gediffnrm1.o  ATL_gegen.o  ATL_genrm1.o  ATL_infnrm.o  ATL_rand.o  ATL_set.o  ATL_synrm.o  ATL_trnrm1.o  ATL_vdiff.o  ATL_zero.o  ATL_df77wrap.o
ar: creating archive libtstatlas.a
ranlib libtstatlas.a
gfortran -o xdl1blastst l1blastst.o libtstatlas.a ../libatlulmblas.a /opt/intel/composer_xe_2013_sp1.3.166/mkl//lib/libmkl_intel_lp64.a /opt/intel/composer_xe_2013_sp1.3.166/mkl//lib/libmkl_core.a /opt/intel/composer_xe_2013_sp1.3.166/mkl//lib/libmkl_sequential.a
gfortran -o xdl3blastst l3blastst.o libtstatlas.a ../libatlulmblas.a /opt/intel/composer_xe_2013_sp1.3.166/mkl//lib/libmkl_intel_lp64.a /opt/intel/composer_xe_2013_sp1.3.166/mkl//lib/libmkl_core.a /opt/intel/composer_xe_2013_sp1.3.166/mkl//lib/libmkl_sequential.a
```

</details>

Run the benchmark and extract the MKL results:

```console
$ ./xdl3blastst -N 100 2000 100 > report
$ grep "\ \-\-\-\-\-$" report > mkl
$ cat mkl
```

<details>
<summary>Show complete Intel MKL benchmark results</summary>

```text
   0 N N  100  100  100   1.0 2000 2000   1.0 2000  0.00 4926.1 1.00 -----
   1 N N  200  200  200   1.0 2000 2000   1.0 2000  0.00 8226.2 1.00 -----
   2 N N  300  300  300   1.0 2000 2000   1.0 2000  0.01 8620.7 1.00 -----
   3 N N  400  400  400   1.0 2000 2000   1.0 2000  0.01 8838.6 1.00 -----
   4 N N  500  500  500   1.0 2000 2000   1.0 2000  0.03 8725.1 1.00 -----
   5 N N  600  600  600   1.0 2000 2000   1.0 2000  0.05 8758.2 1.00 -----
   6 N N  700  700  700   1.0 2000 2000   1.0 2000  0.08 8874.2 1.00 -----
   7 N N  800  800  800   1.0 2000 2000   1.0 2000  0.11 8916.6 1.00 -----
   8 N N  900  900  900   1.0 2000 2000   1.0 2000  0.16 8843.1 1.00 -----
   9 N N 1000 1000 1000   1.0 2000 2000   1.0 2000  0.22 8958.0 1.00 -----
  10 N N 1100 1100 1100   1.0 2000 2000   1.0 2000  0.30 8952.4 1.00 -----
  11 N N 1200 1200 1200   1.0 2000 2000   1.0 2000  0.38 9010.0 1.00 -----
  12 N N 1300 1300 1300   1.0 2000 2000   1.0 2000  0.49 9015.9 1.00 -----
  13 N N 1400 1400 1400   1.0 2000 2000   1.0 2000  0.61 8949.6 1.00 -----
  14 N N 1500 1500 1500   1.0 2000 2000   1.0 2000  0.75 9044.0 1.00 -----
  15 N N 1600 1600 1600   1.0 2000 2000   1.0 2000  0.91 9031.2 1.00 -----
  16 N N 1700 1700 1700   1.0 2000 2000   1.0 2000  1.08 9059.7 1.00 -----
  17 N N 1800 1800 1800   1.0 2000 2000   1.0 2000  1.30 8983.4 1.00 -----
  18 N N 1900 1900 1900   1.0 2000 2000   1.0 2000  1.52 9032.9 1.00 -----
  19 N N 2000 2000 2000   1.0 2000 2000   1.0 2000  1.77 9060.5 1.00 -----
```

</details>

For visualizing the results we use `compare-bench3.gps`:

```gnuplot
set terminal svg size 1140,480
set output "compare-bench3.svg"
set title "Compute C + A*B"
set xlabel "Matrix dimensions N=M=K"
set ylabel "MFLOPS"
set xrange [100:2000]
set yrange [0:9600]
set key outside

plot "eigen" using 4:13 with linespoints lt 2 lw 2 \
         title "Eigen 3.2.1", \
     "atlas" using 4:13 with linespoints lt 3 lw 2 \
         title "ATLAS 3.10.1", \
     "mkl" using 4:13 with linespoints lt 4 lw 2 \
         title "Intel MKL 11.1.3"
```

With

```console
$ gnuplot compare-bench3.gps
```

we get:

![Eigen, ATLAS, and Intel MKL benchmark](compare-bench3.svg)

> **Historical note:** The original tutorial refers to Intel MKL Version 13.1.3 in the text, while the original plot legend says "Intel MKL 11.1.3". This discrepancy is preserved here.

## BLIS

Of course, we also benchmark the original [BLIS](https://github.com/flame/blis) library.

First clean the project:

```console
$ cd ulmBLAS
$ make clean
```

<details>
<summary>Show complete output of <code>make clean</code></summary>

```console
for dir in src refblas test bench; do make -C $dir clean; done
rm -f  auxiliary/xerbla.o  level1/dasum.o level1/daxpy.o level1/dcopy.o level1/ddot.o level1/dnrm2.o level1/drot.o level1/drotg.o level1/drotm.o level1/drotmg.o level1/dscal.o level1/dswap.o level1/idamax.o  level3/dgemm.o level3/dgemm_nn.o level3/dsymm.o level3/stubs.o
rm -f  auxiliary/atl_xerbla.o  level1/atl_dasum.o level1/atl_daxpy.o level1/atl_dcopy.o level1/atl_ddot.o level1/atl_dnrm2.o level1/atl_drot.o level1/atl_drotg.o level1/atl_drotm.o level1/atl_drotmg.o level1/atl_dscal.o level1/atl_dswap.o level1/atl_idamax.o  level3/atl_dgemm.o level3/atl_dgemm_nn.o level3/atl_dsymm.o level3/atl_stubs.o
rm -f ../libulmblas.a
rm -f ../libatlulmblas.a
rm -f caxpy.o ccopy.o cdotc.o cdotu.o cgbmv.o cgemm.o cgemv.o cgerc.o cgeru.o chbmv.o chemm.o chemv.o cher.o cher2.o cher2k.o cherk.o chpmv.o chpr.o chpr2.o crotg.o cscal.o csrot.o csscal.o cswap.o csymm.o csyr2k.o csyrk.o ctbmv.o ctbsv.o ctpmv.o ctpsv.o ctrmm.o ctrmv.o ctrsm.o ctrsv.o dasum.o daxpy.o dcabs1.o dcopy.o ddot.o dgbmv.o dgemm.o dgemv.o dger.o dnrm2.o drot.o drotg.o drotm.o drotmg.o dsbmv.o dscal.o dsdot.o dspmv.o dspr.o dspr2.o dswap.o dsymm.o dsymv.o dsyr.o dsyr2.o dsyr2k.o dsyrk.o dtbmv.o dtbsv.o dtpmv.o dtpsv.o dtrmm.o dtrmv.o dtrsm.o dtrsv.o dzasum.o dznrm2.o icamax.o idamax.o isamax.o izamax.o lsame.o sasum.o saxpy.o scabs1.o scasum.o scnrm2.o scopy.o sdot.o sdsdot.o sgbmv.o sgemm.o sgemv.o sger.o snrm2.o srot.o srotg.o srotm.o srotmg.o ssbmv.o sscal.o sspmv.o sspr.o sspr2.o sswap.o ssymm.o ssymv.o ssyr.o ssyr2.o ssyr2k.o ssyrk.o stbmv.o stbsv.o stpmv.o stpsv.o strmm.o strmv.o strsm.o strsv.o xerbla.o xerbla_array.o zaxpy.o zcopy.o zdotc.o zdotu.o zdrot.o zdscal.o zgbmv.o zgemm.o zgemv.o zgerc.o zgeru.o zhbmv.o zhemm.o zhemv.o zher.o zher2.o zher2k.o zherk.o zhpmv.o zhpr.o zhpr2.o zrotg.o zscal.o zswap.o zsymm.o zsyr2k.o zsyrk.o ztbmv.o ztbsv.o ztpmv.o ztpsv.o ztrmm.o ztrmv.o ztrsm.o ztrsv.o
rm -f ../librefblas.a
rm -f  dblat1_ref  dblat3_ref  dblat1_ulm  dblat3_ulm *.SUMM
rm -f xdl1blastst libtstatlas.a l1blastst.o  ATL_cputime.o  ATL_epsilon.o  ATL_f77amax.o  ATL_f77asum.o  ATL_f77axpy.o  ATL_f77copy.o  ATL_f77dot.o  ATL_f77gemm.o  ATL_f77nrm2.o  ATL_f77rot.o  ATL_f77rotg.o  ATL_f77rotm.o  ATL_f77rotmg.o  ATL_f77scal.o  ATL_f77swap.o  ATL_f77symm.o  ATL_f77syr2k.o  ATL_f77syrk.o  ATL_f77trmm.o  ATL_f77trsm.o  ATL_flushcache.o  ATL_gediffnrm1.o  ATL_gegen.o  ATL_genrm1.o  ATL_infnrm.o  ATL_rand.o  ATL_set.o  ATL_synrm.o  ATL_trnrm1.o  ATL_vdiff.o  ATL_zero.o  ATL_df77wrap.o
```

</details>

Check out the `bench-blis` branch:

```console
$ git checkout -B bench-blis remotes/origin/bench-blis
Switched to a new branch 'bench-blis'
Branch bench-blis set up to track remote branch bench-blis from origin.
```

Then build everything:

```console
$ make
```

<details>
<summary>Show complete build output</summary>

```console
make -C src
clang -Wall -I. -O3 -msse3 -mfpmath=sse -fomit-frame-pointer -DULM_BLOCKED -c -o auxiliary/xerbla.o auxiliary/xerbla.c
clang -Wall -I. -O3 -msse3 -mfpmath=sse -fomit-frame-pointer -DULM_BLOCKED -c -o level1/dasum.o level1/dasum.c
clang -Wall -I. -O3 -msse3 -mfpmath=sse -fomit-frame-pointer -DULM_BLOCKED -c -o level1/daxpy.o level1/daxpy.c
clang -Wall -I. -O3 -msse3 -mfpmath=sse -fomit-frame-pointer -DULM_BLOCKED -c -o level1/dcopy.o level1/dcopy.c
clang -Wall -I. -O3 -msse3 -mfpmath=sse -fomit-frame-pointer -DULM_BLOCKED -c -o level1/ddot.o level1/ddot.c
clang -Wall -I. -O3 -msse3 -mfpmath=sse -fomit-frame-pointer -DULM_BLOCKED -c -o level1/dnrm2.o level1/dnrm2.c
clang -Wall -I. -O3 -msse3 -mfpmath=sse -fomit-frame-pointer -DULM_BLOCKED -c -o level1/drot.o level1/drot.c
clang -Wall -I. -O3 -msse3 -mfpmath=sse -fomit-frame-pointer -DULM_BLOCKED -c -o level1/drotg.o level1/drotg.c
clang -Wall -I. -O3 -msse3 -mfpmath=sse -fomit-frame-pointer -DULM_BLOCKED -c -o level1/drotm.o level1/drotm.c
clang -Wall -I. -O3 -msse3 -mfpmath=sse -fomit-frame-pointer -DULM_BLOCKED -c -o level1/drotmg.o level1/drotmg.c
clang -Wall -I. -O3 -msse3 -mfpmath=sse -fomit-frame-pointer -DULM_BLOCKED -c -o level1/dscal.o level1/dscal.c
clang -Wall -I. -O3 -msse3 -mfpmath=sse -fomit-frame-pointer -DULM_BLOCKED -c -o level1/dswap.o level1/dswap.c
clang -Wall -I. -O3 -msse3 -mfpmath=sse -fomit-frame-pointer -DULM_BLOCKED -c -o level1/idamax.o level1/idamax.c
clang -Wall -I. -O3 -msse3 -mfpmath=sse -fomit-frame-pointer -DULM_BLOCKED -c -o level3/dgemm.o level3/dgemm.c
clang -Wall -I. -O3 -msse3 -mfpmath=sse -fomit-frame-pointer -DULM_BLOCKED -c -o level3/dgemm_nn.o level3/dgemm_nn.c
clang -Wall -I. -O3 -msse3 -mfpmath=sse -fomit-frame-pointer -DULM_BLOCKED -c -o level3/dsymm.o level3/dsymm.c
clang -Wall -I. -O3 -msse3 -mfpmath=sse -fomit-frame-pointer -DULM_BLOCKED -c -o level3/stubs.o level3/stubs.c
ar cru ../libulmblas.a  auxiliary/xerbla.o  level1/dasum.o level1/daxpy.o level1/dcopy.o level1/ddot.o level1/dnrm2.o level1/drot.o level1/drotg.o level1/drotm.o level1/drotmg.o level1/dscal.o level1/dswap.o level1/idamax.o  level3/dgemm.o level3/dgemm_nn.o level3/dsymm.o level3/stubs.o
ranlib ../libulmblas.a
clang -Wall -I. -O3 -msse3 -mfpmath=sse -fomit-frame-pointer -DULM_BLOCKED -DFAKE_ATLAS -c -o auxiliary/atl_xerbla.o auxiliary/xerbla.c
clang -Wall -I. -O3 -msse3 -mfpmath=sse -fomit-frame-pointer -DULM_BLOCKED -DFAKE_ATLAS -c -o level1/atl_dasum.o level1/dasum.c
clang -Wall -I. -O3 -msse3 -mfpmath=sse -fomit-frame-pointer -DULM_BLOCKED -DFAKE_ATLAS -c -o level1/atl_daxpy.o level1/daxpy.c
clang -Wall -I. -O3 -msse3 -mfpmath=sse -fomit-frame-pointer -DULM_BLOCKED -DFAKE_ATLAS -c -o level1/atl_dcopy.o level1/dcopy.c
clang -Wall -I. -O3 -msse3 -mfpmath=sse -fomit-frame-pointer -DULM_BLOCKED -DFAKE_ATLAS -c -o level1/atl_ddot.o level1/ddot.c
clang -Wall -I. -O3 -msse3 -mfpmath=sse -fomit-frame-pointer -DULM_BLOCKED -DFAKE_ATLAS -c -o level1/atl_dnrm2.o level1/dnrm2.c
clang -Wall -I. -O3 -msse3 -mfpmath=sse -fomit-frame-pointer -DULM_BLOCKED -DFAKE_ATLAS -c -o level1/atl_drot.o level1/drot.c
clang -Wall -I. -O3 -msse3 -mfpmath=sse -fomit-frame-pointer -DULM_BLOCKED -DFAKE_ATLAS -c -o level1/atl_drotg.o level1/drotg.c
clang -Wall -I. -O3 -msse3 -mfpmath=sse -fomit-frame-pointer -DULM_BLOCKED -DFAKE_ATLAS -c -o level1/atl_drotm.o level1/drotm.c
clang -Wall -I. -O3 -msse3 -mfpmath=sse -fomit-frame-pointer -DULM_BLOCKED -DFAKE_ATLAS -c -o level1/atl_drotmg.o level1/drotmg.c
clang -Wall -I. -O3 -msse3 -mfpmath=sse -fomit-frame-pointer -DULM_BLOCKED -DFAKE_ATLAS -c -o level1/atl_dscal.o level1/dscal.c
clang -Wall -I. -O3 -msse3 -mfpmath=sse -fomit-frame-pointer -DULM_BLOCKED -DFAKE_ATLAS -c -o level1/atl_dswap.o level1/dswap.c
clang -Wall -I. -O3 -msse3 -mfpmath=sse -fomit-frame-pointer -DULM_BLOCKED -DFAKE_ATLAS -c -o level1/atl_idamax.o level1/idamax.c
clang -Wall -I. -O3 -msse3 -mfpmath=sse -fomit-frame-pointer -DULM_BLOCKED -DFAKE_ATLAS -c -o level3/atl_dgemm.o level3/dgemm.c
clang -Wall -I. -O3 -msse3 -mfpmath=sse -fomit-frame-pointer -DULM_BLOCKED -DFAKE_ATLAS -c -o level3/atl_dgemm_nn.o level3/dgemm_nn.c
clang -Wall -I. -O3 -msse3 -mfpmath=sse -fomit-frame-pointer -DULM_BLOCKED -DFAKE_ATLAS -c -o level3/atl_dsymm.o level3/dsymm.c
clang -Wall -I. -O3 -msse3 -mfpmath=sse -fomit-frame-pointer -DULM_BLOCKED -DFAKE_ATLAS -c -o level3/atl_stubs.o level3/stubs.c
ar cru ../libatlulmblas.a  auxiliary/atl_xerbla.o  level1/atl_dasum.o level1/atl_daxpy.o level1/atl_dcopy.o level1/atl_ddot.o level1/atl_dnrm2.o level1/atl_drot.o level1/atl_drotg.o level1/atl_drotm.o level1/atl_drotmg.o level1/atl_dscal.o level1/atl_dswap.o level1/atl_idamax.o  level3/atl_dgemm.o level3/atl_dgemm_nn.o level3/atl_dsymm.o level3/atl_stubs.o
ranlib ../libatlulmblas.a
make -C refblas
gfortran -fimplicit-none -O3 -c -o caxpy.o caxpy.f
gfortran -fimplicit-none -O3 -c -o ccopy.o ccopy.f
gfortran -fimplicit-none -O3 -c -o cdotc.o cdotc.f
gfortran -fimplicit-none -O3 -c -o cdotu.o cdotu.f
gfortran -fimplicit-none -O3 -c -o cgbmv.o cgbmv.f
gfortran -fimplicit-none -O3 -c -o cgemm.o cgemm.f
gfortran -fimplicit-none -O3 -c -o cgemv.o cgemv.f
gfortran -fimplicit-none -O3 -c -o cgerc.o cgerc.f
gfortran -fimplicit-none -O3 -c -o cgeru.o cgeru.f
gfortran -fimplicit-none -O3 -c -o chbmv.o chbmv.f
gfortran -fimplicit-none -O3 -c -o chemm.o chemm.f
gfortran -fimplicit-none -O3 -c -o chemv.o chemv.f
gfortran -fimplicit-none -O3 -c -o cher.o cher.f
gfortran -fimplicit-none -O3 -c -o cher2.o cher2.f
gfortran -fimplicit-none -O3 -c -o cher2k.o cher2k.f
gfortran -fimplicit-none -O3 -c -o cherk.o cherk.f
gfortran -fimplicit-none -O3 -c -o chpmv.o chpmv.f
gfortran -fimplicit-none -O3 -c -o chpr.o chpr.f
gfortran -fimplicit-none -O3 -c -o chpr2.o chpr2.f
gfortran -fimplicit-none -O3 -c -o crotg.o crotg.f
gfortran -fimplicit-none -O3 -c -o cscal.o cscal.f
gfortran -fimplicit-none -O3 -c -o csrot.o csrot.f
gfortran -fimplicit-none -O3 -c -o csscal.o csscal.f
gfortran -fimplicit-none -O3 -c -o cswap.o cswap.f
gfortran -fimplicit-none -O3 -c -o csymm.o csymm.f
gfortran -fimplicit-none -O3 -c -o csyr2k.o csyr2k.f
gfortran -fimplicit-none -O3 -c -o csyrk.o csyrk.f
gfortran -fimplicit-none -O3 -c -o ctbmv.o ctbmv.f
gfortran -fimplicit-none -O3 -c -o ctbsv.o ctbsv.f
gfortran -fimplicit-none -O3 -c -o ctpmv.o ctpmv.f
gfortran -fimplicit-none -O3 -c -o ctpsv.o ctpsv.f
gfortran -fimplicit-none -O3 -c -o ctrmm.o ctrmm.f
gfortran -fimplicit-none -O3 -c -o ctrmv.o ctrmv.f
gfortran -fimplicit-none -O3 -c -o ctrsm.o ctrsm.f
gfortran -fimplicit-none -O3 -c -o ctrsv.o ctrsv.f
gfortran -fimplicit-none -O3 -c -o dasum.o dasum.f
gfortran -fimplicit-none -O3 -c -o daxpy.o daxpy.f
gfortran -fimplicit-none -O3 -c -o dcabs1.o dcabs1.f
gfortran -fimplicit-none -O3 -c -o dcopy.o dcopy.f
gfortran -fimplicit-none -O3 -c -o ddot.o ddot.f
gfortran -fimplicit-none -O3 -c -o dgbmv.o dgbmv.f
gfortran -fimplicit-none -O3 -c -o dgemm.o dgemm.f
gfortran -fimplicit-none -O3 -c -o dgemv.o dgemv.f
gfortran -fimplicit-none -O3 -c -o dger.o dger.f
gfortran -fimplicit-none -O3 -c -o dnrm2.o dnrm2.f
gfortran -fimplicit-none -O3 -c -o drot.o drot.f
gfortran -fimplicit-none -O3 -c -o drotg.o drotg.f
gfortran -fimplicit-none -O3 -c -o drotm.o drotm.f
gfortran -fimplicit-none -O3 -c -o drotmg.o drotmg.f
gfortran -fimplicit-none -O3 -c -o dsbmv.o dsbmv.f
gfortran -fimplicit-none -O3 -c -o dscal.o dscal.f
gfortran -fimplicit-none -O3 -c -o dsdot.o dsdot.f
gfortran -fimplicit-none -O3 -c -o dspmv.o dspmv.f
gfortran -fimplicit-none -O3 -c -o dspr.o dspr.f
gfortran -fimplicit-none -O3 -c -o dspr2.o dspr2.f
gfortran -fimplicit-none -O3 -c -o dswap.o dswap.f
gfortran -fimplicit-none -O3 -c -o dsymm.o dsymm.f
gfortran -fimplicit-none -O3 -c -o dsymv.o dsymv.f
gfortran -fimplicit-none -O3 -c -o dsyr.o dsyr.f
gfortran -fimplicit-none -O3 -c -o dsyr2.o dsyr2.f
gfortran -fimplicit-none -O3 -c -o dsyr2k.o dsyr2k.f
gfortran -fimplicit-none -O3 -c -o dsyrk.o dsyrk.f
gfortran -fimplicit-none -O3 -c -o dtbmv.o dtbmv.f
gfortran -fimplicit-none -O3 -c -o dtbsv.o dtbsv.f
gfortran -fimplicit-none -O3 -c -o dtpmv.o dtpmv.f
gfortran -fimplicit-none -O3 -c -o dtpsv.o dtpsv.f
gfortran -fimplicit-none -O3 -c -o dtrmm.o dtrmm.f
gfortran -fimplicit-none -O3 -c -o dtrmv.o dtrmv.f
gfortran -fimplicit-none -O3 -c -o dtrsm.o dtrsm.f
gfortran -fimplicit-none -O3 -c -o dtrsv.o dtrsv.f
gfortran -fimplicit-none -O3 -c -o dzasum.o dzasum.f
gfortran -fimplicit-none -O3 -c -o dznrm2.o dznrm2.f
gfortran -fimplicit-none -O3 -c -o icamax.o icamax.f
gfortran -fimplicit-none -O3 -c -o idamax.o idamax.f
gfortran -fimplicit-none -O3 -c -o isamax.o isamax.f
gfortran -fimplicit-none -O3 -c -o izamax.o izamax.f
gfortran -fimplicit-none -O3 -c -o lsame.o lsame.f
gfortran -fimplicit-none -O3 -c -o sasum.o sasum.f
gfortran -fimplicit-none -O3 -c -o saxpy.o saxpy.f
gfortran -fimplicit-none -O3 -c -o scabs1.o scabs1.f
gfortran -fimplicit-none -O3 -c -o scasum.o scasum.f
gfortran -fimplicit-none -O3 -c -o scnrm2.o scnrm2.f
gfortran -fimplicit-none -O3 -c -o scopy.o scopy.f
gfortran -fimplicit-none -O3 -c -o sdot.o sdot.f
gfortran -fimplicit-none -O3 -c -o sdsdot.o sdsdot.f
gfortran -fimplicit-none -O3 -c -o sgbmv.o sgbmv.f
gfortran -fimplicit-none -O3 -c -o sgemm.o sgemm.f
gfortran -fimplicit-none -O3 -c -o sgemv.o sgemv.f
gfortran -fimplicit-none -O3 -c -o sger.o sger.f
gfortran -fimplicit-none -O3 -c -o snrm2.o snrm2.f
gfortran -fimplicit-none -O3 -c -o srot.o srot.f
gfortran -fimplicit-none -O3 -c -o srotg.o srotg.f
gfortran -fimplicit-none -O3 -c -o srotm.o srotm.f
gfortran -fimplicit-none -O3 -c -o srotmg.o srotmg.f
gfortran -fimplicit-none -O3 -c -o ssbmv.o ssbmv.f
gfortran -fimplicit-none -O3 -c -o sscal.o sscal.f
gfortran -fimplicit-none -O3 -c -o sspmv.o sspmv.f
gfortran -fimplicit-none -O3 -c -o sspr.o sspr.f
gfortran -fimplicit-none -O3 -c -o sspr2.o sspr2.f
gfortran -fimplicit-none -O3 -c -o sswap.o sswap.f
gfortran -fimplicit-none -O3 -c -o ssymm.o ssymm.f
gfortran -fimplicit-none -O3 -c -o ssymv.o ssymv.f
gfortran -fimplicit-none -O3 -c -o ssyr.o ssyr.f
gfortran -fimplicit-none -O3 -c -o ssyr2.o ssyr2.f
gfortran -fimplicit-none -O3 -c -o ssyr2k.o ssyr2k.f
gfortran -fimplicit-none -O3 -c -o ssyrk.o ssyrk.f
gfortran -fimplicit-none -O3 -c -o stbmv.o stbmv.f
gfortran -fimplicit-none -O3 -c -o stbsv.o stbsv.f
gfortran -fimplicit-none -O3 -c -o stpmv.o stpmv.f
gfortran -fimplicit-none -O3 -c -o stpsv.o stpsv.f
gfortran -fimplicit-none -O3 -c -o strmm.o strmm.f
gfortran -fimplicit-none -O3 -c -o strmv.o strmv.f
gfortran -fimplicit-none -O3 -c -o strsm.o strsm.f
gfortran -fimplicit-none -O3 -c -o strsv.o strsv.f
gfortran -fimplicit-none -O3 -c -o xerbla.o xerbla.f
gfortran -fimplicit-none -O3 -c -o xerbla_array.o xerbla_array.f
gfortran -fimplicit-none -O3 -c -o zaxpy.o zaxpy.f
gfortran -fimplicit-none -O3 -c -o zcopy.o zcopy.f
gfortran -fimplicit-none -O3 -c -o zdotc.o zdotc.f
gfortran -fimplicit-none -O3 -c -o zdotu.o zdotu.f
gfortran -fimplicit-none -O3 -c -o zdrot.o zdrot.f
gfortran -fimplicit-none -O3 -c -o zdscal.o zdscal.f
gfortran -fimplicit-none -O3 -c -o zgbmv.o zgbmv.f
gfortran -fimplicit-none -O3 -c -o zgemm.o zgemm.f
gfortran -fimplicit-none -O3 -c -o zgemv.o zgemv.f
gfortran -fimplicit-none -O3 -c -o zgerc.o zgerc.f
gfortran -fimplicit-none -O3 -c -o zgeru.o zgeru.f
gfortran -fimplicit-none -O3 -c -o zhbmv.o zhbmv.f
gfortran -fimplicit-none -O3 -c -o zhemm.o zhemm.f
gfortran -fimplicit-none -O3 -c -o zhemv.o zhemv.f
gfortran -fimplicit-none -O3 -c -o zher.o zher.f
gfortran -fimplicit-none -O3 -c -o zher2.o zher2.f
gfortran -fimplicit-none -O3 -c -o zher2k.o zher2k.f
gfortran -fimplicit-none -O3 -c -o zherk.o zherk.f
gfortran -fimplicit-none -O3 -c -o zhpmv.o zhpmv.f
gfortran -fimplicit-none -O3 -c -o zhpr.o zhpr.f
gfortran -fimplicit-none -O3 -c -o zhpr2.o zhpr2.f
gfortran -fimplicit-none -O3 -c -o zrotg.o zrotg.f
gfortran -fimplicit-none -O3 -c -o zscal.o zscal.f
gfortran -fimplicit-none -O3 -c -o zswap.o zswap.f
gfortran -fimplicit-none -O3 -c -o zsymm.o zsymm.f
gfortran -fimplicit-none -O3 -c -o zsyr2k.o zsyr2k.f
gfortran -fimplicit-none -O3 -c -o zsyrk.o zsyrk.f
gfortran -fimplicit-none -O3 -c -o ztbmv.o ztbmv.f
gfortran -fimplicit-none -O3 -c -o ztbsv.o ztbsv.f
gfortran -fimplicit-none -O3 -c -o ztpmv.o ztpmv.f
gfortran -fimplicit-none -O3 -c -o ztpsv.o ztpsv.f
gfortran -fimplicit-none -O3 -c -o ztrmm.o ztrmm.f
gfortran -fimplicit-none -O3 -c -o ztrmv.o ztrmv.f
gfortran -fimplicit-none -O3 -c -o ztrsm.o ztrsm.f
gfortran -fimplicit-none -O3 -c -o ztrsv.o ztrsv.f
ar cru ../librefblas.a caxpy.o ccopy.o cdotc.o cdotu.o cgbmv.o cgemm.o cgemv.o cgerc.o cgeru.o chbmv.o chemm.o chemv.o cher.o cher2.o cher2k.o cherk.o chpmv.o chpr.o chpr2.o crotg.o cscal.o csrot.o csscal.o cswap.o csymm.o csyr2k.o csyrk.o ctbmv.o ctbsv.o ctpmv.o ctpsv.o ctrmm.o ctrmv.o ctrsm.o ctrsv.o dasum.o daxpy.o dcabs1.o dcopy.o ddot.o dgbmv.o dgemm.o dgemv.o dger.o dnrm2.o drot.o drotg.o drotm.o drotmg.o dsbmv.o dscal.o dsdot.o dspmv.o dspr.o dspr2.o dswap.o dsymm.o dsymv.o dsyr.o dsyr2.o dsyr2k.o dsyrk.o dtbmv.o dtbsv.o dtpmv.o dtpsv.o dtrmm.o dtrmv.o dtrsm.o dtrsv.o dzasum.o dznrm2.o icamax.o idamax.o isamax.o izamax.o lsame.o sasum.o saxpy.o scabs1.o scasum.o scnrm2.o scopy.o sdot.o sdsdot.o sgbmv.o sgemm.o sgemv.o sger.o snrm2.o srot.o srotg.o srotm.o srotmg.o ssbmv.o sscal.o sspmv.o sspr.o sspr2.o sswap.o ssymm.o ssymv.o ssyr.o ssyr2.o ssyr2k.o ssyrk.o stbmv.o stbsv.o stpmv.o stpsv.o strmm.o strmv.o strsm.o strsv.o xerbla.o xerbla_array.o zaxpy.o zcopy.o zdotc.o zdotu.o zdrot.o zdscal.o zgbmv.o zgemm.o zgemv.o zgerc.o zgeru.o zhbmv.o zhemm.o zhemv.o zher.o zher2.o zher2k.o zherk.o zhpmv.o zhpr.o zhpr2.o zrotg.o zscal.o zswap.o zsymm.o zsyr2k.o zsyrk.o ztbmv.o ztbsv.o ztpmv.o ztpsv.o ztrmm.o ztrmv.o ztrsm.o ztrsv.o
ranlib ../librefblas.a
make -C test
gfortran dblat1.f -L.. -lrefblas -o dblat1_ref
dblat1.f:215.44:
               CALL STEST1(DNRM2(N,SX,INCX),STEMP,STEMP,SFAC)           
                                            1
Warning: Rank mismatch in argument 'strue1' at (1) (scalar and rank-1)
dblat1.f:219.44:
               CALL STEST1(DASUM(N,SX,INCX),STEMP,STEMP,SFAC)           
                                            1
Warning: Rank mismatch in argument 'strue1' at (1) (scalar and rank-1)
gfortran dblat3.f -L.. -lrefblas -o dblat3_ref
gfortran dblat1.f -L.. -lulmblas -o dblat1_ulm
dblat1.f:215.44:
               CALL STEST1(DNRM2(N,SX,INCX),STEMP,STEMP,SFAC)           
                                            1
Warning: Rank mismatch in argument 'strue1' at (1) (scalar and rank-1)
dblat1.f:219.44:
               CALL STEST1(DASUM(N,SX,INCX),STEMP,STEMP,SFAC)           
                                            1
Warning: Rank mismatch in argument 'strue1' at (1) (scalar and rank-1)
gfortran dblat3.f -L.. -lulmblas -o dblat3_ulm
make -C bench
gcc-4.8 -c -DL2SIZE=4194304 -DAdd_ -DF77_INTEGER=int -DStringSunStyle -DATL_SSE2 -DDREAL   -c -o l1blastst.o l1blastst.c
gcc-4.8 -c -DL2SIZE=4194304 -DAdd_ -DF77_INTEGER=int -DStringSunStyle -DATL_SSE2 -DDREAL   -c -o ATL_cputime.o ATL_cputime.c
gcc-4.8 -c -DL2SIZE=4194304 -DAdd_ -DF77_INTEGER=int -DStringSunStyle -DATL_SSE2 -DDREAL   -c -o ATL_epsilon.o ATL_epsilon.c
gcc-4.8 -c -DL2SIZE=4194304 -DAdd_ -DF77_INTEGER=int -DStringSunStyle -DATL_SSE2 -DDREAL   -c -o ATL_f77amax.o ATL_f77amax.c
gcc-4.8 -c -DL2SIZE=4194304 -DAdd_ -DF77_INTEGER=int -DStringSunStyle -DATL_SSE2 -DDREAL   -c -o ATL_f77asum.o ATL_f77asum.c
gcc-4.8 -c -DL2SIZE=4194304 -DAdd_ -DF77_INTEGER=int -DStringSunStyle -DATL_SSE2 -DDREAL   -c -o ATL_f77axpy.o ATL_f77axpy.c
gcc-4.8 -c -DL2SIZE=4194304 -DAdd_ -DF77_INTEGER=int -DStringSunStyle -DATL_SSE2 -DDREAL   -c -o ATL_f77copy.o ATL_f77copy.c
gcc-4.8 -c -DL2SIZE=4194304 -DAdd_ -DF77_INTEGER=int -DStringSunStyle -DATL_SSE2 -DDREAL   -c -o ATL_f77dot.o ATL_f77dot.c
gcc-4.8 -c -DL2SIZE=4194304 -DAdd_ -DF77_INTEGER=int -DStringSunStyle -DATL_SSE2 -DDREAL   -c -o ATL_f77gemm.o ATL_f77gemm.c
gcc-4.8 -c -DL2SIZE=4194304 -DAdd_ -DF77_INTEGER=int -DStringSunStyle -DATL_SSE2 -DDREAL   -c -o ATL_f77nrm2.o ATL_f77nrm2.c
gcc-4.8 -c -DL2SIZE=4194304 -DAdd_ -DF77_INTEGER=int -DStringSunStyle -DATL_SSE2 -DDREAL   -c -o ATL_f77rot.o ATL_f77rot.c
gcc-4.8 -c -DL2SIZE=4194304 -DAdd_ -DF77_INTEGER=int -DStringSunStyle -DATL_SSE2 -DDREAL   -c -o ATL_f77rotg.o ATL_f77rotg.c
gcc-4.8 -c -DL2SIZE=4194304 -DAdd_ -DF77_INTEGER=int -DStringSunStyle -DATL_SSE2 -DDREAL   -c -o ATL_f77rotm.o ATL_f77rotm.c
gcc-4.8 -c -DL2SIZE=4194304 -DAdd_ -DF77_INTEGER=int -DStringSunStyle -DATL_SSE2 -DDREAL   -c -o ATL_f77rotmg.o ATL_f77rotmg.c
gcc-4.8 -c -DL2SIZE=4194304 -DAdd_ -DF77_INTEGER=int -DStringSunStyle -DATL_SSE2 -DDREAL   -c -o ATL_f77scal.o ATL_f77scal.c
gcc-4.8 -c -DL2SIZE=4194304 -DAdd_ -DF77_INTEGER=int -DStringSunStyle -DATL_SSE2 -DDREAL   -c -o ATL_f77swap.o ATL_f77swap.c
gcc-4.8 -c -DL2SIZE=4194304 -DAdd_ -DF77_INTEGER=int -DStringSunStyle -DATL_SSE2 -DDREAL   -c -o ATL_f77symm.o ATL_f77symm.c
gcc-4.8 -c -DL2SIZE=4194304 -DAdd_ -DF77_INTEGER=int -DStringSunStyle -DATL_SSE2 -DDREAL   -c -o ATL_f77syr2k.o ATL_f77syr2k.c
gcc-4.8 -c -DL2SIZE=4194304 -DAdd_ -DF77_INTEGER=int -DStringSunStyle -DATL_SSE2 -DDREAL   -c -o ATL_f77syrk.o ATL_f77syrk.c
gcc-4.8 -c -DL2SIZE=4194304 -DAdd_ -DF77_INTEGER=int -DStringSunStyle -DATL_SSE2 -DDREAL   -c -o ATL_f77trmm.o ATL_f77trmm.c
gcc-4.8 -c -DL2SIZE=4194304 -DAdd_ -DF77_INTEGER=int -DStringSunStyle -DATL_SSE2 -DDREAL   -c -o ATL_f77trsm.o ATL_f77trsm.c
gcc-4.8 -c -DL2SIZE=4194304 -DAdd_ -DF77_INTEGER=int -DStringSunStyle -DATL_SSE2 -DDREAL   -c -o ATL_flushcache.o ATL_flushcache.c
gcc-4.8 -c -DL2SIZE=4194304 -DAdd_ -DF77_INTEGER=int -DStringSunStyle -DATL_SSE2 -DDREAL   -c -o ATL_gediffnrm1.o ATL_gediffnrm1.c
gcc-4.8 -c -DL2SIZE=4194304 -DAdd_ -DF77_INTEGER=int -DStringSunStyle -DATL_SSE2 -DDREAL   -c -o ATL_gegen.o ATL_gegen.c
gcc-4.8 -c -DL2SIZE=4194304 -DAdd_ -DF77_INTEGER=int -DStringSunStyle -DATL_SSE2 -DDREAL   -c -o ATL_genrm1.o ATL_genrm1.c
gcc-4.8 -c -DL2SIZE=4194304 -DAdd_ -DF77_INTEGER=int -DStringSunStyle -DATL_SSE2 -DDREAL   -c -o ATL_infnrm.o ATL_infnrm.c
gcc-4.8 -c -DL2SIZE=4194304 -DAdd_ -DF77_INTEGER=int -DStringSunStyle -DATL_SSE2 -DDREAL   -c -o ATL_rand.o ATL_rand.c
gcc-4.8 -c -DL2SIZE=4194304 -DAdd_ -DF77_INTEGER=int -DStringSunStyle -DATL_SSE2 -DDREAL   -c -o ATL_set.o ATL_set.c
gcc-4.8 -c -DL2SIZE=4194304 -DAdd_ -DF77_INTEGER=int -DStringSunStyle -DATL_SSE2 -DDREAL   -c -o ATL_synrm.o ATL_synrm.c
gcc-4.8 -c -DL2SIZE=4194304 -DAdd_ -DF77_INTEGER=int -DStringSunStyle -DATL_SSE2 -DDREAL   -c -o ATL_trnrm1.o ATL_trnrm1.c
gcc-4.8 -c -DL2SIZE=4194304 -DAdd_ -DF77_INTEGER=int -DStringSunStyle -DATL_SSE2 -DDREAL   -c -o ATL_vdiff.o ATL_vdiff.c
gcc-4.8 -c -DL2SIZE=4194304 -DAdd_ -DF77_INTEGER=int -DStringSunStyle -DATL_SSE2 -DDREAL   -c -o ATL_zero.o ATL_zero.c
gfortran   -c -o ATL_df77wrap.o ATL_df77wrap.f
ar r libtstatlas.a  ATL_cputime.o  ATL_epsilon.o  ATL_f77amax.o  ATL_f77asum.o  ATL_f77axpy.o  ATL_f77copy.o  ATL_f77dot.o  ATL_f77gemm.o  ATL_f77nrm2.o  ATL_f77rot.o  ATL_f77rotg.o  ATL_f77rotm.o  ATL_f77rotmg.o  ATL_f77scal.o  ATL_f77swap.o  ATL_f77symm.o  ATL_f77syr2k.o  ATL_f77syrk.o  ATL_f77trmm.o  ATL_f77trsm.o  ATL_flushcache.o  ATL_gediffnrm1.o  ATL_gegen.o  ATL_genrm1.o  ATL_infnrm.o  ATL_rand.o  ATL_set.o  ATL_synrm.o  ATL_trnrm1.o  ATL_vdiff.o  ATL_zero.o  ATL_df77wrap.o
ar: creating archive libtstatlas.a
ranlib libtstatlas.a
gfortran -o xdl1blastst l1blastst.o libtstatlas.a ../libatlulmblas.a /Users/clientadmin/work/blis/lib/dunnington/libblis.a
gfortran -o xdl3blastst l3blastst.o libtstatlas.a ../libatlulmblas.a /Users/clientadmin/work/blis/lib/dunnington/libblis.a
```

</details>

Then we do the benchmark:

```console
$ cd bench
$ ./xdl3blastst -N 100 2000 100 > report
$ grep "\ \-\-\-\-\-$" report > blis
$ cat blis
```

<details>
<summary>Show complete BLIS benchmark results</summary>

```text
   0 N N  100  100  100   1.0 2000 2000   1.0 2000  0.00 2958.6 1.00 -----
   1 N N  200  200  200   1.0 2000 2000   1.0 2000  0.00 7568.6 1.00 -----
   2 N N  300  300  300   1.0 2000 2000   1.0 2000  0.01 8016.6 1.00 -----
   3 N N  400  400  400   1.0 2000 2000   1.0 2000  0.02 7961.2 1.00 -----
   4 N N  500  500  500   1.0 2000 2000   1.0 2000  0.03 8157.1 1.00 -----
   5 N N  600  600  600   1.0 2000 2000   1.0 2000  0.05 8337.7 1.00 -----
   6 N N  700  700  700   1.0 2000 2000   1.0 2000  0.08 8477.3 1.00 -----
   7 N N  800  800  800   1.0 2000 2000   1.0 2000  0.12 8433.8 1.00 -----
   8 N N  900  900  900   1.0 2000 2000   1.0 2000  0.17 8565.8 1.00 -----
   9 N N 1000 1000 1000   1.0 2000 2000   1.0 2000  0.23 8643.8 1.00 -----
  10 N N 1100 1100 1100   1.0 2000 2000   1.0 2000  0.31 8685.8 1.00 -----
  11 N N 1200 1200 1200   1.0 2000 2000   1.0 2000  0.40 8653.8 1.00 -----
  12 N N 1300 1300 1300   1.0 2000 2000   1.0 2000  0.50 8716.1 1.00 -----
  13 N N 1400 1400 1400   1.0 2000 2000   1.0 2000  0.63 8765.9 1.00 -----
  14 N N 1500 1500 1500   1.0 2000 2000   1.0 2000  0.77 8757.0 1.00 -----
  15 N N 1600 1600 1600   1.0 2000 2000   1.0 2000  0.94 8737.4 1.00 -----
  16 N N 1700 1700 1700   1.0 2000 2000   1.0 2000  1.12 8792.6 1.00 -----
  17 N N 1800 1800 1800   1.0 2000 2000   1.0 2000  1.32 8810.0 1.00 -----
  18 N N 1900 1900 1900   1.0 2000 2000   1.0 2000  1.55 8828.0 1.00 -----
  19 N N 2000 2000 2000   1.0 2000 2000   1.0 2000  1.82 8805.9 1.00 -----
```

</details>

For visualizing the results we use `compare-bench4.gps`:

```gnuplot
set terminal svg size 1140,480
set output "compare-bench4.svg"
set title "Compute C + A*B"
set xlabel "Matrix dimensions N=M=K"
set ylabel "MFLOPS"
set xrange [100:2000]
set yrange [0:9600]
set key outside

plot "eigen" using 4:13 with linespoints lt 2 lw 2 \
         title "Eigen 3.2.1", \
     "atlas" using 4:13 with linespoints lt 3 lw 2 \
         title "ATLAS 3.10.1", \
     "mkl" using 4:13 with linespoints lt 4 lw 2 \
         title "Intel MKL 11.1.3", \
     "blis" using 4:13 with linespoints lt 5 lw 2 \
         title "BLIS"
```

With

```console
$ gnuplot compare-bench4.gps
```

we get:

![Eigen, ATLAS, Intel MKL, and BLIS benchmark](compare-bench4.svg)

## ulmBLAS

So what about us?

We include the benchmark results from the [previous page](../page13/README.md), where we arrived at the `demo-sse-all-asm-with-prefetching` implementation.

The final gnuplot script `compare-bench5.gps` is:

```gnuplot
set terminal svg size 1140,480
set output "compare-bench5.svg"
set title "Compute C + A*B"
set xlabel "Matrix dimensions N=M=K"
set ylabel "MFLOPS"
set xrange [100:2000]
set yrange [0:9600]
set key outside

plot "eigen" using 4:13 with linespoints lt 2 lw 2 \
         title "Eigen 3.2.1", \
     "atlas" using 4:13 with linespoints lt 3 lw 2 \
         title "ATLAS 3.10.1", \
     "mkl" using 4:13 with linespoints lt 4 lw 2 \
         title "Intel MKL 11.1.3", \
     "blis" using 4:13 with linespoints lt 5 lw 2 \
         title "BLIS", \
     "demo-sse-all-asm-with-prefetching" using 4:13 with linespoints lt 6 lw 2 \
         title "ulmBLAS (demo-sse-all-asm-with-prefetching)"
```

Generate the final comparison with

```console
$ gnuplot compare-bench5.gps
```

and get:

![Final benchmark comparison](compare-bench5.svg)

---

[← Previous](../page13/README.md) | [Main Page](../README.md)

Copyright © 2014 Michael Lehn

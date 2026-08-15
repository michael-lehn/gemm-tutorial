# Loop Unrolling

We optimize the previous implementation by manually unrolling the loop.

Again, the performance achieved here is what I would have expected from a smart compiler for the `demo-pure-c` branch — at least if we provide enough hints through compiler flags and attributes.

## Select the `demo-naive-sse-with-intrinsics-unrolled` Branch

As before, first clean the project:

```console id="i1rgi7"
$ cd ulmBLAS
$ make clean
```

<details>
<summary>Show complete output of <code>make clean</code></summary>

```console id="0bn3d5"
for dir in src refblas test bench; do make -C $dir clean; done
rm -f auxiliary/xerbla.o level1/dasum.o level1/daxpy.o ...
rm -f auxiliary/atl_xerbla.o level1/atl_dasum.o level1/atl_daxpy.o ...
rm -f ../libulmblas.a
rm -f ../libatlulmblas.a
...
```

</details>

Now check out the `demo-naive-sse-with-intrinsics-unrolled` branch:

```console id="26j319"
$ git branch -a
* demo-naive-sse-with-intrinsics
  demo-pure-c
  master
  remotes/origin/HEAD -> origin/master
  remotes/origin/bench-atlas
  remotes/origin/bench-blis
  remotes/origin/bench-eigen
  remotes/origin/bench-mkl
  remotes/origin/blis-avx-microkernel
  remotes/origin/demo-naive-avx-with-intrinsics
  remotes/origin/demo-naive-sse-with-intrinsics
  remotes/origin/demo-naive-sse-with-intrinsics-unrolled
  remotes/origin/demo-pure-c
  remotes/origin/demo-sse-all-asm
  remotes/origin/demo-sse-all-asm-try-prefetching
  remotes/origin/demo-sse-all-asm-try-prefetching-v2
  remotes/origin/demo-sse-all-asm-with-prefetching
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
  remotes/origin/trsm-assignment
  remotes/origin/trsm-pure-c

$ git checkout -B demo-naive-sse-with-intrinsics-unrolled remotes/origin/demo-naive-sse-with-intrinsics-unrolled
Switched to a new branch 'demo-naive-sse-with-intrinsics-unrolled'
Branch demo-naive-sse-with-intrinsics-unrolled set up to track remote branch demo-naive-sse-with-intrinsics-unrolled from origin.
```

Then compile the project:

```console id="kv5h03"
$ make
```

<details>
<summary>Show complete build output</summary>

```console id="9euvyq"
make -C src
gcc-4.8 -Wall -I. -O2 -msse3 -mfpmath=sse -fomit-frame-pointer -c -o auxiliary/xerbla.o auxiliary/xerbla.c
gcc-4.8 -Wall -I. -O2 -msse3 -mfpmath=sse -fomit-frame-pointer -c -o level1/dasum.o level1/dasum.c
gcc-4.8 -Wall -I. -O2 -msse3 -mfpmath=sse -fomit-frame-pointer -c -o level1/daxpy.o level1/daxpy.c
...
gfortran -o xdl1blastst l1blastst.o libtstatlas.a ../libatlulmblas.a ../librefblas.a
gfortran -o xdl3blastst l3blastst.o libtstatlas.a ../libatlulmblas.a ../librefblas.a
```

</details>

## Unrolling

The previous micro kernel performs one rank-one update per loop iteration.

Here we manually unroll the loop by a factor of four. Instead of

```c id="jmj6lv"
for (l=0; l<kc; ++l) {
    /*
     * one update
     */
}
```

we effectively perform

```c id="j7phlr"
for (l=0; l<kc/4; ++l) {
    /*
     * update 1
     * update 2
     * update 3
     * update 4
     */
}
```

followed by a cleanup loop for the remaining `kc % 4` iterations.

The purpose is not to change the arithmetic. We still compute exactly the same sequence of rank-one updates. Unrolling reduces loop overhead and, more importantly, gives the processor and compiler a larger instruction window in which independent operations can be scheduled.

> **TODO from the original 2014 tutorial**
>
> Add some of the lecture notes about pipelining, branch prediction, and prefetching.
>
> Add pictures showing how this is realized in the implementation.

## The `dgemm_nn` Code

The complete implementation is contained in

[`src/level3/dgemm_nn.c`](https://github.com/michael-lehn/ulmBLAS/blob/demo-naive-sse-with-intrinsics-unrolled/src/level3/dgemm_nn.c)

on the `demo-naive-sse-with-intrinsics-unrolled` branch.

The main difference is in the computational loop. Four consecutive updates are written out explicitly:

```c id="9q8v6f"
for (l=0; l<kc/4; ++l) {

    /*
     * Update 1
     */
    a_01 = _mm_load_pd(A);
    a_23 = _mm_load_pd(A+2);

    b_00 = _mm_load_pd1(B);
    b_11 = _mm_load_pd1(B+1);
    b_22 = _mm_load_pd1(B+2);
    b_33 = _mm_load_pd1(B+3);

    tmp1 = a_01;
    tmp2 = a_23;

    // col 0 of AB
    tmp1 = _mm_mul_pd(tmp1, b_00);
    tmp2 = _mm_mul_pd(tmp2, b_00);
    ab_00_10 = _mm_add_pd(tmp1, ab_00_10);
    ab_20_30 = _mm_add_pd(tmp2, ab_20_30);

    // col 1 of AB
    tmp1 = a_01;
    tmp2 = a_23;
    tmp1 = _mm_mul_pd(tmp1, b_11);
    tmp2 = _mm_mul_pd(tmp2, b_11);
    ab_01_11 = _mm_add_pd(tmp1, ab_01_11);
    ab_21_31 = _mm_add_pd(tmp2, ab_21_31);

    // col 2 of AB
    tmp1 = a_01;
    tmp2 = a_23;
    tmp1 = _mm_mul_pd(tmp1, b_22);
    tmp2 = _mm_mul_pd(tmp2, b_22);
    ab_02_12 = _mm_add_pd(tmp1, ab_02_12);
    ab_22_32 = _mm_add_pd(tmp2, ab_22_32);

    // col 3 of AB
    tmp1 = a_01;
    tmp2 = a_23;
    tmp1 = _mm_mul_pd(tmp1, b_33);
    tmp2 = _mm_mul_pd(tmp2, b_33);
    ab_03_13 = _mm_add_pd(tmp1, ab_03_13);
    ab_23_33 = _mm_add_pd(tmp2, ab_23_33);

    A += 4;
    B += 4;

    /*
     * Updates 2, 3 and 4 follow in exactly the same form.
     */
}
```

After the unrolled loop, the remaining updates are handled by

```c id="mkru6f"
for (l=0; l<kc%4; ++l) {
    a_01 = _mm_load_pd(A);
    a_23 = _mm_load_pd(A+2);

    b_00 = _mm_load_pd1(B);
    b_11 = _mm_load_pd1(B+1);
    b_22 = _mm_load_pd1(B+2);
    b_33 = _mm_load_pd1(B+3);

    /*
     * Perform the same four column updates as before.
     */

    A += 4;
    B += 4;
}
```

The complete source file shows all four explicitly unrolled updates.

## Benchmark Results

Run the benchmark:

```console id="1xxl0e"
$ cd bench
$ ./xdl3blastst > report
$ cat report
```

<details>
<summary>Show complete benchmark output</summary>

```text id="xlzp4v"
./xdl3blastst
--------------------------------- GEMM ----------------------------------
TST# A B    M    N    K ALPHA  LDA  LDB  BETA  LDC  TIME MFLOP SpUp  TEST
==== = = ==== ==== ==== ===== ==== ==== ===== ==== ===== ===== ==== =====
   0 N N  100  100  100   1.0 1000 1000   1.0 1000  0.00 1798.6 1.00 -----
   0 N N  100  100  100   1.0 1000 1000   1.0 1000  0.00 4132.2 2.30 PASS
   1 N N  200  200  200   1.0 1000 1000   1.0 1000  0.01 1960.8 1.00 -----
   1 N N  200  200  200   1.0 1000 1000   1.0 1000  0.00 4770.4 2.43 PASS
   2 N N  300  300  300   1.0 1000 1000   1.0 1000  0.03 2000.0 1.00 -----
   2 N N  300  300  300   1.0 1000 1000   1.0 1000  0.01 4873.6 2.44 PASS
   3 N N  400  400  400   1.0 1000 1000   1.0 1000  0.07 1929.6 1.00 -----
   3 N N  400  400  400   1.0 1000 1000   1.0 1000  0.03 4881.0 2.53 PASS
   4 N N  500  500  500   1.0 1000 1000   1.0 1000  0.12 2063.5 1.00 -----
   4 N N  500  500  500   1.0 1000 1000   1.0 1000  0.05 4986.1 2.42 PASS
   5 N N  600  600  600   1.0 1000 1000   1.0 1000  0.32 1339.5 1.00 -----
   5 N N  600  600  600   1.0 1000 1000   1.0 1000  0.09 5059.9 3.78 PASS
   6 N N  700  700  700   1.0 1000 1000   1.0 1000  0.66 1032.4 1.00 -----
   6 N N  700  700  700   1.0 1000 1000   1.0 1000  0.13 5095.1 4.94 PASS
   7 N N  800  800  800   1.0 1000 1000   1.0 1000  0.97 1053.7 1.00 -----
   7 N N  800  800  800   1.0 1000 1000   1.0 1000  0.20 5055.0 4.80 PASS
   8 N N  900  900  900   1.0 1000 1000   1.0 1000  1.38 1053.9 1.00 -----
   8 N N  900  900  900   1.0 1000 1000   1.0 1000  0.29 5099.5 4.84 PASS
   9 N N 1000 1000 1000   1.0 1000 1000   1.0 1000  1.73 1154.4 1.00 -----
   9 N N 1000 1000 1000   1.0 1000 1000   1.0 1000  0.39 5133.4 4.45 PASS

10 tests run, 10 passed
```

</details>

Filter out the results for this implementation:

```console id="tqrmnu"
$ grep PASS report > demo-naive-sse-with-intrinsics-unrolled
```

We can now compare all implementations so far.

```gnuplot id="8xpamc"
set terminal svg size 1140,480 background rgb 'white'
set output "bench4.svg"

set xlabel "Matrix dimensions N=M=K"
set ylabel "MFLOPS"
set yrange [0:9600]
set title "Compute C + A*B"
set key outside

plot "refBLAS" using 4:13 with linespoints lt 2 \
         title "Netlib RefBLAS", \
     "demo-pure-c" using 4:13 with linespoints lt 4 \
         title "demo-pure-c", \
     "demo-naive-sse-with-intrinsics" using 4:13 with linespoints lt 5 \
         title "demo-naive-sse-with-intrinsics", \
     "demo-naive-sse-with-intrinsics-unrolled" using 4:13 with linespoints lt 6 \
         title "demo-naive-sse-with-intrinsics-unrolled"
```

Generate the plot with

```console id="0h6dkf"
$ gnuplot bench4.gps
```

![Benchmark results](bench4.svg)

The manually unrolled version reaches a little over **5.1 GFLOPS** on the original benchmark machine, compared with about **4.8 GFLOPS** for the previous SSE-intrinsics implementation.

---

[← Previous](../page03/README.md) | [Main Page](../README.md) | [Next →](../page05/README.md)

Copyright © 2014 Michael Lehn
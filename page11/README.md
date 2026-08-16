# More Fine-tuning of the Unrolled Assembler Kernel

In the last implementation, the two pointer increments were performed directly after each other:

```asm
addq    $4*32, %rax    # A += 16
addq    $4*32, %rbx    # B += 16
```

These instructions compete for the same execution resources of the CPU. Pulling them apart further improves performance.

## Select the `demo-sse-asm-unrolled-v3` Branch

Again, we do a `make clean` before switching branches:

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

Then we check out the `demo-sse-asm-unrolled-v3` branch:

```console
$ git branch -a
  demo-naive-sse-with-intrinsics
  demo-naive-sse-with-intrinsics-unrolled
  demo-pure-c
  demo-sse-asm
  demo-sse-asm-unrolled
* demo-sse-asm-unrolled-v2
  demo-sse-intrinsics
  demo-sse-intrinsics-v2
  demo-sse-intrinsics-v3
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

$ git checkout -B demo-sse-asm-unrolled-v3 remotes/origin/demo-sse-asm-unrolled-v3
Switched to a new branch 'demo-sse-asm-unrolled-v3'
Branch demo-sse-asm-unrolled-v3 set up to track remote branch demo-sse-asm-unrolled-v3 from origin.
```

Then we compile the project:

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
gfortran -o xdl1blastst l1blastst.o libtstatlas.a ../libatlulmblas.a ../librefblas.a
gfortran -o xdl3blastst l3blastst.o libtstatlas.a ../libatlulmblas.a ../librefblas.a
```

</details>

The historical build uses Clang with

```text
-O3 -msse3 -mfpmath=sse -fomit-frame-pointer
```

for the ulmBLAS sources.

## The `dgemm_nn` Code

The complete implementation is available in

[`src/level3/dgemm_nn.c`](https://github.com/michael-lehn/ulmBLAS/blob/demo-sse-asm-unrolled-v3/src/level3/dgemm_nn.c)

on the `demo-sse-asm-unrolled-v3` branch.

<details>
<summary>Show complete <code>dgemm_nn.c</code></summary>

```c
#include <ulmblas.h>
#include <stdio.h>
#include <emmintrin.h>
#include <immintrin.h>

#define MC  384
#define KC  384
#define NC  4096

#define MR  4
#define NR  4

//
//  Local buffers for storing panels from A, B and C
//
static double _A[MC*KC] __attribute__ ((aligned (16)));
static double _B[KC*NC] __attribute__ ((aligned (16)));
static double _C[MR*NR] __attribute__ ((aligned (16)));

//
//  Packing complete panels from A (i.e. without padding)
//
static void
pack_MRxk(int k, const double *A, int incRowA, int incColA,
          double *buffer)
{
    int i, j;

    for (j=0; j<k; ++j) {
        for (i=0; i<MR; ++i) {
            buffer[i] = A[i*incRowA];
        }
        buffer += MR;
        A      += incColA;
    }
}

//
//  Packing panels from A with padding if required
//
static void
pack_A(int mc, int kc, const double *A, int incRowA, int incColA,
       double *buffer)
{
    int mp  = mc / MR;
    int _mr = mc % MR;

    int i, j;

    for (i=0; i<mp; ++i) {
        pack_MRxk(kc, A, incRowA, incColA, buffer);
        buffer += kc*MR;
        A      += MR*incRowA;
    }
    if (_mr>0) {
        for (j=0; j<kc; ++j) {
            for (i=0; i<_mr; ++i) {
                buffer[i] = A[i*incRowA];
            }
            for (i=_mr; i<MR; ++i) {
                buffer[i] = 0.0;
            }
            buffer += MR;
            A      += incColA;
        }
    }
}

//
//  Packing complete panels from B (i.e. without padding)
//
static void
pack_kxNR(int k, const double *B, int incRowB, int incColB,
          double *buffer)
{
    int i, j;

    for (i=0; i<k; ++i) {
        for (j=0; j<NR; ++j) {
            buffer[j] = B[j*incColB];
        }
        buffer += NR;
        B      += incRowB;
    }
}

//
//  Packing panels from B with padding if required
//
static void
pack_B(int kc, int nc, const double *B, int incRowB, int incColB,
       double *buffer)
{
    int np  = nc / NR;
    int _nr = nc % NR;

    int i, j;

    for (j=0; j<np; ++j) {
        pack_kxNR(kc, B, incRowB, incColB, buffer);
        buffer += kc*NR;
        B      += NR*incColB;
    }
    if (_nr>0) {
        for (i=0; i<kc; ++i) {
            for (j=0; j<_nr; ++j) {
                buffer[j] = B[j*incColB];
            }
            for (j=_nr; j<NR; ++j) {
                buffer[j] = 0.0;
            }
            buffer += NR;
            B      += incRowB;
        }
    }
}

//
//  Micro kernel for multiplying panels from A and B.
//
static void
dgemm_micro_kernel(long kc,
                   double alpha, const double *A, const double *B,
                   double beta,
                   double *C, long incRowC, long incColC,
                   const double *nextA, const double *nextB)
{
    long kb = kc / 4;
    long kl = kc % 4;

//
//  Compute AB = A*B
//
    __asm__ volatile
    (
    "movq      %0,      %%rsi    \n\t"  // kb (32 bit) stored in %rsi
    "movq      %1,      %%rdi    \n\t"  // kl (32 bit) stored in %rdi
    "movq      %2,      %%rax    \n\t"  // Address of A stored in %rax
    "movq      %3,      %%rbx    \n\t"  // Address of B stored in %rbx
    "movq      %9,      %%r9     \n\t"  // Address of nextA stored in %r9
    "movq      %10,     %%r10    \n\t"  // Address of nextB stored in %r10
    "                            \n\t"
    "addq      $128,    %%rax    \n\t"
    "addq      $128,    %%rbx    \n\t"
    "                            \n\t"
    "movapd -128(%%rax),%%xmm0   \n\t"  // tmp0 = _mm_load_pd(A)
    "movapd -112(%%rax),%%xmm1   \n\t"  // tmp1 = _mm_load_pd(A+2)
    "movapd -128(%%rbx),%%xmm2   \n\t"  // tmp2 = _mm_load_pd(B)
    "                            \n\t"
    "xorpd     %%xmm8,  %%xmm8   \n\t"  // ab_00_11 = _mm_setzero_pd()
    "xorpd     %%xmm9,  %%xmm9   \n\t"  // ab_20_31 = _mm_setzero_pd()
    "xorpd     %%xmm10, %%xmm10  \n\t"  // ab_01_10 = _mm_setzero_pd()
    "xorpd     %%xmm11, %%xmm11  \n\t"  // ab_21_30 = _mm_setzero_pd()
    "xorpd     %%xmm12, %%xmm12  \n\t"  // ab_02_13 = _mm_setzero_pd()
    "xorpd     %%xmm13, %%xmm13  \n\t"  // ab_22_33 = _mm_setzero_pd()
    "xorpd     %%xmm14, %%xmm14  \n\t"  // ab_03_12 = _mm_setzero_pd()
    "xorpd     %%xmm15, %%xmm15  \n\t"  // ab_23_32 = _mm_setzero_pd()
    "                            \n\t"
    "xorpd     %%xmm3,  %%xmm3   \n\t"  // tmp3 = _mm_setzero_pd
    "xorpd     %%xmm4,  %%xmm4   \n\t"  // tmp4 = _mm_setzero_pd
    "xorpd     %%xmm5,  %%xmm5   \n\t"  // tmp5 = _mm_setzero_pd
    "xorpd     %%xmm6,  %%xmm6   \n\t"  // tmp6 = _mm_setzero_pd
    "xorpd     %%xmm7,  %%xmm7   \n\t"  // tmp7 = _mm_setzero_pd
    "testq     %%rdi,   %%rdi    \n\t"  // if kl==0 writeback to AB
    "                            \n\t"
    "                            \n\t"
    "testq     %%rsi,   %%rsi    \n\t"  // if kb==0 handle remaining kl
    "je        .DCONSIDERLEFT%=  \n\t"  // update iterations
    "                            \n\t"
    ".DLOOP%=:                   \n\t"  // for l = kb,..,1 do
    "                            \n\t"
    "prefetcht0 (4*35+1)*8(%%rax)\n\t"
    "                            \n\t"
    "                            \n\t"  // 1. update
    "addpd     %%xmm3,  %%xmm12  \n\t"  // ab_02_13 = _mm_add_pd(ab_02_13, tmp3)
    "movaps -112(%%rbx),%%xmm3   \n\t"  // tmp3     = _mm_load_pd(B+2)
    "addpd     %%xmm6,  %%xmm13  \n\t"  // ab_22_33 = _mm_add_pd(ab_22_33, tmp6)
    "movaps    %%xmm2,  %%xmm6   \n\t"  // tmp6     = tmp2
    "pshufd $78,%%xmm2, %%xmm4   \n\t"  // tmp4     = _mm_shuffle_pd(tmp2, tmp2,
    "                            \n\t"  //                   _MM_SHUFFLE2(0, 1))
    "mulpd     %%xmm0,  %%xmm2   \n\t"  // tmp2     = _mm_mul_pd(tmp2, tmp0);
    "mulpd     %%xmm1,  %%xmm6   \n\t"  // tmp6     = _mm_mul_pd(tmp6, tmp1);
    "                            \n\t"
    "                            \n\t"
    "addpd     %%xmm5,  %%xmm14  \n\t"  // ab_03_12 = _mm_add_pd(ab_03_12, tmp5)
    "addpd     %%xmm7,  %%xmm15  \n\t"  // ab_23_32 = _mm_add_pd(ab_23_32, tmp7)
    "movaps    %%xmm4,  %%xmm7   \n\t"  // tmp7     = tmp4
    "mulpd     %%xmm0,  %%xmm4   \n\t"  // tmp4     = _mm_mul_pd(tmp4, tmp0)
    "mulpd     %%xmm1,  %%xmm7   \n\t"  // tmp7     = _mm_mul_pd(tmp7, tmp1)
    "                            \n\t"
    "                            \n\t"
    "addpd     %%xmm2,  %%xmm8   \n\t"  // ab_00_11 = _mm_add_pd(ab_00_11, tmp2)
    "movaps -96(%%rbx), %%xmm2   \n\t"  // tmp2     = _mm_load_pd(B+4)
    "addpd     %%xmm6,  %%xmm9   \n\t"  // ab_20_31 = _mm_add_pd(ab_20_31, tmp6)
    "movaps    %%xmm3,  %%xmm6   \n\t"  // tmp6     = tmp3
    "pshufd $78,%%xmm3, %%xmm5   \n\t"  // tmp5     = _mm_shuffle_pd(tmp3, tmp3,
    "                            \n\t"  //                   _MM_SHUFFLE2(0, 1))
    "mulpd     %%xmm0,  %%xmm3   \n\t"  // tmp3     = _mm_mul_pd(tmp3, tmp0)
    "mulpd     %%xmm1,  %%xmm6   \n\t"  // tmp6     = _mm_mul_pd(tmp6, tmp1)
    "                            \n\t"
    "                            \n\t"
    "addpd     %%xmm4,  %%xmm10  \n\t"  // ab_01_10 = _mm_add_pd(ab_01_10, tmp4)
    "addpd     %%xmm7,  %%xmm11  \n\t"  // ab_21_30 = _mm_add_pd(ab_21_30, tmp7)
    "movaps    %%xmm5,  %%xmm7   \n\t"  // tmp7     = tmp5
    "mulpd     %%xmm0,  %%xmm5   \n\t"  // tmp5     = _mm_mul_pd(tmp5, tmp0)
    "movaps -96(%%rax), %%xmm0   \n\t"  // tmp0     = _mm_load_pd(A+4)
    "mulpd     %%xmm1,  %%xmm7   \n\t"  // tmp7     = _mm_mul_pd(tmp7, tmp1)
    "movaps -80(%%rax), %%xmm1   \n\t"  // tmp1     = _mm_load_pd(A+6)
    "                            \n\t"
    "                            \n\t"
    "                            \n\t"
    "                            \n\t"  // 2. update
    "addpd     %%xmm3,  %%xmm12  \n\t"  // ab_02_13 = _mm_add_pd(ab_02_13, tmp3)
    "movaps -80(%%rbx), %%xmm3   \n\t"  // tmp3     = _mm_load_pd(B+6)
    "addpd     %%xmm6,  %%xmm13  \n\t"  // ab_22_33 = _mm_add_pd(ab_22_33, tmp6)
    "movaps    %%xmm2,  %%xmm6   \n\t"  // tmp6     = tmp2
    "pshufd $78,%%xmm2, %%xmm4   \n\t"  // tmp4     = _mm_shuffle_pd(tmp2, tmp2,
    "                            \n\t"  //                   _MM_SHUFFLE2(0, 1))
    "mulpd     %%xmm0,  %%xmm2   \n\t"  // tmp2     = _mm_mul_pd(tmp2, tmp0);
    "mulpd     %%xmm1,  %%xmm6   \n\t"  // tmp6     = _mm_mul_pd(tmp6, tmp1);
    "                            \n\t"
    "                            \n\t"
    "addpd     %%xmm5,  %%xmm14  \n\t"  // ab_03_12 = _mm_add_pd(ab_03_12, tmp5)
    "addpd     %%xmm7,  %%xmm15  \n\t"  // ab_23_32 = _mm_add_pd(ab_23_32, tmp7)
    "movaps    %%xmm4,  %%xmm7   \n\t"  // tmp7     = tmp4
    "mulpd     %%xmm0,  %%xmm4   \n\t"  // tmp4     = _mm_mul_pd(tmp4, tmp0)
    "mulpd     %%xmm1,  %%xmm7   \n\t"  // tmp7     = _mm_mul_pd(tmp7, tmp1)
    "                            \n\t"
    "                            \n\t"
    "addpd     %%xmm2,  %%xmm8   \n\t"  // ab_00_11 = _mm_add_pd(ab_00_11, tmp2)
    "movaps -64(%%rbx), %%xmm2   \n\t"  // tmp2     = _mm_load_pd(B+8)
    "addpd     %%xmm6,  %%xmm9   \n\t"  // ab_20_31 = _mm_add_pd(ab_20_31, tmp6)
    "movaps    %%xmm3,  %%xmm6   \n\t"  // tmp6     = tmp3
    "pshufd $78,%%xmm3, %%xmm5   \n\t"  // tmp5     = _mm_shuffle_pd(tmp3, tmp3,
    "                            \n\t"  //                   _MM_SHUFFLE2(0, 1))
    "mulpd     %%xmm0,  %%xmm3   \n\t"  // tmp3     = _mm_mul_pd(tmp3, tmp0)
    "mulpd     %%xmm1,  %%xmm6   \n\t"  // tmp6     = _mm_mul_pd(tmp6, tmp1)
    "                            \n\t"
    "                            \n\t"
    "addpd     %%xmm4,  %%xmm10  \n\t"  // ab_01_10 = _mm_add_pd(ab_01_10, tmp4)
    "addpd     %%xmm7,  %%xmm11  \n\t"  // ab_21_30 = _mm_add_pd(ab_21_30, tmp7)
    "movaps    %%xmm5,  %%xmm7   \n\t"  // tmp7     = tmp5
    "mulpd     %%xmm0,  %%xmm5   \n\t"  // tmp5     = _mm_mul_pd(tmp5, tmp0)
    "movaps -64(%%rax), %%xmm0   \n\t"  // tmp0     = _mm_load_pd(A+8)
    "mulpd     %%xmm1,  %%xmm7   \n\t"  // tmp7     = _mm_mul_pd(tmp7, tmp1)
    "movaps -48(%%rax), %%xmm1   \n\t"  // tmp1     = _mm_load_pd(A+10)
    "                            \n\t"
    "                            \n\t"
    "prefetcht0 (4*37+1)*8(%%rax)\n\t"
    "                            \n\t"
    "                            \n\t"  // 3. update
    "addpd     %%xmm3,  %%xmm12  \n\t"  // ab_02_13 = _mm_add_pd(ab_02_13, tmp3)
    "movaps -48(%%rbx), %%xmm3   \n\t"  // tmp3     = _mm_load_pd(B+10)
    "addpd     %%xmm6,  %%xmm13  \n\t"  // ab_22_33 = _mm_add_pd(ab_22_33, tmp6)
    "movaps    %%xmm2,  %%xmm6   \n\t"  // tmp6     = tmp2
    "pshufd $78,%%xmm2, %%xmm4   \n\t"  // tmp4     = _mm_shuffle_pd(tmp2, tmp2,
    "                            \n\t"  //                   _MM_SHUFFLE2(0, 1))
    "mulpd     %%xmm0,  %%xmm2   \n\t"  // tmp2     = _mm_mul_pd(tmp2, tmp0);
    "mulpd     %%xmm1,  %%xmm6   \n\t"  // tmp6     = _mm_mul_pd(tmp6, tmp1);
    "                            \n\t"
    "                            \n\t"
    "addpd     %%xmm5,  %%xmm14  \n\t"  // ab_03_12 = _mm_add_pd(ab_03_12, tmp5)
    "addpd     %%xmm7,  %%xmm15  \n\t"  // ab_23_32 = _mm_add_pd(ab_23_32, tmp7)
    "movaps    %%xmm4,  %%xmm7   \n\t"  // tmp7     = tmp4
    "mulpd     %%xmm0,  %%xmm4   \n\t"  // tmp4     = _mm_mul_pd(tmp4, tmp0)
    "mulpd     %%xmm1,  %%xmm7   \n\t"  // tmp7     = _mm_mul_pd(tmp7, tmp1)
    "                            \n\t"
    "                            \n\t"
    "addpd     %%xmm2,  %%xmm8   \n\t"  // ab_00_11 = _mm_add_pd(ab_00_11, tmp2)
    "movaps -32(%%rbx), %%xmm2   \n\t"  // tmp2     = _mm_load_pd(B+12)
    "addpd     %%xmm6,  %%xmm9   \n\t"  // ab_20_31 = _mm_add_pd(ab_20_31, tmp6)
    "movaps    %%xmm3,  %%xmm6   \n\t"  // tmp6     = tmp3
    "pshufd $78,%%xmm3, %%xmm5   \n\t"  // tmp5     = _mm_shuffle_pd(tmp3, tmp3,
    "                            \n\t"  //                   _MM_SHUFFLE2(0, 1))
    "mulpd     %%xmm0,  %%xmm3   \n\t"  // tmp3     = _mm_mul_pd(tmp3, tmp0)
    "mulpd     %%xmm1,  %%xmm6   \n\t"  // tmp6     = _mm_mul_pd(tmp6, tmp1)
    "                            \n\t"
    "                            \n\t"
    "addpd     %%xmm4,  %%xmm10  \n\t"  // ab_01_10 = _mm_add_pd(ab_01_10, tmp4)
    "addpd     %%xmm7,  %%xmm11  \n\t"  // ab_21_30 = _mm_add_pd(ab_21_30, tmp7)
    "movaps    %%xmm5,  %%xmm7   \n\t"  // tmp7     = tmp5
    "mulpd     %%xmm0,  %%xmm5   \n\t"  // tmp5     = _mm_mul_pd(tmp5, tmp0)
    "movaps -32(%%rax), %%xmm0   \n\t"  // tmp0     = _mm_load_pd(A+12)
    "mulpd     %%xmm1,  %%xmm7   \n\t"  // tmp7     = _mm_mul_pd(tmp7, tmp1)
    "movaps -16(%%rax), %%xmm1   \n\t"  // tmp1     = _mm_load_pd(A+14)
    "                            \n\t"
    "                            \n\t"
    "                            \n\t"  // 4. update
    "addpd     %%xmm3,  %%xmm12  \n\t"  // ab_02_13 = _mm_add_pd(ab_02_13, tmp3)
    "movaps -16(%%rbx), %%xmm3   \n\t"  // tmp3     = _mm_load_pd(B+14)
    "addpd     %%xmm6,  %%xmm13  \n\t"  // ab_22_33 = _mm_add_pd(ab_22_33, tmp6)
    "movaps    %%xmm2,  %%xmm6   \n\t"  // tmp6     = tmp2
    "pshufd $78,%%xmm2, %%xmm4   \n\t"  // tmp4     = _mm_shuffle_pd(tmp2, tmp2,
    "                            \n\t"  //                   _MM_SHUFFLE2(0, 1))
    "mulpd     %%xmm0,  %%xmm2   \n\t"  // tmp2     = _mm_mul_pd(tmp2, tmp0);
    "mulpd     %%xmm1,  %%xmm6   \n\t"  // tmp6     = _mm_mul_pd(tmp6, tmp1);
    "                            \n\t"
    "subq     $-32*4,   %%rax    \n\t"  // A += 16;
    "                            \n\t"
    "addpd     %%xmm5,  %%xmm14  \n\t"  // ab_03_12 = _mm_add_pd(ab_03_12, tmp5)
    "addpd     %%xmm7,  %%xmm15  \n\t"  // ab_23_32 = _mm_add_pd(ab_23_32, tmp7)
    "movaps    %%xmm4,  %%xmm7   \n\t"  // tmp7     = tmp4
    "mulpd     %%xmm0,  %%xmm4   \n\t"  // tmp4     = _mm_mul_pd(tmp4, tmp0)
    "mulpd     %%xmm1,  %%xmm7   \n\t"  // tmp7     = _mm_mul_pd(tmp7, tmp1)
    "                            \n\t"
    "subq     $-128,    %%r10    \n\t"  // nextB += 16
    "                            \n\t"
    "addpd     %%xmm2,  %%xmm8   \n\t"  // ab_00_11 = _mm_add_pd(ab_00_11, tmp2)
    "movaps    (%%rbx),%%xmm2   \n\t"  // tmp2     = _mm_load_pd(B+16)
    "addpd     %%xmm6,  %%xmm9   \n\t"  // ab_20_31 = _mm_add_pd(ab_20_31, tmp6)
    "movaps    %%xmm3,  %%xmm6   \n\t"  // tmp6     = tmp3
    "pshufd $78,%%xmm3, %%xmm5   \n\t"  // tmp5     = _mm_shuffle_pd(tmp3, tmp3,
    "                            \n\t"  //                   _MM_SHUFFLE2(0, 1))
    "mulpd     %%xmm0,  %%xmm3   \n\t"  // tmp3     = _mm_mul_pd(tmp3, tmp0)
    "mulpd     %%xmm1,  %%xmm6   \n\t"  // tmp6     = _mm_mul_pd(tmp6, tmp1)
    "                            \n\t"
    "subq     $-32*4,   %%rbx    \n\t"  // B += 16;
    "                            \n\t"
    "                            \n\t"
    "addpd     %%xmm4,  %%xmm10  \n\t"  // ab_01_10 = _mm_add_pd(ab_01_10, tmp4)
    "addpd     %%xmm7,  %%xmm11  \n\t"  // ab_21_30 = _mm_add_pd(ab_21_30, tmp7)
    "movaps    %%xmm5,  %%xmm7   \n\t"  // tmp7     = tmp5
    "mulpd     %%xmm0,  %%xmm5   \n\t"  // tmp5     = _mm_mul_pd(tmp5, tmp0)
    "movaps -128(%%rax),%%xmm0   \n\t"  // tmp0     = _mm_load_pd(A+16)
    "mulpd     %%xmm1,  %%xmm7   \n\t"  // tmp7     = _mm_mul_pd(tmp7, tmp1)
    "movaps -112(%%rax), %%xmm1   \n\t"  // tmp1     = _mm_load_pd(A+18)
    "                            \n\t"
    "prefetcht2        0(%%r10)  \n\t"  // prefetch nextB[0]
    "prefetcht2       64(%%r10)  \n\t"  // prefetch nextB[8]
    "                            \n\t"
    "decq      %%rsi             \n\t"  // --l
    "jne       .DLOOP%=          \n\t"  // if l>= 1 go back
    "                            \n\t"
    "                            \n\t"
    ".DCONSIDERLEFT%=:           \n\t"
    "testq     %%rdi,   %%rdi    \n\t"  // if kl==0 writeback to AB
    "je        .DPOSTACCUMULATE%=\n\t"
    "                            \n\t"
    ".DLOOPLEFT%=:               \n\t"  // for l = kl,..,1 do
    "                            \n\t"
    "addpd     %%xmm3,  %%xmm12  \n\t"  // ab_02_13 = _mm_add_pd(ab_02_13, tmp3)
    "movapd -112(%%rbx),%%xmm3   \n\t"  // tmp3     = _mm_load_pd(B+2)
    "addpd     %%xmm6,  %%xmm13  \n\t"  // ab_22_33 = _mm_add_pd(ab_22_33, tmp6)
    "movapd    %%xmm2,  %%xmm6   \n\t"  // tmp6     = tmp2
    "pshufd $78,%%xmm2, %%xmm4   \n\t"  // tmp4     = _mm_shuffle_pd(tmp2, tmp2,
    "                            \n\t"  //                   _MM_SHUFFLE2(0, 1))
    "mulpd     %%xmm0,  %%xmm2   \n\t"  // tmp2     = _mm_mul_pd(tmp2, tmp0);
    "mulpd     %%xmm1,  %%xmm6   \n\t"  // tmp6     = _mm_mul_pd(tmp6, tmp1);
    "                            \n\t"
    "                            \n\t"
    "addpd     %%xmm5,  %%xmm14  \n\t"  // ab_03_12 = _mm_add_pd(ab_03_12, tmp5)
    "addpd     %%xmm7,  %%xmm15  \n\t"  // ab_23_32 = _mm_add_pd(ab_23_32, tmp7)
    "movapd    %%xmm4,  %%xmm7   \n\t"  // tmp7     = tmp4
    "mulpd     %%xmm0,  %%xmm4   \n\t"  // tmp4     = _mm_mul_pd(tmp4, tmp0)
    "mulpd     %%xmm1,  %%xmm7   \n\t"  // tmp7     = _mm_mul_pd(tmp7, tmp1)
    "                            \n\t"
    "                            \n\t"
    "addpd     %%xmm2,  %%xmm8   \n\t"  // ab_00_11 = _mm_add_pd(ab_00_11, tmp2)
    "movapd -96(%%rbx), %%xmm2   \n\t"  // tmp2     = _mm_load_pd(B+4)
    "addpd     %%xmm6,  %%xmm9   \n\t"  // ab_20_31 = _mm_add_pd(ab_20_31, tmp6)
    "movapd    %%xmm3,  %%xmm6   \n\t"  // tmp6     = tmp3
    "pshufd $78,%%xmm3, %%xmm5   \n\t"  // tmp5     = _mm_shuffle_pd(tmp3, tmp3,
    "                            \n\t"  //                   _MM_SHUFFLE2(0, 1))
    "mulpd     %%xmm0,  %%xmm3   \n\t"  // tmp3     = _mm_mul_pd(tmp3, tmp0)
    "mulpd     %%xmm1,  %%xmm6   \n\t"  // tmp6     = _mm_mul_pd(tmp6, tmp1)
    "                            \n\t"
    "                            \n\t"
    "addpd     %%xmm4,  %%xmm10  \n\t"  // ab_01_10 = _mm_add_pd(ab_01_10, tmp4)
    "addpd     %%xmm7,  %%xmm11  \n\t"  // ab_21_30 = _mm_add_pd(ab_21_30, tmp7)
    "movapd    %%xmm5,  %%xmm7   \n\t"  // tmp7     = tmp5
    "mulpd     %%xmm0,  %%xmm5   \n\t"  // tmp5     = _mm_mul_pd(tmp5, tmp0)
    "movapd -96(%%rax), %%xmm0   \n\t"  // tmp0     = _mm_load_pd(A+4)
    "mulpd     %%xmm1,  %%xmm7   \n\t"  // tmp7     = _mm_mul_pd(tmp7, tmp1)
    "movapd -80(%%rax), %%xmm1   \n\t"  // tmp1     = _mm_load_pd(A+6)
    "                            \n\t"
    "                            \n\t"
    "addq      $32,     %%rax    \n\t"  // A += 4;
    "addq      $32,     %%rbx    \n\t"  // B += 4;
    "                            \n\t"
    "decq      %%rdi             \n\t"  // --l
    "jne       .DLOOPLEFT%=      \n\t"  // if l>= 1 go back
    "                            \n\t"
    ".DPOSTACCUMULATE%=:         \n\t"  // Update remaining ab_*_* registers
    "                            \n\t"
    "addpd    %%xmm3,   %%xmm12  \n\t"  // ab_02_13 = _mm_add_pd(ab_02_13, tmp3)
    "addpd    %%xmm6,   %%xmm13  \n\t"  // ab_22_33 = _mm_add_pd(ab_22_33, tmp6)
    "                            \n\t"  //
    "addpd    %%xmm5,   %%xmm14  \n\t"  // ab_03_12 = _mm_add_pd(ab_03_12, tmp5)
    "addpd    %%xmm7,   %%xmm15  \n\t"  // ab_23_32 = _mm_add_pd(ab_23_32, tmp7)
    "                            \n\t"
//
//  Update C <- beta*C + alpha*AB
//
//
    "movsd  %4,                  %%xmm0 \n\t"  // load alpha
    "movsd  %5,                  %%xmm1 \n\t"  // load beta 
    "movq   %6,                  %%rcx  \n\t"  // Address of C stored in %rcx

    "movq   %7,                  %%r8   \n\t"  // load incRowC
    "leaq   (,%%r8,8),           %%r8   \n\t"  //      incRowC *= sizeof(double)
    "movq   %8,                  %%r9   \n\t"  // load incColC
    "leaq   (,%%r9,8),           %%r9   \n\t"  //      incRowC *= sizeof(double)
    "                                   \n\t"
    "leaq (%%rcx,%%r9),          %%r10  \n\t"  // Store addr of C01 in %r10
    "leaq (%%rcx,%%r8,2),        %%rdx  \n\t"  // Store addr of C20 in %rdx
    "leaq (%%rdx,%%r9),          %%r11  \n\t"  // Store addr of C21 in %r11
    "                                   \n\t"
    "unpcklpd %%xmm0,            %%xmm0 \n\t"  // duplicate alpha
    "unpcklpd %%xmm1,            %%xmm1 \n\t"  // duplicate beta
    "                                   \n\t"
    "                                   \n\t"
    "movlpd (%%rcx),             %%xmm3 \n\t"  // load (C00,
    "movhpd (%%r10,%%r8),        %%xmm3 \n\t"  //       C11)
    "mulpd  %%xmm0,              %%xmm8 \n\t"  // scale ab_00_11 by alpha
    "mulpd  %%xmm1,              %%xmm3 \n\t"  // scale (C00, C11) by beta
    "addpd  %%xmm8,              %%xmm3 \n\t"  // add results

    "movlpd %%xmm3,        (%%rcx)       \n\t"  // write back (C00,
    "movhpd %%xmm3,        (%%r10,%%r8)  \n\t"  //             C11)
    "                                   \n\t"
    "movlpd (%%rdx),             %%xmm4 \n\t"  // load (C20,
    "movhpd (%%r11,%%r8),        %%xmm4 \n\t"  //       C31)
    "mulpd  %%xmm0,              %%xmm9 \n\t"  // scale ab_20_31 by alpha
    "mulpd  %%xmm1,              %%xmm4 \n\t"  // scale (C20, C31) by beta
    "addpd  %%xmm9,              %%xmm4 \n\t"  // add results
    "movlpd %%xmm4,        (%%rdx)       \n\t"  // write back (C20,
    "movhpd %%xmm4,        (%%r11,%%r8)  \n\t"  //             C31)
    "                                   \n\t"
    "                                   \n\t"
    "movlpd (%%r10),             %%xmm3 \n\t"  // load (C01,
    "movhpd (%%rcx,%%r8),        %%xmm3 \n\t"  //       C10)
    "mulpd  %%xmm0,              %%xmm10\n\t"  // scale ab_01_10 by alpha
    "mulpd  %%xmm1,              %%xmm3 \n\t"  // scale (C01, C10) by beta
    "addpd  %%xmm10,             %%xmm3 \n\t"  // add results
    "movlpd %%xmm3,        (%%r10)      \n\t"  // write back (C01,
    "movhpd %%xmm3,        (%%rcx,%%r8) \n\t"  //             C10)
    "                                   \n\t"
    "movlpd (%%r11),             %%xmm4 \n\t"  // load (C21,
    "movhpd (%%rdx,%%r8),        %%xmm4 \n\t"  //       C30)
    "mulpd  %%xmm0,              %%xmm11\n\t"  // scale ab_21_30 by alpha
    "mulpd  %%xmm1,              %%xmm4 \n\t"  // scale (C21, C30) by beta
    "addpd  %%xmm11,             %%xmm4 \n\t"  // add results
    "movlpd %%xmm4,        (%%r11)      \n\t"  // write back (C21,
    "movhpd %%xmm4,        (%%rdx,%%r8) \n\t"  //             C30)
    "                                   \n\t"
    "                                   \n\t"
    "leaq   (%%rcx,%%r9,2),      %%rcx  \n\t"  // Store addr of C02 in %rcx
    "leaq   (%%r10,%%r9,2),      %%r10  \n\t"  // Store addr of C03 in %r10
    "leaq   (%%rdx,%%r9,2),      %%rdx  \n\t"  // Store addr of C22 in $rdx
    "leaq   (%%r11,%%r9,2),      %%r11  \n\t"  // Store addr of C23 in %r11
    "                                   \n\t"
    "                                   \n\t"
    "movlpd (%%rcx),             %%xmm3 \n\t"  // load (C02,
    "movhpd (%%r10,%%r8),        %%xmm3 \n\t"  //       C13)
    "mulpd  %%xmm0,              %%xmm12\n\t"  // scale ab_02_13 by alpha
    "mulpd  %%xmm1,              %%xmm3 \n\t"  // scale (C02, C13) by beta
    "addpd  %%xmm12,             %%xmm3 \n\t"  // add results
    "movlpd %%xmm3,        (%%rcx)      \n\t"  // write back (C02,
    "movhpd %%xmm3,        (%%r10,%%r8) \n\t"  //             C13)
    "                                   \n\t"
    "movlpd (%%rdx),             %%xmm4 \n\t"  // load (C22,
    "movhpd (%%r11, %%r8),       %%xmm4 \n\t"  //       C33)
    "mulpd  %%xmm0,              %%xmm13\n\t"  // scale ab_22_33 by alpha
    "mulpd  %%xmm1,              %%xmm4 \n\t"  // scale (C22, C33) by beta
    "addpd  %%xmm13,             %%xmm4 \n\t"  // add results
    "movlpd %%xmm4,             (%%rdx) \n\t"  // write back (C22,
    "movhpd %%xmm4,        (%%r11,%%r8) \n\t"  //             C33)
    "                                   \n\t"
    "                                   \n\t"
    "movlpd (%%r10),             %%xmm3 \n\t"  // load (C03,
    "movhpd (%%rcx,%%r8),        %%xmm3 \n\t"  //       C12)
    "mulpd  %%xmm0,              %%xmm14\n\t"  // scale ab_03_12 by alpha
    "mulpd  %%xmm1,              %%xmm3 \n\t"  // scale (C03, C12) by beta
    "addpd  %%xmm14,             %%xmm3 \n\t"  // add results
    "movlpd %%xmm3,        (%%r10)      \n\t"  // write back (C03,
    "movhpd %%xmm3,        (%%rcx,%%r8) \n\t"  //             C12)
    "                                   \n\t"
    "movlpd (%%r11),             %%xmm4 \n\t"  // load (C23,
    "movhpd (%%rdx,%%r8),        %%xmm4 \n\t"  //       C32)
    "mulpd  %%xmm0,              %%xmm15\n\t"  // scale ab_23_32 by alpha
    "mulpd  %%xmm1,              %%xmm4 \n\t"  // scale (C23, C32) by beta
    "addpd  %%xmm15,             %%xmm4 \n\t"  // add results
    "movlpd %%xmm4,        (%%r11)      \n\t"  // write back (C23,
    "movhpd %%xmm4,        (%%rdx,%%r8) \n\t"  //             C32)
    : // output
    : // input
        "m" (kb),       // 0
        "m" (kl),       // 1
        "m" (A),        // 2
        "m" (B),        // 3
        "m" (alpha),    // 4
        "m" (beta),     // 5
        "m" (C),        // 6
        "m" (incRowC),  // 7
        "m" (incColC),  // 8
        "m" (nextA),    // 9
        "m" (nextB)     // 10
    : // register clobber list
        "rax", "rbx", "rcx", "rdx", "rsi", "rdi", "r8", "r9", "r10", "r11",
        "xmm0", "xmm1", "xmm2", "xmm3",
        "xmm4", "xmm5", "xmm6", "xmm7",
        "xmm8", "xmm9", "xmm10", "xmm11",
        "xmm12", "xmm13", "xmm14", "xmm15"
    );
}

//
//  Compute Y += alpha*X
//
static void
dgeaxpy(int           m,
        int           n,
        double        alpha,
        const double  *X,
        int           incRowX,
        int           incColX,
        double        *Y,
        int           incRowY,
        int           incColY)
{
    int i, j;


    if (alpha!=1.0) {
        for (j=0; j<n; ++j) {
            for (i=0; i<m; ++i) {
                Y[i*incRowY+j*incColY] += alpha*X[i*incRowX+j*incColX];
            }
        }
    } else {
        for (j=0; j<n; ++j) {
            for (i=0; i<m; ++i) {
                Y[i*incRowY+j*incColY] += X[i*incRowX+j*incColX];
            }
        }
    }
}

//
//  Compute X *= alpha
//
static void
dgescal(int     m,
        int     n,
        double  alpha,
        double  *X,
        int     incRowX,
        int     incColX)
{
    int i, j;

    if (alpha!=0.0) {
        for (j=0; j<n; ++j) {
            for (i=0; i<m; ++i) {
                X[i*incRowX+j*incColX] *= alpha;
            }
        }
    } else {
        for (j=0; j<n; ++j) {
            for (i=0; i<m; ++i) {
                X[i*incRowX+j*incColX] = 0.0;
            }
        }
    }
}

//
//  Macro Kernel for the multiplication of blocks of A and B.  We assume that
//  these blocks were previously packed to buffers _A and _B.
//
static void
dgemm_macro_kernel(int     mc,
                   int     nc,
                   int     kc,
                   double  alpha,
                   double  beta,
                   double  *C,
                   int     incRowC,
                   int     incColC)
{
    int mp = (mc+MR-1) / MR;
    int np = (nc+NR-1) / NR;

    int _mr = mc % MR;
    int _nr = nc % NR;

    int mr, nr;
    int i, j;

    const double *nextA;
    const double *nextB;

    for (j=0; j<np; ++j) {
        nr    = (j!=np-1 || _nr==0) ? NR : _nr;
        nextB = &_B[j*kc*NR];

        for (i=0; i<mp; ++i) {
            mr    = (i!=mp-1 || _mr==0) ? MR : _mr;
            nextA = &_A[(i+1)*kc*MR];

            if (i==mp-1) {
                nextA = _A;
                nextB = &_B[(j+1)*kc*NR];
                if (j==np-1) {
                    nextB = _B;
                }
            }

            if (mr==MR && nr==NR) {
                dgemm_micro_kernel(kc, alpha, &_A[i*kc*MR], &_B[j*kc*NR],
                                   beta,
                                   &C[i*MR*incRowC+j*NR*incColC],
                                   incRowC, incColC,
                                   nextA, nextB);
            } else {
                dgemm_micro_kernel(kc, alpha, &_A[i*kc*MR], &_B[j*kc*NR],
                                   0.0,
                                   _C, 1, MR,
                                   nextA, nextB);
                dgescal(mr, nr, beta,
                        &C[i*MR*incRowC+j*NR*incColC], incRowC, incColC);
                dgeaxpy(mr, nr, 1.0, _C, 1, MR,
                        &C[i*MR*incRowC+j*NR*incColC], incRowC, incColC);
            }
        }
    }
}

//
//  Compute C <- beta*C + alpha*A*B
//
void
ULMBLAS(dgemm_nn)(int            m,
                  int            n,
                  int            k,
                  double         alpha,
                  const double   *A,
                  int            incRowA,
                  int            incColA,
                  const double   *B,
                  int            incRowB,
                  int            incColB,
                  double         beta,
                  double         *C,
                  int            incRowC,
                  int            incColC)
{
    int mb = (m+MC-1) / MC;
    int nb = (n+NC-1) / NC;
    int kb = (k+KC-1) / KC;

    int _mc = m % MC;
    int _nc = n % NC;
    int _kc = k % KC;

    int mc, nc, kc;
    int i, j, l;

    double _beta;

    if (alpha==0.0 || k==0) {
        dgescal(m, n, beta, C, incRowC, incColC);
        return;
    }

    for (j=0; j<nb; ++j) {
        nc = (j!=nb-1 || _nc==0) ? NC : _nc;

        for (l=0; l<kb; ++l) {
            kc    = (l!=kb-1 || _kc==0) ? KC   : _kc;
            _beta = (l==0) ? beta : 1.0;

            pack_B(kc, nc,
                   &B[l*KC*incRowB+j*NC*incColB], incRowB, incColB,
                   _B);

            for (i=0; i<mb; ++i) {
                mc = (i!=mb-1 || _mc==0) ? MC : _mc;

                pack_A(mc, kc,
                       &A[i*MC*incRowA+l*KC*incColA], incRowA, incColA,
                       _A);

                dgemm_macro_kernel(mc, nc, kc, alpha, _beta,
                                   &C[i*MC*incRowC+j*NC*incColC],
                                   incRowC, incColC);
            }
        }
    }
}
```

</details>

The relevant change from the previous version is very small.

In `demo-sse-asm-unrolled-v2`, the two pointer increments occurred directly after each other. In this version they are separated by independent floating-point instructions.

In particular, the update of the pointer to the packed panel of $A$

```asm
addq    $32*4, %rax    # A += 16
```

is performed earlier. Some independent arithmetic follows before the pointer to the packed panel of $B$ is updated:

```asm
movapd 128(%rbx), %xmm2
addpd      %xmm6, %xmm9
movapd     %xmm3, %xmm6
pshufd $78, %xmm3, %xmm5
mulpd      %xmm0, %xmm3
mulpd      %xmm1, %xmm6

addq    $32*4, %rbx    # B += 16
```

The algorithm, register blocking, unrolling factor, and floating-point operations remain unchanged. Only the scheduling of the two integer pointer updates has changed.

## Benchmark Results

We run the benchmarks:

```console
$ cd bench
$ ./xdl3blastst > report
```

and filter out the results for the `demo-sse-asm-unrolled-v3` branch:

```console
$ grep PASS report > demo-sse-asm-unrolled-v3
```

With the gnuplot script

```gnuplot
set terminal svg size 1140,480
set output "bench12.svg"
set title "Compute C + A*B"
set xlabel "Matrix dimensions N=M=K"
set ylabel "MFLOPS"
set yrange [0:9600]
set key outside

plot "refBLAS" using 4:13 with linespoints lt 2 \
         title "Netlib RefBLAS", \
     "demo-pure-c" using 4:13 with linespoints lt 4 \
         title "demo-pure-c", \
     "demo-naive-sse-with-intrinsics" using 4:13 with linespoints lt 5 \
         title "demo-naive-sse-with-intrinsics", \
     "demo-naive-sse-with-intrinsics-unrolled" using 4:13 with linespoints lt 6 \
         title "demo-naive-sse-with-intrinsics-unrolled", \
     "demo-sse-intrinsics" using 4:13 with linespoints lt 7 \
         title "demo-sse-intrinsics", \
     "demo-sse-intrinsics-v2" using 4:13 with linespoints lt 8 \
         title "demo-sse-intrinsics-v2", \
     "demo-sse-asm" using 4:13 with linespoints lt 9 \
         title "demo-sse-asm", \
     "demo-sse-asm-unrolled" using 4:13 with linespoints lt 10 \
         title "demo-sse-asm-unrolled", \
     "demo-sse-asm-unrolled-v2" using 4:13 with linespoints lt 11 \
         title "demo-sse-asm-unrolled-v2", \
     "demo-sse-asm-unrolled-v3" using 4:13 with linespoints lt 12 \
         title "demo-sse-asm-unrolled-v3"
```

we feed gnuplot:

```console
$ gnuplot bench12.gps
```

and get:

![Benchmark results](bench12.svg)

---

[← Previous](../page10/README.md) | [Main Page](../README.md) | [Next →](../page12/README.md)

Copyright © 2014 Michael Lehn

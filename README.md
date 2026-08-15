# GEMM: From Pure C to SSE Optimized Micro Kernels

This repository is a GitHub port of a tutorial I originally wrote in 2014:

**GEMM: From Pure C to SSE Optimized Micro Kernels**

The original version was published on the Ulm University website:

https://www.mathematik.uni-ulm.de/~lehn/sghpc/gemm/

The tutorial explores, step by step, how a simple implementation of matrix-matrix multiplication can be transformed into a high-performance implementation. For this journey, we set up our own BLAS implementation, [ulmBLAS](https://github.com/michael-lehn/ulmBLAS), following ideas from the BLIS framework.

The original tutorial uses SSE3, which was widely available on x86 processors at the time. Today, one would naturally use newer SIMD instruction sets such as AVX or AVX2, or target other architectures altogether. Nevertheless, the main ideas behind the tutorial remain relevant: memory access patterns, cache-aware blocking, register blocking, vectorization, instruction scheduling, and the design of small computational micro-kernels.

For now, this repository preserves the original 2014 tutorial as closely as possible. At some point, I may add updated versions of the examples using more recent instruction sets.

## Contents

- [Page 1](page01/) — How to obtain the ulmBLAS project.
- [Page 2](page02/) — Pure C implementation.
- [Page 3](page03/) — Naive use of SSE intrinsics.
- [Page 4](page04/) — Applying loop unrolling to the previous implementation.
- [Page 5](page05/) — Another SSE intrinsics approach based on the BLIS micro-kernel for SSE architectures.
- [Page 6](page06/) — Improving pipelining by reordering SSE intrinsics.
- [Page 7](page07/) — Limitations of SSE intrinsics.
- [Page 8](page08/) — We go nuclear and translate the intrinsics to assembler ourselves!
- [Page 9](page09/) — Unrolling the nuke: `demo-asm-unrolled`.
- [Page 10](page10/) — Fine-tuning the unrolled assembler kernel.
- [Page 11](page11/) — More fine-tuning of the unrolled assembler kernel.
- [Page 12](page12/) — Preparation for adding prefetching: porting the rest of the micro-kernel to assembler.
- [Page 13](page13/) — Adding prefetching.
- [Page 14](page14/) — Benchmarking: comparing the performance with MKL, ATLAS, Eigen, and the original BLIS micro-kernel.

## Historical Benchmark Platform

All benchmarks shown in the original tutorial were generated on my MacBook Pro with a **2.4 GHz Intel Core 2 Duo P8600 ("Penryn")**.

The theoretical peak performance of one core is **9.6 GFLOPS**.

The benchmark results should therefore be understood in their original 2014 hardware context. The purpose of the tutorial is not the absolute performance numbers, but the process of understanding where performance comes from and how successive transformations of the implementation affect it.

## Related Projects

- [ulmBLAS](https://github.com/michael-lehn/ulmBLAS)
- [BLIS](https://github.com/flame/blis)

---

Copyright © 2014 Michael Lehn
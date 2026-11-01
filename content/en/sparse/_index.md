---
title: "sparse"
weight: 10
description: "Sparse matrices, fill-reducing orderings and iterative solvers, released v0.2.0."
---

<!-- Summary of go-sparse/sparse v0.2.0, README.md and docs/perf.md. Change the library's text first. -->

**Status: released v0.2.0.**

Sparse matrices, fill-reducing orderings and iterative solvers in pure Go
(`CGO_ENABLED=0`), module `github.com/go-sparse/sparse`, BSD-3-Clause.

```sh
go get github.com/go-sparse/sparse@v0.2.0
```

```go
a := gallery.Laplacian2D(100, 100)           // *sparse.CSR[float64]
b := make([]float64, a.Rows)
ic, _ := iter.NewIC0(a)                      // incomplete Cholesky
res, _ := iter.CG(a, b, &iter.Options{RTol: 1e-8, M: ic})
fmt.Println(res.Info, res.Iterations)        // 0 = converged, as in SciPy
```

## What is provided

| package | provides |
| --- | --- |
| `sparse` | `COO`, `CSR`, `CSC`, `DOK` over `float64` and `complex128`; conversions; O(1) transpose; SpMV, SpMM, SpGEMM; `Add`, `Sub`, `Axpby`; scaling; `Diagonal`, `Tril`, `Triu`, `Permute`, `Prune`, norms; `Eye`, `Diags`; `LinearOperator` |
| `nd` | `*ndarray.Array` in and out: vectors, dense matrices, SpMV and SpMM |
| `mtx` | Matrix Market read and write: coordinate and array, real, integer, pattern and complex, general, symmetric, skew-symmetric and hermitian, gzip input |
| `order` | `RCM`, `AMD`, `COLAMD`, and `AMDWith`/`COLAMDWith` on a reusable `Workspace` (no allocation once it is sized); `Bandwidth`, `Profile`, `CholeskyNNZ`, `QRFillNNZ` |
| `ldl` | sparse LDLᵀ of quasi-definite symmetric matrices given as an upper-triangle CSC, a port of [QDLDL](https://github.com/osqp/qdldl) v0.1.9: `Analyze` (elimination tree, column counts), `Factor.Refactor` (numeric phase, redone with new values on the same pattern, without allocation; an exact zero or non-finite pivot is an error), `Solve`, `LSolve`, `LTSolve`, `DSolve`, `Permuted` (with a fill-reducing ordering such as `order.AMD`; `Reset` redoes the symbolic phase for the next problem on the same storage, without allocation) |
| `iter` | `CG`, `GMRES(m)`, `BiCGSTAB`, `MINRES`, `LSQR`, `LSMR`; preconditioners `Jacobi`, `SSOR`, `IC0`, `ILU0`, `ILUT` |
| `gallery` | Laplacians (1-D, 2-D, 3-D), convection-diffusion, seeded random sparse matrices |

Solvers follow `scipy.sparse.linalg`: the stopping rule is
`‖b − A x‖ ≤ max(rtol·‖b‖, atol)`, `rtol` defaults to 1e-5, `maxiter` to 10n
(5n for MINRES), and `info` is 0 on success, the iteration limit when it was
reached, and negative on breakdown.

Not provided yet: sparse LU, Cholesky and QR factorisations (`ldl` covers
quasi-definite LDLᵀ only: no pivoting), nested dissection, eigensolvers,
algebraic multigrid, Rutherford-Boeing I/O.

`(*CSC).MulVecSymUpper(dst, x)` multiplies by a symmetric matrix stored as its
upper triangle only, the layout OSQP uses for its cost matrix.

## LDLᵀ: provenance, tests and speed

`ldl` is a port of QDLDL v0.1.9 (Apache-2.0, attribution in the library's
`NOTICE`), written from its source; Davis's LDL (LGPL) was not consulted. It is
tested against QDLDL's own nine test cases, against a dense unpivoted LDLᵀ (the
values and signs of D) and a dense LU solve from go-ndarray on random
quasi-definite KKT matrices, and fuzzed: malformed or non-quasi-definite input
returns an error and never panics.

Against the C QDLDL (`-O3`) on one core, 9 interleaved rounds, measured on
2026-11-01 at load 2.3 (Apple M4 Max; Go time / C time; KKT matrices of a grid
Laplacian plus coupling constraints; for the AMD rows `order.AMD` is applied
once and both sides factor the permuted pattern):

| order, nnz(K), nnz(L) | ordering | numeric factor | solve |
|---|---|---:|---:|
| 72, 204, 1516 | natural | 1.06 | 0.96 |
| 288, 840, 22474 | natural | 0.99 | 1.01 |
| 800, 2360, 168018 | natural | 1.00 | 1.01 |
| 800, 2360, 4885 | AMD | 1.21 | 1.13 |
| 3200, 9520, 28309 | AMD | 1.13 | 0.96 |
| 9800, 29260, 102299 | AMD | 1.11 | 1.10 |

With long columns of L (natural order) Go is on par with C. With the short
columns AMD produces (6 to 10 entries each) the numeric factorisation is 11 to
21% slower and the solve up to 13% slower, which is an open defect. v0.2.1
split the row loop of the numeric phase in two (pattern, then elimination),
which removed 3 to 6% of it; the rest is bounds checks on data-dependent
indices and per-row overhead that C does not pay. The profile, the assembly
comparison and what was not tried are in `docs/perf.md` of the repository.

The table is v0.2.1's. v0.3.0 (issue #10) fused `Solve` into two sweeps (D
inside the L sweep, the permutation inside the last sweep; 2 to 4% faster,
bit-identical), tried an interleaved layout of L (index and value in one array
of structs) and did not keep it because it was never faster, and made the
symbolic side of a solver setup cheaper: AMD followed by `NewPermuted` on the
same KKT grids takes about half the time of v0.2.1 with fresh storage (order 72:
28.5 µs and 652 allocations, now 15.4 µs and 79) and, on a reused
`order.Workspace` with `Permuted.Reset`, 9.8 µs and no allocation. The ratios
against QDLDL are re-measured in `docs/perf.md` when the machine is quiet.

## Orderings: provenance

`AMD` and `COLAMD` are written from the papers (Amestoy, Davis and Duff, 1996;
Davis, Gilbert, Larimore and Ng, 2004). SuiteSparse AMD and COLAMD are the
reference algorithms; CHOLMOD, UMFPACK and CSparse (GPL/LGPL) were not
consulted. Tie-breaking differs, so permutations differ; quality is compared
by fill.

## Verification

Every solver, LSQR and LSMR and the orderings are compared with SciPy 1.18.1
and NumPy 2.5.4 (and SuiteSparse AMD through cvxopt as a binary oracle). A Go
harness drives SciPy over a pipe and records the answers in a committed
fixture; the regression tests replay it, so they need neither Python nor
network. The full method is the library's
[docs/verification.md](https://github.com/go-sparse/sparse/blob/main/docs/verification.md).

On the committed suite: CG, GMRES, MINRES, LSQR and LSMR iteration counts
equal SciPy's; BiCGSTAB agrees within a few iterations. AMD Cholesky fill is
within 1% of SuiteSparse AMD; COLAMD LU fill is within 4% of SuperLU's COLAMD;
RCM bandwidth equals SciPy's.

## Performance

One core, Go 1.27.2 on darwin/arm64, SciPy 1.18.1, medians of 7 interleaved
rounds. Ratio is Go time / SciPy time; below 1 means Go is faster. From the
library's
[docs/perf.md](https://github.com/go-sparse/sparse/blob/main/docs/perf.md),
measured on 2026-10-20 at load 5.

| work | SpMV | CG | GMRES(30) |
| --- | ---: | ---: | ---: |
| 2-D Laplacian, 1.1e5 nonzeros | 0.97 | 0.70 | 0.56 |
| 2-D Laplacian, 1.0e6 nonzeros | 0.98 | 0.89 | 0.82 |

Column indices are `int32`, as in `scipy.sparse`; dot products and vector
updates use the fleet's SIMD kernels, and the Krylov solvers fuse each vector
update with the dot product that follows it.

## Untrusted input

`mtx.ReadLimited` takes limits on dimensions, entries, line length,
decompressed size and comment lines; declared sizes are checked before
allocation, and a gzip bomb is cut off at the byte limit. `mtx.FuzzRead` has a
committed corpus.

## Quality gates

`gofmt`, `go vet`, `go test -race` with 100% statement coverage over all
packages, 386 and the other architectures in CI, and `govulncheck`.

---
title: "go-sparse documentation"
linkTitle: "Home"
type: docs
cascade:
  type: docs
description: "Sparse matrices, fill-reducing orderings and iterative solvers in pure Go (CGO=0), following scipy.sparse.linalg."
---

**go-sparse is sparse linear algebra in pure Go.** One library provides the
sparse matrix formats, Matrix Market I/O, fill-reducing orderings and iterative
solvers with preconditioners, following `scipy.sparse.linalg`, and tests them
against recorded SciPy 1.18.1 answers.

## The libraries

| Library | Version | What it does |
| --- | --- | --- |
| [sparse]({{< relref "sparse" >}}) | released v0.2.1 | COO, CSR, CSC and DOK matrices over float64 and complex128, Matrix Market I/O, RCM, AMD and COLAMD orderings, a QDLDL-based sparse LDLᵀ for quasi-definite matrices, CG, GMRES, BiCGSTAB, MINRES, LSQR and LSMR with preconditioners, a matrix gallery |

## Install

```sh
go get github.com/go-sparse/sparse@v0.2.0
```

The module is pure Go (`CGO_ENABLED=0`) and its CI runs on every architecture
the fleet tests.

## Not provided yet

Sparse direct factorisations (LU, Cholesky, QR), nested dissection,
eigensolvers, algebraic multigrid and Rutherford-Boeing I/O.

## Built on

[go-ndarray](https://github.com/go-ndarray/ndarray): the `nd` package takes
and returns `*ndarray.Array` for vectors, dense matrices, SpMV and SpMM.

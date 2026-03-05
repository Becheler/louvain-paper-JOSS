---
title: 'A Generic Louvain Community Detection Algorithm for the Boost Graph Library'
tags:
  - C++
  - Boost Graph Library
  - community detection
  - Louvain algorithm
  - modularity
authors:
  - name: Arnaud Becheler
    orcid: 0000-0002-5729-9897
    affiliation: 1
affiliations:
  - name: Independent Researcher
    index: 1
date: 5 March 2026
bibliography: paper.bib
---

# Summary

We present a header-only C++14 implementation of the Louvain community detection algorithm for the Boost Graph Library (BGL).

The Louvain algorithm [@Blondel2008] finds a partition of graph vertices into communities that maximizes a quality function (*e.g*., Newman-Girvan modularity [@Newman2004]). The algorithm repeatedly alternates between a local optimization phase that moves each vertex to the neighboring community for best quality gain, and an aggregation phase that collapses each community into a single super-vertex. 

# Statement of Need

The Boost Graph Library is widely used but lacks community detection methods such as Louvain [@Blondel2008], Leiden [@Traag2019], or the Stochastic Block Model [@Holland1983]. Existing Louvain implementations like igraph (written in C) [@Csardi2006], NetworkX (Python) [@Hagberg2008], or gen-louvain (C++) [@Campigotto2013software] hardcode their graph data structures into the algorithm, requiring users to convert their data before use. This calls for a generic implementation built on the abstract data access patterns that BGL graph concepts already provide.

# Key Design Decisions

The implementation is templated on graph type, quality function with support for incremental optimization and termination conditions.

We templated on quality function, as its choice is an important driver of runtime cost. A function exposing only a basic evaluation interface requires a full O(V+E) quality recomputation for every candidate vertex move. A function that additionally exposes an incremental interface [@Campigotto2014] can evaluate each candidate move in O(1) through local bookkeeping, reducing the total per-vertex cost to O(degree). The algorithm detects which interface is present at compile time and selects the faster code path automatically, at no runtime cost. The practical consequence for users is straightforward: a non-incremental quality function is always sufficient to obtain correct results, while an incremental one is a drop-in optimization whenever the user can provide it.

We templated on local optimization and aggregation termination predicates as different application domains may require different stopping conditions, including fixed gain thresholds [@Campigotto2014], threshold scaling [@halappanavar2017scalable], decisions learned from gain decay patterns, or number of vertices moved. Exposing the predicate as a template parameter lets users substitute any criterion at compile time without modifying the algorithm.

# Benchmarks

Our comparative benchmarks serve three purposes. They validate genericity by showing that different BGL data structures can be injected into the algorithm, with different performance profiles but similar output quality. They confirm correctness by showing that the communities we detect are consistent with established implementations. Finally, they demonstrate that high C++ genericity carries little to no overhead in practice, with runtime matching or outperforming competitors.

[](communities.png)
[](speedup.png)

# Availability and Quality

The algorithm is integrated into the Boost Graph Library under the Boost Software License, with full CI and unit test coverage. The code has been documented and reviewed during the pull request process. Because the algorithm is a heuristic, a dedicated benchmarking repository has been set up with a correctness and runtime comparison suite run in CI. The implementation has been validated on multiple standard graphs, both synthetic and real-world, across several BGL graph types, and compared against competing implementations.

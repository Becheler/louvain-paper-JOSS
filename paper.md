# Summary

We present a header-only C++14 implementation of the Louvain community detection algorithm for the Boost Graph Library. The implementation is templated on graph type, quality function, and termination conditions, with support for incremental modularity optimization.

The Louvain algorithm [@Blondel2008] finds a partition of graph vertices into communities that approximately maximizes a quality function (by default, Newman-Girvan modularity [@Newman2004]). The algorithm alternates two phases: (i) local optimization, where each vertex is moved to the neighboring community yielding the largest improvement in quality, and (ii) aggregation, where the graph is contracted by collapsing each community into a single super-vertex. These two phases are applied repeatedly on the coarsened graph, discovering communities of communities. Once every level has converged, the algorithm traces community assignments from the coarsest level back to the original graph, writing the final community labels into the output `components` map.

# Statement of Need

The Boost Graph Library is widely used but lacks community detection methods such as Louvain [@Blondel2008], Leiden [@Traag2019], or the Stochastic Block Model [@Holland1983]. Existing C++ Louvain implementations — gen-louvain [@Jutla2011], igraph [@Csardi2006], NetworkX [@Hagberg2008] — are not BGL-compatible: they hardcode their graph data structure into the algorithm, requiring users to convert their data before use. A user who already holds graph data in a BGL structure should not need to serialize it into a foreign format simply to run community detection. This calls for a generic implementation built on the abstract data access patterns that BGL graph concepts already provide.

# Key Design Decisions

The algorithm is templated on both the graph representation type and the quality function. Newman-Girvan modularity is the default, but other quality measures are legitimate research targets (see XXX), and the generic interface accommodates them without modification to the algorithm itself.

The choice of quality function also determines runtime cost. A function exposing only a basic evaluation interface requires a full O(V+E) quality recomputation for every candidate vertex move. A function that additionally exposes an incremental interface (see XXX) can evaluate each candidate move in O(1) through local bookkeeping, reducing the total per-vertex cost to O(degree). The algorithm detects which interface is present at compile time and selects the faster code path automatically, at no runtime cost. This differs from the approach taken in (XXX), which uses inheritance and virtual dispatch to achieve the same polymorphism at runtime. The practical consequence for users is straightforward: a non-incremental quality function is always sufficient to obtain correct results, while an incremental one is a drop-in optimization whenever the user can provide it.

We also templated the local optimization and aggregation termination predicates. Fixed numerical thresholds are an arbitrary design choice, and different application domains may require different stopping conditions — including ones learned from gain decay patterns. Exposing the predicate as a template parameter lets users substitute any criterion at compile time without modifying the algorithm.

# Benchmarks

# Availability and Quality

The algorithm is integrated into the Boost Graph Library under the Boost Software License, with full CI and unit test coverage. The code has been documented and reviewed during the pull request process. Because the algorithm is a heuristic, a dedicated benchmarking repository has been set up with a correctness and runtime comparison suite run in CI. The implementation has been validated on multiple standard graphs — both synthetic and real-world — across several BGL graph types, and compared against competing implementations.

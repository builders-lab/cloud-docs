# Engineering Roadmap

This document outlines the strategic development phases for **Trailblazer**. The project follows an iterative approach, moving from a monolithic build prototype to a distributed, DAG-based execution engine.

## Phase 1: The Monolith (MVP)
**Objective:** Establish the baseline CI loop.

- [ ] **VCS Polling:** Implement a watcher service to detect changes in the target GitHub repository.
- [ ] **Sandboxed Build:** On change detection, pull source code into a single, isolated Docker container.
- [ ] **Artifact Simulation:** execute the build command and mock the artifact upload process (logging success/failure only).
- [ ] **Outcome:** A working "Input -> Process -> Output" loop with zero caching or parallelism.

## Phase 2: DAG-Based Execution Engine (Core)
**Objective:** Transition to a graph-based build system for parallelism.

This is the core architectural shift. We move from executing a script to executing a **Directed Acyclic Graph (DAG)**.

- [ ] **Manifest Parsing:** Implement a parser for `BUILD` files to identify build targets and dependencies.
- [ ] **Graph Generation:** Algorithmically plot the dependency graph from the parsed targets.
- [ ] **Container orchestration:**
    - Provide a `Dockerfile` defining the build environment (compilers, linkers).
    - Spin up **separate containers** for independent graph nodes.
- [ ] **Aggregation:** A final "Linker" container collects outputs from upstream nodes and produces the final binary.
- [ ] **Constraint:** No cross-dependencies between parallel threads in this phase to reduce complexity.

## Phase 3: Optimization & Caching
**Objective:** Reduce build times through artifact reuse.

- [ ] **Fingerprinting:** Implement hashing for source files (content-addressable storage).
- [ ] **Object Caching:** Store compiled `.o` files mapped to their source hash.
- [ ] **Cache Invalidation:** Implement Time-To-Live (TTL) or LRU policies for the build cache.

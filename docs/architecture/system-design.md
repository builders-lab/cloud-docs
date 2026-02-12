System Architecture
Overview

Trailblazer is a distributed build orchestration system designed to parallelize compilation workflows through containerized execution. Unlike traditional CI pipelines that execute linear scripts, Trailblazer constructs a dependency-aware execution graph and schedules independent tasks concurrently.

The system emphasizes deterministic builds, horizontal scalability, and efficient resource utilization through ephemeral execution environments.

Execution Flow (Phase 2)

Trailblazer follows a Split–Merge execution model composed of four primary components.

1. Trailblazer Controller (Orchestration Layer)

The Controller acts as the central coordination unit responsible for managing the lifecycle of a build pipeline.

Responsibilities

Ingestion
Receives repository change events through webhooks or polling mechanisms and initializes build pipelines.

DAG Generation
Parses the project’s BUILD configuration to construct a Directed Acyclic Graph (DAG) representing task dependencies.

Scheduling
Continuously evaluates the DAG to identify executable nodes (tasks with no unresolved dependencies) and dispatches them to the execution layer.

2. Ephemeral Workers (Execution Layer)

Workers are stateless, short-lived Docker containers created dynamically for each node in the dependency graph.

Key Characteristics

Isolation
Each worker executes within a clean container environment defined by the project’s Dockerfile, ensuring reproducible builds.

Parallel Execution
Independent compilation targets run concurrently. For example, utils.c and network.c may compile simultaneously if no dependency exists between them.

Lifecycle Management
Containers terminate immediately after task completion, freeing compute and memory resources.

3. Shared Volume / Artifact Store

A temporary storage layer mounted across all worker containers.

Purpose

Serves as the data exchange mechanism between isolated execution units.

Stores intermediate artifacts such as object files (.o) produced during compilation.

Workflow

Workers write intermediate outputs to the shared volume.

Downstream nodes consume these artifacts as inputs for subsequent stages.

4. Linker (Aggregation Node)

The Linker represents the terminal node within the dependency graph.

Behavior

Trigger Condition
Executes only after all upstream dependencies complete successfully.

Output Stage
Aggregates object files into a final executable and publishes the result to the long-term artifact repository.

Design Principles
Hermetic Execution

Every build step runs within a freshly instantiated container environment.
This eliminates configuration drift and ensures that builds remain deterministic across machines and environments.

DAG-Driven Parallelism

Traditional CI systems execute tasks sequentially:

Step A → Step B → Step C


Trailblazer analyzes dependency relationships to maximize concurrency:

Independent nodes execute immediately.

Dependent nodes wait only for required upstream tasks.

As a result, total build time approaches:

Max(Critical Path Duration)


instead of:

Sum(All Step Durations)

Future Roadmap (Phase 3)
Content-Addressable Caching

Input hashing (source code, flags, environment metadata) will allow Trailblazer to skip execution for unchanged nodes, reducing redundant work.

Distributed Worker Clusters

Execution will extend beyond a single host, enabling horizontal scaling across multiple physical nodes in cluster mode.

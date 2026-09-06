## Domain Lifecycle API

The framework defines domain semantics but defers the specification of the domain lifecycle management API. Any public interface for creating, loading, or recycling domain instances requires unambiguous identification. However, raw string-based identifiers are vulnerable to build-pipeline transformations (e.g., symbol renaming, string encryption, and aggressive code stripping).

> **Resolution Vector:** Establish an invariant, obfuscation-resistant identifier mapping system paired with an explicit asynchronous lifecycle API, decoupling developer-facing identifiers from internal domain runtime metadata.

## Multiplayer State Synchronization

Preserving deterministic execution across multiplayer clients requires establishing domain synchronization protocols that minimize replicated state payload size and network bandwidth utilization without violating domain isolation guarantees.

## Hierarchical Granularity for Logical Entities

Instantiating dedicated execution domains for high-frequency, short-lived gameplay objects (e.g., projectiles) introduces potential allocation and structural overhead.

> **Resolution Vector:** Determine whether lightweight entities warrant dedicated domain execution contexts or should be governed by a lightweight scope-grouping mechanism that enforces domain lifecycle contracts with reduced computational overhead.

## Structural Variants

Domain definitions frequently share identical dependency topologies while requiring dynamic implementation substitution (e.g., an `Enemy` domain binding either `Orc` or `Zombie` service implementations).

> **Resolution Vector:** Evaluate whether structural variability should be represented explicitly at the fluent API layer or handled via runtime instantiation parameters applied to unified domain definitions.

## Fluent API Modularity and Namespace Decoupling

As fluent topology definitions scale across multi-team repositories, centralized topology declarations become an architectural bottleneck.

> **Resolution Vector:** Introduce nested execution sub-scopes (`SubScopes`) allowing additive extension (e.g., expanding event publication rights and descendant domain visibility) while strictly guaranteeing the immutability of parent service bindings.

## Verification Integrity Chain

The framework currently cannot guarantee structural integrity parity between editor-verified topologies and the code the compile-time generator ultimately emits. The absence of a continuous verification mechanism leaves an explicit trust gap between editor-time validation and the generated bootstrap code that actually executes.

> **Legacy Resolution Vector:** Implement deterministic graph fingerprinting, build-time metadata embedding, or structural checksum validation to establish verifiable parity between verified topologies and generated execution models.

> **Current Resolution Vector:** Restrict formal topology verification exclusively to editor-time workflows, treating the compile-time generated bootstrap code as passive execution of the verified graph payload. Detecting or recovering from post-validation graph corruption, external tampering, or build pipeline artifact drift is reclassified as an application-level operational concern outside the framework's architectural scope.

## Code Protection and Obfuscation Integration

Post-compile transformations (such as symbol renaming and assembly encryption) risk invalidating metadata bindings and type resolution within the fluent topology definition.

> **Resolution Vector:** Derive persistent architectural hashes from original type metadata to maintain invariant bindings under obfuscation. This mitigates mapping indirection between obfuscated assemblies and source declarations.

## Explicit Creation Triggers for Domain Objects

The `.Owns<T>()` call currently conflates scope membership with automated initialization during domain loading.

> **Resolution Vector:** Extend key-based addressing to non-service domain objects (`.Owns("Key/Identifier")`), enabling on-demand instantiation via generated factories while preserving framework-managed dependency injection.

## Dynamic Instantiation Parameters

Dynamic parameters (e.g., spawn coordinates or initial state snapshots) cannot be declared statically via the fluent API, yet manual constructor invocation is prohibited.

> **Resolution Vector:** Formalize a two-phase initialization model or strongly typed factory payloads to cleanly decouple structural dependency injection from dynamic runtime initialization.
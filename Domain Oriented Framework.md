# Introduction

The framework is an architectural governance system for Unity applications. Rather than acting as a conventional dependency injection container, it provides a declarative model for defining application topology, dependency relationships, object lifecycles, and communication boundaries.

The architecture intentionally favors deterministic execution, explicit ownership of application composition, and compile-time verification over dynamic graph mutation. C# classes serve purely as type definitions and contracts, and both injection artifacts (factories) and the topology graph itself are source-generated at compile time: the fluent API is declared in C#, but the dependency graph it describes is fully resolved into static bootstrap code before the application ever runs.

Business logic remains fully decoupled from architectural configuration. Domain services, components, events, and logic are implemented as ordinary C# classes. Dependency injection follows explicit conventions: plain C# classes declare dependencies through constructors, while `MonoBehaviour` components utilize `[Inject]` method targets.

This separation establishes a strict ownership boundary: application domain engineers implement functional behavior, whereas architectural composition is defined exclusively through a dedicated fluent API.

---

# Architectural Terminology and Execution Stages

To establish the framework's governance model and verification layers, the application lifecycle is divided into distinct execution phases:

- **Compile-time:** The Roslyn C# compiler analyzes source code strictly as type definitions and structural contracts. Source generators process the fluent topology declarations to produce type metadata, strongly typed factory call sites, final structural integrity checks, and the fully static topology bootstrap code — domain hierarchy, service bindings, ownership, and event permissions are all resolved and validated at this stage.
- **Editor-time:** Development environment interaction phase. Editor tooling and Roslyn analyzers statically inspect fluent topology declarations, perform structural validation, and provide real-time feedback prior to compilation.
- **Build-time:** Post-processing pipeline encompassing asset compilation, code stripping, conditional compilation, assembly modification, and platform-specific packaging between code compilation and binary generation.
- **Execution:** Active application state from process launch onward. The compile-time generated bootstrap code constructs the topology graph directly; strongly typed factories and direct method invocations operate continuously without graph mutation, interpretation, or reflection-based resolution.

---

# Domains

Applications are organized into hierarchical domains representing bounded execution contexts. Every domain encapsulates its own dependency graph, object lifecycles, and communication boundaries while participating in a parent-child topology.

System components interact exclusively within their local domain, adjacent scopes, or ancestor contexts rather than accessing a global application registry. This topology enforces strict visibility boundaries, minimizes coupling, and permits independent domain evolution.

Domains may locally override inherited service implementations without mutating parallel or ancestor branches of the hierarchy.

---

# Architecture Fluent API

Architectural topology is described through a dedicated fluent API detailing:

- Domain hierarchy and parent-child scoping;
- Service registrations and local overrides;
- Object ownership boundaries;
- Event publication permissions.

Syntax illustration:

```cs
/*
 * FLUENT API CHEATSHEET
 *
 * Scope & Inheritance Rules:
 * - `root.Domain(name, cfg => ...)` -> Declares a child scope (inherits services from its parent).
 * Any domain builder (including a child's own `cfg`) may itself call `.Domain(...)`
 * to declare a further-nested sub-domain, so hierarchies can go arbitrarily deep.
 * - `root domain` -> The single topology root, configured in the entry-point TopologyDefinition.
 * Uninstantiable by user code, managed exclusively by the framework, and lives for the
 * entire application lifetime.
 * - Domains: Configuration lambdas act as structural blueprints. Domain instances
 * live strictly inside their parent domain scope (unlimited instances allowed).
 * - Services: `.Service<TContract, TImpl>()` -> Inherited down the domain tree;
 * overridable via `.OverrideService<TContract, TImpl>()`.
 * - Ownership (`.Owns<T>()` / `.OwnsPrefab(path)`): Entities exclusively owned by a
 * specific domain. They live strictly within its scope and are NOT inherited by
 * child domains. Discriminated by call:
 * .Owns<TClass>()          -> C# factory-instantiated object
 * .OwnsPrefab("Path")      -> Unity prefab asset
 * - Events: `.Event<TSignal>()` -> Domain-permitted C# signal types.
 */

using App.Services;
using Meta.Services;
using Meta.UI;
using Game.Services;
using Game.Entities;

public sealed class AppTopology : TopologyDefinition
{
    protected override void Configure(ITopologyBuilder root)
    {
        // Root domain: Framework-managed root topology and global infrastructure services
        root.Service<IAnalyticsService, AppAnalyticsService>(isPrivate: true)
            .Service<IAudioService, CoreAudioService>();

        // Child domain: Meta/UI logic scope
        root.Domain("meta", meta => meta
            .Service<IShopService, MetaShopService>()

            // Domain-permitted C# event type
            .Event<PurchaseCompletedEvent>()

            // Domain-owned entities (encapsulated, unavailable to child domains)
            .Owns<MainMenuController>()
            .Owns<ShopManager>()
            .OwnsPrefab("UI/Windows/ShopWindow"));

        // Child domain: Gameplay logic scope
        root.Domain("gameplay", gameplay => gameplay
            // Explicitly overriding inherited C# service from root app scope
            .OverrideService<IAudioService, GameplayAudioService>()

            // Domain-permitted C# event types
            .Event<DamageEvent>()
            .Event<PlayerDiedEvent>()

            // Domain-owned gameplay objects
            .Owns<PlayerController>()
            .Owns<EnemySpawner>()
            .OwnsPrefab("Gameplay/Units/Player")
            .OwnsPrefab("Gameplay/Weapons/Bullet")

            // Nested child domain: scoped strictly within gameplay
            .Domain("combat", combat => combat
                .Service<IHitResolutionService, HitResolutionService>()

                // Domain-permitted C# event type
                .Event<CriticalHitEvent>()

                // Domain-owned combat-scoped objects
                .Owns<ProjectilePool>()
                .OwnsPrefab("Gameplay/VFX/HitImpact")));
    }
}
```

The fluent topology is authored directly as ordinary C# code rather than parsed from an external file format. C# code therefore serves two roles side by side: type definitions and contracts for business logic, and the fluent builder calls that describe architectural composition. Compile-time source generation produces both the dependency injection factories and the static bootstrap code that constructs the topology — the fluent calls are consumed entirely by the generator and never execute as-written when the application starts.

Because factory generation is decoupled from topological composition, modifications to domain hierarchies, service bindings, or event permissions require updating only the fluent topology definition, provided the set of injectable C# types remains unchanged.

---

# Dependency Graph Generation

Unlike conventional dynamic containers, the compilation phase processes user C# code strictly as structural definitions to generate type metadata and strongly typed factories. The application topology is declared via the fluent API and is fully resolved into a static graph by the same compile-time source generator.

Upon successful compile-time validation, the source generator emits the complete executable topology as static bootstrap code, invoking call sites via pre-generated metadata and static factories. This architecture combines the flexibility of a fluently composed topology with single-stage, compile-time structural validation, avoiding any dynamic reflection or ad hoc graph assembly during application execution.

---

# Framework Restrictions Validation

While architectural topology and service bindings undergo editor-time and compile-time validation, compile-time Roslyn analyzers enforce localized code governance.

Analyzers do not evaluate global application topology. Their scope is strictly restricted to verifying that C# code respects framework access boundaries—specifically detecting unauthorized invocations of `[Inject]` methods, internal lifecycle hooks, or restricted framework APIs.

Decoupling local C# code inspection from topology validation ensures compile-time detection of unauthorized API access while reserving topological graph verification for dedicated validation layers.

---

# Framework Architecture

The framework comprises three integrated layers, each maintaining explicit functional boundaries:

|Layer|Responsibility|
|---|---|
|**Business Logic (C#)**|Provides type definitions and contracts. Implements services, components, events, and domain behavior.|
|**Fluent API**|Defines architectural topology, domain hierarchies, service bindings, and communication rules.|
|**Editor Tooling / Compile-Time Generators**|Executes static validation, incremental analysis, and source-generates both DI factories and the fully static topology bootstrap code that constructs and sequences lifecycles.|

Business logic never dictates architectural composition, and dependency discovery never happens during application execution — it is fully resolved by the compile-time generators. This separation guarantees complete architectural validation prior to execution while keeping domain code free of infrastructure overhead.

---

# Object Lifecycle

Object lifecycle management is fully deterministic and governed exclusively by the framework:

1. Strongly typed generated factories construct the object instance.
2. Dependencies are injected via generated constructor or `[Inject]` method invocations.
3. Event subscription bindings are registered according to validated fluent topology permissions.
4. The instance transitions to the initialized state following owning domain initialization.
5. The instance is disposed upon owning domain unloading.

This deterministic sequence ensures predictable initialization ordering, dependency availability, and resource cleanup across all application contexts.

---

# Domain Pooling

Domains are lightweight execution contexts rather than immutable application subsystems.

To eliminate scope allocation overhead, domains support recycling via domain pooling. A pooled domain preserves its validated dependency graph structure while re-initializing instance state for new context instances.

This mechanism allows short-lived gameplay contexts (e.g., localized simulations, combat encounters, or transient entities) to execute within isolated domain scopes without container allocation overhead.

---

# Developer Restrictions

To preserve deterministic guarantees and static verification invariants, the framework enforces strict operational constraints:

- Services cannot be registered after compile time.
- Dependency bindings and type contracts are defined strictly at compile-time.
- Dependency graph topology is immutable during execution.
- Domain hierarchies cannot be mutated dynamically during execution.
- Dynamic dependency discovery is not supported at any stage after compilation.
- Reflection-based injection is prohibited.
- Injection occurs exclusively through generated constructors and `[Inject]` invocations.
- Direct user invocation of `[Inject]` methods is restricted by Roslyn analyzers.
- Event publication is constrained by fluent topology permissions.
- Instantiation must execute via framework-managed generated factories.
- Execution operates exclusively on the statically generated, previously validated architectural model.
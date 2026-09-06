## Native Plugin and Runtime Extension Boundaries

> **Decision:** The framework explicitly omits native plugin extension subsystems and dynamic execution pipelines.

> **Rationale:** Dynamic extension models introduce severe friction under IL2CPP ahead-of-time compilation pipelines and compromise deterministic verification guarantees. Game modifiability and dynamic extensibility are classified as application-level concerns to be implemented via localized hooks within domain logic rather than framework-level infrastructure.

## Inherited Service Encapsulation

> **Decision:** Services remain transitively visible to descendant domains by default, but may be declared with an explicit private-visibility call (`.Service<IService, ServiceImpl>(isPrivate: true)`).

> **Rationale:** The private-visibility flag provides strict encapsulation for intra-domain infrastructure without altering default inheritance semantics or introducing structural ambiguity into verified graphs.

## Asset Reference Resolution in the Fluent API

> **Decision:** Unity prefab assets are declared using explicit path-based discriminators (`.OwnsPrefab("Relative/Path")`).

> **Rationale:** Unlike C# compile-time symbols, Unity prefabs are serialized asset artifacts lacking static type representation. Explicit string paths relative to designated asset roots provide deterministic editor-time validation and unambiguous resolution during engine-native instantiation.

## Dynamically Loaded Content Boundaries

> **Decision:** Non-deterministic content loading pipelines (e.g., Addressables or remote asset bundles) operate outside framework topology management.

> **Rationale:** Runtime-resolved asset availability is fundamentally incompatible with pre-validated execution models. Dynamically loaded entities must be managed within dedicated wrapper domains, isolating non-deterministic lifecycle logic from the framework's validated core.
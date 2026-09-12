# Rust Service Shape

Use when choosing between generic and dynamic dispatch for a service, or placing trait seams.

## Traits As Seams

- Use traits for real external or test seams: repositories, outbound clients, object storage, authorization clients, license verifiers, clocks, and deterministic ID generators.
- Do not create a trait for every struct. Prefer concrete types for deterministic internal logic.
- Treat dependency injection as a testing and replacement tool, not a universal rule.

## Dispatch Choice

Prefer generic services when static dispatch stays readable:

```rust
pub struct NodeService<R, A> {
    repository: R,
    authorization: A,
}

impl<R, A> NodeService<R, A>
where
    R: NodeRepository,
    A: AuthorizationClient,
{
    pub async fn get_node(&self, request: GetNodeRequest) -> ProductResult<RegisteredNode> {
        // orchestration only
    }
}
```

Use dynamic dispatch when runtime composition, heterogeneous implementations, or simpler type signatures justify it:

```rust
pub struct NodeService {
    repository: Arc<dyn NodeRepository + Send + Sync>,
    authorization: Arc<dyn AuthorizationClient + Send + Sync>,
}
```

Generic parameters propagate into every caller's type signature. When that propagation starts appearing in unrelated modules, the readability argument has flipped and `Arc<dyn _>` is the better trade.

## Module Shape

- Split large `lib.rs` files; `lib.rs` exposes intentional APIs and does not contain all implementation.
- Separate type-heavy modules from behavior when a domain grows. Common names are `model`, `command`, `service`, `ports`, `adapters`, `error`, and `constants`, but follow local project style.
- Keep domain modules free of web framework types, RPC framework types, database clients, vendor SDK clients, and tracing setup.

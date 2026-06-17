# API-First Methodology

## Philosophy

Design APIs through **domain-driven thinking**, not mechanical CRUD. The guiding
sentence: *APIs should reflect how users interact, not how developers
implement.* Endpoints are derived from valid **state transitions**, not from
database tables.

Core principles:
1. **Resource-centric** — design around real domain entities, not DB operations.
2. **Domain knowledge first** — domain experts decide which transitions are
   valid; engineers encode them.
3. **Semantic clarity** — every operation should be immediately understandable.
4. **Clear scope** — be explicit about what is and isn't possible.
5. **Separation of concerns** — the API provides *building blocks*; applications
   orchestrate business logic. Don't bake workflow orchestration into endpoints.

## The five areas of API work

1. **API Design** — data models, methods, workflows, validation.
2. **API Architecture** — gateways, caching, performance, protocols, monitoring.
3. **API Governance** — versioning, adoption, lifecycle management.
4. **API Implementation Patterns** — async operations, pagination, idempotence,
   batching.
5. **API Engineering** — dev process, testing, collaboration, documentation.

## Standard vs Custom methods

- **Standard methods** = HTTP verbs for CRUD and structural changes:
  `GET, POST, PUT, PATCH, DELETE`.
- **Custom methods** = domain actions beyond CRUD, expressed as a `:verb`
  suffix on the resource. Reach for these when a transition is a domain action,
  not a field update.

| Intent | Use |
|---|---|
| Create resource (initial transition) | `POST /resources` |
| Read one / list | `GET /resources/{id}` / `GET /resources` |
| Replace / partial update fields | `PUT` / `PATCH /resources/{id}` |
| Terminal transition (delete) | `DELETE /resources/{id}` |
| Domain transition (start/stop/cancel/approve…) | `POST /resources/{id}:verb` |

### EC2 instance example

Valid domain verbs: `launch, start, stop, reboot, terminate`.

```
POST   /instances              # launch (create)
POST   /instances/{id}:start   # start a stopped instance
POST   /instances/{id}:stop    # stop a running instance
POST   /instances/{id}:reboot  # reboot a running instance
DELETE /instances/{id}         # terminate
```

Each maps to exactly one allowed state transition. There is deliberately no
`:start` on an already-running or terminated instance — the transition table
forbids it, so the API does too.

## Conventions

- **Naming**: plural nouns for collections (`/orders`), resource id in the path
  (`/orders/{id}`), domain verbs (not nouns) for custom methods.
- **Versioning**: treat versioning as a governance decision; version the
  contract, and plan adoption + lifecycle (deprecation) explicitly.
- **Pagination / async / idempotence / batching**: treat these as named
  implementation patterns, decided per endpoint — not ad hoc.
- **Errors**: return open-ended failure detail as *status* data (reason,
  message), distinct from the resource's finite *state*.

## OpenAPI as the contract

- The spec is the **product requirement document** and the **contract** — the
  most concrete, complete artifact available. Write it **first**.
- Avoid the anti-pattern of coding first and reverse-engineering the spec later.
- Combine: **FSM model** (states/transitions) + **class diagram** (structure) +
  **transition table** (exhaustive operation list) + **role-based authorization**
  (who does what).
- Known OpenAPI v3.0 limitations: weak modularity for large systems; `$ref` path
  constraints complicate modular decomposition — plan file structure accordingly.

## Common API-First pitfalls to avoid

- Underestimating the OpenAPI spec; treating it as optional rather than the
  foundational contract.
- Weak OOAD (object-oriented analysis/design) skills leading to anemic models.
- Embedding business logic in APIs, creating tight coupling.
- Writing code first and back-filling the spec.
- Needing separate use-case docs because the API isn't self-describing.
- Letting unresolved communication problems hide behind missing specs.

## Recommended reading

- *API Design Patterns* (and Google's public API style guide)
- *Continuous API Management*
- *Mastering API Architecture*
- Martin Fowler — "Anemic Domain Model"

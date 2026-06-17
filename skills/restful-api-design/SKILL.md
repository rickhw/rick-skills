---
name: restful-api-design
description: Design RESTful APIs using a Finite-State-Machine-first, domain-driven methodology. Use when designing or reviewing HTTP/REST APIs, defining resources and endpoints, modeling resource lifecycles and state transitions, choosing between standard and custom methods, writing OpenAPI specs, or deciding which operations an API should expose. Triggers on requests like "design an API for X", "what endpoints should this resource have", "model the lifecycle of this resource", or "review this API design".
---

# RESTful API Design (FSM-First)

## Overview

This skill designs RESTful APIs by modeling each resource as a **Finite State
Machine (FSM)** rather than as a set of database CRUD operations. The core
belief: **an API should reflect how users interact with a resource, not how
developers store it.** Endpoints are *derived* from valid state transitions, so
the API exposes exactly the operations the domain allows — no more, no fewer.

This is a methodology, not a CRUD generator. Domain knowledge drives the design.

## When to use this skill

- Designing a new API or a new resource within an existing API
- Deciding what endpoints/operations a resource should expose
- Modeling a resource's lifecycle (e.g. order, instance, bug, subscription)
- Choosing between standard HTTP methods and custom methods
- Reviewing an existing API for missing/invalid/illogical operations
- Writing or reviewing an OpenAPI specification

## The method (follow in order)

### Step 1 — Identify the controlled object (the resource)

State the single noun this API controls (Order, Instance, Bug, Subscription).
If you cannot name one clear noun, the scope is wrong — split it first. One FSM
controls exactly one resource type. See `references/fsm-design-principles.md`.

### Step 2 — Enumerate the states

List the resource's states as **nouns or adjectives**, each a single lexical
unit (e.g. `pending`, `running`, `stopped`, `terminated`).

Rules:
- Keep states **≤ 10, ideally ~5**.
- **Exactly one initial state.** Multiple end states are allowed.
- Distinguish **state** (finite, enumerable, countable — belongs in the FSM)
  from **status** (open-ended conditions like error reasons — does *not* drive
  the FSM). See the State vs Status note in the principles reference.
- If you exceed ~10 states, decompose with a **Nested / Hierarchical FSM**
  (divide and conquer) instead of one giant machine.

### Step 3 — Build the state transition table

Create a from → to matrix. For every cell mark:
- ✅ allowed, with the **domain verb** that triggers it (e.g. `start`, `stop`)
- ❌ prohibited

This table is the heart of the design. Invalid transitions are eliminated here,
which is what stops the API from sprouting meaningless endpoints. Use the
template in `references/state-transition-table-template.md`.

### Step 4 — Derive endpoints from transitions

Map each resource and each allowed transition to HTTP:

- **Standard methods** for lifecycle and structural operations:
  - `POST   /orders`            — create (the initial transition)
  - `GET    /orders` / `GET /orders/{id}` — read / list
  - `PUT|PATCH /orders/{id}`    — update fields
  - `DELETE /orders/{id}`       — terminal transition
- **Custom methods** for domain transitions that aren't plain CRUD, using a
  `:verb` suffix on the resource:
  - `POST /instances/{id}:start`
  - `POST /instances/{id}:stop`
  - `POST /orders/{id}:cancel`

Only create an endpoint if a ✅ transition justifies it. If the table forbids a
transition, the API must not offer it.

See `references/api-first-methodology.md` for standard-vs-custom guidance and
naming conventions.

### Step 5 — Attach events and roles (authorization)

- Transitions may fire **events** before/after execution (hooks, notifications).
- Define **roles** (who may trigger each transition). The same transition is
  often allowed for one role and forbidden for another — encode this alongside
  the transition table, not as an afterthought.

### Step 6 — Capture it as a contract (OpenAPI)

Treat the spec as the **product requirement document and the contract**, written
*before* implementation — not reverse-engineered from code afterward. Keep
business-logic orchestration out of the API surface: the API provides building
blocks; applications compose them. See the methodology reference for the
API-First pitfalls to avoid.

## Output you should produce

When applying this skill, deliver (at minimum):
1. The controlled object (one noun) and a one-line scope statement.
2. The state list (with the single initial state and end states marked).
3. The state transition table (from → to, verbs, ✅/❌).
4. The derived endpoint list (method + path + the transition it serves).
5. Roles per transition, if more than one actor exists.

Prefer an FSM/state diagram (Mermaid `stateDiagram-v2`) plus the table — they
make invalid transitions obvious.

## References

- `references/fsm-design-principles.md` — the 5 FSM design principles, state vs
  status, nested/hierarchical FSMs, Mealy/Moore, complexity limits.
- `references/api-first-methodology.md` — API-First philosophy, standard vs
  custom methods, naming/versioning/pagination conventions, common pitfalls.
- `references/state-transition-table-template.md` — fill-in templates for the
  state table and a worked EC2-instance example.

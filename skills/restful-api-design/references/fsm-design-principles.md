# FSM Design Principles

Why FSM: the *finite* nature of states establishes clear boundaries, which is
what makes a system reliable and consistent. Without a finite constraint,
systems drift and consistency becomes hard to guarantee. FSM is a planning tool
for system design and business flow, not just an implementation detail.

## The 5 design principles

### 1. Define the controlled object
Before anything else, name the single thing you are controlling — almost always
a noun: an API resource, a work item, a business entity (Order, BOM, Defect,
Feature). Skipping this step is the most common cause of chaotic designs.

### 2. Limited states
- State names are **nouns or adjectives**, each a **single lexical unit**.
- Keep states **under 10, ideally around 5**.
- When you exceed that, apply **divide and conquer**: use a **Nested FSM (NFSM)**
  or **Hierarchical State Machine (HSM)** rather than one sprawling machine.
- Rationale: the simplified TCP state machine has ~11 states and is already hard
  to hold in your head — beyond ~10 states cognitive load grows sharply.

### 3. Clear start and end points
- Exactly **one initial state**.
- **Multiple end states** are allowed.
- Design by fixing the head (initial) and tail (terminal) states first, then
  fill in the middle.

### 4. State transitions
- Transitions are **verb phrases** and are recorded in a **state transition
  table** connecting every from → to pair.
- The connections are programmable / enumerable.
- RESTful APIs align naturally with this: each allowed transition becomes an
  operation.

### 5. Events and roles
- A transition can trigger **events** before and/or after it executes.
- **Roles** (developer, manager, QA, reporter, ...) have *different* permitted
  transitions. The same transition may be allowed for one role and forbidden for
  another. Handle each role's events explicitly.

## State vs Status — a critical distinction

- **State**: limited, enumerable, countable. Like a traffic light's three
  colors. This is what belongs in the FSM and what drives API operations.
- **Status**: open-ended, uncertain, not cleanly countable. Like HTTP status
  codes or arbitrary failure reasons. This does **not** drive the FSM.

If a "state" can take unbounded values, it's really a status field — model it as
data, not as an FSM node.

## When NOT to force an FSM

Some things look stateful but aren't finite. Example: a CI/CD pipeline. A single
stage (build → test → deploy) has a clean FSM, but orchestrating, say, 10
projects yields 100+ pipeline variations — an "infinite divergence machine,"
not a finite-state one. If states multiply combinatorially, FSM is the wrong
lens (or you need hierarchical decomposition).

## Worked example: Bug workflow

- **Resource**: Bug
- **End states**: New, Closed, Canceled, Limitation
- **Roles**: Developer, Team Leader, Reporter (QA/Tester)
- **Constraint**: each role has a restricted set of allowed transitions.

## Theory (background)

- **Mealy machine (1955)**: output depends on current state *and* input; uses
  states more efficiently.
- **Moore machine (1956)**: output depends only on current state; more stable
  and resistant to signal glitches.
- **Replicated State Machine (Raft, 2014)**: multiple servers reach consensus by
  executing identical commands from identical initial state.
- **Statecharts** (Harel) are the basis of UI libraries like XState — the
  practical realization of hierarchical FSMs.

## Foundational mindset

Two principles underpin all five rules:
- **Divide and conquer** (decompose large machines).
- **High cohesion, low coupling** (each FSM owns one resource cleanly).

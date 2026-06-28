# rick-skills — Rick's Claude Skills

A collection of publishable [Agent Skills](https://docs.claude.com/en/docs/claude-code/skills)
for [Claude Code](https://docs.claude.com/en/docs/claude-code). Each skill lives
in its own folder under `skills/` and can be installed independently.

## Skill catalog

| Skill | What it does |
|---|---|
| [`restful-api-design`](skills/restful-api-design/) | Design RESTful APIs with a Finite-State-Machine-first, domain-driven method. Model each resource as an FSM, build a state transition table, and *derive* endpoints from valid transitions. (Traditional Chinese / 正體中文) |
| [`r3-model`](skills/r3-model/) | Describe & design system architecture with the R3 Model — three views (High Level / Logical / Physical) centered on Role & Responsibility. A C4-like, R&R-driven methodology. (Traditional Chinese / 正體中文) |
| [`software-versioning`](skills/software-versioning/) | Plan version & artifact management across the SDLC — SemVer numbering, dev-vs-release artifacts, artifact≠config, Version-First vs Version-Late, traceable delivery. (Traditional Chinese / 正體中文) |
| [`iterative-delivery`](skills/iterative-delivery/) | End-to-end product delivery workflow — version-line iterations with gated design docs (Req→URD→SystemDesign→Tasks), branch/tag conventions, "tag as release" + deploy-from-registry, test-gated releases with a regression test per bug, async-write/cached-read API principles, and a hard-won operational-pitfalls checklist. (Traditional Chinese / 正體中文) |
| _more coming…_ | |

## Repo layout

```
.
├── README.md
├── LICENSE
├── skills-lock.json
└── skills/
    ├── restful-api-design/
    │   ├── SKILL.md                                   # the skill (entry point)
    │   └── references/
    │       ├── fsm-design-principles.md               # the 5 FSM principles, state vs status
    │       ├── api-first-methodology.md               # standard vs custom methods, conventions, pitfalls
    │       ├── state-transition-table-template.md      # fill-in templates + worked EC2 example
    │       └── identifier-design.md                    # ID design: scope, countability, UUID/ULID/Snowflake/KGS
    ├── r3-model/
    │   ├── SKILL.md                                   # the skill (entry point)
    │   └── references/
    │       └── three-views.md                          # per-view templates + R&R checklist + worked example
    ├── software-versioning/
    │   ├── SKILL.md                                   # the skill (entry point)
    │   └── references/
    │       ├── semver-and-numbering.md                 # X.Y.Z(.Q), build metadata, naming conventions
    │       ├── artifact-management.md                  # dev vs rel, artifact≠config, deliver-on-day-1
    │       └── versioning-in-sdlc.md                   # Version-First vs Version-Late, version flow through SDLC
    └── iterative-delivery/
        ├── SKILL.md                                   # the skill (entry point)
        ├── references/
        │   ├── iteration-workflow.md                   # version-line dirs, gated Req→URD→Design→Tasks, doc layering
        │   ├── branch-and-release.md                   # branch/tag roles, "tag as release", deploy-from-registry, /readyz, rollback
        │   ├── async-api-runtime.md                    # async writes (202+event_id+idempotency), cached reads, OpenAPI, logging
        │   └── operational-pitfalls.md                 # real production incidents: bytes-vs-chars, real-IP, compose orphans, …
        └── templates/                                  # 00-Req / 01-URD / 02-SystemDesign / 03-Tasks / ReleaseNotes
```

Each `SKILL.md` is an entry point. Its YAML frontmatter (`name` + `description`)
controls when Claude automatically invokes the skill; the reference files are
loaded on demand to keep the main file lean.

## Installing a skill

### Option A — personal/project skills directory (simplest)

Copy a skill folder into your skills directory:

```bash
# user-level (available in every project)
cp -r skills/restful-api-design ~/.claude/skills/

# or project-level (checked in with a repo)
cp -r skills/restful-api-design <your-project>/.claude/skills/
```

Claude Code discovers it automatically. Confirm with `/skills`.

### Option B — install from GitHub via skills-lock.json

If you manage skills through a `skills-lock.json`, add an entry per skill
pointing at this repository (`skillPath` selects which skill in the repo):

```json
{
  "version": 1,
  "skills": {
    "restful-api-design": {
      "source": "rickhw/rick-skills",
      "sourceType": "github",
      "skillPath": "skills/restful-api-design/SKILL.md"
    }
  }
}
```

### Option C — ship it as a plugin

Bundle a skill folder inside a Claude Code plugin's `skills/` directory and
distribute it through a plugin marketplace.

---

## Skill: `restful-api-design`

Encodes a **Finite-State-Machine-first, domain-driven** approach to RESTful API
design, distilled from three articles:

- [API First Journey](https://rickhw.github.io/2025/03/08/Design/Gossip-API-First-Joureny/)
- [Finite State Machine](https://rickhw.github.io/2026/04/20/Design/Finite-State-Machine/)
- [Identifiers Design Consideration](https://rickhw.github.io/2024/03/24/Design/Identifiers-Design-Consideration/)

The central idea: **design an API to reflect how users interact with a resource,
not how it is stored.** Model each resource as a finite state machine, build a
state transition table, and *derive* endpoints from the valid transitions — so
the API exposes exactly the operations the domain allows, and nothing else.

### How to use it

Once installed, you don't call it explicitly — Claude triggers the skill when
your request matches its description. Prompts that activate it (Chinese or
English both work):

- 「幫我設計一個訂單管理的 REST API。」
- "What endpoints should a **subscription** resource expose?"
- 「把支援工單的生命週期建模成 API。」
- "Review this API design — are any operations missing or invalid?"

The skill then walks the method and produces:

1. **Controlled object** — the one noun the API controls, plus a scope line.
2. **States** — ≤10 (ideally ~5), one initial state, end states marked.
3. **State transition table** — from → to, the domain verb per transition,
   ✅/❌, and roles.
4. **Derived endpoints** — HTTP method + path, each tied to one transition,
   with a deliberate identifier scheme for `{id}`.
5. A Mermaid `stateDiagram-v2` so invalid transitions are obvious at a glance.

### The method in one screen

1. **Identify the controlled object** — one clear noun, or split the scope.
2. **Enumerate states** — nouns/adjectives, ≤10, one initial / many end. Keep
   finite *state* separate from open-ended *status*.
3. **Build the transition table** — mark every from→to as a verb or ❌.
4. **Derive endpoints** — standard methods (`POST/GET/PUT/PATCH/DELETE`) for
   CRUD/lifecycle; custom `POST /resource/{id}:verb` for domain transitions.
   No endpoint exists without a ✅ transition behind it. Choose the `{id}`
   scheme deliberately (scope, internal-vs-external, sortability).
5. **Attach events & roles** — who may trigger each transition; pre/post hooks.
6. **Capture as OpenAPI** — write the contract first; it *is* the requirement.

### Design principles behind the skill

- **Resource-centric, domain-first** — domain experts decide valid transitions.
- **Finite states create reliability** — boundaries make systems consistent.
- **State ≠ status** — finite/enumerable states drive the FSM; open-ended
  conditions (error reasons, HTTP codes) are data, not FSM nodes.
- **Separation of concerns** — the API offers building blocks; applications
  orchestrate workflow. Don't embed business logic in endpoints.
- **Divide and conquer** — past ~10 states, decompose into nested/hierarchical
  FSMs rather than one giant machine.
- **Identifiers are a deliberate choice** — pick by scope, countability, and
  whether the ID is user-facing; don't default to auto-increment or UUID.

---

## Skill: `r3-model`

Describes and designs system architecture with the **R3 Model** — three views of
the same system at different abstraction levels, all driven by **Role &
Responsibility (R&R)**. Distilled from:

- [R3 Model](https://rickhw.github.io/2025/06/29/Design/R3-Model/)

Conceptually close to the C4 Model (R3 predates the author's discovery of C4),
but with R&R as the through-line, grounded in OOP encapsulation and Conway's Law.

### The three views

1. **High Level View** — for non-technical stakeholders: users, internal/external
   dependencies, and access control (ACL) expressed as public/protected/private.
2. **Logical View** — service roles (Web, API, DB, Cache, Queue…) with their
   responsibilities, data flow tied to User Stories, and throughput/frequency on
   hot paths. The goal is shared understanding, not implementation.
3. **Physical View** — deployable form: pods/containers, N replicas, technology
   choices, and observability (metrics/logging/tracing) context.

No fourth code-level view — use sequence diagrams and state machines for detailed
behavior instead.

### How to use it

Trigger prompts (Chinese or English):

- 「幫我畫這個系統的架構圖。」
- "Write an architecture doc for this service."
- 「這個系統有哪些角色與職責？」
- 「規劃這個系統的部署架構與副本數。」

---

## Skill: `software-versioning`

Plans **version management and artifact management** across the SDLC. Core idea:
**version numbers are a communication interface** — between PM, dev, QA, and ops.
Distilled from three articles:

- [Version Control](https://rickhw.github.io/2015/02/11/SoftwareEngineering/Version-Control/)
- [Artifact Management and Version Control](https://rickhw.github.io/2022/04/06/SoftwareEngineering/Artifact-Management-and-Version-Control/)
- [Versioning in SDLC](https://rickhw.github.io/2024/09/14/SoftwareEngineering/Versioning-in-SDLC/)

### What it covers

- **SemVer `X.Y.Z` (+ optional `Q`)** numbering and build metadata (build id,
  revision, tag, customer id); a version is a *feature set*, not a schedule, and
  only increments *after* the release procedure.
- **dev vs release artifacts** — internal/unvalidated vs certified/customer-ready
  (exactly one per release cycle).
- **artifact ≠ config** (1-to-many): one artifact, per-environment config —
  Single Codebase, Multiple Deployment.
- **Version-First (products) vs Version-Late (projects)** — and the failure modes
  of leaving a project un-versioned.
- **Branch strategy vs artifact management vs version control** solve *different*
  problems — don't conflate them.
- **Deliver "Hello World" on day one** — CI-built artifacts from project start.

### How to use it

Trigger prompts (Chinese or English):

- 「這個專案的版本號要怎麼訂？SemVer 還是時間戳？」
- "Design an artifact naming and release flow for this service."
- 「dev build 跟 release build 要怎麼區分？」
- "How should a version flow through dev/test/staging/prod with traceability?"

---

## Adding more skills to this repo

1. Create a new folder under `skills/<skill-name>/` with its own `SKILL.md`
   (and an optional `references/` folder).
2. Write the frontmatter `name` (kebab-case slug) and a `description` that
   clearly states *when* Claude should trigger it.
3. Add a row to the **Skill catalog** table above.
4. Publish, then install via Option B with
   `skillPath: skills/<skill-name>/SKILL.md`.

## Publishing changes

```bash
git add .
git commit -m "…"
git push
```

## License

MIT — see [LICENSE](LICENSE).

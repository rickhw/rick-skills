# rick-skills — Rick's Claude Skills

A collection of publishable [Agent Skills](https://docs.claude.com/en/docs/claude-code/skills)
for [Claude Code](https://docs.claude.com/en/docs/claude-code). Each skill lives
in its own folder under `skills/` and can be installed independently.

## Skill catalog

| Skill | What it does |
|---|---|
| [`restful-api-design`](skills/restful-api-design/) | Design RESTful APIs with a Finite-State-Machine-first, domain-driven method. Model each resource as an FSM, build a state transition table, and *derive* endpoints from valid transitions. (Traditional Chinese / 正體中文) |
| _more coming…_ | |

## Repo layout

```
.
├── README.md
├── LICENSE
├── skills-lock.json
└── skills/
    └── restful-api-design/
        ├── SKILL.md                                   # the skill (entry point)
        └── references/
            ├── fsm-design-principles.md               # the 5 FSM principles, state vs status
            ├── api-first-methodology.md               # standard vs custom methods, conventions, pitfalls
            ├── state-transition-table-template.md      # fill-in templates + worked EC2 example
            └── identifier-design.md                    # ID design: scope, countability, UUID/ULID/Snowflake/KGS
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

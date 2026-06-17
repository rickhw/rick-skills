# restful-api-design — Claude Skills

A publishable [Agent Skill](https://docs.claude.com/en/docs/claude-code/skills)
that encodes a **Finite-State-Machine-first, domain-driven** approach to
RESTful API design.

It is distilled from two articles:

- [API First Journey](https://rickhw.github.io/2025/03/08/Design/Gossip-API-First-Joureny/)
- [Finite State Machine](https://rickhw.github.io/2026/04/20/Design/Finite-State-Machine/)

The central idea: **design an API to reflect how users interact with a resource,
not how it is stored.** Model each resource as a finite state machine, build a
state transition table, and *derive* endpoints from the valid transitions — so
the API exposes exactly the operations the domain allows, and nothing else.

## What's in this repo

```
.
├── README.md
└── skills/
    └── restful-api-design/
        ├── SKILL.md                                   # the skill (entry point)
        └── references/
            ├── fsm-design-principles.md               # the 5 FSM principles, state vs status
            ├── api-first-methodology.md               # standard vs custom methods, conventions, pitfalls
            └── state-transition-table-template.md      # fill-in templates + worked EC2 example
```

`SKILL.md` is the entry point. Its YAML frontmatter (`name` + `description`)
controls when Claude automatically invokes the skill; the reference files are
loaded on demand to keep the main file lean.

## Installing the skill

### Option A — personal/project skills directory (simplest)

Copy the skill folder into your skills directory:

```bash
# user-level (available in every project)
cp -r skills/restful-api-design ~/.claude/skills/

# or project-level (checked in with a repo)
cp -r skills/restful-api-design <your-project>/.claude/skills/
```

Claude Code discovers it automatically. Confirm with `/skills`.

### Option B — install from GitHub via skills-lock.json

If you manage skills through a `skills-lock.json` (one already exists in this
repo as an example), add an entry pointing at this repository:

```json
{
  "version": 1,
  "skills": {
    "restful-api-design": {
      "source": "<your-gh-user>/restful-api-design",
      "sourceType": "github",
      "skillPath": "skills/restful-api-design/SKILL.md"
    }
  }
}
```

### Option C — ship it as a plugin

Bundle `skills/restful-api-design/` inside a Claude Code plugin's `skills/`
directory and distribute it through a plugin marketplace.

## Publishing this repo

```bash
git add .
git commit -m "Add restful-api-design skill"
gh repo create restful-api-design --public --source=. --push
```

Once it's on GitHub, anyone can install it with Option B (point `source` at
`<your-gh-user>/restful-api-design`).

## How to use it

Once installed, you don't call it explicitly — Claude triggers the skill when
your request matches its description. Prompts that activate it:

- "Design a REST API for managing **orders**."
- "What endpoints should a **subscription** resource expose?"
- "Model the **lifecycle** of a support ticket as an API."
- "Review this API design — are any operations missing or invalid?"

The skill then walks the method and produces:

1. **Controlled object** — the one noun the API controls, plus a scope line.
2. **States** — ≤10 (ideally ~5), one initial state, end states marked.
3. **State transition table** — from → to, the domain verb per transition,
   ✅/❌, and roles.
4. **Derived endpoints** — HTTP method + path, each tied to one transition.
5. A Mermaid `stateDiagram-v2` so invalid transitions are obvious at a glance.

### The method in one screen

1. **Identify the controlled object** — one clear noun, or split the scope.
2. **Enumerate states** — nouns/adjectives, ≤10, one initial / many end. Keep
   finite *state* separate from open-ended *status*.
3. **Build the transition table** — mark every from→to as a verb or ❌.
4. **Derive endpoints** — standard methods (`POST/GET/PUT/PATCH/DELETE`) for
   CRUD/lifecycle; custom `POST /resource/{id}:verb` for domain transitions.
   No endpoint exists without a ✅ transition behind it.
5. **Attach events & roles** — who may trigger each transition; pre/post hooks.
6. **Capture as OpenAPI** — write the contract first; it *is* the requirement.

## Design principles behind the skill

- **Resource-centric, domain-first** — domain experts decide valid transitions.
- **Finite states create reliability** — boundaries make systems consistent.
- **State ≠ status** — finite/enumerable states drive the FSM; open-ended
  conditions (error reasons, HTTP codes) are data, not FSM nodes.
- **Separation of concerns** — the API offers building blocks; applications
  orchestrate workflow. Don't embed business logic in endpoints.
- **Divide and conquer** — past ~10 states, decompose into nested/hierarchical
  FSMs rather than one giant machine.

## Adding more skills to this repo

Create another folder under `skills/` with its own `SKILL.md` (and optional
`references/`), then add it to your `skills-lock.json` / plugin manifest. The
repo is structured to hold a family of API-design skills.

## License

MIT — see [LICENSE](LICENSE).

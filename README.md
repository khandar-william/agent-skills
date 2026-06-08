# agent-skills

A collection of [Agent Skills](https://agentskills.io/home) — portable, version-controlled instruction sets that extend AI agents with specialized workflows and domain expertise. Skills follow the open [Agent Skills format](https://agentskills.io/specification) and work across any compatible agent (Cursor, Claude Code, Copilot, Gemini CLI, OpenCode, and [many others](https://agentskills.io/clients)).

## Skills

| Skill | Description |
|-------|-------------|
| [commit-message](commit-message/) | Writes conventional commit messages from git diffs with bullet-point body details |
| [deep-research](deep-research/) | Produces long-form research reports using a Planner + Writer methodology with multi-source synthesis and citations |
| [pr-explainer](pr-explainer/) | Generates detailed pull request descriptions from code diffs |
| [writing-code](writing-code/) | Guides implementation toward simple, deep, maintainable code using *Philosophy of Software Design* and *Five Lines of Code* principles |

### commit-message

Use when you need a commit message, help committing staged changes, or a diff summarized for git. Reads the diff, infers intent, and outputs a [Conventional Commits](https://www.conventionalcommits.org/) subject line plus a bullet-point body describing what changed and why.

### deep-research

Use for deep dives, comprehensive analysis, or long-form research reports (2,000–10,000+ words). Follows a dual-agent Planner + Writer pipeline inspired by Alibaba's WebWeaver: search and outline first, then section-by-section synthesis with inline citations. Includes reference files for extraction patterns, outline structure, source reliability, Mermaid diagram rendering, and writer prompts.

### pr-explainer

Use when you have a diff and need a PR description or summary. Produces a structured description covering overview, changes made, motivation, and notes for the reviewer — focused on impact and intent rather than restating the diff.

### writing-code

Use when implementing features, refactoring, or when you want high design quality with minimal surgical changes. Encodes principles from John Ousterhout's *A Philosophy of Software Design* and Christian Clausen's *Five Lines of Code*: deep modules, information hiding, simplicity, and goal-driven execution.

## Installation

### Manual install

Copy a skill directory into your agent skills folder:

```bash
# User-level (available across all projects)
cp -r commit-message ~/.agents/skills/

# Project-level (shared with anyone using the repository)
mkdir -p .agents/skills
cp -r commit-message .agents/skills/
```

Repeat for any other skills you want. Your agent discovers skills at startup and loads full instructions only when a task matches a skill's description.

### Skills CLI

You can also install individual skills with the [Skills CLI](https://skills.sh/):

```bash
npx skills add khandar-william/agent-skills@commit-message
npx skills add khandar-william/agent-skills@deep-research
npx skills add khandar-william/agent-skills@pr-explainer
npx skills add khandar-william/agent-skills@writing-code
```

Add `-g -y` to install globally and skip confirmation prompts.

## Repository structure

Each skill is a directory with a `SKILL.md` file (required) and optional supporting files:

```
agent-skills/
├── commit-message/
│   └── SKILL.md
├── deep-research/
│   ├── SKILL.md
│   └── references/
│       ├── extraction-pattern.md
│       ├── mermaid-diagram-rendering.md
│       ├── outline-pattern.md
│       ├── source-reliability.md
│       └── writer-system-prompt.md
├── pr-explainer/
│   └── SKILL.md
└── writing-code/
    └── SKILL.md
```

Agents load skills through [progressive disclosure](https://agentskills.io/home): they read each skill's `name` and `description` at startup, pull in the full `SKILL.md` when a task matches, and load referenced files only as needed.

## Authoring

To add a new skill, create a directory with a `SKILL.md` that includes YAML frontmatter (`name` and `description` are required). Keep the main file concise; put detailed reference material in sibling files and link to them from `SKILL.md`.

See the [Agent Skills specification](https://agentskills.io/specification) and [quickstart guide](https://agentskills.io/skill-creation/quickstart) for the full authoring guide.

## License

[MIT](LICENSE)

---

Update this file with the following prompt:
```
Read this codebase and update its README.md appropriately
```

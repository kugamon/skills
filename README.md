# Kugamon Skills

Reusable skills for Salesforce ISVs and consulting partners.

This repository is both a **Claude plugin** (`kugamon-skills`) and a **marketplace** (`kugamon`), hosted by [Kugamon LLC](https://kugamon.com). Each skill lives in its own folder under `skills/` with a `SKILL.md`, a `README.md`, and a `CHANGELOG.md`.

## Available Skills

| Skill | What it does |
|---|---|
| [sf-message-catalog](skills/sf-message-catalog) | Generates a customer-facing catalog of every user-facing error, warning, info, and confirmation message in a Salesforce managed package. |
| [sf-message-grammar-audit](skills/sf-message-grammar-audit) | Audits every user-facing message in a Salesforce managed package for grammatical issues and produces a prioritized remediation report. |

## Installation

There are two ways to install:

### Option 1: Install the whole plugin (recommended)

Get all skills at once and receive updates automatically as new skills are added.

From Claude Code or Claude Desktop:

```
/plugin marketplace add kugamon/skills
/plugin install kugamon-skills@kugamon
```

When new skills or updates ship, run:

```
/plugin update kugamon-skills
```

### Option 2: Copy an individual skill

If you only want one skill and prefer not to install the whole bundle:

1. Clone the repo (or download the skill folder you want):
   ```bash
   git clone https://github.com/kugamon/skills.git
   ```
2. Copy the skill folder into your local skills directory:
   ```bash
   cp -R skills/sf-message-catalog ~/.claude/skills/
   ```
3. Restart your agent client so the new skill is picked up.

Some skills have additional dependencies — see each skill's `README.md` for requirements.

## Repository Layout

```
.claude-plugin/
  plugin.json            <-- plugin manifest (for /plugin install)
  marketplace.json       <-- Kugamon marketplace definition
skills/                  <-- one folder per skill
  sf-message-catalog/
    SKILL.md             <-- skill definition (frontmatter + instructions)
    README.md            <-- human-facing description
    CHANGELOG.md         <-- version history
  sf-message-grammar-audit/
    ...
LICENSE                  <-- MIT, applies to the whole repo
CONTRIBUTING.md
README.md                <-- this file
```

## Versioning

The plugin follows [semantic versioning](https://semver.org/):

- **Patch** (`1.0.x`) — bug fixes to existing skills
- **Minor** (`1.x.0`) — new skill added or non-breaking feature to an existing skill
- **Major** (`x.0.0`) — breaking change to a skill's inputs or outputs

See each skill's `CHANGELOG.md` for skill-level history, and the [GitHub Releases](https://github.com/kugamon/skills/releases) page for plugin-level releases.

## Contributing

Pull requests welcome. See [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines — including the SKILL.md frontmatter contract, testing conventions, and how to propose a new skill.

## License

[MIT](LICENSE) — © 2026 Kugamon LLC. You are free to use, modify, and redistribute any skill in this repository, including commercially, provided the license and copyright notice are preserved.

## About Kugamon

[Kugamon](https://kugamon.com) builds Quote-to-Cash, Subscription, and Digital Sales Room apps native to Salesforce. These skills are extracted from our internal tooling and shared with the Salesforce ISV community.

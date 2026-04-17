# Kugamon Skills

Reusable skills for Salesforce ISVs and consulting partners.

This repository is a monorepo of open-source agent skills authored by [Kugamon LLC](https://kugamon.com). Each skill lives in its own folder under `skills/` with a `SKILL.md`, a `README.md`, and a `CHANGELOG.md`.

## Available Skills

| Skill | What it does |
|---|---|
| [sf-message-catalog](skills/sf-message-catalog) | Generates a customer-facing catalog of every user-facing error, warning, info, and confirmation message in a Salesforce managed package. |
| [sf-message-grammar-audit](skills/sf-message-grammar-audit) | Audits every user-facing message in a Salesforce managed package for grammatical issues and produces a prioritized remediation report. |

## Installation

Each skill is a standalone folder. To use one:

1. Clone this repo (or download the skill folder you want):
   ```bash
   git clone https://github.com/kugamon/skills.git
   ```
2. Copy the skill folder into your local skills directory:
   ```bash
   cp -R skills/sf-message-catalog ~/.claude/skills/
   ```
3. Restart your agent client (Claude Code, Claude Desktop / Cowork, etc.) so the new skill is picked up.

Some skills have additional dependencies — see the individual skill's `README.md` for requirements.

## Repository Layout

```
skills/                  <-- one folder per skill
  sf-message-catalog/
    SKILL.md             <-- skill definition (frontmatter + instructions)
    README.md            <-- human-facing description
    CHANGELOG.md         <-- version history
  sf-message-grammar-audit/
    ...
LICENSE                  <-- MIT, applies to the whole monorepo
CONTRIBUTING.md
README.md                <-- this file
```

## Contributing

Pull requests welcome. See [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines — including the SKILL.md frontmatter contract, testing conventions, and how to propose a new skill.

## License

[MIT](LICENSE) — © 2026 Kugamon LLC. You are free to use, modify, and redistribute any skill in this repository, including commercially, provided the license and copyright notice are preserved.

## About Kugamon

[Kugamon](https://kugamon.com) builds Quote-to-Cash, Subscription, and Digital Sales Room apps native to Salesforce. These skills are extracted from our internal tooling and shared with the Salesforce ISV community.

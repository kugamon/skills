# Contributing to kugamon/skills

Thank you for considering a contribution. This repo is a home for reusable, well-documented agent skills focused on (but not limited to) the Salesforce ecosystem.

## How to Propose a New Skill

1. **Open an issue first** describing the skill, the problem it solves, the inputs it needs, and the outputs it produces. This lets us discuss scope before you invest time.
2. **Wait for feedback** — we'll label the issue `accepted` once we agree it's a fit.
3. **Fork and submit a pull request** with the new skill folder.

## Skill Folder Structure

Every skill lives in `plugins/kugamon-skills/skills/<skill-name>/` and contains at minimum:

| File | Required | Purpose |
|---|---|---|
| `SKILL.md` | ✅ | The skill definition. Must start with YAML frontmatter containing `name` and `description`. |
| `README.md` | ✅ | Human-facing description of what the skill does, who it's for, requirements, and usage. |
| `CHANGELOG.md` | ✅ | Semantic-versioned list of changes (start with `1.0.0`). |
| `scripts/` | optional | Supporting scripts (Python, shell, Node) referenced by SKILL.md. |
| `templates/` | optional | Template files (docx, xlsx, markdown, etc.) used by the skill. |

## SKILL.md Frontmatter Contract

Every `SKILL.md` must begin with YAML frontmatter:

```yaml
---
name: my-skill-name
description: "A single paragraph describing what this skill does, when to use it, and the trigger phrases (e.g., 'generate a message catalog', 'audit messages for grammar'). Keep this description rich and trigger-heavy — it's how agents decide whether to invoke the skill."
---
```

**Rules:**

- `name` must match the folder name exactly and must be lowercase with hyphens (`kebab-case`).
- `description` should be one paragraph, 400–1200 characters. List the scenarios and phrases that should trigger the skill.
- Keep the body of the SKILL.md actionable — phase-by-phase instructions the agent can follow.

## Style Guidelines

- **Write for an agent, not a human.** Use imperative instructions ("Query the Tooling API...", "Save the output as...").
- **Be explicit about prerequisites.** If your skill needs a specific MCP connector or npm package, say so up front.
- **Prefer vendor-neutral language.** These skills are distributed via the Claude ecosystem today but should work with any compatible agent. Avoid Claude-specific phrasing where possible.
- **Keep examples real.** Sample outputs should reflect what the skill actually produces.
- **Use Markdown tables** for pattern/severity reference lists — they render well in SKILL.md and are easy to scan.

## Testing Your Skill

Before opening a PR:

1. Copy your skill folder into `~/.claude/skills/` locally.
2. Restart your agent client.
3. Run the skill end-to-end against a realistic input.
4. Verify the output matches what your README claims.
5. Fix any issues and iterate.

## Pull Request Checklist

- [ ] Folder name and `name:` frontmatter match.
- [ ] `README.md` explains requirements, usage, and example output.
- [ ] `CHANGELOG.md` starts at `1.0.0` with a dated entry.
- [ ] The top-level `README.md` has a new row in the "Available Skills" table.
- [ ] No secrets, customer data, or PII committed.
- [ ] From inside a skill `README.md`, use `../../LICENSE` to link to the license. This resolves to the plugin-local `LICENSE` at `plugins/kugamon-skills/LICENSE` (a duplicate of the repo-root `LICENSE`), which is necessary because plugins are copied to a cache directory at install time and can't reference files outside their root.

## Licensing

All contributions are licensed under MIT (see [LICENSE](LICENSE) at the repo root). By opening a pull request, you agree your contribution is licensed the same way.

## Code of Conduct

Be kind, be precise, assume good intent. Harassment, discrimination, and spam are grounds for removal.

## Questions

For questions about the repo or a specific skill, open an issue or email **support@kugamon.com**.

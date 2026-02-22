# Skills

A collection of skills for AI agents (Claude Code compatible). Each skill gives an agent deep, domain-specific knowledge and a structured workflow for a particular problem space.

## Available Skills

| Skill | Description |
|-------|-------------|
| [`amazon-sp-api`](./amazon-sp-api/) | Expert guidance for building with the Amazon Selling Partner API — orders, inventory, listings, fulfillment, reports, finances, and all 51+ SP-API domains |

## Usage

Copy a skill directory into your project's `.claude/skills/` folder (or wherever your agent loads skills from), then reference it in your agent configuration.

Each skill is self-contained:
- `SKILL.md` — the skill prompt loaded by the agent
- `references/` — supporting reference docs the skill reads at runtime

## Contributing

To add a new skill:
1. Create a directory named after the domain (e.g., `stripe-api/`)
2. Add a `SKILL.md` with a YAML frontmatter `name` and `description`, followed by the skill instructions
3. Add any reference files under `references/`

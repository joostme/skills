# Skills

A collection of agent skills for interacting with self-hosted services.

## Installation

Requires [skills CLI](https://github.com/vercel-labs/skills).

```bash
# Install all skills
npx skills add joostme/skills --all

# Install a specific skill
npx skills add joostme/skills --skill komodo
npx skills add joostme/skills --skill tandoor

# Install globally (available across all projects)
npx skills add joostme/skills --skill komodo -g

# List available skills
npx skills add joostme/skills --list
```

## Skills

| Skill | Description |
| --- | --- |
| [komodo](komodo/SKILL.md) | Manage a Komodo Core instance — inspect and control stacks, deployments, builds, repos, servers, syncs, and more through its JSON API. |
| [tandoor](tandoor/SKILL.md) | Interact with a Tandoor Recipes instance — manage recipes, meal plans, shopping lists, foods, keywords, and units through its REST API. |

---
name: changesets
description: Create changesets for the current project. Use this skill whenever you need to add a changeset, prepare a release, write a changelog entry, or commit a change that needs a changeset in the current repo. This includes any time the user says "add a changeset", "create a changeset", "prepare for release", or when committing a feature/fix/breaking change that should be tracked.
---

## What this skill does

This project uses [Changesets](https://github.com/changesets/changesets) to track changes and automate versioning/changelog generation. Every user-facing change needs a changeset file before it can be released.

## How to create a changeset

Create a markdown file in the `.changeset/` directory at the repo root. The filename should be descriptive kebab-case (e.g., `mise-auto-update.md`, `embed-web-ui-binary.md`, `bugfix-shell-env.md`).

### File format

The file has YAML frontmatter specifying the package name and semver bump level, followed by a markdown description of the change:

```markdown
---
"my-project": patch
---

Short summary of the change on the first line.

Optional longer description with more detail about what changed and why.
Markdown formatting is supported — bullet lists, bold, code blocks, etc.
```

### Bump levels

- **`patch`** — Bug fixes, minor improvements, dependency updates, config tweaks
- **`minor`** — New features, new capabilities, non-breaking enhancements
- **`major`** — Breaking changes that require users to update their setup (e.g., changed mount paths, renamed env vars)

### Important details

- The package name in the frontmatter is always `"my-project"` (with quotes).
- The `.changeset/config.json` file and `.changeset/README.md` are permanent fixtures — don't modify or delete them.
- Only one changeset file per PR/commit is typical, but multiple are allowed if a single PR includes several distinct changes.
- The description becomes the CHANGELOG.md entry verbatim, so write it as you'd want it to appear in the changelog — clear, concise, and useful to someone reading release notes.

### Example: patch

```markdown
---
"my-project": patch
---

Auto-update mise to the latest version on container startup.

The entrypoint now runs `mise self-update --yes` before installing user-defined
toolchains, so the container always picks up the latest mise release without
needing an image rebuild.
```

### Example: minor

```markdown
---
"my-project": minor
---

Build OpenCode from source with the web UI embedded directly into the binary.

The Dockerfile now includes a multi-stage build that patches OpenCode to compile
the upstream web UI assets into the binary at build time. This removes the
runtime dependency on `app.opencode.ai` for serving the web interface.

Additional improvements:

- **Multi-platform build support**: Uses Docker's `TARGETARCH` for arm64 builds.
- **Renovate auto-updates**: Added renovate tracking for pinned versions.
```

### Example: major

```markdown
---
"my-project": major
---

Rework the container's persisted directory layout to mount full `~/.config`,
`~/.local/share`, and `~/.agents` paths.

This simplifies configuration but requires existing installs to migrate their
mounted data and config files.
```

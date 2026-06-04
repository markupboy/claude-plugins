# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

This is a **Claude Code plugin marketplace** repository. It contains Claude Code plugins distributed via a marketplace manifest. There is no build system, no compiled code, no tests, and no CI/CD — the repository is entirely Markdown skills and JSON configuration.

## Repository Structure

- `.claude-plugin/marketplace.json` — Root marketplace manifest that registers all plugins
- Each plugin is a self-contained directory with:
  - `.claude-plugin/plugin.json` — Plugin metadata (name, version, author)
  - `skills/<skill-name>/SKILL.md` — Skill definitions (YAML frontmatter + Markdown instructions)
  - `README.md` — User-facing documentation

## Plugins

### pr-review-autosave
Wraps `pr-review-toolkit` and auto-saves review output. Single skill:
- `review` — Auto-detects PR number (preferred) or falls back to commit hash for WIP, saves to `review_{PR_NUMBER}.md` or `review_{SHORT_HASH}.md`. Accepts optional filename argument. Saves under `pr_reviews/` in the active git worktree.

### noteplan-import
Saves Claude's long-form markdown research output into NotePlan 3 on macOS as a searchable note. Single skill:
- `import` — Identifies the most recent substantial markdown research output in the conversation, resolves the NotePlan documents directory (iCloud Drive, sandbox container, or legacy fallback), writes the note under `Notes/Research/` (or an overridden folder) with a `#research` tag, and opens it via the `noteplan://` URL scheme. Accepts an optional folder argument.

### vulnerability-review
Triages open GitHub Dependabot alerts and recommends the smallest set of dependency bumps that clear them. Depends on `gh` CLI. Single skill:
- `triage` — Lists open Dependabot alerts, traces transitive alerts back to the actionable parent dependency to bump, assesses exploitability of any residual High/Critical alerts, cites CVE/GHSA links, and flags application code changes implied by the bumps. Accepts an optional filename argument for saving the triage output.

## Conventions

- `gh` CLI is required for all GitHub interactions (not web fetch)
- Skill files use YAML frontmatter: `description` and `argument-hint` are required; `allowed-tools` and `disable-model-invocation` are optional and currently unused by the shipped skills
- Plugin versions follow semver in `plugin.json`
- When modifying a plugin's skills, bump the patch version in that plugin's `.claude-plugin/plugin.json`
- PR review output files are named `review_{PR_NUMBER}.md` (prefer PR number over branch name)

## Adding a Plugin

1. Create a directory: `<plugin-name>/`
2. Add `.claude-plugin/plugin.json` with name, description, version, author
3. Add skills under `<plugin-name>/skills/<skill-name>/SKILL.md`
4. Add a `README.md` for user-facing docs
5. Register the plugin in `.claude-plugin/marketplace.json` under `plugins`

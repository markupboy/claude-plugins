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

### code-review-local
Automated PR code review that launches parallel agents (CLAUDE.md audit, bug detection, git blame analysis, comment compliance), scores issues 0-100 confidence, filters below 80, and writes results to a local `review_{PR_NUMBER}.md` file. Depends on `gh` CLI.

### pr-review-autosave
Wraps `pr-review-toolkit` and auto-saves review output. Single skill:
- `review` — Auto-detects PR number (preferred) or falls back to branch name, saves to `review_{PR_NUMBER}.md`. Accepts optional filename argument.

Saves to a `pr_reviews/` directory if one exists in the project.

## Conventions

- Skill files use YAML frontmatter for `allowed-tools`, `description`, `argument-hint`, and `disable-model-invocation`
- Plugin versions follow semver in `plugin.json`
- Review output files are named `review_{PR_NUMBER}.md` (prefer PR number over branch name)
- Use `gh` CLI for all GitHub interactions, not web fetch

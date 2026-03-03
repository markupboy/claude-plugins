# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

This is a **Claude Code plugin marketplace** repository. It contains Claude Code plugins distributed via a marketplace manifest. There is no build system, no compiled code, no tests, and no CI/CD — the repository is entirely Markdown commands and JSON configuration.

## Repository Structure

- `.claude-plugin/marketplace.json` — Root marketplace manifest that registers all plugins
- Each plugin is a self-contained directory with:
  - `.claude-plugin/plugin.json` — Plugin metadata (name, version, author)
  - `commands/*.md` — Slash command definitions (YAML frontmatter + Markdown instructions)
  - `README.md` — User-facing documentation

## Plugins

### code-review-local
Automated PR code review that launches parallel agents (CLAUDE.md audit, bug detection, git blame analysis, comment compliance), scores issues 0-100 confidence, filters below 80, and writes results to a local `review_{PR_NUMBER}.md` file. Depends on `gh` CLI.

### pr-review-autosave
Wraps `pr-review-toolkit` and auto-saves review output. Two commands:
- `review` — Auto-detects PR number from branch/log, saves to `review_{PR_NUMBER}.md`
- `review-to-file` — Accepts custom filename, ensures `.md` extension

Both commands save to a `pr_reviews/` directory if one exists in the project.

## Conventions

- Command files use YAML frontmatter for `allowed-tools`, `description`, and `disable-model-invocation`
- Plugin versions follow semver in `plugin.json`
- Review output files are named `review_{PR_NUMBER}.md` (prefer PR number over branch name)
- Use `gh` CLI for all GitHub interactions, not web fetch

# PR Review Autosave Plugin

Automatically saves PR review results to markdown files.

## Usage

Basic usage (auto-detects PR number):

```
/pr-review-autosave:review
```

With specific analyzers:

```
/pr-review-autosave:review comment-analyzer security-analyzer
```

Save to a custom filename:

```
/pr-review-autosave:review my_review.md
```

## What it does

1. Runs pr-review-toolkit to review the PR
2. Detects the PR number via `gh pr view`
3. Detects a linked Linear ticket (from the branch name, PR title/body, or recent commit messages) and, when a Linear MCP server is available, fetches the ticket so the review can check the PR against its acceptance criteria
4. Saves output to `review_{PR_NUMBER}.md`
5. Falls back to `review_{branch_name}.md` if no PR exists for the branch

## Linear integration

When a Linear issue key (e.g. `SCR-123`) can be found for the PR, the saved review will include:

- A `**Linear:**` header line linking to the ticket
- A `**Linear ticket alignment:** X/10` score alongside the existing description-accuracy score
- A `Linear ticket coverage` subsection breaking the ticket's acceptance criteria into **Met / Partial / Not met / Out of scope / Scope creep**, with cross-references to the numbered findings

The review uses the Linear ticket — not the PR description — as the source of truth for *what was supposed to be done*, so missing requirements show up as `Important` findings and unrelated changes show up as scope-creep `Suggestion` findings.

If no Linear key is detected, or the Linear MCP server is unavailable or unauthenticated, the skill silently skips the ticket-alignment sections rather than fabricating ticket contents.

## Installation

Enable it in your `~/.claude/settings.json`:

```json
"enabledPlugins": {
  "pr-review-autosave@markupboy-claude": true
}
```

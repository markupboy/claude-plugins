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
3. Saves output to `review_{PR_NUMBER}.md`
4. Falls back to `review_{branch_name}.md` if no PR exists for the branch

## Installation

Enable it in your `~/.claude/settings.json`:

```json
"enabledPlugins": {
  "pr-review-autosave@markupboy-claude": true
}
```

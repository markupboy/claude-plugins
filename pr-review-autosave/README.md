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

Save to specific file:
```
/pr-review-autosave:review-to-file filename=review_custom.md
```

## What it does

1. Runs pr-review-toolkit to review the PR
2. Extracts the PR number from git branch/context
3. Saves output to `review_{PR_NUMBER}.md`
4. Falls back to `review_{timestamp}.md` if PR number not found

## Installation

Enable it in your `~/.claude/settings.json`:

```json
"enabledPlugins": {
  "pr-review-autosave@cts-claude-marketplace": true
}
```

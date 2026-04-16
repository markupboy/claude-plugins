# Claude Plugins

Custom Claude Code plugins (borrowed from Clint Shryock).

## Plugins

- **pr-review-autosave** - PR review that automatically saves results to markdown files with a consistent, versioned format

## Review Output Format

Reviews are saved in a consistent format with sequential issue numbering, per-issue metadata, and version tracking for re-reviews. Example:

```markdown
# PR#42 - Add user authentication

**Branch:** `feat/auth`
**Commit:** `abc1234`
**Reviewed:** 2026-04-14
**Files changed:** 8 (120 insertions, 15 deletions)

## Description

Adds JWT-based authentication middleware and login endpoint.

**Description accuracy:** 8/10

---

### Critical

[#1] **SQL injection in login query** - `auth/login.go:47`
_Introduced: v1 | Status: open_
User input is interpolated directly into the SQL query string.

### Important

[#2] **JWT secret loaded from hardcoded string** - `auth/token.go:12`
_Introduced: v1 | Status: open_
Secret should come from environment config, not source code.

### Suggestions

[#3] **Consider rate limiting on login endpoint** - `auth/routes.go:8`
_Introduced: v1 | Status: open_
No rate limiting on the login route; vulnerable to brute force.
```

Re-reviews (v2+) track issue status across versions -- fixed, open, or dismissed -- so you can see progress over time.

## Installation

Add the marketplace to your Claude Code settings:

```json
"pluginMarketplaces": ["https://github.com/markupboy/claude-plugins"]
```

# Review My Assigned PRs

Fetch all PRs I'm assigned to review on GitHub, then for each one: create a
worktree, spawn a subagent, and run the pr-review-autosave:review skill.

## Steps

### 1. Fetch PRs assigned to me for review

Run this gh CLI command to get all open PRs where I'm a requested reviewer:

```bash
gh search prs --review-requested=@me --state=open --json number,title,url,headRefName,headRepository
```

Parse the JSON output to get: PR number, branch name, and repo owner/name.

### 2. For each PR, set up a worktree and spawn a subagent

For each PR in the list:

**a. Clone the repo if not already local, then create a worktree:**

```bash
# Clone if needed
gh repo clone "$OWNER/$REPO" "/tmp/pr-reviews/$REPO_NAME" -- --no-checkout

# Create a worktree on the PR branch
WORKTREE_PATH="/tmp/pr-reviews/$REPO_NAME-pr-$PR_NUMBER"
git -C "/tmp/pr-reviews/$REPO_NAME" worktree add "$WORKTREE_PATH" "origin/$HEAD_BRANCH"
```

**b. Spawn a subagent for this PR using the Task tool:**

```
Task: Review PR #$PR_NUMBER in $OWNER/$REPO
Working directory: $WORKTREE_PATH
Instructions:
  - The PR branch is already checked out in this worktree
  - PR URL: $PR_URL
  - Run the /pr-review-autosave:review skill on this PR
```

### 3. Cleanup (optional)

After all subagents complete, remove the worktrees:

```bash
git -C "/tmp/pr-reviews/$REPO_NAME" worktree remove "$WORKTREE_PATH" --force
```

## Notes

- Requires `gh` CLI to be authenticated (`gh auth status`)
- Requires the `pr-review-autosave:review` command to exist in each repo's
  `.claude/commands/` or globally in `~/.claude/commands/`
- Subagents run in parallel by default — all PRs are reviewed concurrently
- Add "run one at a time" to the Task instructions if you want sequential reviews
- Scope to a specific org by adding `--owner=myorg` to the gh search command

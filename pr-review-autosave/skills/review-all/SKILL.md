# Review My Assigned PRs

Fetch all PRs I'm assigned to review on GitHub, then for each one: locate the
matching repo checkout in the current directory, create a worktree, spawn a
subagent, and run the pr-review-autosave:review skill.

## Steps

### 1. Fetch PRs assigned to me for review

Run this gh CLI command to get all open PRs where I'm a requested reviewer:

```bash
gh search prs --review-requested=@me --state=open --json number,title,url,headRefName,headRepository
```

Parse the JSON output to get: PR number, branch name, and repo owner/name.

### 2. For each PR, locate the local repo and set up a worktree

For each PR in the list:

**a. Find the matching repo checkout under the current directory:**

Search immediate subdirectories of the current working directory (`$PWD`) for a
git checkout whose `origin` remote matches the PR's repo (either
`$OWNER/$REPO` or just `$REPO`). Do not clone — if no matching local checkout
exists, skip the PR and report it at the end.

```bash
# Find a local checkout matching $OWNER/$REPO under the current directory
LOCAL_REPO=""
for dir in "$PWD"/*/; do
  [ -d "$dir/.git" ] || continue
  remote_url=$(git -C "$dir" remote get-url origin 2>/dev/null) || continue
  case "$remote_url" in
    *"$OWNER/$REPO"*|*":$REPO".git|*"/$REPO".git)
      LOCAL_REPO="${dir%/}"
      break
      ;;
  esac
done

if [ -z "$LOCAL_REPO" ]; then
  echo "SKIP: no local checkout of $OWNER/$REPO found under $PWD"
  continue
fi
```

**b. Fetch the PR branch and create a worktree:**

```bash
# Make sure we have the latest ref for the PR branch
git -C "$LOCAL_REPO" fetch origin "$HEAD_BRANCH"

# Create a worktree on the PR branch, kept alongside the repo
WORKTREE_PATH="$LOCAL_REPO/../.pr-review-worktrees/$(basename "$LOCAL_REPO")-pr-$PR_NUMBER"
mkdir -p "$(dirname "$WORKTREE_PATH")"
git -C "$LOCAL_REPO" worktree add "$WORKTREE_PATH" "origin/$HEAD_BRANCH"
```

**c. Spawn a subagent for this PR using the Task tool:**

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
git -C "$LOCAL_REPO" worktree remove "$WORKTREE_PATH" --force
```

## Notes

- Requires `gh` CLI to be authenticated (`gh auth status`)
- Requires the `pr-review-autosave:review` command to exist in each repo's
  `.claude/commands/` or globally in `~/.claude/commands/`
- Only PRs whose repo is already checked out as a subdirectory of `$PWD` are
  reviewed — others are reported as skipped at the end of the run
- Worktrees are created under `.pr-review-worktrees/` next to each repo so the
  original checkout is left untouched
- Subagents run in parallel by default — all PRs are reviewed concurrently
- Add "run one at a time" to the Task instructions if you want sequential reviews
- Scope to a specific org by adding `--owner=myorg` to the gh search command

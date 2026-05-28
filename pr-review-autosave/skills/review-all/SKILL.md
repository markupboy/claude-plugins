# Review My Assigned PRs

Fetch all PRs I'm assigned to review on GitHub, then for each one: locate the
matching repo checkout in the current directory, create a worktree, spawn a
subagent, and run the pr-review-autosave:review skill.

## Steps

### 1. Fetch PRs assigned to me for review

Run this gh CLI command to get all open PRs where I'm a requested reviewer:

```bash
gh search prs --review-requested=@me --state=open --json number,title,url,repository
```

`gh search prs` does not expose the head branch or head SHA as JSON fields, so
for each PR you also need to fetch them separately. Grab both in a single call:

```bash
gh pr view "$PR_NUMBER" --repo "$OWNER/$REPO" --json headRefName,headRefOid
```

For each PR, capture: PR number, title, URL, repo owner/name (from the
`repository` field's `nameWithOwner`), head branch name (`headRefName`), and
current head commit SHA (`headRefOid`, full 40-char SHA). The head SHA is used
in step 2b to skip PRs that haven't changed since the last review.

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

**b. Skip if the PR is unchanged since the last review:**

The `pr-review-autosave:review` skill records the short HEAD SHA on a
`**Commit:**` line in each saved review file under `$LOCAL_REPO/pr_reviews/`.
If the latest review for this PR was taken at the PR's current head commit,
skip the PR — there's nothing new to review.

```bash
# Look for the most recent review file for this PR (handles versioned
# re-reviews like review_123_v2.md by sorting -V so v10 > v9 > v2 > v1).
LATEST_REVIEW=$(ls -1 "$LOCAL_REPO/pr_reviews/review_${PR_NUMBER}".md \
                       "$LOCAL_REPO/pr_reviews/review_${PR_NUMBER}_v"*.md \
                       2>/dev/null | sort -V | tail -n 1)

if [ -n "$LATEST_REVIEW" ]; then
  # Extract the short SHA from the `**Commit:** \`abc1234\`` header line.
  LAST_REVIEWED_SHA=$(grep -m 1 -E '^\*\*Commit:\*\*' "$LATEST_REVIEW" \
                      | sed -E 's/.*`([0-9a-fA-F]+)`.*/\1/')

  # Compare as a prefix match — the review records the short hash, the PR API
  # returns the full 40-char SHA. Lowercase both sides to be safe.
  if [ -n "$LAST_REVIEWED_SHA" ] \
     && [ "${HEAD_SHA:0:${#LAST_REVIEWED_SHA}}" = "$(echo "$LAST_REVIEWED_SHA" | tr '[:upper:]' '[:lower:]')" ]; then
    echo "SKIP: PR #$PR_NUMBER unchanged since last review ($LAST_REVIEWED_SHA in $(basename "$LATEST_REVIEW"))"
    continue
  fi
fi
```

Record skipped-as-unchanged PRs separately from skipped-as-no-local-checkout
PRs so the final summary can distinguish them.

**c. Fetch the PR branch and create a worktree:**

```bash
# Make sure we have the latest ref for the PR branch
git -C "$LOCAL_REPO" fetch origin "$HEAD_BRANCH"

# Create a worktree on the PR branch, kept alongside the repo
WORKTREE_PATH="$LOCAL_REPO/../.pr-review-worktrees/$(basename "$LOCAL_REPO")-pr-$PR_NUMBER"
mkdir -p "$(dirname "$WORKTREE_PATH")"
git -C "$LOCAL_REPO" worktree add "$WORKTREE_PATH" "origin/$HEAD_BRANCH"
```

**d. Spawn a subagent for this PR using the Task tool:**

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
- PRs whose latest existing review (in `$LOCAL_REPO/pr_reviews/`) was taken at
  the PR's current head commit are also skipped automatically. To force a
  re-review of an unchanged PR, delete or rename its review file before running
  this skill.
- Worktrees are created under `.pr-review-worktrees/` next to each repo so the
  original checkout is left untouched
- Subagents run in parallel by default — all PRs are reviewed concurrently
- Add "run one at a time" to the Task instructions if you want sequential reviews
- Scope to a specific org by adding `--owner=myorg` to the gh search command

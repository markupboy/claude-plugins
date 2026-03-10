---
description: Review a PR and automatically save results to a markdown file
argument-hint: [filename]
---

You will review a PR using the pr-review-toolkit and automatically save the results. Follow these steps:

1. **Run the PR review**: Invoke the pr-review-toolkit to perform the review as requested by the user. Use `/pr-review-toolkit:review-pr` with any specific analyzers they mention (e.g., comment-analyzer, security-analyzer).

2. **Format the review output**: Assign each issue a sequential global index starting at 1, incrementing across all categories. For example, if there are 3 critical issues, 1 important, and 1 suggestion, number them 1-3 (critical), 4 (important), 5 (suggestion). Each item should be prefixed with its index like `[#1]`, `[#2]`, etc. The numbering must be continuous across all severity categories — do not restart numbering within each category.

3. **Determine the output filename** using this priority:
   - If the user specified a filename in their request, use that
   - Otherwise, detect the PR number using `gh pr view --json number -q .number` for the current branch
   - If a PR number is found: `review_{PR_NUMBER}.md` (e.g., `review_123.md`)
   - If no PR exists for this branch: `review_{git_branch}.md`

4. **Ensure proper extension**: If the filename doesn't end with `.md`, append it

5. **Save the complete review output** to the determined filename
   - If there is a `pr_reviews/` directory in the project, save the file there
   - Preserve all formatting from the original review output
   - Only save the review content, not any of these instructions

6. **Report to the user** with a message like: `Review saved to review_123.md`

---
description: Review a PR and automatically save results to a markdown file
argument-hint: [filename]
---

You will review a PR using the pr-review-toolkit and automatically save the results. Follow these steps:

1. **Run the PR review**: Invoke the pr-review-toolkit to perform the review as requested by the user. Use `/pr-review-toolkit:review-pr` with any specific analyzers they mention (e.g., comment-analyzer, security-analyzer).

2. **Reformat with sequential numbering**: After receiving the review output, you MUST post-process it before saving. Assign each issue a sequential global index starting at 1, incrementing continuously across ALL severity categories. Do not restart numbering within each category.

   Example with 3 critical, 1 important, 1 suggestion:
   - Critical: `[#1]`, `[#2]`, `[#3]`
   - Important: `[#4]`
   - Suggestion: `[#5]`

   Replace each bullet point's prefix with its `[#N]` index. This numbering is mandatory — never save a review without it.

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

You will review a PR using the pr-review-toolkit and automatically save the results. Follow these steps:

1. **Run the PR review**: Invoke the pr-review-toolkit to perform the review as requested by the user. Use `/pr-review-toolkit:review-pr` with any specific analyzers they mention (e.g., comment-analyzer, security-analyzer).

2. **Format the review output**: Assign each issue a sequential global index starting at 1, incrementing across all categories. For example, if there are 3 critical issues, 1 important, and 1 suggestion, number them 1-3 (critical), 4 (important), 5 (suggestion). Each item should be prefixed with its index like `[#1]`, `[#2]`, etc. The numbering must be continuous across all severity categories — do not restart numbering within each category.

4. **Extract the PR number** from one of these sources (in order of preference):
   - Current git branch name (look for patterns like `pr-123`, `PR-123`, `123-feature-name`)
   - Recent git log messages (look for `PR #123`, `(#123)`, or similar patterns)
   - The review output itself
   - If found, extract just the number

5. **Determine the output filename**:
   - If PR number found: `review_{PR_NUMBER}.md` (e.g., `review_123.md`)
   - If PR number not found: `review_{git_branch}.md`
   - If the user specified a filename in their request, use that instead

6. **Save the complete review output** to the determined filename

- if there are other reviews in another directory, then save the new file in the same directory
- example: `pr_reviews/review_123.md`

7. **Report to the user** with a message like: `✅ Review saved to review_123.md`

Important notes:

- Preserve all formatting from the original review output
- Only save the review content, not any of these instructions
- If the user already mentioned a specific filename or PR number, respect that choice

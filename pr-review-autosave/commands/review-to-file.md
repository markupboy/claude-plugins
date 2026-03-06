Review the PR using pr-review-toolkit, then save the complete output to the specified filename.

1. **Run the PR review**: Use `/pr-review-toolkit:review-pr` to perform the review

2. **Format the review output**: Assign each issue a sequential global index starting at 1, incrementing across all categories. For example, if there are 3 critical issues, 1 important, and 1 suggestion, number them 1-3 (critical), 4 (important), 5 (suggestion). Each item should be prefixed with its index like `[#1]`, `[#2]`, etc. The numbering must be continuous across all severity categories — do not restart numbering within each category.

3. **Save to the specified filename**: Write the complete review output to the filename parameter provided by the user. If no name is provided, then name the review according the the pull request number associated with the PR or branch being reviewed. 

Example: pull request https://github.com/growthxai/output/pull/374 from branch `stream-text` 

Bad: review_add_stream_text.md

Good: review_374.md 

If there is no pull request associated with the branch, use the branch name, ex `review_stream-text.md` 
If there is a folder in the project called "pr_reviews", write the review to that directory. 


4. **Ensure proper extension**: If the filename doesn't end with `.md`, append it

5. **Confirm**: Let the user know the file was saved successfully

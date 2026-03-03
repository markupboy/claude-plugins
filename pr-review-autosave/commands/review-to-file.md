Review the PR using pr-review-toolkit, then save the complete output to the specified filename.

1. **Run the PR review**: Use `/pr-review-toolkit:review-pr` to perform the review

2. **Save to the specified filename**: Write the complete review output to the filename parameter provided by the user. If no name is provided, then name the review according the the pull request number associated with the PR or branch being reviewed. 

Example: pull request https://github.com/growthxai/output/pull/374 from branch `stream-text` 

Bad: review_add_stream_text.md

Good: review_374.md 

If there is no pull request associated with the branch, use the branch name, ex `review_stream-text.md` 
If there is a folder in the project called "pr_reviews", write the review to that directory. 


3. **Ensure proper extension**: If the filename doesn't end with `.md`, append it

4. **Confirm**: Let the user know the file was saved successfully

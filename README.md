# workflow-review

Reusable GitHub Actions workflows for running an OpenAI Codex-powered pull-request review on demand.

## Usage
- Add a repository secret named `OPENAI_API_KEY` that contains the key for the OpenAI account you want to bill.
- Create `.github/workflows/review.yaml` in your repository with the contents below.
- Replace `<ref>` with a semantic version tag or commit SHA from this repository and update the `allowlist` with the GitHub usernames allowed to trigger the review.

```yaml
name: Perform a code review

on:
  issue_comment:
    types: [created]

permissions:
	pull-requests: write
	issues: write
	checks: read
	contents: read

jobs:
  review:
    uses: rasmus/workflow-review/.github/workflows/pipeline.yaml@v<ref>
      with:
        pull_request_number: ${{ github.event.issue.number }}
        allowlist: "octocat"
        allow_drafts: "true"
        skip_ci: "true"
      secrets:
        openai_api_key: ${{ secrets.OPENAI_API_KEY }}
```

## Triggering a review
- On a pull request, add a new comment containing only `.review`.
- The commenter must appear in the configured `allowlist`; otherwise the workflow exits immediately.
- When accepted, the workflow reacts to the comment with 👀 and posts the Codex feedback as a pull-request comment when finished.

## Optional inputs
- `allow_drafts`: Allow reviews on draft pull requests (`"false"` by default).
- `allow_forks`: Allow commands from forked repositories (`"false"` by default).
- `allowlist`: Comma-separated usernames permitted to trigger `.review` (`"rasmus"` by default).
- `skip_ci`: Forwarded to `github/command` to skip workflows when `skip-ci` is present (`"false"` by default).

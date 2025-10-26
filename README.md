# workflow-review

Reusable GitHub Actions workflows that let maintainers trigger an OpenAI Codex analysis on demand.

## Workflows at a glance
- `review`: Performs a pull-request code review when someone comments `.review` on the PR.
- `analyze`: Provides deep analysis on a GitHub issue when someone comments `.analyze` on the issue.

Both workflows share the same prerequisites and authentication model, so once the secret is in place you can opt into either command independently.

## Prerequisites
- Add a repository secret named `OPENAI_API_KEY` that contains the key for the OpenAI account you want to bill.
- Decide on a version of this repository to consume (`v1`, `v1.0.0`, or a commit SHA) and replace `<ref>` in the snippets below with that reference.

## Pull-request reviews (`review`)
Create `.github/workflows/review.yaml` in your repository:

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
    uses: rasmus/workflow-review/.github/workflows/review.yaml@v<ref>
    with:
      pull_request_number: ${{ github.event.issue.number }}
      allowlist: "octocat"
    secrets:
      openai_api_key: ${{ secrets.OPENAI_API_KEY }}
```

Trigger the workflow by leaving a new pull-request comment that contains only `.review`. The commenter must be part of the configured `allowlist`; otherwise the workflow exits immediately. When the workflow proceeds it reacts with 👀 and posts the Codex findings back to the pull request once the analysis finishes.

### Inputs
| Input | Default | Description |
| --- | --- | --- |
| `pull_request_number` | (required) | The PR number to review, typically `${{ github.event.issue.number }}` when triggered from a comment. |
| `allowlist` | `"rasmus"` | Comma-separated GitHub usernames allowed to run `.review`. |
| `allow_drafts` | `"false"` | Set to `"true"` to allow draft pull requests to be reviewed. |
| `allow_forks` | `"false"` | Permit commands from forked repositories. |
| `model` | `"gpt-5-codex"` | OpenAI model passed to `openai/codex-action`. |
| `sandbox` | `"danger-full-access"` | Codex sandbox profile to use during execution. |
| `skip_ci` | `"false"` | Forwarded to `github/command`'s `skip_ci` input. |

### Outputs
- `final_message`: Returned by the `codex` job, contains the formatted review posted back to the pull request.

## Issue analysis (`analyze`)
Create `.github/workflows/analyze.yaml` in your repository:

```yaml
name: Analyze an issue

on:
  issue_comment:
    types: [created]

permissions:
  issues: write
  checks: read
  contents: read

jobs:
  analyze:
    uses: rasmus/workflow-review/.github/workflows/analyze.yaml@v<ref>
    with:
      issue_number: ${{ github.event.issue.number }}
      allowlist: "octocat"
    secrets:
      openai_api_key: ${{ secrets.OPENAI_API_KEY }}
```

Trigger the workflow by leaving a new issue comment containing only `.analyze`. Approved commenters receive a Codex-authored response with guidance, reproduction steps, or implementation ideas depending on the issue type.

### Inputs
| Input | Default | Description |
| --- | --- | --- |
| `issue_number` | (required) | The issue to analyze, usually `${{ github.event.issue.number }}`. |
| `allowlist` | `"rasmus"` | Comma-separated GitHub usernames allowed to run `.analyze`. |
| `model` | `"gpt-5-codex"` | OpenAI model passed to `openai/codex-action`. |
| `sandbox` | `"danger-full-access"` | Codex sandbox profile to use during execution. |

### Outputs
- `final_message`: Returned by the `codex` job, contains the formatted analysis posted back to the issue.

## Tips
- Always keep the reusable workflow reference pinned to a specific tag or commit to avoid unexpected changes.
- Adjust the `allowlist` inputs to match the trusted maintainers in your repository.
- Use workflow-level `concurrency` or branch protections in your consumer repository if you want to limit simultaneous reviews or analyses.

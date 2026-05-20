# claude-code-review-config

Reusable GitHub Actions workflows for AI-powered code review.

## Workflows

### `claude-review.yml` — Claude Code Review

Runs a multi-turn code review using Claude via the `claude-code-action`.

**Inputs:**

| Input | Type | Default | Description |
|-------|------|---------|-------------|
| `no-nit` | boolean | `true` | Suppress style/naming/minor suggestions |
| `trigger-method` | string | `'auto'` | `'auto'` fires on every PR push; `'comment'` fires on `/claude-review` comment |

**Secrets:** `CLAUDE_CODE_OAUTH_TOKEN` (required)

#### Trigger modes

**Auto (default)** — fires on every non-draft PR push. No `trigger-method` input needed:

```yaml
# .github/workflows/code-review.yml
name: Code Review
on:
  pull_request:
    types: [opened, synchronize, ready_for_review, reopened]

jobs:
  claude-review:
    uses: xukai92/claude-code-review-config/.github/workflows/claude-review.yml@main
    secrets:
      CLAUDE_CODE_OAUTH_TOKEN: ${{ secrets.CLAUDE_CODE_OAUTH_TOKEN }}
```

**Comment** — fires only when an authorized collaborator comments `/claude-review` on a PR. The consumer wrapper must subscribe to both `pull_request` and `issue_comment` events so the workflow_call has a chance to fire on both:

```yaml
# .github/workflows/code-review.yml
name: Code Review
on:
  pull_request:
    types: [opened, synchronize, ready_for_review, reopened]
  issue_comment:
    types: [created]

jobs:
  claude-review:
    uses: xukai92/claude-code-review-config/.github/workflows/claude-review.yml@main
    with:
      trigger-method: comment
    secrets:
      CLAUDE_CODE_OAUTH_TOKEN: ${{ secrets.CLAUDE_CODE_OAUTH_TOKEN }}
```

**Notes:**
- The `author_association` gate (`OWNER`, `MEMBER`, `COLLABORATOR`) prevents drive-by commenters from triggering paid LLM calls.
- In comment mode, `pull_request` events are silently skipped by the job-level `if:` — the wrapper subscription is needed so the parent workflow fires on `issue_comment`, not to run auto reviews.
- The two modes are mutually exclusive per job. For a manual re-trigger in auto mode, use the GitHub Actions "Re-run" button.

---

### `cursor-review.yml` — Cursor Code Review

Runs a three-step (context → review → validate+post) code review using the Cursor agent CLI.

**Inputs:**

| Input | Type | Default | Description |
|-------|------|---------|-------------|
| `no-nit` | boolean | `true` | Suppress style/naming/minor suggestions |
| `expert-mode` | boolean | `false` | Use top-tier Cursor models instead of cost-effective defaults |
| `trigger-method` | string | `'auto'` | `'auto'` fires on every PR push; `'comment'` fires on `/cursor-review` comment |

**Secrets:** `CURSOR_API_KEY` (required)

#### Trigger modes

**Auto (default)** — fires on every non-draft PR push:

```yaml
# .github/workflows/code-review.yml
name: Code Review
on:
  pull_request:
    types: [opened, synchronize, ready_for_review, reopened]

jobs:
  cursor-review:
    uses: xukai92/claude-code-review-config/.github/workflows/cursor-review.yml@main
    secrets:
      CURSOR_API_KEY: ${{ secrets.CURSOR_API_KEY }}
```

**Comment** — fires only when an authorized collaborator comments `/cursor-review` on a PR:

```yaml
# .github/workflows/code-review.yml
name: Code Review
on:
  pull_request:
    types: [opened, synchronize, ready_for_review, reopened]
  issue_comment:
    types: [created]

jobs:
  cursor-review:
    uses: xukai92/claude-code-review-config/.github/workflows/cursor-review.yml@main
    with:
      trigger-method: comment
    secrets:
      CURSOR_API_KEY: ${{ secrets.CURSOR_API_KEY }}
```

**Notes:**
- The `author_association` gate (`OWNER`, `MEMBER`, `COLLABORATOR`) prevents drive-by commenters from triggering paid API calls.
- In comment mode, the workflow reacts to the triggering comment with 👀 to confirm the run was picked up.
- In comment mode, `pull_request` events are silently skipped by the job-level `if:`.
- The two modes are mutually exclusive per job.

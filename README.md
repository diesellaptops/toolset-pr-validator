# toolset-pr-validator

A reusable GitHub Actions workflow that fails a pull request whose description still contains
unfilled pull request template text. It is public so that repositories in the Diesel Laptops and
Preteckt organizations, public or private, can all call it.

## What it checks

The workflow reads the PR description and fails when:

1. The description is empty.
2. The **Summary** placeholder is still there.
3. The **Linked issue / ticket** placeholder is still there.
4. A **Critical Changes** box is checked but the **Required Test Coverage** table still has placeholder values.
5. The **Deployment Notes** placeholder is still there (write "N/A" if nothing applies).

On failure it posts one comment listing what is missing and updates that comment on every
re-run. Once the description passes, the comment is deleted.

The checks match the placeholder text of the Diesel Laptops pull request template. A repository
with a different template passes any non-empty description.

## Using it

Add a caller workflow to the repository, for example `.github/workflows/validate-pr.yml`:

```yaml
name: Validate PR description
on:
  pull_request:
    types: [opened, edited, reopened, synchronize]
permissions:
  pull-requests: write
jobs:
  validate:
    uses: diesellaptops/toolset-pr-validator/.github/workflows/validate-pr.yml@<full-commit-sha>
```

- **Pin to a full commit SHA**, not `@main`. A pinned caller keeps running the version it
  reviewed until someone updates the SHA on purpose.
- **Permissions:** the job only needs `pull-requests: write` (to post and delete its comment).
  Do not grant more.
- **Trigger:** keep `pull_request`. Do not switch to `pull_request_target`.

To make the check required, add it as a required status check (or a required workflow) in the
calling repository's ruleset.

## Changing it

Every change to `validate-pr.yml` affects every repository that calls it. Open a pull request.
Callers pinned to a SHA pick up the change only when they update their pin.

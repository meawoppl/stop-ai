# stop-ai

GitHub composite action that fails a PR check when the title or body
contains AI-attribution strings (Claude co-author trailers, "generated
with" / "created with" phrasings, etc.). One central home for the
attribution policy so consuming repos don't each maintain their own copy.

## Usage

Add a workflow to the consuming repo at `.github/workflows/pr-check.yml`:

```yaml
name: PR Check

on:
  pull_request:
    types: [opened, edited, synchronize]

jobs:
  check-attribution:
    runs-on: ubuntu-latest
    steps:
      - uses: meawoppl/stop-ai@main
```

Pin to a tag (`@v1`) instead of `@main` once stable, so a drive-by
commit to this repo can't break every downstream PR check at once.

## What it blocks

Case-insensitive regex against `title ||| body`:

```
(claude\.com|@claude|co-authored-by.*claude|generated.*with.*claude|created.*with.*claude)
```

## Updating the policy

Edit `action.yml`, tag a release, bump the `@ref` in each consuming repo.

# stop-ai

GitHub composite action that fails a PR check when AI-attribution
strings appear in the PR title, body, branch name, or any commit
message on the PR. Covers the major assistants (Claude, Copilot,
ChatGPT, Cursor, Gemini, Codex) and common forms: co-author trailers,
bot @mentions, vendor domains, and "generated/created with X"
phrasings. One central home for the attribution policy so consuming
repos don't each maintain their own copy.

## Why

AI coding tools are powerful, and we use them. This action is not an
anti-AI policy — it's a responsibility policy.

Co-authorship is a claim about accountability, not keystrokes. A
co-author is someone who can stand behind the work: explain why a
change was made, defend the tradeoffs in review, fix it when it
breaks at 2am, and carry the design forward as the codebase evolves.
An LLM cannot do any of those things. It cannot be paged, it cannot
be interviewed about its intent six months later, and it cannot own
the consequences of a decision it "made."

When AI-generated text is committed under a human's name with no
machine co-author trailer, that human is asserting: *I read this, I
understand it, I vouch for it, and I am the one responsible if it is
wrong.* That is the contract we want every change in a consuming repo
to carry. Adding `Co-Authored-By: <an-llm>` dilutes that contract by
spreading responsibility onto an entity that cannot hold any.

So: use the tools, use them aggressively, but own the output. If you
wouldn't defend a line in review, don't commit it — regardless of who
or what typed it.

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

## Inputs

| Name | Default | Purpose |
| --- | --- | --- |
| `rejection-message` | `This repository does not include AI attribution in PRs.` | Message printed (and posted as a comment, if enabled) when a match is found. |
| `extra-patterns` | `""` | Newline-separated list of extra regex alternatives to OR onto the built-in pattern. Each non-empty line is treated as one alternative. |
| `comment-on-failure` | `"false"` | When `"true"`, post the rejection message as a sticky PR comment (deduplicated via an HTML marker) in addition to failing the check. |

Example with every input:

```yaml
permissions:
  contents: read
  pull-requests: write  # only needed when comment-on-failure: true

jobs:
  check-attribution:
    runs-on: ubuntu-latest
    steps:
      - uses: meawoppl/stop-ai@v1
        with:
          rejection-message: |
            Please remove AI co-author trailers before merging.
            See CONTRIBUTING.md for the attribution policy.
          extra-patterns: |
            internal-ai-bot@example\.com
            @acme-ai
          comment-on-failure: "true"
```

## What it checks

Four surfaces, all case-insensitive:

1. **PR title and body** — matched against the main pattern.
2. **Branch name** (`pull_request.head.ref`) — matched against the main pattern plus a tighter branch-name pattern that catches segments like `claude/…`, `copilot/…`, `cursor/…`.
3. **Every commit message on the PR** — fetched via `gh api repos/{owner}/{repo}/pulls/{n}/commits`. Catches `Co-Authored-By:` trailers that never appear in the PR body.
4. **Custom `extra-patterns`** — OR'd onto the main pattern.

The built-in pattern covers vendor domains (`claude.com`, `anthropic.com`, `openai.com`, `chatgpt.com`, `cursor.sh`, `cursor.com`, `codeium.com`, `tabnine.com`, `gemini.google`), bot `@mentions` (`@claude`, `@copilot`, `@chatgpt`, `@openai`, `@cursor`, `@gemini`, `@codex`), co-author trailers naming any of those, "generated/created/written/authored with/by X" phrasings, product names (`claude-code`, `github-copilot`), and known AI noreply addresses (`noreply@anthropic.com`, `noreply@openai.com`).

## Permissions

The default `GITHUB_TOKEN` has enough access to read PR commits. If you enable `comment-on-failure`, add `pull-requests: write` to the workflow's `permissions:` block.

## Updating the policy

Edit `action.yml`, tag a release (`v1.x.y`), re-point the moving `v1` tag, and consuming repos tracking `@v1` pick up the change on the next run.

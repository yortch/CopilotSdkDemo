# Session Plan: GitHub Copilot CLI + GitHub Actions

**Audience:** Team evaluating overlap between an external SDLC platform and the GitHub CLI/SDK.
**Duration:** ~60 minutes (30 min demo, 20 min Q&A, 10 min buffer).
**Goal:** Show that common SDLC hooks (LLMs, issue trackers, CI/CD, code scanning, Azure DevOps bridges) can be composed on GitHub using **Copilot CLI** + **GitHub Actions** + the **`gh` CLI**.

---

## 1. Framing (5 min)

Map common SDLC platform capabilities -> GitHub equivalents:

| SDLC capability              | GitHub equivalent                                            |
| ---------------------------- | ------------------------------------------------------------ |
| LLM integrations             | Copilot CLI (`copilot`) + Copilot SDK (.NET / Python)        |
| Issue tracker integration    | `gh issue` + `GITHUB_TOKEN` in Actions                       |
| CI/CD pipelines              | GitHub Actions                                               |
| Code scanning                | CodeQL + `gh code-scanning` + Copilot CLI for triage         |
| Azure DevOps bridge          | `azure/login` + `az devops` CLI in Actions                   |
| Agentic workflows            | Copilot CLI in headless mode + `--allow-tool` permissions    |

Key talking point: **Copilot CLI runs headless inside Actions**, so every workflow step can be "LLM-augmented" without writing a custom integration.

---

## 2. Prereqs to show on screen (2 min)

- `permissions:` block in workflows is least-privilege
- `copilot --version` in a local terminal to show the same binary CI will use
- Repo secret `COPILOT_CLI_TOKEN` configured (see below)

### Creating the `COPILOT_CLI_TOKEN` secret

The workflow authenticates the Copilot CLI via the `COPILOT_GITHUB_TOKEN` env var, sourced from a repo secret named `COPILOT_CLI_TOKEN`.

**1. Create a fine-grained PAT** at https://github.com/settings/personal-access-tokens/new

- **Resource owner:** your org or user (e.g. `yortch`)
- **Repository access:** *Only select repositories* → this repo
- **Repository permissions:**
  - Contents: Read
  - Issues: Read and write
  - Pull requests: Read and write
  - Metadata: Read (auto)
- **Account permissions:**
  - Copilot Chat / Copilot Requests: Read

> Classic PAT alternative: scopes `repo`, `workflow` (and Copilot access on the account).

**2. Store it as a repo secret** (paste at the prompt):

```bash
gh secret set COPILOT_CLI_TOKEN --repo <owner>/<repo>
```

Non-interactive variant:

```bash
echo -n "github_pat_xxx" | gh secret set COPILOT_CLI_TOKEN --repo <owner>/<repo>
```

**3. Verify:**

```bash
gh secret list --repo <owner>/<repo>
# NAME               UPDATED
# COPILOT_CLI_TOKEN  <timestamp>
```

**4. Smoke test** (no PR required):

```bash
gh workflow run "Copilot CLI Demo" --repo <owner>/<repo> -f scenario=review
gh run watch $(gh run list --repo <owner>/<repo> -L 1 --json databaseId -q '.[0].databaseId') --repo <owner>/<repo>
```

> `GITHUB_TOKEN` is auto-provided by Actions — no setup needed for `gh issue comment` / `gh pr comment` / `gh api` calls.

---

## 3. Demo flow (30 min)

Four short scenarios, each ~6-8 min. Each one maps to a common SDLC platform capability.

### Scenario A — Issue triage bot (LLM + issue tracker)

- Trigger: `issues: [opened]`
- Copilot CLI reads the issue body, classifies (`bug|feature|question`), proposes labels, posts a summary comment via `gh issue comment`.
- Shows: LLM-driven triage without a separate SaaS.

### Scenario B — PR review assistant (LLM + CI)

- Trigger: `pull_request: [opened, synchronize]`
- Workflow checks out the PR, runs `copilot -p "Review this diff for bugs, security, style. Output a markdown review."` against `git diff origin/main...HEAD`.
- Posts result as a PR review comment.
- Shows: pre-human-review gating, similar to an LLM review step in a third-party platform.

### Scenario C — CodeQL finding explainer (code scanning)

- Trigger: `workflow_run` after CodeQL completes, or manual `workflow_dispatch`.
- Pull open alerts with `gh api /repos/:owner/:repo/code-scanning/alerts`.
- Pipe each alert JSON to Copilot CLI: "Explain this finding in plain English and propose a fix."
- Post the explanation back as an issue or comment.
- Shows: same value prop as an LLM layer on top of code scanning.

### Scenario D — Release notes + Azure DevOps work-item sync (CI/CD + ADO bridge)

- Trigger: `release: [published]` or tag push.
- Copilot CLI summarizes commits between tags.
- `az boards work-item update` (via `azure/login`) pushes the summary to a matching ADO work item.
- Shows: GitHub -> ADO crossover without leaving Actions.

Use `demo/.github/workflows/copilot-demo.yml` as the canonical example for Scenario B, then just describe A/C/D verbally with snippets if time runs short.

---

## 4. Comparison slide (3 min)

Position this not as "GitHub replaces your existing platform", but as:

- **Overlap:** LLM-in-pipeline, issue enrichment, PR review, scan triage.
- **Unique to GitHub path:** tight coupling with `GITHUB_TOKEN`, zero extra infra, Copilot billing already in place.
- **Where a dedicated platform may still win:** cross-tool orchestration across non-GitHub systems, central policy UI, multi-tenant governance.

---

## 5. Q&A prompts to seed (5 min)

- "How do you keep prompts versioned?" -> prompts live in repo as files, diffable in PRs.
- "How do you control cost?" -> Copilot CLI uses existing Copilot entitlement; no per-call billing surprise.
- "How do you prevent prompt injection from issue bodies?" -> `--allow-tool` allowlist + strip/sandbox inputs + don't give the bot write scopes it doesn't need.
- "Can this run on self-hosted runners for regulated code?" -> yes, Copilot CLI works on any runner with network egress to the API.

---

## 6. Leave-behinds

- This repo (`CopilotSdkDemo`) — shows the SDK side in C# and Python.
- `demo/.github/workflows/copilot-demo.yml` — working Actions example.
- Links:
  - Copilot CLI docs: https://docs.github.com/copilot/github-copilot-in-the-cli
  - Copilot SDK: https://github.com/github/copilot-sdk
  - `gh` CLI manual: https://cli.github.com/manual/

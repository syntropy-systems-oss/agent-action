# Syntropy Agent action

Run your org's Syntropy agent on a branch of your repository, from your own
GitHub Actions workflows. **This repository is the interface, not the
implementation** — one composite action, no runtime, no secrets, nothing to
configure beyond a permissions block.

## The model

Your agent is the **memory**: the skills and judgment your org's worker has
accumulated. Everything you pass this action is a **frame** that memory is
placed into — a merge-conflict frame, a fix-red-CI frame, a review frame.
New use cases are new wrapper workflows around this one step, never new
agents and never new integrations. Your wrapper YAML owns the business
logic — *when* the agent is called and *why*; Syntropy only ever sees the
frame.

The corollary you should design for: **clean paths never call Syntropy at
all.** A branch that merges cleanly or a CI run that passes should cost you
nothing and involve no agent — the agent is an immune response, not a
resident. Wrapper workflows for the common frames are provided during
onboarding.

## Identity — no secrets, ever

- **Calling**: the action authenticates with your workflow run's own OIDC
  token (`id-token: write`). GitHub itself attests which repository and
  workflow is calling; nothing is stored.
- **Acting**: the agent works your branch through the **Syntropy GitHub
  App installation** on your repository. Its commits and comments carry the
  App's identity, so your CI triggers on its pushes like anyone else's.
- **Status**: the agent's reply lands as an App comment on the thread you
  name in `comment-on` — native GitHub UI, and what this action watches
  when `wait` is on.

Setup is the App install (one click, done once per repository) plus copying
a wrapper workflow. There is no step three.

## Inputs

| input | required | default | meaning |
|---|---|---|---|
| `prompt` | yes | — | The frame's job on the branch. |
| `branch` | no | the run's own branch | Working branch; the default branch is refused. |
| `comment-on` | no | — | Issue/PR number for the reply. |
| `agent` | no | the org's single agent | Which memory answers (multi-agent orgs only). |
| `wait` | no | `true` | Watch the thread for the App's reply and mirror the outcome. |
| `wait-minutes` | no | `30` | Ceiling on the wait. |
| `server` | no | `https://mod.syntropy.systems` | Control plane origin. |

Minimal permissions for a calling workflow:

```yaml
permissions:
  id-token: write   # the OIDC proof — the whole identity
  issues: read      # watching the thread for the App's reply
```

(Your wrapper's other steps may need more — e.g. `contents: write` if the
wrapper itself pushes.)

Pin this action to a release tag:

```yaml
- uses: syntropy-systems-oss/agent-action@v0
```

**Pin tags, not `@main`** — you are executing this action in your repo's
identity context; review the tag you pin (it is one file).

## What this action does not do

It does not run any agent code in your workflow, download a runtime, read
your secrets, or touch anything beyond the HTTPS call you can read in
`action.yml` and the thread watch. Everything the agent does to your
repository happens server-side through the App installation you granted,
visible in your repo's audit surface like any App.

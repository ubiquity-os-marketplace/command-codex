# `command-codex` (skeleton)

This is a starter skeleton for a marketplace plugin repo `ubiquity-os-marketplace/command-codex`.

It is intended to be run as a **GitHub Actions plugin** (via `workflow_dispatch`) and invoked from the kernel via:

- `/codex <task>`
- `@ubiquityos codex <task>`

## What’s included

- `manifest.json`: declares a single `codex` command (no tool parameters; task is extracted from the comment body).
- `action.yml`: the action implementation that:
  - decompresses the kernel event payload
  - validates explicit invocation + basic authorization
  - checks out the target repo using the kernel-provided `authToken`
  - runs Codex (`codex exec`)
  - commits & pushes changes to a branch
  - creates a draft PR and comments back with the PR link
- `.github/workflows/action.yml`: the workflow entrypoint invoked via `workflow_dispatch` by the kernel.

This mirrors the patterns used by:

- `lib/command-ask/.github/workflows/action.yml` (workflow wrapper calling `./action.yml`)
- `lib/hello-world-plugin/manifest.json` (simple command declaration)

## Required secrets (in the plugin repo)

Subscription auth (preferred):

- `CODEX_AUTH_JSON_B64` (base64 of Codex CLI `auth.json`)

Optional fallback (pay-per-request):

- `OPENAI_API_KEY`

Set secrets in GitHub Actions **Environments** (`development` and `main`) or repo secrets.

## Kernel config (example)

In the target repo’s `.github/.ubiquity-os.config.yml` (or dev variant), enable the plugin:

```yml
plugins:
  ubiquity-os-marketplace/command-codex@development:
    with:
      allowedAuthorAssociations:
        - OWNER
        - MEMBER
        - COLLABORATOR
      denyForkPRs: true
      requireExplicitInvocation: true
      sandbox: workspace-write # set to danger-full-access for full runner + network access
      safetyStrategy: drop-sudo
      codexArgs: "" # extra `codex exec` flags (optional)
      passAuthTokenToCodex: false # set true to expose GH_TOKEN to Codex (dangerous)
      draftPr: true
```

## Notes

- This skeleton is intentionally conservative: it does not pass GitHub tokens to Codex.
- For fork PRs and other edge cases, refine policies in `action.yml`.

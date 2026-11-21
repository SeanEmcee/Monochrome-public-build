# Monochrome — public CI, private source

**Public build orchestration only.** All Monochrome source, app bundles, Worker source, maps, private tests, and credentials remain in the separate **private** [SeanEmcee/Monochrome-Deploy](https://github.com/SeanEmcee/Monochrome-Deploy) repository. This public repository contains no original site source or generated output.

This follows the Nexaris/Nexaris-public-build architecture, but uses Ubuntu/Node.js and Cloudflare Workers Static Assets rather than a macOS IPA compiler.

## Workflow

[Open Monochrome private-source build and Cloudflare deployment](https://github.com/SeanEmcee/Monochrome-public-build/actions/workflows/monochrome-private-build.yml)

Manual `workflow_dispatch` only. The workflow runs only when initiated by the owner from the public repository's `main` branch. It temporarily checks out the private source branch `feature/worker-request-reduction-20261121` using a read-only secret, then runs private Node tests and reconstructs the complete Cloudflare asset manifest. It prints only coarse PASS/FAIL messages; it does not upload source or public build artifacts.

Run modes:

- **validate (default):** test source and reconstruct/verify the complete asset bundle; no Cloudflare write
- **stage:** same tests, then upload the **complete** asset manifest and stage a Worker version without deploying
- **deploy:** same checks followed by an explicitly selected production deployment. Requires typing `DEPLOY PLAYER 28` in the production confirmation field and can be protected by GitHub Environment reviewers. The deploy script validates the original 20 bindings, current Worker version, and Durable Object migration.

The initial baseline is Cloudflare Worker 365 at version `3f46418b-0628-4cb1-82e2-9af1d8342924`, and the intended next release is `2026-11-21-player-28`. Updating the production baseline after deployment requires a reviewed private-source change.

## Required secrets

Configure in **this repository's** Settings → Secrets and variables → Actions → New repository secret. Never paste tokens into public files, Actions logs, or chat.

1. `MONO_PRIVATE_SOURCE_TOKEN` — **fine-grained GitHub token**, selected repository **SeanEmcee/Monochrome-Deploy**, **Contents: Read-only**. Required for all modes. Choose a short expiry and rotate when needed.
2. `MONO_CLOUDFLARE_API_TOKEN` — **Cloudflare API token** scoped to the relevant account and necessary Worker script/assets/versions/deployment editing permissions. Required for `stage` and `deploy`. Do not use a Global API Key or commit the credential.
3. `MONO_PRIVATE_OUTPUT_TOKEN` — optional fine-grained GitHub token scoped only to the private source repository with **Contents: Read and write**. Enables optional **draft release** private diagnostics; omit it if you don't want these logs stored. Never add public Actions artifact uploads.

Use separate credentials for the private-source checkout and private diagnostics; do not give a read/write credential to the source checkout. Avoid giving access to anyone who can edit the public workflow's `main` branch.

## Production review

In repository Settings → Environments, create **monochrome-production** and configure required reviewers if available on your plan. A named environment alone does **not** guarantee approval is enforced. Optionally create **monochrome-nonproduction** for the validation/staging modes. Restrict deployments to `main` if supported.

**Run `validate` first, then `stage`, then `deploy` only after reviewing staging.** Deployment is a live site change. If a release fails, inspect Cloudflare deployment history before retrying; the original upload script rejects unexpected production-version changes.

## Privacy boundaries and limitations

- The source repository and its git history stay private; the public workflow temporarily reads that private source inside a hosted runner.
- Actions workflow source and logs in **this public repository** are public. No build command should echo private source, tokens, or full logs; generated binaries and maps are never uploaded as public artifacts.
- Browser JavaScript published to the Monochrome website is inherently inspectable by visitors, regardless of repository visibility.
- Any contributor who can modify a workflow that has access to GitHub/Cloudflare secrets could potentially exfiltrate them; use branch protections and restrict collaborators accordingly.
- A public repository does not guarantee eligibility for free hosted-runner minutes. GitHub's additional Actions terms apply, and GitHub determines whether this split-build arrangement complies; a self-hosted runner or authorized private billing is the fallback.

This workflow handles the **first guarded Worker-traffic reduction patch**, not a complete migration of audio streaming/search routes. Never claim the latter is fully Worker-free without browser CORS and playback validation.

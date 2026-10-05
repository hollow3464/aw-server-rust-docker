# activitywatch-checker

Automated CI/CD pipeline that tracks upstream [ActivityWatch/aw-server-rust](https://github.com/ActivityWatch/aw-server-rust) releases and rebuilds/redeploys a Docker image when it changes.

## How it works

1. **`check-upstream.yml`** (daily cron + manual) fetches the latest commit hash of the upstream `master` branch via `git ls-remote` and compares it with the hash stored in `upstream-hash.txt`. If the hash changed, the file is updated and committed, which triggers the next workflow.

2. **`build-deploy.yml`** (on push to `main` touching `upstream-hash.txt`, or manual):
   - **build** job: fetches secrets from [Infisical](https://infisical.com) via OIDC, logs in to Docker Hub, clones `aw-server-rust` at the stored hash, builds the image with the repo's own `scripts/Dockerfile`, and pushes it as:
     - `daved3464/activitywatch:<commit-hash>`
     - `daved3464/activitywatch:latest`
   - **deploy** job: fetches secrets from Infisical and sends an HMAC-SHA256 signed `push` webhook event to a [doco-cd](https://github.com/kimdre/doco-cd) instance to trigger deployment.

## Repository layout

```
.github/workflows/
  check-upstream.yml   # Hash checker
  build-deploy.yml     # Build, push, deploy
upstream-hash.txt      # Last seen upstream commit hash
```

## Secrets (via Infisical, project `ci-secrets`, env `dev`)

| Variable | Purpose |
|---|---|
| `DOCKERHUB_USERNAME` | Docker Hub user |
| `DOCKERHUB_TOKEN` | Docker Hub access token |
| `WEBHOOK_URL` | doco-cd webhook endpoint |
| `WEBHOOK_SECRET` | HMAC secret for the webhook |

OIDC authentication to Infisical uses `Infisical/secrets-action` — no long-lived tokens stored in GitHub.

## Manual usage

- Run the checker manually: *Actions → Check upstream ActivityWatch → Run workflow*.
- Rebuild/redeploy manually: *Actions → Build, push and deploy → Run workflow*.

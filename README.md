# ci-workflows

Reusable GitHub Actions workflows shared across kylofon's project repositories.

## `sourceforge-mirror.yml`

Force-pushes the calling repository (branch + tags) to a SourceForge git repo over SSH.

### Inputs

| input | required | default | notes |
|---|---|---|---|
| `sf_project` | yes | – | SourceForge project **unix name** — the `SLUG` in `https://sourceforge.net/p/SLUG/code` |
| `sf_user` | no | `kylofon` | SourceForge account used for SSH auth |
| `branch` | no | triggering branch | Branch name to create/update on SourceForge |

### Secrets

| secret | notes |
|---|---|
| `SF_SSH_KEY` | Private ed25519 key whose public half is registered under SourceForge → Account → SSH Keys |

### Caller workflow

Add this as `.github/workflows/mirror.yml` in the project repo. Set `branches:` to
that repo's **actual default branch** (`main` or `master` — they are not uniform).

```yaml
name: Mirror to SourceForge
on:
  push:
    branches: [main]      # <- match this repo's default branch
  workflow_dispatch:

jobs:
  call-mirror:
    uses: kylofon/ci-workflows/.github/workflows/sourceforge-mirror.yml@master
    with:
      sf_project: SF_SLUG_HERE
    secrets:
      SF_SSH_KEY: ${{ secrets.SF_SSH_KEY }}
```

Notes:

- The `@master` ref is this repo's default branch. Change it only if this repo is renamed to `main`.
- `SF_SSH_KEY` must exist as an Actions secret in **each calling repo**.
- The SourceForge project must already exist; the mirror does not create it.
  Verify a slug with `curl -o /dev/null -w '%{http_code}' https://sourceforge.net/rest/p/SLUG`
  (`200` = exists, `404` = does not).
- Pushes are `--force`; SourceForge is a mirror, never a source of truth.

# Workflow Usage

Alle Workflows sind als Reusable Workflows verfügbar und werden per `uses` aufgerufen.

---

## Pipeline Übersicht

```
PR auf main gemergt
  └── release.yml – Tag + GitHub Release erstellen
        └── ci-build.yml – Stack erkennen, Abhängigkeiten, Lint, Build
              └── publish.yml – Docker Image bauen und in GHCR pushen
                    └── cd-deploy.yml – Via SSH auf Server deployen
```

---

## Release – `release.yml`

Trigger: PR wird auf `main` gemergt.

```yaml
name: Release

on:
  pull_request:
    types:
      - closed
    branches:
      - main

jobs:
  release:
    if: github.event.pull_request.merged == true
    uses: AgentGG00/workflows/.github/workflows/release.yml@main
    secrets: inherit
```

---

## CI Build – `ci-build.yml`

Trigger: Release wird published.

```yaml
name: CI Build

on:
  release:
    types: [published]

jobs:
  ci:
    uses: AgentGG00/workflows/.github/workflows/ci-build.yml@main
    secrets: inherit
```

---

## Publish – `publish.yml`

Trigger: CI Build erfolgreich – baut Docker Image und pusht in GitHub Container Registry.

```yaml
name: Publish

on:
  workflow_run:
    workflows: ["CI Build"]
    types:
      - completed

jobs:
  publish:
    if: ${{ github.event.workflow_run.conclusion == 'success' }}
    uses: AgentGG00/workflows/.github/workflows/publish.yml@main
    secrets: inherit
```

---

## CD Deploy – `cd-deploy.yml`

Trigger: Package published in GitHub Container Registry.

```yaml
name: CD Deploy

on:
  registry_package:
    types: [published]

jobs:
  deploy:
    uses: AgentGG00/workflows/.github/workflows/cd-deploy.yml@main
    with:
      project_name: "mein-projekt"
      bootstrap: false
    secrets:
      host: ${{ secrets.SERVER_HOST }}
      user: ${{ secrets.SERVER_USER }}
      ssh_key: ${{ secrets.SERVER_SSH_KEY }}
```

**Erstes Deployment:**

```yaml
    with:
      project_name: "mein-projekt"
      bootstrap: true
```

**Benötigte Secrets im Repo:**

| Secret | Inhalt |
|---|---|
| `SERVER_HOST` | IP oder Hostname des Servers |
| `SERVER_USER` | SSH-Benutzer |
| `SERVER_SSH_KEY` | Privater SSH-Key |

---

## Review – `review.yml`

Trigger: PR auf `main` – kommentiert das Ergebnis direkt im PR.

```yaml
name: Review

on:
  pull_request:
    branches:
      - main

jobs:
  review:
    uses: AgentGG00/workflows/.github/workflows/review.yml@main
    secrets: inherit
```

**Pflichtdateien die geprüft werden:**

| Datei | Ort |
|---|---|
| `README.md` | Root |
| `.gitignore` | Root |
| `.env.example` | Root |
| `Dockerfile` | Root |
| `docker-compose.yml` | Root |
| `requirements.txt` oder `package.json` | Root |
| `docs/projekt-plan.md` | `docs/` |
| `docs/projekt-stand.md` | `docs/` |
| `docs/cliff.toml` | `docs/` |

---

## Create Issue – `create-issue.yml`

Trigger: Push auf beliebigem Branch wenn `docs/issues.md` geändert wird.

```yaml
name: Create Issue

on:
  push:
    paths:
      - 'docs/issues.md'

jobs:
  issue:
    uses: AgentGG00/workflows/.github/workflows/create-issue.yml@main
    secrets: inherit
```

**Format `docs/issues.md`:**

```markdown
# Issues

## Titel des Issues
labels: bug
body: Kurze Beschreibung was das Problem ist und wo es auftritt.

---

## Zweites Issue
labels: bug, ux
body: Beschreibung des zweiten Issues.

---
```

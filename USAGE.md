# Workflow Usage

Alle Workflows sind als Reusable Workflows verfügbar und werden per `uses` aufgerufen.

---

## Release – `release.yml`

Erstellt automatisch einen neuen Release wenn ein PR auf `main` gemergt wird.
Benötigt eine `docs/cliff.toml` im aufrufenden Repo.

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

## CD Deploy – `cd-deploy.yml`

Deployed via SSH auf einen Server. Beim ersten Deployment `bootstrap: true` setzen.

```yaml
name: CD Deploy

on:
  release:
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

## CI Build – `ci-build.yml`

Erkennt den Stack automatisch anhand von `requirements.txt` oder `package.json` und führt den passenden Build- und Lint-Check aus.

```yaml
name: CI Build

on:
  push:
    branches:
      - main
      - dev
  pull_request:
    branches:
      - main

jobs:
  ci:
    uses: AgentGG00/workflows/.github/workflows/ci-build.yml@main
    secrets: inherit
```

---

## Review – `review.yml`

Prüft Pflichtdateien und scannt auf hartcodierte Secrets. Gibt das Ergebnis als PR-Comment aus.

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
| `requirements.txt` oder `package.json` | Root |
| `docs/projekt-plan.md` | docs/ |
| `docs/projekt-stand.md` | docs/ |
| `docs/cliff.toml` | docs/ |

---

## Create Issue – `create-issue.yml`

Legt automatisch ein GitHub Issue an wenn ein neuer Eintrag in `docs/issues.md` committet wird.

```yaml
name: Create Issue

on:
  push:
    branches:
      - main
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
body: Beschreibung des Issues.

---

## Zweites Issue
labels: bug, ux
body: Beschreibung des zweiten Issues.

---
```

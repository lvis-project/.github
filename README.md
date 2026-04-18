# .github (lvis-project)

Org-level meta: reusable workflows and workflow templates.

- Reusable CI: `.github/workflows/plugin-ci.yml` (tag `@v1`)
- Template: `.github/workflow-templates/plugin-ci.yml`

## Consumer wrapper (each lvis-plugin-* repo)

```yaml
name: CI
on:
  push:
    branches: [main]
  pull_request:
    branches: [main]
jobs:
  ci:
    uses: lvis-project/.github/.github/workflows/plugin-ci.yml@v1
```

# lvis-project — Governance

## Tag Policy (manual — free plan)

The `lvis-project` org is on the **free** GitHub plan, which does not support repository or org rulesets. Tag immutability / force-push protection is therefore enforced **by convention**, not by GitHub.

### Protected tags across the org

| Repo | Tag | Meaning |
|---|---|---|
| `lvis-project/.github` | `v1`, `v1.0.0` | Reusable CI workflow (5 consumer repos pin here) |
| `lvis-project/lvis-plugin-sdk` | `v1`, `v1.0.0`, `v0.1.0` | Type contract pinned by all plugin repos + host app |

### Rules

1. **`vN` (major) tags are *moving*** — only re-pointed to newer compatible commits. Use `git push --force-with-lease`, never plain `--force`.
2. **`vN.M.P` (full SemVer) tags are *immutable*** — never re-point. Cut a new patch release instead.
3. **Re-pointing `v1`** requires (a) passing drift-check (SDK) or CI (reusable workflow), (b) changelog entry.
4. Tag changes land via PR review (no direct push to tag); PR description must justify the move.

### Audit

- SDK drift-check (nightly) — surfaces if SDK's `v1` regression-introduces host divergence.
- Manual quarterly review of tag history (`git log v1.0.0..v1`) for each protected tag.

### Upgrade path

When the org upgrades to GitHub Team/Pro, replace this manual policy with rulesets:

```bash
# .github repo
gh api -X POST /repos/lvis-project/.github/rulesets --input - <<'EOF'
{ "name": "protect-reusable-workflow-tags", "target": "tag", "enforcement": "active",
  "conditions": { "ref_name": { "include": ["refs/tags/v*"], "exclude": [] } },
  "rules": [{ "type": "deletion" }, { "type": "non_fast_forward" }] }
EOF

# SDK repo
gh api -X POST /repos/lvis-project/lvis-plugin-sdk/rulesets --input - <<'EOF'
{ "name": "protect-sdk-v-tags", "target": "tag", "enforcement": "active",
  "conditions": { "ref_name": { "include": ["refs/tags/v*"], "exclude": [] } },
  "rules": [{ "type": "deletion" }, { "type": "non_fast_forward" }] }
EOF
```

After upgrade, delete this manual-policy section and link to the ruleset URLs.

## CI Architecture

- **Central reusable workflow:** `.github/workflows/plugin-ci.yml@v1`
- **Consumers:** every `lvis-plugin-*` repo (excluding `lvis-plugin-sdk`) via a ~10-line wrapper at `.github/workflows/ci.yml`
- **Template:** `lvis-project/lvis-plugin-template` pre-wired with SDK submodule + CI wrapper

## Repo Matrix

| Repo | Submodule | CI Wrapper | Purpose |
|---|---|---|---|
| `lvis-app` | `packages/plugin-sdk` → SDK@v1.0.0 | — (host app CI separate) | Electron host |
| `lvis-plugin-sdk` | — | — (own drift-check workflow) | Type-only SDK |
| `lvis-plugin-meeting` | `plugin-sdk/` → SDK@v1.0.0 | `@v1` | Meeting plugin |
| `lvis-plugin-pageindex` | `plugin-sdk/` → SDK@v1.0.0 | `@v1` | PageIndex plugin |
| `lvis-plugin-email` | `plugin-sdk/` → SDK@v1.0.0 | `@v1` | Email plugin |
| `lvis-plugin-calendar` | `plugin-sdk/` → SDK@v1.0.0 | `@v1` | Calendar plugin |
| `lvis-plugin-template` | `plugin-sdk/` → SDK@v1.0.0 | `@v1` | Scaffold for new plugins |
| `.github` | — | — | Org meta (reusable workflows) |

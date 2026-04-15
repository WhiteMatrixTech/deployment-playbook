# Test Report

> Copy this template and fill in before requesting deployment approval.

## Metadata

| Field | Value |
|-------|-------|
| **Service** | `<service-name>` |
| **Version / Tag** | `<git-tag or image-tag>` |
| **Branch** | `<branch-name>` |
| **Commit SHA** | `<sha>` |
| **Author** | `<deployer>` |
| **Date** | `YYYY-MM-DD` |
| **Environment** | staging / production |

## Change Summary

<!-- 1-3 sentences: what changed and why. Link to PR/issue. -->

- PR: #___
- Summary: ___

## Test Matrix

### Unit Tests

| Suite | Passed | Failed | Skipped | Coverage |
|-------|--------|--------|---------|----------|
| `<workspace/module>` | | | | % |

```bash
# Command used
npm test --workspace=<name>
```

<details>
<summary>Test output (truncated)</summary>

```
<paste relevant output>
```
</details>

### Integration Tests

| Suite | Passed | Failed | Notes |
|-------|--------|--------|-------|
| API routes | | | |
| Database migrations | | | |
| Auth middleware | | | |

### End-to-End Tests

| Scenario | Status | Evidence |
|----------|--------|----------|
| Happy path: `<primary user flow>` | PASS / FAIL | screenshot / link |
| Edge case: `<describe>` | PASS / FAIL | |
| Regression: `<known bug area>` | PASS / FAIL | |

### Performance / Load (if applicable)

| Metric | Before | After | Threshold | Status |
|--------|--------|-------|-----------|--------|
| p50 latency | ms | ms | < ___ms | |
| p99 latency | ms | ms | < ___ms | |
| Error rate | % | % | < ___% | |
| Memory usage | MB | MB | < ___MB | |

### Security Scan

| Tool | Findings | Critical | High | Notes |
|------|----------|----------|------|-------|
| Secret scan (AgentShield / gitleaks) | | | | |
| Dependency audit (`npm audit`) | | | | |
| SAST (if configured) | | | | |

## Known Issues / Risks

<!-- List any known issues that are shipping intentionally, with justification. -->

- [ ] None
- [ ] `<issue>`: accepted because `<reason>`

## Test Verdict

- [ ] **PASS** — All critical tests pass; no blocking issues
- [ ] **CONDITIONAL PASS** — Passes with known issues listed above
- [ ] **FAIL** — Do not deploy; issues must be resolved first

**Signed off by**: `<name>` on `<date>`

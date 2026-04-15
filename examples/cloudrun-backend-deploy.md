# Example: Cloud Run Backend Deployment

> Real-world example based on OpenClawCloud backend (Express/TypeScript on Cloud Run).

---

## Test Report

| Field | Value |
|-------|-------|
| **Service** | `openclaw-backend` |
| **Version / Tag** | `prod-20260414-a1b2c3d` |
| **Branch** | `main` |
| **Commit SHA** | `a1b2c3d4e5f6` |
| **Author** | shuyi |
| **Date** | 2026-04-14 |
| **Environment** | production |

### Change Summary

- PR: #592
- Summary: Fix pool manager WeChat bind QR capture (stdout → internal log polling)

### Test Matrix

| Suite | Passed | Failed | Coverage |
|-------|--------|--------|----------|
| backend (vitest) | 47 | 0 | 82% |

| E2E Scenario | Status |
|----------|--------|
| WeChat bind QR → poll → success | PASS |
| WeChat bind timeout (6min) | PASS |
| Agent create → deploy → chat | PASS |

### Security Scan

| Tool | Critical | High |
|------|----------|------|
| AgentShield | 0 | 0 |
| npm audit | 0 | 0 |

**Verdict**: PASS

---

## Deployment Plan

### Metadata

| Field | Value |
|-------|-------|
| **Service** | backend |
| **Target env** | production |
| **Deploy window** | 2026-04-14 02:00-03:00 UTC |
| **Risk level** | Low (single service, no migration) |

### Steps

- [x] 1.1 Tag: `git tag v1.5.2 && git push origin v1.5.2`
- [x] 1.2 CI auto-triggers `deploy-prod.yml` for backend
- [x] 1.3 Verify build: `prod-20260414-a1b2c3d` in Artifact Registry
- [x] 2.1 Pre-deploy checklist: all items checked
- [x] 2.2 Baseline: error rate 0.1%, p50 45ms, p99 320ms
- [x] 3.1 CI deploys via `deploy.sh prod backend --build --infra --yes`
- [x] 3.2 New revision serving: `curl api.tapclaw.app/health → v1.5.2`
- [x] 5.1 Smoke test: `deploy.sh prod --verify` PASS
- [x] 6.1 Monitor 15 min: error rate 0.08% (within baseline)
- [x] 6.4 Notified #deployments: "v1.5.2 backend deployed. SUCCESS"

---

## Canary Plan

Strategy: **Cloud Run traffic splitting**

| Stage | Traffic | Bake | Error rate | p50 | Decision |
|-------|---------|------|------------|-----|----------|
| 1 | 10% canary | 5 min | 0.09% (< 1.1%) | 43ms (< 54ms) | GO |
| 2 | 50% canary | 10 min | 0.10% (< 1.1%) | 44ms (< 54ms) | GO |
| 3 | 100% | — | 0.08% | 45ms | Complete |

---

## Rollback Plan

| Field | Value |
|-------|-------|
| **Previous version** | `prod-20260413-f9e8d7c` (revision `openclaw-backend-00147`) |
| **Rollback method** | Cloud Run revision rollback |
| **Estimated time** | 30 seconds |

```bash
# Rollback command (pre-tested)
gcloud run services update-traffic openclaw-backend \
  --region=us-central1 \
  --to-revisions=openclaw-backend-00147=100
```

**Outcome**: Rollback was NOT needed. Deployment succeeded.

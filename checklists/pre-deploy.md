# Pre-Deploy Checklist

> Run through EVERY item before executing the deployment plan. All items must be checked or explicitly marked N/A.

## Code Readiness

- [ ] PR merged and CI green on target branch
- [ ] No open BLOCKING or CRITICAL issues on the PR
- [ ] Test report completed with verdict = PASS or CONDITIONAL PASS
- [ ] Secret scan clean (no new credentials committed)
- [ ] Dependency audit: no new critical/high CVEs (`npm audit` / `pip audit`)
- [ ] Build artifact exists in registry with expected tag
- [ ] Image digest recorded in deployment plan

## Environment Readiness

- [ ] Target environment is healthy (no active incidents)
- [ ] No other deployments in progress to the same environment
- [ ] Deployment window confirmed (for production: team notified)
- [ ] Required secrets/env vars are set in target environment
- [ ] Database migrations are compatible (if applicable):
  - [ ] Forward-compatible with current code (for zero-downtime)
  - [ ] Reversible (or documented as non-reversible)
  - [ ] Tested on staging/dev first

## Observability

- [ ] Monitoring dashboards bookmarked and accessible:
  - Error rate dashboard: `<link>`
  - Latency dashboard: `<link>`
  - Resource utilization: `<link>`
- [ ] Alerts are configured and routing to on-call
- [ ] Log aggregation is accessible for the target service
- [ ] Baseline metrics recorded:
  - Current error rate: ___%
  - Current p50 latency: ___ms
  - Current p99 latency: ___ms

## People

- [ ] Deployer identified and available for the full deployment window
- [ ] Rollback owner identified and reachable
- [ ] On-call engineer aware of the deployment
- [ ] For High/Critical risk: approver has signed off on deployment plan

## Rollback Readiness

- [ ] Rollback plan written and reviewed
- [ ] Previous stable version/revision identified: `<version>`
- [ ] Rollback command tested on staging (for first-time procedures)
- [ ] Estimated rollback time: `<N>` minutes

## Final Gate

- [ ] **All above items checked or marked N/A**
- [ ] **Deployer confirms: "Ready to deploy"**

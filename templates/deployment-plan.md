# Deployment Plan

> Step-by-step deployment actions. Every step is a checkbox. Do not skip steps.

## Metadata

| Field | Value |
|-------|-------|
| **Service(s)** | `<backend, frontend, litellm, pool, ...>` |
| **Target env** | dev / staging / production |
| **Deploy window** | `YYYY-MM-DD HH:MM – HH:MM UTC` |
| **Deployer** | `<name>` |
| **Approver** | `<name>` |
| **Rollback owner** | `<name>` |
| **Estimated duration** | `<N>` minutes |
| **Risk level** | Low / Medium / High / Critical |

## Prerequisites

- [ ] Test report completed and verdict = PASS ([link to test report](#))
- [ ] PR merged to target branch (or tag pushed)
- [ ] No active incidents on target environment
- [ ] Deployment window confirmed with team (for High/Critical)
- [ ] Rollback plan reviewed and understood by rollback owner
- [ ] Canary plan reviewed (if applicable)
- [ ] Database migrations tested on staging (if applicable)
- [ ] Feature flags configured (if applicable)
- [ ] Monitoring dashboards open: `<links>`

## Deployment Steps

### Phase 1: Build

- [ ] **1.1** Trigger build: `<exact command or CI workflow>`
  ```bash
  # Example: deploy.sh beta backend --build --yes
  ```
- [ ] **1.2** Verify build succeeds — check CI/CD status: `<link>`
- [ ] **1.3** Verify image tag: `<registry-url>:<expected-tag>`
- [ ] **1.4** Record image digest: `sha256:___`

### Phase 2: Pre-deploy Verification

- [ ] **2.1** Run pre-deploy checklist: [checklists/pre-deploy.md](../checklists/pre-deploy.md)
- [ ] **2.2** Verify current production is healthy (baseline metrics):
  - Error rate: ___%
  - p50 latency: ___ms
  - Active users: ___
- [ ] **2.3** Notify team in `#deployments` channel: "Starting deploy of `<service>` `<version>` to `<env>`"

### Phase 3: Deploy

- [ ] **3.1** Execute deployment:
  ```bash
  # Example: deploy.sh prod backend --build --infra --yes
  ```
- [ ] **3.2** Wait for rollout completion (all replicas healthy)
- [ ] **3.3** Verify new revision is serving traffic:
  ```bash
  # Example: curl -s https://api.example.com/health | jq .version
  ```

### Phase 4: Canary (if applicable)

- [ ] **4.1** Follow canary plan: [link to canary-plan.md](#)
- [ ] **4.2** Record canary metrics at each gate
- [ ] **4.3** Canary verdict: GO / NO-GO

### Phase 5: Full Rollout

- [ ] **5.1** Shift 100% traffic to new revision
- [ ] **5.2** Verify all health endpoints return 200
- [ ] **5.3** Run post-deploy checklist: [checklists/post-deploy.md](../checklists/post-deploy.md)
- [ ] **5.4** Run smoke tests:
  ```bash
  # Example: deploy.sh prod --verify
  ```

### Phase 6: Post-deploy

- [ ] **6.1** Monitor for 15 minutes (dashboards: `<links>`)
- [ ] **6.2** Check error rate is within baseline +/- 5%
- [ ] **6.3** Check no new error types in logs
- [ ] **6.4** Notify team: "Deploy of `<service>` `<version>` to `<env>` complete. Status: SUCCESS / WATCHING"
- [ ] **6.5** Close deployment issue / update status

## Abort Criteria

If ANY of these occur, **stop deployment and execute rollback plan**:

- [ ] Error rate exceeds baseline by >10%
- [ ] p99 latency exceeds 2x baseline
- [ ] Health endpoint returns non-200 for >60 seconds
- [ ] Critical alert fires from monitoring
- [ ] Build or infrastructure step fails

## Notes

<!-- Any deployment-specific notes, dependencies, or coordination needed. -->

---

**Plan approved by**: `<approver>` on `<date>`

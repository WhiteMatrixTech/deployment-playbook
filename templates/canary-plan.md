# Canary / Staging Plan

> Gradual traffic shift with go/no-go decision gates at each stage.

## Metadata

| Field | Value |
|-------|-------|
| **Service** | `<service-name>` |
| **Canary method** | Traffic split / Rolling VM / Preview URL / Feature flag |
| **Total stages** | `<N>` |
| **Total duration** | `<N>` minutes |
| **Rollback trigger** | Auto (metrics) / Manual (human) |

## Strategy Selection

Choose the strategy that matches your platform:

### A. Cloud Run — Traffic Splitting

```
Stage 1: 10% canary → 90% stable     (5 min bake)
Stage 2: 50% canary → 50% stable     (10 min bake)
Stage 3: 100% canary                  (full rollout)
```

```bash
# Stage 1
gcloud run services update-traffic <service> \
  --to-revisions=<new-rev>=10,<old-rev>=90

# Stage 2
gcloud run services update-traffic <service> \
  --to-revisions=<new-rev>=50,<old-rev>=50

# Stage 3 (full rollout)
gcloud run services update-traffic <service> \
  --to-revisions=<new-rev>=100
```

### B. VM Pool — Rolling Restart

```
Stage 1: 1 VM patched, N-1 on old version    (5 min bake)
Stage 2: 50% VMs patched                     (10 min bake)
Stage 3: All VMs patched                     (full rollout)
```

```bash
# Example (OpenClawCloud)
deploy.sh prod --pool=hot-patch   # patches 1 VM at a time, health-checks between
```

### C. Kubernetes — Weighted Routing

```
Stage 1: 10% via VirtualService/HTTPRoute    (5 min bake)
Stage 2: 50%                                 (10 min bake)
Stage 3: 100%                                (full rollout)
```

### D. Feature Flag — Percentage Rollout

```
Stage 1: 5% of users see new feature         (1 hour bake)
Stage 2: 25% of users                        (4 hour bake)
Stage 3: 100% of users                       (full rollout)
```

---

## Canary Execution

### Gate Criteria (apply at EVERY stage transition)

| Metric | Threshold | Check method |
|--------|-----------|-------------|
| Error rate (5xx) | < baseline + 1% | `<dashboard-link>` |
| p50 latency | < baseline + 20% | `<dashboard-link>` |
| p99 latency | < baseline + 50% | `<dashboard-link>` |
| Memory usage | < limit * 80% | `<dashboard-link>` |
| Custom: `<metric>` | `<threshold>` | `<method>` |

**GO** = all metrics within thresholds for the full bake duration
**NO-GO** = any metric breached → execute rollback plan immediately

### Stage Log

Record actuals during execution:

#### Stage 1: `___%` traffic

- [ ] Traffic shifted at `HH:MM UTC`
- [ ] Bake duration: `___` minutes
- Metrics observed:
  - Error rate: ___%  (threshold: < ___%)
  - p50 latency: ___ms  (threshold: < ___ms)
  - p99 latency: ___ms  (threshold: < ___ms)
- [ ] **Decision: GO / NO-GO**
- [ ] Decision by: `<name>` at `HH:MM UTC`

#### Stage 2: `___%` traffic

- [ ] Traffic shifted at `HH:MM UTC`
- [ ] Bake duration: `___` minutes
- Metrics observed:
  - Error rate: ___%
  - p50 latency: ___ms
  - p99 latency: ___ms
- [ ] **Decision: GO / NO-GO**
- [ ] Decision by: `<name>` at `HH:MM UTC`

#### Stage 3: 100% traffic (full rollout)

- [ ] Traffic shifted at `HH:MM UTC`
- [ ] Post-rollout monitoring: `___` minutes
- [ ] All metrics within threshold
- [ ] **Canary complete**

## Rollback During Canary

If NO-GO at any stage:

1. **Immediately** shift 100% traffic back to stable revision
2. Log the metrics that triggered NO-GO
3. Notify team in `#deployments`
4. Follow full [rollback plan](./rollback-plan.md) if needed
5. File post-mortem if canary caught a real issue (this is a success!)

---

**Canary approved by**: `<approver>` on `<date>`

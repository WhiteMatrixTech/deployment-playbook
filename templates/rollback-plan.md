# Rollback Plan

> Pre-planned recovery steps. This plan is written BEFORE deployment and executed IF things go wrong.

## Metadata

| Field | Value |
|-------|-------|
| **Service** | `<service-name>` |
| **Current (pre-deploy) version** | `<tag/revision/sha>` |
| **New version being deployed** | `<tag/revision/sha>` |
| **Rollback owner** | `<name>` |
| **Max acceptable downtime** | `<N>` minutes |
| **Rollback method** | Revision revert / Image swap / Feature flag / DB restore |

## Rollback Triggers

Execute this plan if ANY of the following occur:

- [ ] Error rate exceeds `___`% for > 60 seconds
- [ ] p99 latency exceeds `___`ms for > 60 seconds
- [ ] Health endpoint returns non-200 for > 60 seconds
- [ ] Critical alert fires: `<alert-name>`
- [ ] Data integrity issue detected
- [ ] Customer-reported issue confirmed
- [ ] Deployer or on-call makes a judgment call

## Rollback Procedures

### Method A: Cloud Run Revision Rollback (~30 seconds)

```bash
# Step 1: Identify the last known good revision
gcloud run revisions list --service=<service> --region=<region> --limit=5

# Step 2: Shift 100% traffic to the previous revision
gcloud run services update-traffic <service> \
  --region=<region> \
  --to-revisions=<previous-revision>=100

# Step 3: Verify
curl -s https://<service-url>/health | jq .version
# Expected: <previous-version>
```

**Estimated time**: 30 seconds
**Data impact**: None (stateless)

### Method B: VM Pool Image Rollback (~5-10 minutes)

```bash
# Step 1: Identify previous image family
gcloud compute images list --filter="family=<pool-image-family>" --sort-by=~creationTimestamp --limit=3

# Step 2: Hot-patch VMs back to previous image
deploy.sh <env> --pool=hot-patch --image-family=<previous-family>

# Step 3: Verify each VM
curl -s http://<vm-ip>:9000/health
```

**Estimated time**: 5-10 minutes (per VM)
**Data impact**: Active sessions may be disrupted; stateless containers restart

### Method C: Database Migration Rollback

```bash
# Step 1: Run the down migration
<migration-tool> migrate down --to=<previous-version>

# Step 2: Verify schema
<migration-tool> status

# Step 3: Deploy previous application version (Method A or B above)
```

**Estimated time**: Varies (1-30 minutes depending on data volume)
**Data impact**: HIGH — verify reversibility before deploying. Non-reversible migrations need a separate strategy.

### Method D: Feature Flag Rollback (~10 seconds)

```bash
# Step 1: Disable the feature flag
<flag-tool> disable <flag-name>

# Step 2: Verify
# Users should immediately see the old behavior
```

**Estimated time**: 10 seconds
**Data impact**: None if flag is UI-only; verify if flag gates data writes

### Method E: Frontend Rollback (Vercel / CDN)

```bash
# Vercel
vercel rollback --scope=<team>

# Or: redeploy last known good commit
git checkout <previous-sha>
vercel deploy --prod
```

**Estimated time**: 1-2 minutes
**Data impact**: None (static assets)

---

## Post-Rollback Verification

After executing rollback:

- [ ] **1.** Verify service returns to previous version: `curl health | jq .version`
- [ ] **2.** Verify error rate returns to baseline within 5 minutes
- [ ] **3.** Verify no data corruption (spot-check key records)
- [ ] **4.** Notify team: "Rollback of `<service>` complete. Now on `<previous-version>`"
- [ ] **5.** Page on-call if rollback was triggered by data integrity issue

## Communication

| When | Who | Channel | Message |
|------|-----|---------|---------|
| Rollback initiated | Deployer | #deployments | "Rolling back `<service>` from `<new>` to `<old>`. Reason: `<trigger>`" |
| Rollback complete | Deployer | #deployments | "Rollback complete. `<service>` on `<old>`. Investigating root cause." |
| If customer-facing | Deployer + PM | #incidents | "Service degradation detected. Rollback in progress. ETA: `<N>` min" |
| Post-mortem scheduled | On-call lead | #incidents | "Post-mortem for `<date>` deploy incident scheduled for `<date/time>`" |

## Post-Mortem Requirements

If rollback was executed, a post-mortem is MANDATORY within 48 hours:

- [ ] Timeline of events (deploy start → issue detected → rollback → recovery)
- [ ] Root cause analysis (5 Whys)
- [ ] What the canary plan caught or missed
- [ ] Action items with owners and deadlines
- [ ] Template/checklist updates if process gap identified

---

**Rollback plan reviewed by**: `<reviewer>` on `<date>`

# Deployment Incident Response

> When a deployment causes an incident. Follow this runbook top-to-bottom.

## Severity Classification

| Level | Criteria | Response time | Rollback? |
|-------|----------|--------------|-----------|
| **SEV-1** | Service down, data loss, security breach | Immediate | Mandatory |
| **SEV-2** | Major feature broken, >10% users affected | < 15 min | Strongly recommended |
| **SEV-3** | Minor feature broken, <10% users affected | < 1 hour | Case-by-case |
| **SEV-4** | Cosmetic, no user impact | Next business day | No |

## Step 1: Assess (< 2 minutes)

- [ ] **What is broken?** ___
- [ ] **When did it start?** ___
- [ ] **What was deployed?** Service: ___, Version: ___, Time: ___
- [ ] **Who is affected?** All users / Subset / Internal only
- [ ] **Severity**: SEV-1 / SEV-2 / SEV-3 / SEV-4

## Step 2: Communicate (< 5 minutes)

- [ ] Post in `#incidents`:
  ```
  INCIDENT: [SEV-X] <service> — <brief description>
  Started: HH:MM UTC
  Impact: <who is affected>
  Status: INVESTIGATING
  Lead: <your name>
  ```
- [ ] If SEV-1/SEV-2: page on-call if not already alerted
- [ ] If customer-facing: notify PM for external comms

## Step 3: Mitigate (< 15 minutes)

**Decision: Rollback or Fix-Forward?**

| Situation | Action |
|-----------|--------|
| Root cause is unknown | **Rollback** |
| Fix is trivial and tested (<5 min) | Fix-forward (with team agreement) |
| Data is being corrupted | **Rollback immediately** |
| Users are unable to use the service | **Rollback** |

### If rolling back:

- [ ] Execute rollback plan: [rollback-plan.md](../templates/rollback-plan.md)
- [ ] Verify service recovers
- [ ] Update `#incidents`: "Rolled back to `<version>`. Service recovering."

### If fixing forward:

- [ ] Implement fix on a branch
- [ ] Get one reviewer to approve (expedited)
- [ ] Deploy fix following abbreviated deployment plan
- [ ] Update `#incidents`: "Fix deployed. Monitoring."

## Step 4: Verify Recovery (< 30 minutes)

- [ ] Error rate returned to baseline
- [ ] Latency returned to baseline
- [ ] No data corruption (spot-check critical records)
- [ ] User reports stopped
- [ ] Update `#incidents`: "RESOLVED at HH:MM UTC. Root cause: `<brief>`"

## Step 5: Post-Mortem (within 48 hours)

### Template

```markdown
# Post-Mortem: [date] [service] deployment incident

## Timeline
| Time (UTC) | Event |
|------------|-------|
| HH:MM | Deploy started |
| HH:MM | Issue detected (how: alert/user/manual) |
| HH:MM | Rollback initiated |
| HH:MM | Service recovered |

## Impact
- Duration: ___ minutes
- Users affected: ___
- Revenue impact: $___  (or N/A)
- Data impact: none / <describe>

## Root Cause
<5 Whys analysis>

## What Went Well
- <what caught the issue early>
- <what worked in the response>

## What Went Wrong
- <what failed or was too slow>
- <what the canary/staging missed>

## Action Items
| Item | Owner | Deadline | Status |
|------|-------|----------|--------|
| <action> | <name> | <date> | Open |

## Process Improvements
- [ ] Update deployment checklist with: ___
- [ ] Update canary plan with: ___
- [ ] Update monitoring with: ___
- [ ] Add regression test for: ___
```

### Post-Mortem Principles

1. **Blameless** — focus on systems and processes, not individuals
2. **Actionable** — every finding leads to a concrete improvement
3. **Shared** — post-mortem document shared with the team
4. **Template-feeding** — improvements feed back into this playbook

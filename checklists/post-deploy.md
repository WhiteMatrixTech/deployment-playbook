# Post-Deploy Checklist

> Run through after deployment completes. All items must pass within the monitoring window.

## Immediate Verification (0-5 minutes)

- [ ] Health endpoint returns 200 with expected version
  ```bash
  curl -s https://<service-url>/health | jq '{version, status}'
  ```
- [ ] No new 5xx errors in logs (check last 5 minutes)
- [ ] Service is accepting traffic (not stuck in startup)
- [ ] DNS resolves correctly (if domain changed)

## Functional Smoke Tests (5-15 minutes)

- [ ] Primary user flow works end-to-end:
  - Flow: `<describe primary happy path>`
  - Status: PASS / FAIL
- [ ] Authentication/authorization works:
  - Login: PASS / FAIL
  - Protected endpoint: PASS / FAIL
- [ ] Critical API endpoints respond:
  - `GET /api/<endpoint>`: ___ms, status ___
  - `POST /api/<endpoint>`: ___ms, status ___
- [ ] Background jobs/workers are processing (if applicable)
- [ ] WebSocket connections establish (if applicable)

## Metrics Verification (15-30 minutes)

- [ ] Error rate within baseline +/- 5%
  - Baseline: ___% → Current: ___%
- [ ] p50 latency within baseline +/- 20%
  - Baseline: ___ms → Current: ___ms
- [ ] p99 latency within baseline +/- 50%
  - Baseline: ___ms → Current: ___ms
- [ ] Memory usage stable (no leak trend)
- [ ] CPU usage within expected range
- [ ] No new error types in structured logs

## Integration Points (if applicable)

- [ ] Downstream services receiving data correctly
- [ ] Webhook deliveries succeeding
- [ ] Third-party API integrations responding
- [ ] Cache hit rate stable (no cold-cache cliff)
- [ ] Queue depth stable (no processing backlog)

## Communication

- [ ] Team notified: "Deploy complete. Status: SUCCESS"
- [ ] Deployment issue/ticket updated with:
  - [ ] Final image tag/revision
  - [ ] Deployment timestamp
  - [ ] Verification results
  - [ ] Link to monitoring dashboard

## Cleanup

- [ ] Old revisions/images marked for cleanup (if retention policy)
- [ ] Temporary feature flags reviewed (remove if no longer needed)
- [ ] Documentation updated (if deployment changed behavior)

## Final Verdict

- [ ] **SUCCESS** — All checks pass; deployment is complete
- [ ] **WATCHING** — Minor anomaly detected; monitoring extended to `<time>`
- [ ] **ROLLBACK** — Critical issue detected; executing rollback plan

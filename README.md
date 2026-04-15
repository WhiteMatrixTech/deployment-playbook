# Deployment Playbook

Standardized deployment SOP for Matrix Labs projects. Every production deployment follows the same 4-artifact process — no exceptions.

**[中文版 README](README.zh.md)**

---

## Why This Exists

Deployments fail for predictable reasons: untested code, no rollback plan, full traffic shift without canary, unclear ownership during incidents. This playbook eliminates those failure modes by requiring 4 artifacts before any production deployment.

## The 4 Artifacts

| # | Artifact | Template | Purpose |
|---|----------|----------|---------|
| 1 | **Test Report** | [`templates/test-report.md`](templates/test-report.md) | Prove the build is safe to ship |
| 2 | **Deployment Plan** | [`templates/deployment-plan.md`](templates/deployment-plan.md) | Step-by-step actions with checkboxes |
| 3 | **Canary / Staging Plan** | [`templates/canary-plan.md`](templates/canary-plan.md) | Gradual rollout with go/no-go gates |
| 4 | **Rollback Plan** | [`templates/rollback-plan.md`](templates/rollback-plan.md) | How to undo if things break |

---

## How to Use

### Method 1: GitHub Issue Template (Recommended)

The fastest way to adopt this playbook in your project.

**Step 1: Copy the issue template to your project**

```bash
# From your project root
mkdir -p .github/ISSUE_TEMPLATE
curl -o .github/ISSUE_TEMPLATE/deployment-request.yml \
  https://raw.githubusercontent.com/WhiteMatrixTech/deployment-playbook/main/.github/ISSUE_TEMPLATE/deployment-request.yml
git add .github/ISSUE_TEMPLATE/deployment-request.yml
git commit -m "chore: add deployment request issue template"
```

**Step 2: Create a deployment request**

Go to your repo → Issues → New Issue → "部署请求 / Deployment Request"

The issue form is **in Chinese by default** (field labels, section headers, pre-populated skeletons). Fill in all 4 sections (产物 1–4).

**Step 3: Get approval**

- Low risk: self-approve, proceed
- Medium risk: one reviewer approves the issue
- High/Critical risk: team lead + on-call must approve

**Step 4: Execute**

Follow the checkboxes in your deployment plan. Check each box as you complete each step. The issue becomes a living log of the deployment.

---

### Method 2: Copy Templates into Your Project

For teams that prefer docs-in-repo over GitHub Issues.

```bash
# Clone this playbook
git clone https://github.com/WhiteMatrixTech/deployment-playbook.git /tmp/dp

# Copy templates to your project
mkdir -p docs/deploy-templates
cp /tmp/dp/templates/*.md docs/deploy-templates/
cp /tmp/dp/checklists/*.md docs/deploy-templates/

# For each deployment, create a dated directory
mkdir -p docs/deploys/2026-04-15-backend-v1.5.3/
cp docs/deploy-templates/*.md docs/deploys/2026-04-15-backend-v1.5.3/
# Fill in the templates, commit, get review
```

---

### Method 3: AI Agent Generation

For teams using Claude Code, Cursor, Copilot, or other AI coding assistants.

**Step 1: Add to your project's agent instructions**

In your `CLAUDE.md` (or equivalent):

```markdown
## Deployment SOP

Before any production deployment, generate a deployment plan following the templates at:
https://github.com/WhiteMatrixTech/deployment-playbook/templates/

Required artifacts:
1. Test Report (templates/test-report.md)
2. Deployment Plan (templates/deployment-plan.md)
3. Canary Plan (templates/canary-plan.md)
4. Rollback Plan (templates/rollback-plan.md)

Use the project's actual deploy commands, not generic examples.
Populate thresholds from real baseline metrics.
Output as a GitHub Issue using the deployment-request template.
```

**Step 2: Ask your agent**

```
Generate a deployment plan for backend v1.5.3 to production.
Service: openclaw-backend
Changes: PR #592 (fix WeChat QR bind)
Deploy command: deploy.sh prod backend --build --infra --yes
Rollback: gcloud run services update-traffic
```

The agent will read the templates and generate a complete deployment request with all 4 artifacts filled in.

**Step 3: Review and execute**

Review the generated plan. Adjust thresholds and commands as needed. Create the GitHub Issue. Execute.

---

### Method 4: CI/CD Integration

Enforce the playbook in your CI pipeline.

```yaml
# .github/workflows/deploy-gate.yml
name: Deploy Gate
on:
  issues:
    types: [labeled]

jobs:
  check-deployment-artifacts:
    if: contains(github.event.label.name, 'deployment')
    runs-on: ubuntu-latest
    steps:
      - name: Verify all 4 artifacts present
        run: |
          BODY="${{ github.event.issue.body }}"
          for section in "Test Report" "Deployment Plan" "Canary" "Rollback Plan"; do
            if ! echo "$BODY" | grep -qi "$section"; then
              echo "::error::Missing required artifact: $section"
              exit 1
            fi
          done
          echo "All 4 artifacts present"
```

---

## Repo Structure

```
deployment-playbook/
├── README.md                           # This file (English)
├── README.zh.md                        # Chinese version
├── CLAUDE.md                           # AI agent instructions
├── templates/
│   ├── test-report.md                  # Test report template
│   ├── deployment-plan.md              # Deployment plan with checkboxes
│   ├── canary-plan.md                  # Gradual rollout stages + gates
│   └── rollback-plan.md               # Pre-planned recovery steps
├── .github/
│   └── ISSUE_TEMPLATE/
│       └── deployment-request.yml      # GitHub Issue form (all 4 artifacts)
├── examples/
│   └── cloudrun-backend-deploy.md      # Real-world Cloud Run example
├── checklists/
│   ├── pre-deploy.md                   # Gate checks before deploying
│   ├── post-deploy.md                  # Verification after deploying
│   └── incident-response.md           # When a deployment goes wrong
└── CLAUDE.md                           # Agent instructions for using templates
```

## Supported Platforms

Templates are platform-agnostic but include specific guidance for:

| Platform | Canary method | Rollback method | Example |
|----------|--------------|-----------------|---------|
| **Google Cloud Run** | Traffic splitting (revision %) | Revision revert (~30s) | Included |
| **GCE / VM pools** | Rolling restart (1→N) | Image family revert | Included |
| **Kubernetes** | Istio/Linkerd traffic shifting | Rollback deployment | Guidance |
| **Serverless** (Functions/Lambda) | Alias-based canary | Alias revert | Guidance |
| **Static hosting** (Vercel/CF Pages) | Preview deployments | `vercel rollback` | Guidance |

## Workflow Diagram

```
              ┌─────────────┐
              │  Code Ready  │
              │  (PR merged) │
              └──────┬───────┘
                     │
              ┌──────▼───────┐
              │ Test Report   │◄── Must be PASS
              │ (Artifact 1)  │
              └──────┬───────┘
                     │
              ┌──────▼───────┐
              │ Deploy Plan   │◄── All steps as checkboxes
              │ (Artifact 2)  │
              └──────┬───────┘
                     │
              ┌──────▼───────┐
              │ Canary Plan   │◄── Staged rollout + gates
              │ (Artifact 3)  │
              └──────┬───────┘
                     │
              ┌──────▼───────┐
              │ Rollback Plan │◄── Pre-planned, pre-tested
              │ (Artifact 4)  │
              └──────┬───────┘
                     │
              ┌──────▼───────┐     ┌──────────────┐
              │  Pre-Deploy   │────►│   Approved?   │
              │  Checklist    │     └──────┬───────┘
              └───────────────┘            │
                                    ┌──────▼───────┐
                                    │    Deploy     │
                                    │   (canary)    │
                                    └──────┬───────┘
                                           │
                                ┌──────────▼──────────┐
                           YES  │   Metrics OK?       │  NO
                          ┌─────┤   (go/no-go gate)   ├─────┐
                          │     └─────────────────────┘     │
                   ┌──────▼───────┐              ┌──────────▼─────────┐
                   │  Full rollout │              │  Execute rollback  │
                   └──────┬───────┘              └──────────┬─────────┘
                          │                                 │
                   ┌──────▼───────┐              ┌──────────▼─────────┐
                   │  Post-Deploy  │              │  Incident Response │
                   │  Checklist    │              │  + Post-Mortem     │
                   └──────────────┘              └────────────────────┘
```

## Principles

1. **No deployment without a plan.** Even "quick fixes" get a lightweight plan.
2. **Rollback is always pre-planned.** Never figure out rollback during an incident.
3. **Canary before full rollout.** 100% traffic shift without canary is an incident waiting to happen.
4. **Automated where possible, documented where not.** If a step is manual, it's in a checkbox.
5. **Post-mortems feed templates.** After every incident, update the relevant checklist.
6. **Blameless culture.** Incidents are system failures, not people failures.

## FAQ

**Q: Do I need all 4 artifacts for dev/staging deploys?**
A: No. Dev deploys need only the Deployment Plan. Staging needs Deployment Plan + Test Report. Production needs all 4.

**Q: What if my change is tiny (one-line fix)?**
A: Use a lightweight version: Test Report (just verdict line) + Deployment Plan (3-4 checkboxes) + Rollback Plan (just the command). Canary can be "N/A — hotfix, direct rollout with monitoring."

**Q: Can I skip canary for urgent hotfixes?**
A: Only if SEV-1 (service down). Document the skip in the deployment plan with justification. Do a post-deploy canary analysis after the fire is out.

**Q: How do I adapt this for my project?**
A: Fork this repo, customize the templates (especially deploy commands and metric thresholds), and update the GitHub Issue template. The structure stays the same.

**Q: Can AI agents use this?**
A: Yes. Add the repo URL to your agent's context (CLAUDE.md, .cursorrules, etc.). See Method 3 above.

## Contributing

1. Fork this repo
2. Edit or add templates under `templates/` or `checklists/`
3. Open a PR with a clear description of what changed and why
4. At least one reviewer must approve

## License

MIT

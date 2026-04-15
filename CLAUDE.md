# Deployment Playbook — Agent Instructions

## What This Repo Is

Standardized deployment SOP templates for Matrix Labs projects. Every production deployment must produce 4 artifacts: Test Report, Deployment Plan, Canary Plan, Rollback Plan.

## How to Use These Templates

When a user asks you to generate a deployment plan:

1. **Read the relevant templates** from `templates/` directory
2. **Fill in the template** with project-specific details (service name, version, commands, thresholds)
3. **Output as a GitHub Issue** using the format from `.github/ISSUE_TEMPLATE/deployment-request.yml`
4. **Include all 4 artifacts** — none may be skipped
5. **Default language is Chinese** — issue titles, section headers, and skeleton content are in Chinese. Keep technical terms (commands, env names) in English.

## Rules

- NEVER skip the rollback plan, even for "small" changes
- NEVER approve a deployment without a test report verdict
- Canary plan may be "N/A" ONLY for dev/staging environments — production always requires canary
- Use the project's actual deploy commands (e.g., `deploy.sh`, not raw `gcloud run deploy`)
- Populate metric thresholds from the project's real baseline, not generic values
- Reference the `checklists/` for pre-deploy and post-deploy verification

## Example Prompt

User: "Generate a deployment plan for backend v1.5.3 to production"

You should:
1. Read `templates/deployment-plan.md`, `templates/test-report.md`, `templates/canary-plan.md`, `templates/rollback-plan.md`
2. Fill in with the project's backend service details
3. Output a complete deployment request with all 4 sections

# Development Workflow: Intake to Production

This document outlines the end-to-end development workflow, covering each stage from initial idea intake through deployment to production.

---

## Overview

```
Intake → Azure DevOps → GitHub Copilot → UAT → Production
```

### Automation Architecture

The table below shows where automation takes over and where a human approval gate is required.

| Stage | Automation | Human Quality Gate |
|---|---|---|
| Intake | GitHub Issue templates enforce structured request data | Product owner / stakeholder must triage and approve before work begins |
| Planning | Azure DevOps Boards tracks work; items are linked to PRs | Sprint planning sign-off by product owner |
| Development | CI workflow runs lint, build, and tests on every PR | **Code review:** at least one CODEOWNER approval required before merge (branch protection) |
| UAT Deployment | `deploy-uat.yml` triggers automatically after merge to `main` | **Environment approval:** a designated reviewer must approve the `uat` GitHub Environment before deployment proceeds |
| UAT Testing | Automated smoke tests run against the UAT environment after deployment | **UAT sign-off:** product owner or designated tester formally approves the release candidate |
| Production Deployment | `deploy-production.yml` is triggered (manually or automatically); reruns the full test suite before deploying | **Environment approval:** a designated reviewer must approve the `production` GitHub Environment; creates a GitHub Release automatically |
| Post-release | Post-deployment smoke tests and stakeholder notification run automatically | On-call engineer monitors dashboards; manual rollback step available |

### Files Added

```
.github/
├── CODEOWNERS                          # Enforces required code reviewers
├── PULL_REQUEST_TEMPLATE.md            # Consistent PR quality checklist
├── ISSUE_TEMPLATE/
│   ├── feature_request.yml             # Structured intake for new features
│   └── bug_report.yml                  # Structured intake for defects
└── workflows/
    ├── ci.yml                          # Lint → Build → Test on every PR
    ├── deploy-uat.yml                  # Deploy to UAT after merge to main
    └── deploy-production.yml           # Deploy to production (manual sign-off)
```

### Required One-Time GitHub Setup

The following settings must be configured once by a repository administrator for the automation to enforce human gates:

1. **Branch protection on `main`**
   - Require a pull request before merging
   - Require approvals: 1 (or more)
   - Require review from Code Owners
   - Require status checks to pass: `lint`, `build`, `test`

2. **`uat` GitHub Environment**  _(Settings → Environments → New environment)_
   - Required reviewers: add the UAT approver(s)
   - (Optional) Deployment branches: `main` only
   - Add environment variable: `UAT_URL`
   - Add environment secret: `UAT_DEPLOY_TOKEN`

3. **`production` GitHub Environment**  _(Settings → Environments → New environment)_
   - Required reviewers: add the production approver(s)
   - (Optional) Wait timer: e.g. 30 minutes after UAT sign-off
   - Deployment branches: `main` only
   - Add environment variable: `PRODUCTION_URL`
   - Add environment secret: `PROD_DEPLOY_TOKEN`

---

---

## Stage 1: Intake

**Goal:** Capture, triage, and prioritize incoming work.

- **Sources of intake** can include:
  - Customer feedback or support tickets
  - Internal stakeholder requests
  - Bug reports
  - Regulatory or compliance requirements
  - Strategic initiatives

- **Activities:**
  1. Log the request with enough detail to be actionable (description, business value, acceptance criteria, priority).
  2. Triage and assign a severity or priority level.
  3. Review in a regular intake meeting (e.g., weekly backlog refinement).
  4. Obtain sign-off from a product owner or stakeholder before moving the item forward.

- **Output:** An approved, clearly described work item ready for planning.

---

## Stage 2: Azure DevOps

**Goal:** Plan, track, and manage the work.

- **Activities:**
  1. Create a **Work Item** (Epic, Feature, User Story, Bug, or Task) in Azure Boards with:
     - Title and description
     - Acceptance criteria
     - Priority and story points/effort estimate
     - Linked parent items (Epic → Feature → Story)
  2. Add the item to the appropriate **Sprint** or **Iteration** during sprint planning.
  3. Use **Azure Repos** (or link to GitHub) to track branch and PR activity against the work item.
  4. Monitor progress on the **Azure Boards Kanban/Sprint board** (columns: New → Active → In Review → Done).
  5. Use **Azure Pipelines** to define CI/CD pipelines for build, test, and deployment gates.

- **Best practices:**
  - Keep work items small enough to complete within a single sprint.
  - Attach designs, mockups, or specification documents as attachments or links.
  - Use tags to enable reporting and filtering.

- **Output:** A sprint-committed work item with a linked branch/PR and defined CI/CD pipeline.

---

## Stage 3: Development with GitHub Copilot

**Goal:** Implement the feature or fix efficiently with AI assistance.

- **Activities:**
  1. **Branch** from `main` (or the designated integration branch) using a naming convention tied to the Azure DevOps work item (e.g., `feature/AB#1234-short-description`).
  2. Use **GitHub Copilot** throughout development to:
     - Generate boilerplate and scaffolding.
     - Suggest implementations for functions, classes, and tests.
     - Explain unfamiliar code or third-party APIs.
     - Refactor and improve code quality.
     - Write unit and integration tests alongside the implementation.
  3. Commit frequently with clear, atomic commit messages.
  4. Open a **Pull Request (PR)** against the integration/main branch:
     - Link the PR to the Azure DevOps work item.
     - Fill in the PR template (summary, testing notes, screenshots where applicable).
     - Request code review from relevant teammates.
  5. Address review feedback, update the PR, and ensure all CI checks pass (build, lint, unit tests).
  6. Merge the PR once approved and CI is green.

- **Copilot tips:**
  - Use inline comments to guide Copilot's suggestions (e.g., `// Function that validates an email address and returns a boolean`).
  - Review every suggestion critically — Copilot accelerates but doesn't replace developer judgment.
  - Use Copilot Chat for deeper explanations or architectural questions.

- **Output:** Merged, reviewed, and tested code in the integration branch; CI pipeline passing.

---

## Stage 4: User Acceptance Testing (UAT)

**Goal:** Validate that the implementation meets business requirements before production release.

- **Activities:**
  1. **Deploy** the integration branch to the UAT environment via the CI/CD pipeline (Azure Pipelines / GitHub Actions).
  2. Share the UAT environment URL and test credentials with stakeholders/testers.
  3. Testers execute **UAT test cases** derived from the acceptance criteria defined at intake.
  4. Log any defects or change requests back as Azure DevOps work items linked to the original story.
  5. Developers fix defects, redeploy to UAT, and re-test.
  6. Obtain **formal sign-off** from the product owner or designated UAT approver.

- **Best practices:**
  - Keep UAT as close to production configuration as possible (same data shape, feature flags, integrations).
  - Time-box UAT rounds to prevent indefinite feedback loops.
  - Automate regression tests that run on every UAT deployment to catch regressions quickly.

- **Output:** Signed-off release candidate ready for production deployment.

---

## Stage 5: Production Deployment

**Goal:** Release the signed-off change to end users safely and reliably.

- **Activities:**
  1. Create a **release** in Azure DevOps / GitHub Releases with release notes summarizing changes.
  2. Use the CI/CD pipeline to deploy to production, leveraging:
     - **Approval gates** — require a manual sign-off step in the pipeline before deploying.
     - **Blue/green or canary deployments** where appropriate to reduce blast radius.
     - **Feature flags** to enable controlled rollout or instant rollback without redeployment.
  3. Monitor post-deployment health with dashboards, alerts, and logs (e.g., Application Insights, Azure Monitor).
  4. Confirm the work item status is updated to **Done** in Azure Boards.
  5. Communicate the release to stakeholders (release notes email, changelog, etc.).

- **Rollback plan:**
  - Document a rollback procedure before every deployment.
  - Ensure the previous build artifact is retained and the pipeline has a one-click rollback step.

- **Output:** Change live in production, monitored, and work item closed.

---

## Summary Table

| Stage | Tooling | Key Output |
|---|---|---|
| Intake | Ticketing / email / meetings | Approved, prioritized work item |
| Azure DevOps | Azure Boards + Azure Pipelines | Sprint-planned item, CI/CD pipeline |
| GitHub Copilot | GitHub + Copilot | Merged, tested code |
| UAT | UAT environment + test scripts | Stakeholder sign-off |
| Production | CI/CD pipeline + monitoring | Live, monitored release |

---

## Tips for a Smooth Flow

- **Shift left on quality** — write tests and perform code review early so issues surface before UAT.
- **Keep work items small** — smaller stories move through the pipeline faster and are easier to test and roll back.
- **Automate everything repeatable** — builds, tests, deployments, and environment provisioning should all be scripted.
- **Use Copilot for test coverage** — generating comprehensive tests is one of Copilot's strongest use cases and pays dividends throughout UAT and production.
- **Establish clear definition of done (DoD)** — everyone should agree on what "done" means at each stage before work begins.

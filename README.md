# super-duper-broccoli
Let it ride

## Development Workflow

See [WORKFLOW.md](./WORKFLOW.md) for our end-to-end development process:

**Intake → Azure DevOps → GitHub Copilot → UAT → Production**

### Automation at a glance

| Trigger | Workflow | Human Gate |
|---|---|---|
| Open / update a PR | [CI](.github/workflows/ci.yml) — lint, build, test | PR review approval (CODEOWNER) |
| Merge to `main` | [Deploy to UAT](.github/workflows/deploy-uat.yml) | `uat` Environment approval |
| Manual / post-UAT | [Deploy to Production](.github/workflows/deploy-production.yml) | `production` Environment approval |

See [WORKFLOW.md](./WORKFLOW.md) for setup instructions and full stage details.

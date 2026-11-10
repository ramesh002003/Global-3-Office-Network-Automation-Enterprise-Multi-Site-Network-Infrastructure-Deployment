# CI/CD Pipeline — GitHub Actions

## Full Pipeline Flow

```mermaid
flowchart TD
    DEV["👨‍💻 Developer<br/>Push to feature/*"] --> PR["📋 Pull Request<br/>Created"]

    PR --> LINT["🔍 Lint<br/>ansible-lint + yamllint"]
    LINT -->|Fail| FIX["❌ Block<br/>Notify Developer"]
    FIX --> DEV

    LINT -->|Pass| SYNTAX["📝 Syntax Check<br/>ansible-playbook --syntax-check"]
    SYNTAX -->|Fail| FIX
    SYNTAX -->|Pass| DRYRUN["🧪 Dry Run<br/>--check mode (staging)"]

    DRYRUN -->|Fail| FIX
    DRYRUN -->|Pass| REVIEW{"👀 Code Review<br/>1 Approval Required"}
    REVIEW -->|Rejected| FIX
    REVIEW -->|Approved| MERGE["🔀 Merge to develop"]

    MERGE --> STAGING["🚀 Deploy to STAGING<br/>(EVE-NG / Lab)"]
    STAGING --> INTEG["🔗 Integration Tests<br/>(Molecule)"]
    INTEG -->|Fail| ROLLBACK["⏪ Rollback<br/>Hotfix branch"]
    INTEG -->|Pass| RELEASE["🏷️ Tag Release<br/>vX.Y.Z"]

    RELEASE --> STAGING_FULL["🚀 Full Staging Deploy<br/>(All Playbooks)"]
    STAGING_FULL --> VALIDATE["✅ Validation<br/>(STP, LACP, BGP, SD-WAN)"]
    VALIDATE -->|Fail| ROLLBACK
    VALIDATE -->|Pass| APPROVE{"🔐 Production<br/>Approval Gate"}

    APPROVE -->|Approved| PROD["🚨 Deploy to PRODUCTION<br/>(Serial: 5 at a time)"]
    APPROVE -->|Rejected| WAIT["⏳ Wait"]

    PROD --> PROD_VALIDATE["✅ Production Validation"]
    PROD_VALIDATE --> BACKUP["💾 Config Backup<br/>Git Commit + Push"]
    BACKUP --> DONE["🎉 Deployment Complete"]
```

## GitHub Actions Workflow Matrix

```mermaid
flowchart LR
    subgraph TRIGGERS["Triggers"]
        PUSH["Push to main"]
        PR["Pull Request"]
        SCHEDULE["Cron (Nightly)"]
        MANUAL["Manual Dispatch"]
    end

    subgraph JOBS["Jobs"]
        LINT_JOB["lint"]
        VALIDATE_JOB["validate"]
        DEPLOY_STAGING["deploy-staging"]
        DEPLOY_PROD["deploy-production"]
        BACKUP_JOB["backup-configs"]
        AUDIT_JOB["nightly-audit"]
    end

    subgraph GATES["Gates"]
        STATUS["Required Status Checks"]
        ENV["Environment Protection"]
        APPROVAL["Manual Approval"]
    end

    PUSH --> LINT_JOB
    PR --> LINT_JOB
    LINT_JOB --> STATUS
    STATUS --> VALIDATE_JOB
    VALIDATE_JOB --> DEPLOY_STAGING
    DEPLOY_STAGING --> ENV
    ENV --> DEPLOY_PROD
    DEPLOY_PROD --> APPROVAL
    APPROVAL --> BACKUP_JOB
    SCHEDULE --> AUDIT_JOB
    MANUAL --> DEPLOY_PROD
```

## Environment Protection Rules

| Environment | Protection | Approval Required |
|-------------|-----------|-------------------|
| `staging` | Required status checks: `lint`, `validate` | Auto (on merge to develop) |
| `production` | Required status checks + environment | **2 reviewers** from CODEOWNERS |
| `emergency` | No checks (hotfix) | 1 reviewer + CTO |   

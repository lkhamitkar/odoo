CI/CD Pipeline – Jenkinsfile

This repository uses a Jenkins Declarative Pipeline integrated with GitHub Webhooks for automated CI/CD.
The pipeline enforces branching rules, PR validation, and multi-stage quality gates to ensure code quality and controlled releases.

🔧 Pipeline Configuration
General Setting

Trigger: githubPush()

Builds are triggered via GitHub webhooks when a PR or branch update occurs.

Agent: Runs on nodes labeled dev.

Options:

timestamps() → logs include timestamps.

timeout(90 minutes) → prevents stuck builds.

buildDiscarder(logRotator(numToKeepStr: '50')) → retains the last 50 builds.

disableConcurrentBuilds() → avoids overlapping builds for the same job.

Parameters

RELEASE_VERSION: release version string (used during production deploys).

Environment Variables

ARTIFACTORY_SERVER: Artifactory server ID.

ARTIFACTORY_REPO: target repository (odoo-custom-modules-local).

ARTIFACT_VERSION: defaults to Jenkins build number.

MODULES: list of Odoo custom modules to build/test.

📦 Pipeline Stages
1. Checkout from SCM

Checks out the repository code from GitHub.

Prints last 3 commits for debugging.

Logs the current branch name (env.BRANCH_NAME).

2. Validate Branch Rules

Enforces strict branch and PR flows:

✅ Allowed PR flows:

feature/* → develop

develop → staging

staging → main

hotfix/* → main

❌ Any other branch → target combinations are rejected.

The build fails with an error:

Invalid PR flow detected: <source> → <target>

3. Static Analysis and Security Scan

Runs only for:

PRs targeting develop.

Hotfix PRs (hotfix/* → main).

Parallel checks:

Python Linting

Executes static analysis tools (e.g., pylint, black).

Manifest & XML Validation

Ensures module metadata and XML files are valid.

4. Target-specific Test Stages

PR to develop: Runs tests for feature integration.

PR to staging: Runs tests for staging release validation.

PR to main: Runs tests for production readiness.

Each stage runs only if env.CHANGE_TARGET matches the branch.

5. Manual Approval (Production)

Triggered only for PRs targeting main.

Requires human approval before deployment:

input prompt with 2-hour timeout.

Restricted to authorized submitter (name-release).

Prevents accidental production deployments.

✅ Post-build Actions

On Success:

Sends GitHub commit status → CI/CD Pipeline: SUCCESS.

Logs success with branch name.

On Failure:

Sends GitHub commit status → CI/CD Pipeline: FAILURE.

Logs failure with branch name.

🔒 Branching & PR Policy (Enforced by Pipeline)

Developers must:

Create feature/* branches for new work.

Open PRs only into develop.

Deployment flow:

feature/* → develop → staging → main

Exception:

hotfix/* → main (for urgent fixes, later merged back into develop).

Any deviation fails the build.

🚀 Summary

This pipeline ensures:

Enforced Git workflow and branching strategy.

Automated code quality checks and static analysis.

Controlled promotion of code across environments (develop → staging → main).

Mandatory manual approval for production releases.

GitHub integration with status reporting for CI/CD visibility.
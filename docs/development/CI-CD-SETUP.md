# CI/CD Setup Guide

This guide covers the GitHub Actions workflows, security scanning, and automated dependency management configured for Agentic-Flow.

## Table of Contents

- [Overview](#overview)
- [GitHub Actions Workflows](#github-actions-workflows)
  - [Node.js CI (`ci.yml`)](#nodejs-ci-ciyml)
  - [Security Scanning (`security.yml`)](#security-scanning-securityyml)
- [Dependabot](#dependabot)
- [Interpreting Workflow Results](#interpreting-workflow-results)
- [Running Checks Locally](#running-checks-locally)

---

## Overview

The repository uses three automated pipeline configurations:

| File | Purpose |
|------|---------|
| `.github/workflows/ci.yml` | Lint, build, test, and audit on every push and pull request |
| `.github/workflows/security.yml` | CodeQL analysis, secret detection, and dependency review |
| `.github/dependabot.yml` | Automated dependency update pull requests |

All three are triggered automatically. No manual configuration is needed to use them after cloning the repository and opening a pull request.

---

## GitHub Actions Workflows

### Node.js CI (`ci.yml`)

Runs on:
- Every push to `main`, `master`, or `develop`
- Every pull request targeting those branches
- Manual dispatch via the GitHub Actions UI

#### Jobs

**`lint` — Lint & Format**

Runs ESLint and Prettier on all TypeScript, JavaScript, and JSX/TSX files.

- Uses Node.js 22
- Fails the build on any ESLint warning (`--max-warnings=0`)
- Prettier check is informational (exits without failing) when no Prettier config is present

**`build` — Build**

Runs a matrix build across Node.js versions **20** and **22** on Ubuntu, macOS, and Windows.

- Installs dependencies with `npm ci`, `yarn install --frozen-lockfile`, or `pnpm install --frozen-lockfile` depending on the lock file present
- Runs `npm run build` (skipped gracefully when no build script exists)
- Runs `npx tsc --noEmit` for type checking when `tsconfig.json` is present

**`test` — Test**

Runs after `lint` passes (using `needs: [lint]`).

- Uses Node.js 22 on Ubuntu
- Runs `npm test -- --coverage --passWithNoTests`
- Uploads coverage to [Codecov](https://codecov.io/) (failures are non-blocking)

**`audit` — Security Audit**

Runs `npm audit --audit-level=high` when a `package-lock.json` is present. Findings are reported but do not block merges.

#### Concurrency

A single instance of this workflow runs per branch at a time. New pushes automatically cancel any in-progress run for the same branch.

---

### Security Scanning (`security.yml`)

Runs on:
- Every push to `main` or `master`
- Every pull request targeting those branches
- Weekly schedule — Mondays at 09:00 UTC
- Manual dispatch via the GitHub Actions UI

#### Jobs

**`codeql` — CodeQL Analysis**

Uses [GitHub CodeQL](https://codeql.github.com/) to perform static analysis for security vulnerabilities and code quality issues.

- Scans JavaScript/TypeScript, Python, and Go sources
- Uses the `security-and-quality` query suite
- Results appear under **Security → Code scanning alerts** in the repository

**`secret-scan` — Secret Detection**

Uses [TruffleHog](https://github.com/trufflesecurity/trufflehog) to detect committed secrets or credentials in the full git history.

- Scans the entire commit range between the default branch and HEAD
- Only reports verified (confirmed active) secrets

**`dependency-review` — Dependency Review**

Runs on pull requests only. Uses [GitHub's Dependency Review Action](https://github.com/actions/dependency-review-action) to block merges when a PR introduces:

- Packages with known vulnerabilities rated **high** or above
- Packages with GPL-3.0 or AGPL-3.0 licenses

---

## Dependabot

Dependabot is configured in `.github/dependabot.yml` to open automated update pull requests every **Monday at 09:00 ET** for five ecosystems:

| Ecosystem | Scope | PR limit |
|-----------|-------|----------|
| `github-actions` | `.github/workflows/` | 5 open PRs |
| `npm` | `/` (root) | 10 open PRs |
| `pip` | `/` (root) | 10 open PRs |
| `gomod` | `/` (root) | 10 open PRs |
| `docker` | `/` (root) | 5 open PRs |

### Update strategy

- **Minor and patch** updates are grouped into a single PR per ecosystem to reduce noise.
- **Major** version updates are **excluded** (you must opt in manually).
- All Dependabot commits use the conventional commit prefix `chore(deps):` for npm/pip/go, `chore(docker):` for Docker, and `ci:` for Actions.

### Reviewing Dependabot PRs

1. Open the PR opened by `dependabot[bot]`.
2. Review the changelog and release notes linked in the PR description.
3. Check that CI passes (all jobs in `ci.yml` are green).
4. If the Dependency Review job is present, verify no new vulnerabilities are introduced.
5. Approve and merge, or request changes as needed.

---

## Interpreting Workflow Results

### All checks are green ✅

The PR is safe to review and merge from a CI perspective.

### `lint` fails

Run the following locally and commit the fixes:

```bash
npm run lint:fix
npm run format
```

### `build` fails on a specific platform

Check the job log for the failing OS/Node.js combination. Common causes:

- Shell-specific syntax in `package.json` scripts (use `cross-env` for cross-platform env vars)
- Missing peer dependencies
- Platform-specific file paths (`path.join` vs hard-coded `/`)

### `test` fails

Run tests locally to reproduce:

```bash
npm test
```

For coverage failures:

```bash
npm run test:coverage
```

### CodeQL alert raised

Navigate to **Security → Code scanning alerts**, open the alert, and follow the remediation guidance. Fix the identified issue and push a new commit — CodeQL re-runs automatically.

### TruffleHog secret detected

If TruffleHog flags a real secret:

1. **Rotate the credential immediately** — assume it is compromised.
2. Remove the secret from the code and history (use `git filter-repo` or contact GitHub support for assistance).
3. Add the value to your secrets manager and reference it via an environment variable or GitHub Actions secret.

### Dependency Review blocks a PR

Review the introduced dependency:

- If the vulnerability is a false positive or already mitigated, document the justification and request a maintainer override.
- Otherwise, upgrade to a patched version or remove the dependency.

---

## Running Checks Locally

You can reproduce most CI checks in your local environment before pushing:

```bash
# Lint
npm run lint

# Format check
npm run format:check

# Type check
npx tsc --noEmit

# Tests with coverage
npm run test:coverage

# Security audit
npm audit --audit-level=high

# Build
npm run build
```

For CodeQL and TruffleHog scans, use the GitHub Actions environment (push to a branch and review the Security tab results) or run the respective CLIs locally if you have them installed.

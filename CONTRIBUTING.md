# Contributing to Xconfess

Thank you for your interest in contributing to Xconfess - an anonymous confession platform built on the Stellar blockchain. This guide covers everything you need to get started.

---

## Table of Contents.

- [Prerequisites](#prerequisites)
- [Environment Setup](#environment-setup)
- [Development Workflow](#development-workflow)
- [Code Style](#code-style)
- [Testing Requirements](#testing-requirements)
- [Accessibility Contributor Test Harness](#accessibility-contributor-test-harness)
- [Validation Command Matrix](#validation-command-matrix)
- [Pull Request Process](#pull-request-process)
- [Wave / Drips Contribution Guidelines](#wave--drips-contribution-guidelines)

---
.
## Prerequisites

Make sure you have the following installed before cloning:

| Tool | Minimum Version | Purpose |
|------|----------------|---------|
| Node.js | 18.0.0+ | Backend + Frontend |
| npm | 9.0.0+ | Package manager |
| Rust | stable (latest) | Smart contracts |
| Docker + Docker Compose | 24.0+ | PostgreSQL + Redis |
| Git | any recent | Version control |

### Install Rust (if not installed)

    curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
    rustup target add wasm32-unknown-unknown

### Install Stellar CLI

    cargo install --locked stellar-cli

---

## Environment Setup

### 1. Clone the repository

    git clone https://github.com/Xconfess/Xconfess.git
    cd Xconfess

### 2. Install dependencies

    npm install

This installs dependencies for all three workspaces:
- xconfess-backend - NestJS API
- xconfess-frontend - React frontend
- xconfess-contracts - Soroban smart contracts

### 3. Set up environment variables

    cp xconfess-backend/.env.example xconfess-backend/.env

Open xconfess-backend/.env and fill in the required values. At minimum:

    DATABASE_URL=postgresql://postgres:postgres@localhost:55432/xconfess
    REDIS_URL=redis://localhost:6379

### 4. Start Docker services (PostgreSQL + Redis)

    docker compose up -d

Verify both services are healthy:

    docker compose ps

Both xconfess-postgres and xconfess-redis should show healthy.

### 5. Start the development server

    # Run backend + frontend together
    npm run dev

    # Or run separately
    npm run dev:backend
    npm run dev:frontend

---

## Development Workflow

### Picking an OSS issue

If you are contributing through an OSS campaign or grant program:

- Start with issues labeled `good first issue`, `help wanted`, `Stellar Wave`, `Maybe Rewarded`, or `Official Campaign`.
- Comment on the issue before starting so maintainers can confirm it is still available.
- Keep the PR focused on one issue. Do not bundle unrelated cleanup.
- Follow the acceptance criteria and validation commands listed in the issue.
- Ask for clarification in the issue thread instead of guessing when scope is unclear.

Recommended first picks:

| Area | Good starter work |
|------|-------------------|
| Docs | Setup guides, troubleshooting, diagrams, screenshots |
| Frontend | Empty states, auth polish, proxy route tests, mobile fixes |
| Backend | DTO validation, error handling, health checks, focused tests |
| Stellar | Contract metadata checks, docs, testnet smoke scripts |
| Ops | CI checks, deployment preflight, env validation |

### Branch Naming

Always branch off main. Use this naming convention:

| Type | Pattern | Example |
|------|---------|---------|
| Feature | feat/short-description | feat/gdpr-export |
| Bug fix | fix/short-description | fix/token-expiry |
| Tests | test/short-description | test/audit-events |
| Docs | docs/short-description | docs/contributing |
| Refactor | refactor/short-description | refactor/auth-module |

    git checkout main
    git pull origin main
    git checkout -b feat/your-feature-name

### Commit Messages

Follow the Conventional Commits format:

    <type>(<scope>): <short summary>
    [optional body]
    [optional footer]

Types: feat, fix, test, docs, refactor, chore, perf

Examples:

    feat(backend): add GDPR data export endpoint
    fix(contracts): correct token expiry calculation
    test(backend): add audit event unit tests
    docs: add CONTRIBUTING.md

Rules:
- Summary line under 72 characters
- Use present tense (add not added)
- Reference issues in footer: Closes #123

### PR Size Policy

- Ideal: under 400 lines changed
- Maximum: 800 lines changed
- If your change is larger, split it into multiple PRs

---

## Code Style

### Backend (TypeScript / NestJS)

    npm run backend:lint

Prettier config (xconfess-backend/.prettierrc): single quotes, trailing commas, 2-space indentation.

### Frontend (TypeScript / React)

    npm run frontend:lint

### Smart Contracts (Rust / Soroban)

    npm run contract:fmt
    npm run contract:fmt:check
    npm run contract:lint

All Rust code must pass rustfmt and clippy before submission.

---

## Testing Requirements

| Area | Command | Required |
|------|---------|----------|
| Backend unit tests | npm run backend:test | Yes |
| Contract tests | npm run contract:test | Yes |
| Frontend tests | npm run frontend:test | Yes |
| Backend E2E tests | npm run backend:test:e2e | Optional |
| Contract integration | npm run contract:test:integration | Optional |
| Frontend smoke tests | npm run frontend:test:smoke | Optional |

Run all required tests at once:

    npm run test

Run the full CI check locally before opening a PR:

    npm run ci

All CI checks must pass before a PR will be reviewed.

---

## Accessibility Contributor Test Harness

Contributors can run a consistent set of accessibility checks locally against the frontend routes. The harness is wired into the standard frontend validation commands, so a clean checkout can run it without extra setup beyond `npm install`.

### What it covers

- Automated axe-core rule checks against the frontend routes using the shared fixtures in `xconfess-frontend/tests/a11y/fixtures`.
- A Playwright-driven browser pass that exercises the same routes in a real browser context.
- Output that names both the route under test and the violated rule, so failures are triageable without re-running by hand.

### Running the harness

From the repository root, after `npm install`:

    # Static + component-level accessibility checks
    npm run frontend:lint
    npm run frontend:test

    # Browser-level accessibility checks
    npm run frontend:test:e2e

These are the same commands listed in the issue validation block, so a passing run here is the expected signal for a PR that touches frontend accessibility.

### Browser setup

- The browser pass uses Playwright. Install the browser binaries once per machine:

      npx playwright install --with-deps chromium

- If you are running in a container or CI image, use the `--with-deps` flag so system libraries are installed alongside the browser.
- The harness targets Chromium by default. Other browsers can be added locally, but Chromium is the supported baseline for contributor runs.

### Reading the output

Each failure is reported with the route and the rule that failed, for example:

    [a11y] route=/confessions rule=color-contrast
    [a11y] route=/profile rule=label

Use the route to locate the page and the rule name to look up the corresponding axe-core rule documentation before making changes.

### Known limitations

- Automated checks cannot verify screen reader announcements, focus order intent, or the quality of alt text. These still require manual review.
- Dynamic content that only appears after user interaction may not be exercised by the default fixtures.
- Third-party embeds and iframes are excluded from the automated pass.

### When manual testing is required

Run a manual pass, in addition to the harness, whenever a change touches:

- Keyboard navigation, focus management, or modal/dialog behavior.
- Form validation messaging or error announcements.
- Color, contrast, or theming tokens.
- Any interactive flow that the fixtures do not cover.

### Failure triage

1. Re-run the failing command with the route filter to confirm the failure is reproducible.
2. Check whether the rule is a genuine violation or a fixture gap; if the fixture is missing state, extend it rather than suppressing the rule.
3. Fix the underlying markup or styling. Do not disable rules globally to make the harness pass.
4. If a rule cannot be satisfied for a documented reason, note the exception in the PR description with the route and rule name.

---

## Validation Command Matrix

Not sure which commands to run for your change? Refer to the **[Validation Command Matrix](docs/VALIDATION_COMMAND_MATRIX.md)** for a complete table mapping each change area (docs, frontend component, frontend route, backend service, migration, Stellar contract, ops script) to the exact copy-pasteable commands required.

---

## Pull Request Process

### Before opening a PR

- Branch is up to date with main
- npm run ci passes locally
- New code has corresponding unit tests
- No console.log or debug statements left in
- Environment variables are documented in .env.example if added

### PR Title

Use Conventional Commits format: feat(backend): add GDPR data export endpoint

### PR Description Template

    ## Summary
    Brief description of what this PR does.

    ## Changes
    - List of specific changes made

    ## Testing
    - How was this tested?
    - Which test commands were run?

    ## Related Issues
    Closes #<issue-number>

### Auto-merge Criteria

- All CI checks pass
- At least 1 approving review from a maintainer
- No unresolved review comments
- PR is not marked as Draft

---

## Wave / Drips Contribution Guidelines

Xconfess participates in multiple OSS programs, including Stellar Wave. If your contribution is tied to a program issue:

- Reference the issue number in your PR description
- Keep each program contribution as a single focused PR - one issue, one PR
- Do not bundle multiple program issues into one PR
- Ensure your implementation matches the acceptance criteria listed in the issue exactly
- Add or update tests that validate the acceptance criteria

### Contribution checklist for program PRs

- Branch named after the feature area
- PR title references the feature area
- All acceptance criteria from the issue are met
- Tests cover the new behaviour
- No unrelated changes are bundled in
- npm run ci passes

---

## Getting Help

- Open a GitHub Discussion
- Comment on the relevant issue
- Check existing PRs for examples of similar contributions

We appreciate every contribution, no matter how small. Thank you for helping build Xconfess!

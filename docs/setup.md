# Post-Repo Setup Guide for Monorepo with Dev/Main Branches

This document outlines the steps and configuration needed after setting up a new GitHub repository using a **mono repo style**, where:

- The `main` branch is production-ready.
- The `dev` branch is used for active integration of feature branches.
- All development work is done in `dev`, and `main` only receives changes via pull requests from `dev`.
- GitHub Actions (GHA) is used for CI/CD workflows.

---

## Enable GitHub Discussions & Issues

1. Navigate to your repository on GitHub.
2. Go to **Settings** → **General**.
3. Under **Features**, check the boxes for:
   - [x] Issues
   - [x] Discussions

---

## Branch Protection Rules

### 1. Protect `main` branch

Go to **Settings** → **Branches** → **Branch protection rules**:

- **Branch name pattern**: `main`
- Enable the following:
  - [x] Require a pull request before merging
    - [x] Require approvals (e.g., 1–2 reviewers)
    - [x] Dismiss stale pull request approvals
  - [x] Require status checks to pass before merging (select your GHA checks)
  - [x] Require linear history
  - [x] Include administrators (optional)
  - [x] Restrict who can push to matching branches
    - Allow only:
      - Code owners
      - GitHub Actions bot (if needed)

### 2. Protect `dev` branch

Same steps as above, but:

- **Branch name pattern**: `dev`
- Additionally:
  - [x] Restrict push access
    - Allow only feature branches to be merged via PRs
  - [x] Enforce PR review from code owners
  - [x] Allow code owners to override protection (see below)

---

## CODEOWNERS Setup

1. Create a file at `.github/CODEOWNERS`:
   ```text
   # Example CODEOWNERS
   * @your-team-handle @your-user
   ```

2. This gives listed users/teams:
   - Review responsibility
   - Override ability on protected branches (if configured in branch protection rules)

---

## Feature Branch Workflow

### Rules:

- Create branches off of `dev`:  
  ```bash
  git checkout dev
  git checkout -b feature/your-feature-name
  ```

- Push to remote and open a PR into `dev`.
- After successful review and tests (via GHA), merge into `dev`.

- When `dev` is stable and ready for release, open a PR from `dev` to `main`.

---

## GitHub Actions Setup

- Store your workflows in `.github/workflows/`

- Suggested workflow:
  - Build/test on PR to `dev`
  - Build/deploy on PR merge to `main`

- Example triggers:
  ```yaml
  on:
    pull_request:
      branches:
        - dev
        - main
    push:
      branches:
        - main
  ```

- Use environment-specific secrets and environment protection rules (e.g., for production deployment)

---

## Mono Repo Considerations

If using multiple packages (e.g., via `pnpm`, `turborepo`, or `nx`):

- Set up workspaces in `package.json`
- Organize code into logical subdirectories, for example:

  ```text
  /apps
  /packages
  /scripts
  /libs
  ```

- Configure GHA caching and parallelized builds as needed.

---

## Optional: GitHub Environments

Create environments for `dev`, `staging`, and `production`:

- Add in **Settings → Environments**
- Use with `environment:` in your GHA workflows for secret scoping and approvals

---

## Summary of Protections and Access

| Branch     | Direct Push | PR Required               | Code Owner Override | GHA Deploy |
|------------|-------------|---------------------------|----------------------|------------|
| main       | No          | Yes (from `dev`)          | Yes                  | Yes        |
| dev        | No          | Yes (from `feature/*`)    | Yes                  | Optional   |
| feature/*  | Yes         | No                        | No                   | No         |

---

## Tips

- Regularly prune stale feature branches
- Use labels and templates for issues and PRs
- Optionally configure auto-merge on approved PRs for `dev`

---

## What Not To Do

- Do not push directly to `main` or `dev`
- Avoid long-running feature branches
- Do not merge to `main` without a full integration test from `dev`

---

## Conclusion
Following this guide will help maintain a clean and efficient workflow in your mono repo setup, ensuring that your `main` branch remains stable and production-ready while allowing for active development in the `dev` branch.

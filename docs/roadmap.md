# Astra Build Roadmap

> Working name: **Astra Build** — an Astra-owned fork of `stackblitz-labs/bolt.diy` for a multi-user AI web-app builder with GitHub-backed project workflows and a single managed model/provider path.

## Operating rules

1. **One feature at a time.** Do not start implementation for the next feature until the operator explicitly says to continue.
2. **PR gate after every feature.** Every feature must end with a GitHub pull request for operator review. No direct pushes to the default branch.
3. **Review before continuation.** After opening a PR, stop. Continue only after the operator reviews/approves/merges or explicitly instructs what to change.
4. **Small, reversible changes.** Prefer narrow branches, focused commits, and easy rollback over sweeping rewrites.
5. **Astra-owned provider path.** Users should not manage arbitrary provider keys in the final product. Model access should route through Astra-controlled server-side configuration.
6. **Web stack scope discipline.** MVP supports JavaScript/TypeScript web apps that fit the WebContainer model. Arbitrary VM/container workloads are out of scope until a server-side workspace phase is approved.

## Product target

Astra Build should become a self-hosted, multi-user version of the Bolt-style web app builder:

- users authenticate into Astra Build;
- users create and resume projects from any browser;
- projects persist server-side instead of only in browser storage;
- projects can connect to GitHub repos;
- model calls go through a single Astra-managed provider configuration;
- each major feature ships behind a PR review gate.

## Baseline assumptions from upstream bolt.diy

Upstream currently provides useful foundations:

- Remix/Vite/React application shell;
- WebContainer-based project runtime;
- AI provider registry with OpenAI, Anthropic, Gemini, OpenRouter, OpenAI-like providers, and others;
- browser-stored provider configuration/API keys;
- local/browser project persistence mechanisms;
- GitHub/import/export related code paths;
- Supabase and MCP integrations;
- deployment helpers for Vercel, Netlify, and GitHub Pages.

Known gap for Astra Build:

- upstream is not currently a complete multi-tenant SaaS product;
- project data and provider configuration are tied heavily to browser storage;
- true user/workspace/project ownership must be added;
- GitHub integration must be reviewed and likely hardened for organization/team use.

## Feature sequence

### Feature 0 — Fork hygiene and project identity

**Goal:** Establish the Astra Build fork as a maintainable product branch without changing core behavior.

**Scope:**

- rename visible product references where appropriate;
- add `docs/roadmap.md`;
- document contribution/review workflow;
- verify baseline install/build commands;
- keep upstream sync path clear.

**Likely files:**

- `docs/roadmap.md`
- `README.md`
- package metadata if product rename is required
- any visible app branding files discovered during inspection

**Validation:**

- `pnpm install` or documented package install path;
- `pnpm run typecheck` if dependencies install cleanly;
- `pnpm run build` if environment permits.

**PR gate:** Open PR `docs: add Astra Build roadmap` and stop.

---

### Feature 1 — Authentication foundation

**Goal:** Add a minimal user authentication boundary before project persistence is introduced.

**Recommended approach:** Use a provider that can support production OAuth/passwordless flows without overbuilding custom auth. Candidate choices:

- Supabase Auth if we want to align with existing Supabase integration;
- Auth.js if we want framework-native flexibility;
- Clerk only if managed auth dependency is acceptable.

**Initial acceptance criteria:**

- anonymous users can reach marketing/login surface only;
- authenticated users can reach the builder;
- user identity is available to server routes/loaders/actions;
- auth configuration is environment-driven;
- tests or route-level verification prove protected route behavior.

**Likely files:**

- Remix root/routes for auth entry points;
- server/session/auth utility modules under `app/`;
- environment example files;
- docs for local auth setup.

**PR gate:** Open PR, wait for review, then stop.

---

### Feature 2 — Server-side project model

**Goal:** Persist project records server-side and associate them with authenticated users.

**Scope:**

- define project metadata schema;
- create create/list/get/update/delete project APIs or Remix actions/loaders;
- store ownership via `user_id`;
- keep file/content persistence minimal at first.

**Initial project metadata fields:**

- `id`
- `owner_user_id`
- `name`
- `description`
- `created_at`
- `updated_at`
- `last_opened_at`
- optional `github_repo_owner`
- optional `github_repo_name`
- optional `github_default_branch`

**Validation:**

- user A cannot read user B projects;
- authenticated user can create/list/open own projects;
- unauthenticated requests fail cleanly.

**PR gate:** Open PR, wait for review, then stop.

---

### Feature 3 — Project file snapshot persistence

**Goal:** Persist enough project file state server-side for cross-browser resume.

**Recommended approach:** Start with snapshot persistence before collaborative granular sync.

**Scope:**

- serialize WebContainer project files into a server-side snapshot;
- load latest snapshot when opening a project;
- store snapshots in database JSON/blob or object storage, based on size limits discovered during implementation;
- retain browser storage only as cache/offline fallback, not source of truth.

**Validation:**

- create project in browser/session A;
- save snapshot;
- open in fresh browser/session B;
- same files restore;
- owner isolation still holds.

**PR gate:** Open PR, wait for review, then stop.

---

### Feature 4 — Astra-managed single provider gateway

**Goal:** Remove user-managed provider/API-key selection from the MVP path and route all model calls through Astra-managed configuration.

**Scope:**

- identify current provider selection flow;
- add server-side model gateway/config abstraction;
- force a default provider/model from environment;
- hide or disable user-facing provider key settings for MVP;
- preserve upstream provider code where possible for future admin-only use.

**Initial provider candidates:**

- direct OpenAI API provider;
- OpenAI-compatible proxy;
- Hermes proxy only if Codex OAuth/subscription routing is explicitly approved later.

**Validation:**

- user cannot enter arbitrary API key in normal UI;
- prompt requests use configured server provider;
- missing provider config fails with a clear admin-facing error;
- no provider secrets are shipped to the browser.

**PR gate:** Open PR, wait for review, then stop.

---

### Feature 5 — GitHub account/repository connection

**Goal:** Support GitHub-backed project workflows for authenticated users.

**Scope:**

- decide between GitHub OAuth app and GitHub App installation;
- connect a user's GitHub identity or installation;
- list accessible repos;
- attach one repo to an Astra Build project;
- record repo metadata server-side.

**Validation:**

- user can connect/disconnect GitHub;
- user can attach an allowed repo to a project;
- token/installation credentials are encrypted or stored in a managed secret location;
- user cannot attach repos they cannot access.

**PR gate:** Open PR, wait for review, then stop.

---

### Feature 6 — Commit/push project to GitHub

**Goal:** Let users push saved project state to the attached GitHub repository.

**Scope:**

- create commits from server-side snapshot or browser file tree;
- support branch selection or generated feature branch;
- create commit with user-provided message;
- optionally open pull request.

**Validation:**

- commit appears in GitHub;
- generated branch contains expected files;
- failed GitHub calls produce recoverable UI errors;
- project metadata records last pushed commit/branch.

**PR gate:** Open PR, wait for review, then stop.

---

### Feature 7 — Import/resume from GitHub

**Goal:** Create or refresh an Astra Build project from a GitHub repository.

**Scope:**

- import repo contents into project snapshot;
- preserve selected branch/ref;
- protect against oversized repos unsupported by WebContainer/browser constraints;
- document unsupported repo shapes.

**Validation:**

- import small JS/TS repo;
- open project and run preview;
- reject or warn on unsupported large/native stacks.

**PR gate:** Open PR, wait for review, then stop.

---

### Feature 8 — Usage tracking and quotas

**Goal:** Track model and project usage per user/workspace before broader rollout.

**Scope:**

- record prompt/model usage metadata;
- record project count and storage size;
- enforce configurable limits;
- expose admin/operator diagnostics.

**Validation:**

- usage increments after model calls;
- limits block excess usage with clear UX;
- no sensitive prompt/provider secrets leak into logs by default.

**PR gate:** Open PR, wait for review, then stop.

---

### Feature 9 — Admin controls and workspace model

**Goal:** Add organization/workspace concepts for team use.

**Scope:**

- define workspace membership;
- assign project ownership to user or workspace;
- basic roles: owner/admin/member;
- admin view for projects/users/provider health.

**Validation:**

- members can access workspace projects according to role;
- non-members cannot;
- admin operations are audited.

**PR gate:** Open PR, wait for review, then stop.

---

### Feature 10 — Server-side agent/workspace evaluation

**Goal:** Decide whether Astra Build needs Hermes Agent/OpenCode/server containers for heavier workflows.

**This is explicitly not part of MVP implementation.**

Evaluate only after the WebContainer MVP proves demand for:

- long-running background repo edits;
- PR-generating agents;
- non-JS stacks;
- Docker/native tooling;
- backend tests that cannot run inside WebContainer.

Candidate architectures:

- Hermes Agent workers for PR-oriented tasks, scheduling, and tool-rich automation;
- OpenCode workers for repository coding-agent workflows;
- containerized workspaces for arbitrary stack execution.

**Decision artifact:** Write a technical design proposal and open a docs-only PR before implementation.

## Non-goals for MVP

- arbitrary backend/runtime support beyond WebContainer-compatible JS/TS web apps;
- real-time multi-user collaborative editing;
- mobile native app builds;
- Docker-in-browser or unrestricted server containers;
- user-provided arbitrary model keys in the normal product path;
- background autonomous PR agents before core project persistence and GitHub workflows work.

## Initial Kanban task breakdown

Create cards assigned to the `astra` profile in this order:

1. **Astra Build: Feature 0 — fork hygiene and roadmap PR**
   - create/maintain roadmap;
   - verify repository baseline;
   - open docs-only PR;
   - stop for review.
2. **Astra Build: Feature 1 — authentication foundation**
   - blocked until Feature 0 PR is reviewed/merged or operator says continue.
3. **Astra Build: Feature 2 — server-side project model**
   - blocked until Feature 1 PR is reviewed/merged or operator says continue.
4. **Astra Build: Feature 3 — project file snapshot persistence**
   - blocked until Feature 2 PR is reviewed/merged or operator says continue.
5. **Astra Build: Feature 4 — Astra-managed single provider gateway**
   - blocked until Feature 3 PR is reviewed/merged or operator says continue.
6. **Astra Build: Feature 5 — GitHub account/repository connection**
   - blocked until Feature 4 PR is reviewed/merged or operator says continue.
7. **Astra Build: Feature 6 — commit/push project to GitHub**
   - blocked until Feature 5 PR is reviewed/merged or operator says continue.
8. **Astra Build: Feature 7 — import/resume from GitHub**
   - blocked until Feature 6 PR is reviewed/merged or operator says continue.
9. **Astra Build: Feature 8 — usage tracking and quotas**
   - blocked until Feature 7 PR is reviewed/merged or operator says continue.
10. **Astra Build: Feature 9 — admin controls and workspace model**
    - blocked until Feature 8 PR is reviewed/merged or operator says continue.
11. **Astra Build: Feature 10 — server-side agent/workspace evaluation**
    - blocked until MVP evidence justifies it.

## PR template for each feature

Each feature PR should include:

```markdown
## Summary
- What changed
- What stayed intentionally out of scope

## Validation
- [ ] Command/result
- [ ] Manual check/result

## Review gate
This PR completes one feature only. Do not continue to the next feature until the operator explicitly approves continuation.
```

## Immediate next action after this roadmap PR

Stop and wait for operator review. Once the operator says to continue, begin **Feature 1 — Authentication foundation** only.

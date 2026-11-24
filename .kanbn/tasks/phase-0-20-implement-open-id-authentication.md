---
tags:
  - 'type:feature'
  - 'phase:0'
  - 'area:network'
  - 'priority:medium'
assigned: Qwen
progress: 0
created: 2026-11-24T04:09:41.620Z
updated: 2026-11-24T04:56:44.229Z
---

# Phase 0.20: Implement OpenID authentication

**Task ID:** 0.20
**Phase:** 0 — Foundation
**Dependencies:** Phase 0.1 (Project scaffolding)
**Type:** Feature | **Priority:** Medium

---

**References:**
- Consult `design/Orbit.md` §11 (Network/Multiplayer — authentication flow)
- Consult `design/implementation_plan.md` §4.3 task 0.20 — OpenID authentication
- Consult `design/gap_analysis.md` §1.1 (Network Provider Architecture — OpenID for non-Steam players)

---

**Objective:**
Implement OpenID Connect authentication for players without Steam. Provides identity verification and session management for web-based multiplayer.

**Deliverables:**
- `src/auth/OpenIDManager.ts` — OpenID authentication manager
- `tests/auth/openid-manager.test.ts` — login, token validation tests

**Subtasks:**
- [ ] Implement `OpenIDManager` class: initiateLogin, handleCallback, logout
- [ ] Configure OpenID provider (Auth0 or similar)
- [ ] Implement login flow: redirect to provider, handle callback
- [ ] Validate ID token: verify signature, expiration, issuer
- [ ] Extract user identity: Steam-like ID, display name, avatar URL
- [ ] Store session: JWT in localStorage/sessionStorage
- [ ] Implement logout: clear session, revoke token
- [ ] Write unit tests

**Unit Tests Required (≥90% coverage):**
- **Happy paths:**
  - Login flow: redirect to provider → callback with valid ID token → user authenticated
  - Token validation: valid token accepted, user identity extracted correctly
  - Session storage: JWT stored and retrieved for subsequent requests
  - Logout: session cleared, token revoked
- **Sad paths:**
  - Expired token: rejected, user redirected to login
  - Invalid signature: rejected with error logged
  - Callback without code: rejected, error message shown
  - Token from wrong issuer: rejected, security warning

**Completion Criteria:** All tests pass with ≥90% coverage. OpenID authentication flow works end-to-end.

---

⚠️ **This task is NOT complete without thorough unit tests. Both happy path (login flow works) and sad path (expired/invalid tokens rejected) tests must pass.**

## Sub-tasks

- [ ] Implement OpenIDManager class: initiateLogin, handleCallback, logout
- [ ] Configure OpenID provider (Auth0 or similar)
- [ ] Implement login flow: redirect to provider, handle callback
- [ ] Validate ID token: verify signature, expiration, issuer
- [ ] Extract user identity: Steam-like ID, display name, avatar URL
- [ ] Write unit tests with ≥90% coverage including happy and sad paths

## Relations

- [depends-on phase-0-1-initialize-project-scaffolding-package-json-tsconfig-vite-config](phase-0-1-initialize-project-scaffolding-package-json-tsconfig-vite-config.md)

## History

- type: created
  date: 2026-11-24T04:09:41.620Z
  column: Backlog
  fromProgress: 0
  toProgress: 0
  author: Kevin J. Duling

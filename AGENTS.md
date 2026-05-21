<laravel-boost-guidelines>
=== foundation rules ===

# AGENTS.md — Sakala Dashboard

This document extends the Laravel Boost guidelines with project-specific instructions for AI agents, automation tools, and human contributors working on **Sakala Dashboard**.

Sakala Dashboard is the Laravel + Inertia + Svelte control plane for the Sakala developer deployment platform by **PT Media Sarana Data / GMEDIA**.

---

## 1. Project Identity

- Project name: **Sakala Dashboard**
- Product name: **Sakala**
- Domain: `sakala.dev`
- Organization: **PT Media Sarana Data / GMEDIA**
- License: **Apache License 2.0**
- Primary language for product copy and documentation: **Bahasa Indonesia**
- Technical codebase language: English identifiers, English class names, English method names

Sakala helps users deploy projects from Git repositories to public URLs without manually managing VPS, SSL, reverse proxy, Docker networking, or infrastructure wiring.

---

## 2. Installed Stack

This application uses:

- PHP 8.5
- Laravel 13
- Laravel Fortify
- Laravel Socialite
- Laravel Reverb
- Laravel Wayfinder
- Laravel Boost
- Laravel Sail
- Laravel Pint
- Pest
- Inertia Laravel v3
- Inertia Svelte v3
- Svelte
- Tailwind CSS v4
- Spatie Laravel Permission
- Spatie Laravel Data
- Spatie Laravel Activitylog
- Spatie Laravel Query Builder
- Larastan
- Prettier
- Husky
- lint-staged

Always follow version-specific documentation and existing conventions.

---

## 3. Required Skills

When working in a relevant domain, activate/read the matching skill from `.agents/skills`:

```txt
fortify-development
inertia-svelte-development
laravel-best-practices
pest-testing
socialite-development
tailwindcss-development
wayfinder-development
```

For Laravel code, always follow Laravel best practices and prefer Artisan generators.

For Inertia + Svelte UI, follow the Inertia Svelte skill before creating pages, forms, layouts, or realtime UI.

For authentication and GitHub login, follow Fortify and Socialite skills.

For tests, follow Pest skill.

---

## 4. Product Principles

All changes should respect these principles:

1. **Deploy flow must remain simple**
   ```txt
   Repository
   → Sakala reads project
   → Deploy
   → Public URL
   ```

2. **Do not overload beginners**
   Advanced settings must be hidden by default.

3. **Dashboard is a control plane**
   It must not run Docker, Caddy, Railpack, or host-level commands directly.

4. **Agent owns runtime execution**
   Runtime operations belong to `sakala-agent`.

5. **Onboarding is not a survey**
   Initial onboarding should ask only lightweight questions, especially acquisition source.

6. **Admin dashboard is not just a template**
   Admin should show validation pulse, adoption, retention, failed deployment reasons, and operational health.

7. **Use Bahasa Indonesia for user-facing copy**
   Code identifiers remain English.

---

## 5. Architecture Boundary

### sakala-dashboard

Responsible for:

- authentication
- authorization
- workspace/project/deployment metadata
- onboarding data
- admin validation dashboard
- agent API
- GitHub OAuth
- GitHub webhook handling
- deployment events/logs storage
- realtime broadcasting via Reverb

Not responsible for:

- Docker build
- Docker run
- Caddy config reload
- Railpack execution
- host/container lifecycle

### sakala-agent

Responsible for:

- Docker/Railpack/Caddy integration
- build/run/stop/restart/sleep/wake
- command execution
- node heartbeat
- log/event reporting

### sakala-infra

Responsible for:

- local orchestration
- Caddy config
- Docker Compose
- staging/prod config
- infra documentation

---

## 6. Agent Communication Model

Use this MVP model:

```txt
Laravel Dashboard
→ creates deployment record
→ creates agent command in database
→ Rust Agent polls dashboard
→ Agent claims command
→ Agent executes command
→ Agent reports events/logs/status
→ Laravel broadcasts realtime updates via Reverb
```

Do not implement dashboard-to-agent direct Docker access.

Do not expose Docker socket to the web application.

Expected endpoint namespace:

```txt
/api/agent/v1/*
```

Suggested endpoints:

```txt
POST /api/agent/v1/register
POST /api/agent/v1/heartbeat
GET  /api/agent/v1/commands
POST /api/agent/v1/commands/{command}/claim
POST /api/agent/v1/commands/{command}/events
POST /api/agent/v1/commands/{command}/logs
POST /api/agent/v1/commands/{command}/complete
POST /api/agent/v1/commands/{command}/fail
```

Agent authentication:

```txt
Authorization: Bearer <agent-token>
X-Agent-Id: <agent-node-id>
```

Agent tokens must be stored hashed.

---

## 7. Directory Conventions

Backend:

```txt
app/
├── Actions/
│   ├── Agents/
│   ├── Auth/
│   ├── Deployments/
│   ├── Onboarding/
│   ├── Projects/
│   └── Teams/
├── Data/
│   ├── Agents/
│   ├── Deployments/
│   ├── Projects/
│   └── Webhooks/
├── Enums/
├── Events/
├── Http/
│   ├── Controllers/
│   │   ├── AgentApi/
│   │   ├── Auth/
│   │   ├── Dashboard/
│   │   ├── Projects/
│   │   └── Webhooks/
│   └── Middleware/
├── Jobs/
├── Listeners/
├── Models/
├── Notifications/
├── Policies/
├── Rules/
├── Services/
│   ├── Agents/
│   ├── Deployments/
│   ├── Domains/
│   └── Git/
└── Support/
```

Frontend:

```txt
resources/js/
├── actions/
├── components/
├── layouts/
├── lib/
├── pages/
├── routes/
├── types/
└── wayfinder/
```

Component convention:

```txt
resources/js/components/
├── app/
├── sakala/
└── ui/
```

- `ui`: generic UI components
- `app`: application shell/layout components
- `sakala`: domain-specific components

---

## 8. Laravel Coding Rules

Follow these rules:

- Use Artisan generators when possible.
- Use Actions for use-case logic.
- Use Data objects for structured payloads.
- Use Enums for states/types.
- Use Policies for authorization.
- Keep controllers thin.
- Use explicit return types.
- Use constructor property promotion.
- Use named routes.
- Use Eloquent relationships clearly.
- Avoid direct query logic inside Svelte pages.
- Avoid putting business logic inside controllers.
- Avoid running infrastructure commands from Laravel.

Recommended examples:

```txt
CreateProjectAction
CreateDeploymentCommandAction
RecordAgentHeartbeatAction
StoreOnboardingSourceAction
```

---

## 9. Frontend Rules

Use Svelte + Inertia.

UI direction:

- light-first
- warm clean palette
- teal as primary brand color
- topbar-based dashboard
- minimal input
- logs in dark terminal-style panel
- clear empty states
- friendly Bahasa Indonesia copy

Avoid:

- AI-glow dashboard
- cyberpunk/hacker UI
- generic admin template
- too many fields in deploy wizard
- exposing infra jargon to beginners

Deploy wizard must stay simple:

```txt
Repository
→ Auto-detect
→ Deploy
```

Environment variables and advanced options must be hidden by default.

---

## 10. Commit Rules

All commits must follow **Conventional Commits 1.0.0**.

Format:

```txt
<type>[optional scope]: <description>
```

Examples:

```txt
feat(auth): add GitHub login
feat(onboarding): store acquisition source
fix(projects): prevent cross-workspace access
docs(readme): add Sail setup guide
test(agents): cover heartbeat authentication
refactor(deployments): move command creation into action
```

Allowed types:

```txt
feat
fix
docs
style
refactor
perf
test
build
ci
chore
revert
```

Recommended scopes:

```txt
auth
onboarding
projects
deployments
agents
webhooks
admin
ui
docs
tests
infra
```

Breaking changes:

```txt
feat(agents)!: change command payload format
```

Or:

```txt
BREAKING CHANGE: Agent command payload now requires idempotency_key.
```

---

## 11. Testing Rules

Use Pest.

Create tests with:

```bash
php artisan make:test --pest ProjectCreationTest
```

Most tests should be feature tests.

Prioritize tests for:

- auth
- onboarding
- project CRUD
- workspace authorization
- deployment records
- agent API authentication
- agent command lifecycle
- admin validation metrics

Run focused tests before finalizing changes:

```bash
php artisan test --compact
```

---

## 12. Quality Commands

Before finalizing changes, run the relevant checks.

PHP format:

```bash
vendor/bin/pint
```

Static analysis:

```bash
./vendor/bin/phpstan analyse
```

Tests:

```bash
php artisan test --compact
```

Frontend build:

```bash
npm run build
```

Prettier:

```bash
npx prettier --check resources/js
```

---

## 13. Documentation Rules

Update documentation when changing:

- setup steps
- environment variables
- architecture
- agent API
- directory convention
- deployment flow
- contribution rules
- commit rules

Important docs:

```txt
README.md
CONTRIBUTING.md
ARCHITECTURE.md
AGENTS.md
CHANGELOG.md
SECURITY.md
CODE_OF_CONDUCT.md
```

---

## 14. Security Rules

- Never expose Docker socket from dashboard.
- Never store agent token in plain text.
- Never expose secret environment values in UI or logs.
- Always authorize workspace/project access.
- Validate GitHub webhook signatures.
- Rate limit sensitive endpoints.
- Audit admin actions.
- Treat agent API as privileged.
- Use HTTPS in non-local environments.

---

## 15. Existing Laravel Boost Guidelines

Keep following the generated Laravel Boost guidelines in this repository.

If there is a conflict between generic Laravel Boost guidance and Sakala product architecture, prefer Sakala architecture for product boundaries, especially:

- dashboard is control plane only
- agent executes runtime commands
- deploy flow stays minimal
- user-facing copy uses Bahasa Indonesia
- commits follow Conventional Commits

</laravel-boost-guidelines>

# Security Policy

Sakala Dashboard is the control plane for Sakala. Security issues may affect authentication, project metadata, agent API behavior, secrets metadata, or deployment state.

## Reporting

Do not open public issues for exploitable vulnerabilities. Report security concerns privately to the Sakala maintainers through the repository security policy when available, with reproduction steps and impact.

During the MVP phase, GMEDIA may help with security triage as founding sponsor and infrastructure supporter.

## Boundaries

- Do not commit real tokens, credentials, webhook secrets, API keys, database credentials, or private keys.
- Do not expose Docker socket access from the dashboard.
- Do not add dashboard-to-runtime direct execution paths; runtime execution belongs to `sakala-agent`.
- Keep authorization and policy changes covered by tests.

## Supported Versions

Sakala is still in active MVP development. Security fixes are applied to the active `main` branch unless a release policy is introduced later.

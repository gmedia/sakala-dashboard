# Sakala Dashboard Architecture

Dokumen ini menjelaskan arsitektur awal **Sakala Dashboard**, posisinya dalam ekosistem Sakala, dan batas tanggung jawab antara dashboard, agent, dan infra.

Sakala Dashboard adalah **control plane** untuk Sakala. Ia mengelola data, UI, workflow, authorization, API, deployment state, onboarding, dan admin validation dashboard. Sakala Dashboard tidak menjalankan Docker, Caddy, Railpack, atau proses runtime secara langsung.

---

## 1. Product Context

Sakala adalah developer deployment platform dari **PT Media Sarana Data / GMEDIA**.

Tujuannya adalah membantu user menjalankan project dari Git ke URL publik tanpa harus mengelola VPS, SSL, reverse proxy, container networking, atau konfigurasi infrastruktur manual.

Target awal:

- mahasiswa
- dosen/kampus
- developer pemula
- komunitas
- tim kecil
- program magang/pilot internal

Prinsip utama:

```txt
Repository
→ Sakala membaca project
→ Deploy
→ Public URL
```

Sakala tidak langsung dimulai sebagai cloud besar. Sakala dimulai dari deployment experience kecil yang tervalidasi.

---

## 2. Repository Boundary

Sakala dipisah menjadi tiga repository utama.

```txt
sakala-dashboard
- Laravel + Inertia + Svelte
- Auth, workspace, project, deployment record
- Admin validation dashboard
- API endpoint untuk agent
- Webhook handler GitHub

sakala-agent
- Rust service
- Docker/Railpack/Caddy integration
- Build/run/stop/restart app
- Report deployment status
- Stream/send logs
- Heartbeat ke dashboard

sakala-infra
- docker-compose
- Caddy config
- local dev orchestration
- staging/prod deployment config
- sample node setup
- documentation for infra
```

---

## 3. High-level Architecture

```txt
User Browser
  ↓
Svelte + Inertia UI
  ↓
Laravel Dashboard / Control Plane
  ↓
PostgreSQL metadata + command table
  ↓
Rust Agent polling commands
  ↓
Docker / Railpack / Caddy
  ↓
Agent reports status/events/logs
  ↓
Laravel broadcasts realtime updates via Reverb
  ↓
Svelte UI updates deployment progress/logs
```

---

## 4. Control Plane Responsibilities

Sakala Dashboard bertanggung jawab untuk:

- authentication
- authorization
- workspace/team management
- user onboarding
- project metadata
- deployment metadata
- environment variable metadata
- agent node registration
- agent heartbeat tracking
- agent command creation
- deployment event/log ingestion
- admin validation dashboard
- GitHub OAuth integration
- GitHub webhook handling
- realtime event broadcasting to frontend
- audit log
- documentation-driven contributor workflow

Dashboard **tidak** bertanggung jawab untuk:

- Docker build
- container lifecycle
- Caddy route update
- Railpack execution
- host resource collection directly
- runtime log collection directly
- app wake execution directly

Semua operasi runtime dilakukan oleh `sakala-agent`.

---

## 5. Runtime / Agent Responsibilities

`sakala-agent` bertanggung jawab untuk:

- register ke dashboard
- mengirim heartbeat
- polling command dari dashboard
- claim command
- clone/pull repository jika diperlukan
- menjalankan Dockerfile/Railpack build
- membuat container
- start/stop/restart/sleep/wake container
- update route Caddy
- mengirim deployment events
- mengirim deployment logs
- mengirim status akhir command
- melaporkan resource/node health

Agent tidak boleh mengambil keputusan product/business seperti:

- user boleh deploy atau tidak
- user punya quota berapa
- project milik siapa
- policy billing
- workspace permission

Keputusan tersebut tetap berada di dashboard/control plane.

---

## 6. Agent Communication Model

Untuk MVP, Sakala menggunakan pendekatan:

```txt
PostgreSQL-backed command table
+ agent polling
+ HTTP report API
+ Reverb broadcast to UI
```

Alur:

```txt
1. User klik Deploy di dashboard.
2. Laravel membuat Deployment record.
3. Laravel membuat AgentCommand dengan type DEPLOY_PROJECT.
4. Rust Agent polling command dari dashboard.
5. Agent claim command.
6. Agent menjalankan build/run/update route.
7. Agent mengirim events/logs/status ke dashboard.
8. Laravel menyimpan log/event.
9. Laravel broadcast progress ke UI via Reverb.
10. UI menampilkan progress/log realtime.
```

Alasan memilih polling untuk MVP:

- agent tidak perlu expose public API
- dashboard tidak perlu reach agent secara langsung
- lebih aman untuk NAT/private network
- mudah diaudit melalui database
- mudah dipahami oleh tim awal
- cukup scalable untuk fase pilot

Broker seperti Redis/Valkey Streams atau NATS dapat dipertimbangkan setelah jumlah node dan traffic meningkat.

---

## 7. Suggested Agent API

Namespace endpoint:

```txt
/api/agent/v1/*
```

Endpoint awal:

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

Authentication:

```txt
Authorization: Bearer <agent-token>
X-Agent-Id: <agent-node-id>
```

Token agent harus disimpan dalam bentuk hash.

Future hardening:

- mTLS
- signed payload
- rotating token
- IP allowlist
- command idempotency enforcement
- per-agent capability policy

---

## 8. Core Domain Concepts

### 8.1 Workspace

Workspace adalah ruang kerja user. Secara backend, starter kit Laravel dapat menggunakan konsep `Team`, tetapi di UI sebaiknya disebut sebagai `Workspace`.

Contoh workspace:

- Personal Workspace
- Kelas Pemrograman Web
- Komunitas Backend Jogja
- Tim Internal

### 8.2 Project

Project adalah representasi aplikasi yang ingin dideploy.

Contoh field:

```txt
id
workspace_id
user_id
name
slug
repository_url
repository_provider
repository_full_name
branch
default_domain
status
runtime_status
last_deployed_at
last_request_at
created_at
updated_at
```

### 8.3 Deployment

Deployment adalah satu attempt untuk membangun dan menjalankan project.

Contoh field:

```txt
id
project_id
agent_node_id
commit_sha
commit_message
status
image_tag
started_at
finished_at
duration_seconds
triggered_by
error_message
created_at
updated_at
```

### 8.4 Agent Node

Agent Node adalah host yang menjalankan `sakala-agent`.

Contoh field:

```txt
id
name
token_hash
status
hostname
ip_address
capabilities
last_heartbeat_at
created_at
updated_at
```

### 8.5 Agent Command

Agent Command adalah instruksi dari dashboard untuk agent.

Contoh field:

```txt
id
agent_node_id nullable
deployment_id nullable
project_id
type
status
payload json
attempts
available_at
claimed_at
claimed_by
started_at
finished_at
error_message
idempotency_key
created_at
updated_at
```

Command type:

```txt
DEPLOY_PROJECT
STOP_PROJECT
RESTART_PROJECT
SLEEP_PROJECT
WAKE_PROJECT
REMOVE_PROJECT
REFRESH_ROUTE
HEALTH_CHECK
```

Command status:

```txt
pending
claimed
running
succeeded
failed
cancelled
expired
```

### 8.6 Deployment Event

Deployment Event mencatat state transition atau event penting.

Contoh:

```txt
DEPLOYMENT_QUEUED
REPOSITORY_CLONED
BUILD_STARTED
BUILD_SUCCEEDED
CONTAINER_STARTED
ROUTE_CONFIGURED
HEALTH_CHECK_PASSED
DEPLOYMENT_SUCCEEDED
DEPLOYMENT_FAILED
```

### 8.7 Deployment Log

Deployment Log menyimpan baris log build/runtime.

Untuk MVP, log dapat disimpan di database dengan retention terbatas. Setelah usage meningkat, log dapat dipindah ke object storage atau log system khusus.

---

## 9. Backend Structure

Struktur backend:

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

### 9.1 Actions

Actions berisi use-case kecil yang bisa dites.

Contoh:

```txt
Actions/Projects/CreateProjectAction.php
Actions/Deployments/CreateDeploymentCommandAction.php
Actions/Agents/RecordAgentHeartbeatAction.php
Actions/Onboarding/StoreAcquisitionSourceAction.php
```

### 9.2 Data

Data berisi DTO menggunakan Spatie Laravel Data.

Contoh:

```txt
Data/Projects/CreateProjectData.php
Data/Agents/AgentHeartbeatData.php
Data/Deployments/DeploymentLogData.php
Data/Webhooks/GitHubPushWebhookData.php
```

### 9.3 Enums

Enums berisi status dan type.

Contoh:

```txt
DeploymentStatus
RuntimeStatus
AgentNodeStatus
AgentCommandStatus
AgentCommandType
ProjectSourceProvider
```

### 9.4 Services

Services berisi integrasi atau logic yang lebih panjang.

Contoh:

```txt
Services/Git/GitHubService.php
Services/Agents/AgentCommandDispatcher.php
Services/Deployments/DeploymentStateMachine.php
Services/Domains/DefaultDomainGenerator.php
```

---

## 10. Frontend Structure

Struktur frontend mengikuti Inertia + Svelte.

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

Rekomendasi organisasi komponen:

```txt
resources/js/components/
├── app/
│   ├── AppTopbar.svelte
│   ├── AppShell.svelte
│   └── CommandMenu.svelte
├── sakala/
│   ├── SakalaLogo.svelte
│   ├── DeploymentTimeline.svelte
│   ├── TerminalLogs.svelte
│   └── ProjectStatusBadge.svelte
└── ui/
    ├── Button.svelte
    ├── Card.svelte
    ├── Badge.svelte
    └── EmptyState.svelte
```

Pages:

```txt
resources/js/pages/
├── Auth/
├── Onboarding/
├── Dashboard/
├── Projects/
├── Deployments/
└── Admin/
```

---

## 11. UX Architecture

Sakala Dashboard menggunakan prinsip:

```txt
login
→ onboarding ringan
→ deploy project pertama
→ deployment progress
→ project live
→ feedback ringan setelah value terasa
```

Onboarding awal hanya boleh meminta data yang benar-benar ringan.

Rekomendasi onboarding awal:

```txt
Kamu tahu Sakala dari mana?
- Dosen / Kampus
- Teman
- Komunitas
- Workshop
- Media Sosial
- GMEDIA
- GitHub
- Lainnya
```

Pertanyaan lain seperti “project ini untuk apa?” sebaiknya ditanyakan setelah user mulai membuat project atau setelah deployment berhasil.

Deploy wizard harus mengikuti:

```txt
Repository
→ Auto-detect
→ Deploy
```

Advanced settings seperti environment variables, root directory, port manual, Dockerfile path, build command, start command, dan health check harus disembunyikan di section advanced/collapsible.

---

## 12. Realtime Architecture

Realtime digunakan untuk:

- deployment progress
- build logs
- runtime logs
- agent heartbeat updates
- admin operational events

Flow:

```txt
Agent POST event/log ke Laravel
→ Laravel simpan event/log
→ Laravel broadcast event via Reverb
→ Svelte UI listen private channel
→ UI update timeline/logs
```

Channel contoh:

```txt
private-deployment.{deploymentId}
private-project.{projectId}
private-admin.operations
```

Event contoh:

```txt
DeploymentStatusUpdated
DeploymentLogReceived
AgentHeartbeatUpdated
```

---

## 13. Security Boundaries

Prinsip keamanan:

1. Dashboard tidak boleh expose Docker socket.
2. Agent token harus disimpan hashed.
3. Agent API harus diautentikasi.
4. User tidak boleh mengakses project workspace lain.
5. Environment variables harus disimpan terenkripsi.
6. Logs tidak boleh menampilkan secret jika bisa dihindari.
7. Admin operation harus diaudit.
8. Webhook GitHub harus diverifikasi signature-nya.
9. Command agent harus idempotent.
10. Deployment operation harus punya timeout.

---

## 14. MVP Scope

MVP dashboard berfokus pada:

- GitHub login
- onboarding sumber user
- workspace dasar
- project CRUD
- deployment simulation
- deployment status
- deployment logs UI
- project detail
- admin validation pulse
- agent node registration model
- agent command model
- agent heartbeat API draft

MVP belum fokus pada:

- billing
- managed database
- object storage
- multi-region
- production SLA
- Kubernetes
- microVM
- serverless runtime
- AI provider marketplace

---

## 15. Growth Path

### Phase 1: Dashboard Foundation

- UI shell
- auth
- onboarding
- project CRUD
- deployment simulation

### Phase 2: Agent Integration

- agent register
- heartbeat
- command polling
- event/log report
- realtime UI

### Phase 3: Real Deployment

- Dockerfile deploy
- Railpack deploy
- Caddy route update
- generated subdomain

### Phase 4: Pilot Validation

- mahasiswa/dosen/community pilot
- validation dashboard
- feedback loop
- failed deployment reason tracking

### Phase 5: Managed Platform Direction

- always-on plan
- custom domain
- team/workspace improvements
- managed database
- storage
- monitoring
- institutional package

---

## 16. Decision Log

### 16.1 Dashboard sebagai Control Plane

Dashboard hanya mengelola state, API, UI, authorization, dan commands. Runtime execution dilakukan agent.

### 16.2 Agent Polling untuk MVP

Agent melakukan polling command dari dashboard agar tidak perlu expose public API ke agent node.

### 16.3 Reverb untuk Realtime UI

Laravel Reverb digunakan untuk progress/log realtime di UI.

### 16.4 Light-first UI

Sakala menggunakan UI clean, ramah, dan light-first. Logs tetap menggunakan terminal dark panel.

### 16.5 Onboarding Ringan

Onboarding awal tidak menjadi survei panjang. Pertanyaan tambahan dilakukan setelah user mendapat value.

---

## 17. Open Questions

1. Bagaimana format final payload `AgentCommand`?
2. Apakah command assignment dilakukan ke node tertentu atau scheduler sederhana memilih node?
3. Bagaimana retention policy untuk deployment logs?
4. Apakah generated domain langsung memakai `*.sakala.dev` atau domain staging terpisah?
5. Kapan GitHub webhook auto-deploy mulai diaktifkan?
6. Kapan environment variables dienkripsi dan bagaimana key management-nya?
7. Bagaimana batas resource per project pada fase pilot?

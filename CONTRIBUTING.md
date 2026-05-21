# Contributing to Sakala Dashboard

Terima kasih sudah tertarik berkontribusi ke **Sakala Dashboard**.

Sakala Dashboard adalah control plane untuk Sakala, sebuah developer deployment platform dari **PT Media Sarana Data / GMEDIA**. Dashboard ini bertugas mengelola authentication, workspace, project, deployment record, onboarding, admin validation dashboard, API untuk agent, dan integrasi Git provider.

Project ini masih berada pada fase awal/MVP. Fokus kontribusi saat ini adalah membangun fondasi yang rapi, mudah dipahami, aman untuk dikembangkan bersama anak magang/contributor, dan cukup fleksibel untuk berkembang menjadi platform yang lebih besar.

---

## 1. Prinsip Kontribusi

Kontribusi ke Sakala Dashboard harus mengikuti prinsip berikut:

1. **UX-first, bukan infra-first**
   User utama Sakala adalah mahasiswa, dosen, developer pemula, komunitas, dan tim kecil. Hindari membuat flow yang terlalu teknis di awal.

2. **MVP kecil, tetapi fondasi rapi**
   Jangan membangun fitur besar sebelum flow utama stabil: login, onboarding ringan, create project, deployment simulation, deployment status, logs, dan admin validation dashboard.

3. **Control plane only**
   Sakala Dashboard tidak menjalankan Docker, Railpack, atau Caddy secara langsung. Tugas eksekusi runtime dilakukan oleh `sakala-agent`.

4. **Progressive disclosure**
   Jangan menampilkan terlalu banyak input teknis ke user sejak awal. Advanced settings harus disembunyikan sampai dibutuhkan.

5. **Test setiap perubahan penting**
   Setiap feature, bugfix, policy, dan logic backend harus punya test yang relevan.

6. **Dokumentasi ikut hidup**
   Jika perubahanmu mengubah cara setup, flow, arsitektur, atau command, update dokumentasinya.

---

## 2. Tech Stack

Sakala Dashboard menggunakan:

- Laravel 13
- PHP 8.5
- Inertia v3
- Svelte
- Tailwind CSS v4
- Laravel Fortify
- Laravel Socialite
- Laravel Reverb
- Laravel Wayfinder
- Laravel Sail
- Laravel Pint
- Pest
- Larastan
- Spatie Laravel Permission
- Spatie Laravel Data
- Spatie Laravel Activitylog
- Spatie Laravel Query Builder

---

## 3. Repository dalam Ekosistem Sakala

Sakala dipisah menjadi beberapa repository:

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

## 4. Local Development

### 4.1 Clone Repository

```bash
git clone https://github.com/gmedia/sakala-dashboard.git
cd sakala-dashboard
```

### 4.2 Install Dependencies

```bash
composer install
npm install
```

### 4.3 Setup Environment

```bash
cp .env.example .env
php artisan key:generate
```

Sesuaikan konfigurasi database, Redis/Valkey, Reverb, dan GitHub OAuth di `.env`.

### 4.4 Run with Laravel Sail

Jika menggunakan Sail:

```bash
./vendor/bin/sail up -d
./vendor/bin/sail artisan migrate
./vendor/bin/sail npm run dev
```

Atau jika menggunakan script bawaan Laravel:

```bash
composer run dev
```

### 4.5 Build Frontend

```bash
npm run build
```

---

## 5. Struktur Project

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

Rekomendasi struktur komponen frontend:

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

---

## 6. Branching Strategy

Gunakan branch kecil berdasarkan issue.

Format branch:

```txt
<type>/<short-description>
```

Contoh:

```txt
feat/github-login
feat/project-create-flow
fix/deployment-status-badge
docs/update-local-setup
refactor/project-actions
test/project-policy
```

Hindari branch besar seperti:

```txt
feature/all-dashboard
update-everything
final
```

---

## 7. Commit Convention

Semua commit harus mengikuti **Conventional Commits 1.0.0**.

Format:

```txt
<type>[optional scope]: <description>

[optional body]

[optional footer(s)]
```

Contoh:

```txt
feat(auth): add GitHub login callback

fix(projects): prevent users from viewing other workspace projects

docs(readme): add Sail setup guide

test(projects): cover project creation policy

refactor(deployments): move status transition into action class
```

### 7.1 Tipe Commit yang Digunakan

Gunakan tipe berikut:

- `feat`: fitur baru
- `fix`: perbaikan bug
- `docs`: perubahan dokumentasi
- `style`: perubahan formatting tanpa mengubah logic
- `refactor`: perubahan struktur kode tanpa mengubah behavior
- `perf`: peningkatan performa
- `test`: penambahan/perubahan test
- `build`: perubahan build system/dependency
- `ci`: perubahan CI/CD
- `chore`: maintenance umum
- `revert`: revert commit

### 7.2 Scope yang Direkomendasikan

Scope boleh dipakai untuk menjelaskan area perubahan:

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

Contoh:

```txt
feat(onboarding): store acquisition source
feat(agents): add heartbeat data object
fix(ui): improve sleeping status badge contrast
docs(architecture): explain agent polling flow
```

### 7.3 Breaking Changes

Jika ada breaking change:

```txt
feat(agents)!: change command payload format
```

Atau gunakan footer:

```txt
BREAKING CHANGE: Agent command payload now requires an idempotency_key field.
```

---

## 8. Pull Request Guidelines

Sebelum membuka PR:

1. Pastikan branch berasal dari branch terbaru.
2. Pastikan perubahan kecil dan fokus pada satu issue.
3. Pastikan format dan test berjalan.
4. Jelaskan perubahan dengan jelas di deskripsi PR.
5. Tambahkan screenshot/video jika mengubah UI.
6. Tambahkan catatan migration/config jika ada.
7. Hubungkan PR ke issue jika ada.

Template deskripsi PR:

```md
## Summary

Jelaskan perubahan utama secara singkat.

## Changes

- ...
- ...
- ...

## Screenshots

Tambahkan screenshot jika ada perubahan UI.

## Test Plan

- [ ] `php artisan test --compact`
- [ ] `vendor/bin/pint --test`
- [ ] `./vendor/bin/phpstan analyse`
- [ ] `npm run build`

## Related Issue

Closes #...
```

---

## 9. Quality Checks

Jalankan check berikut sebelum merge.

### PHP Formatting

```bash
vendor/bin/pint
```

### Static Analysis

```bash
./vendor/bin/phpstan analyse
```

### Test

```bash
php artisan test --compact
```

### Frontend Build

```bash
npm run build
```

### Frontend Formatting

```bash
npx prettier --check resources/js
```

Jika menggunakan Sail:

```bash
./vendor/bin/sail artisan test --compact
./vendor/bin/sail npm run build
```

---

## 10. Testing Guidelines

Gunakan Pest.

Buat test dengan Artisan:

```bash
php artisan make:test --pest ProjectCreationTest
```

Prioritas test:

- authentication flow
- authorization/policy
- workspace access
- project CRUD
- onboarding data
- agent API authentication
- deployment command lifecycle
- admin validation dashboard query

Sebagian besar test harus berupa feature test.

---

## 11. UI/UX Contribution Guidelines

Sakala bukan dashboard admin template biasa. Hindari UI yang terlalu enterprise, terlalu AI-glow, atau terlalu hacker/cyberpunk.

Prinsip UI:

- light-first
- clean
- ramah untuk pemula
- topbar-based dashboard
- minimal input
- advanced settings disembunyikan
- status jelas
- error message membantu
- logs tetap terminal-like, tetapi tidak mendominasi seluruh UI

Deploy flow utama:

```txt
Repository
→ Sakala membaca project
→ Deploy
→ Public URL
```

Jangan menampilkan terlalu banyak input teknis di awal. Environment variables, port manual, root directory, health check, dan Dockerfile path masuk ke advanced/collapsible section.

---

## 12. Agent API Contribution Guidelines

Dashboard tidak menjalankan Docker/Caddy/Railpack secara langsung.

Pola komunikasi MVP:

```txt
Laravel Dashboard
→ membuat deployment record
→ membuat agent command
→ Rust Agent polling command
→ Agent execute command
→ Agent report event/log/status
→ Laravel broadcast progress ke UI via Reverb
```

Endpoint agent harus berada di namespace:

```txt
/api/agent/v1/*
```

Contoh endpoint:

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

Gunakan authentication berbasis bearer token untuk agent. Token harus disimpan dalam bentuk hash.

---

## 13. Documentation Guidelines

Dokumentasi sebaiknya jelas, ringkas, dan membantu contributor baru.

Dokumentasi awal yang disarankan:

```txt
README.md
CONTRIBUTING.md
ARCHITECTURE.md
AGENTS.md
CHANGELOG.md
CODE_OF_CONDUCT.md
SECURITY.md
```

Jika ada pertanyaan yang berulang dari contributor/magang, ubah jawabannya menjadi dokumentasi.

---

## 14. License

Sakala Dashboard menggunakan **Apache License 2.0**.

Lihat:

```txt
LICENSE
NOTICE
```

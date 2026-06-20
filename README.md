# Sakala Dashboard

**Sakala Dashboard** adalah control plane untuk platform deployment **Sakala** — project deployment open-source yang diinisiasi oleh maintainer Sakala dan didukung oleh **PT Media Sarana Data / GMEDIA** sebagai founding sponsor dan infrastructure supporter.

Sakala membantu developer menjalankan project dari Git ke URL publik tanpa harus mengelola VPS, reverse proxy, SSL, Docker networking, dan konfigurasi infrastruktur secara manual.

> **Tagline:** Tempat kode menjadi aplikasi nyata.

GMEDIA menyediakan dukungan awal berupa domain, infrastruktur, ruang eksperimen, dan dukungan teknis. Dukungan ini tidak mengubah prinsip Sakala sebagai project open-source dengan roadmap, dokumentasi, issue, dan kontribusi yang dikembangkan secara terbuka.

---

## Tentang Sakala

Sakala dibangun untuk menjawab masalah sederhana tetapi sering terjadi: banyak developer, terutama mahasiswa dan developer pemula, bisa membuat aplikasi tetapi masih kesulitan saat harus menjalankannya secara online.

Pada tahap awal, Sakala berfokus pada use case berikut:

- deployment tugas kuliah,
- portfolio developer,
- demo project,
- prototype aplikasi,
- project komunitas,
- validasi awal kebutuhan platform deployment.

Sakala tidak dimulai sebagai cloud besar atau pengganti hyperscaler. Sakala dimulai sebagai platform deployment kecil yang membantu user melakukan alur paling penting terlebih dahulu:

```txt
Repository Git → Deteksi Project → Deploy → URL Publik
```

---

## Posisi Repository Ini

Repository ini adalah **sakala-dashboard**, yaitu aplikasi utama berbasis Laravel yang berperan sebagai dashboard, control plane, dan API pusat untuk Sakala.

Sakala saat ini dipisah menjadi beberapa repository:

```txt
sakala-dashboard
- Laravel + Inertia + Svelte
- Auth, workspace, project, deployment record
- Admin validation dashboard
- API endpoint untuk agent
- Webhook handler GitHub

sakala-agent
- Rust service
- Docker / Railpack / Caddy integration
- Build, run, stop, restart app
- Report deployment status
- Stream/send logs
- Heartbeat ke dashboard

sakala-infra
- docker-compose
- Caddy config
- local development orchestration
- staging / production deployment config
- sample node setup
- documentation for infrastructure
```

---

## Tanggung Jawab Sakala Dashboard

Sakala Dashboard bertanggung jawab untuk:

- autentikasi user,
- login GitHub,
- onboarding awal,
- workspace/team management,
- project management,
- deployment records,
- deployment progress UI,
- deployment logs UI,
- environment variables metadata,
- domain metadata,
- agent node registry,
- agent command queue,
- agent heartbeat API,
- admin validation dashboard,
- webhook GitHub,
- realtime update ke UI.

Sakala Dashboard **tidak** bertanggung jawab langsung untuk:

- menjalankan Docker container,
- build image secara langsung,
- mengubah konfigurasi Caddy secara langsung,
- mengakses Docker socket secara langsung,
- menjalankan command runtime di host server.

Tugas-tugas tersebut dijalankan oleh **sakala-agent**.

---

## Arsitektur Sederhana

Alur komunikasi awal yang direncanakan:

```txt
User
  ↓
Svelte / Inertia Dashboard
  ↓
Laravel Control Plane
  ↓
PostgreSQL: deployment + agent command record
  ↓
Rust Agent polling commands
  ↓
Docker / Railpack / Caddy
  ↓
Agent reports events, logs, and heartbeat
  ↓
Laravel broadcasts updates via Reverb
  ↓
Svelte UI updates in realtime
```

Untuk MVP, komunikasi dashboard ke agent menggunakan pola:

```txt
Dashboard membuat command
Agent mengambil command melalui polling
Agent menjalankan command
Agent mengirim status, log, dan hasil eksekusi ke dashboard
```

Pendekatan ini dipilih agar:

- agent tidak perlu membuka public API yang kompleks,
- dashboard tidak perlu mengetahui IP agent secara langsung,
- command lebih mudah diaudit dari database,
- lebih mudah di-debug pada fase MVP,
- tetap bisa berkembang ke message broker di masa depan.

---

## Tech Stack

### Backend

- PHP 8.5
- Laravel 13
- Laravel Fortify
- Laravel Socialite
- Laravel Reverb
- Laravel Wayfinder
- Laravel Sail
- Laravel Boost
- Pest
- Laravel Pint
- Larastan

### Frontend

- Inertia.js
- Svelte
- Tailwind CSS 4
- Vite
- Wayfinder Vite Plugin
- ESLint
- Prettier
- Prettier Plugin Svelte

### Supporting Packages

- `spatie/laravel-permission`
- `spatie/laravel-data`
- `spatie/laravel-activitylog`
- `spatie/laravel-query-builder`

### Development Tools

- Laravel Sail
- Husky
- lint-staged
- Laravel Boost MCP guidelines

---

## Struktur Project

Struktur utama backend:

```txt
app/
├── Actions/
│   ├── Agents/
│   ├── Auth/
│   ├── Deployments/
│   ├── Fortify/
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

Struktur utama frontend:

```txt
resources/js/
├── actions/
├── app.ts
├── components/
├── layouts/
├── lib/
├── pages/
├── routes/
├── types/
└── wayfinder/
```

Rekomendasi pengelompokan komponen frontend:

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

## Konsep Domain Utama

Beberapa konsep domain yang akan muncul di Sakala Dashboard:

- **User**: pengguna Sakala.
- **Workspace**: ruang kerja personal, kelas, komunitas, atau organisasi.
- **Project**: aplikasi yang ingin dideploy.
- **Deployment**: satu proses deploy dari repository ke runtime.
- **Deployment Log**: log build/runtime yang dikirim oleh agent.
- **Agent Node**: server/node yang menjalankan sakala-agent.
- **Agent Command**: perintah yang dibuat dashboard untuk dijalankan agent.
- **Agent Heartbeat**: laporan berkala dari agent ke dashboard.
- **Audit Event**: catatan aktivitas penting di platform.

---

## Local Development

### 1. Clone repository

```bash
git clone https://github.com/gmedia/sakala-dashboard.git
cd sakala-dashboard
```

### 2. Install dependencies

```bash
composer install
npm install
```

### 3. Setup environment

```bash
cp .env.example .env
php artisan key:generate
```

Sesuaikan konfigurasi database, Redis/Valkey, Reverb, GitHub OAuth, dan service lain di `.env`.

### 4. Jalankan dengan Laravel Sail

```bash
./vendor/bin/sail up -d
```

### 5. Jalankan migration

```bash
./vendor/bin/sail artisan migrate
```

### 6. Jalankan development server

Jika menggunakan workflow Laravel bawaan:

```bash
composer run dev
```

Atau jalankan service secara terpisah sesuai kebutuhan:

```bash
npm run dev
./vendor/bin/sail artisan queue:work
./vendor/bin/sail artisan reverb:start
```

---

## Quality Check

Sebelum membuat pull request, jalankan pengecekan berikut:

```bash
vendor/bin/pint
php artisan test --compact
npm run build
```

Jika Larastan sudah dikonfigurasi:

```bash
vendor/bin/phpstan analyse
```

Untuk frontend formatting:

```bash
npm run format
```

---

## Development Principles

### 1. User mendapat value secepat mungkin

Sakala harus membantu user deploy project pertama dengan input seminimal mungkin.

Prioritas UX:

```txt
Login → Pilih Repository → Auto Detect → Deploy → URL Publik
```

### 2. Progressive disclosure

Jangan tampilkan semua konfigurasi teknis di awal. Pengaturan seperti environment variables, root directory, port manual, health check, dan resource limit harus disembunyikan sampai dibutuhkan.

### 3. Dashboard bukan executor

Laravel Dashboard adalah control plane. Eksekusi Docker, Railpack, Caddy, dan runtime dilakukan oleh Rust Agent.

### 4. Setiap command harus bisa diaudit

Perintah ke agent harus tersimpan sebagai record agar bisa dilacak, di-debug, dan dianalisis.

### 5. Admin dashboard harus menjawab validasi produk

Admin tidak hanya melihat resource teknis, tetapi juga sinyal validasi:

- berapa user berhasil deploy pertama,
- berapa user deploy ulang,
- dari mana user mengetahui Sakala,
- error deployment paling sering,
- fitur apa yang paling banyak diminta.

### 6. Dokumentasi adalah bagian dari produk

Jika ada pertanyaan yang berulang dari user atau contributor, jawaban tersebut harus dipindahkan ke dokumentasi.

---

## Onboarding Product Direction

Onboarding awal tidak boleh terasa seperti survei panjang.

Pertanyaan awal cukup satu:

```txt
Kamu tahu Sakala dari mana?
```

Contoh pilihan:

- Dosen / Kampus
- Teman
- Komunitas
- Workshop
- Media sosial
- GMEDIA
- GitHub
- Lainnya

Pertanyaan lain seperti “project ini untuk apa?” lebih baik ditanyakan setelah user mendapatkan value, misalnya setelah deployment pertama berhasil.

Prinsipnya:

```txt
Sebelum user mendapat value: tanya sesedikit mungkin.
Setelah user mendapat value: boleh minta feedback ringan.
```

---

## Agent Communication Direction

Untuk MVP, Sakala menggunakan pola agent polling:

```txt
1. User klik Deploy.
2. Laravel membuat Deployment record.
3. Laravel membuat AgentCommand.
4. Agent polling command dari dashboard.
5. Agent claim command.
6. Agent menjalankan build/run/update route.
7. Agent mengirim event, status, log, dan hasil eksekusi.
8. Laravel menyimpan data dan broadcast update ke UI.
```

Contoh endpoint agent API:

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

Authentication awal:

```txt
Authorization: Bearer <agent-token>
X-Agent-Id: <agent-id>
```

---

## Git Workflow

Rekomendasi workflow awal:

```txt
main
└── develop
    └── feature/<short-description>
```

Aturan umum:

- Jangan push langsung ke `main`.
- Gunakan branch kecil untuk setiap task.
- Satu pull request sebaiknya menyelesaikan satu scope kecil.
- Pull request harus punya deskripsi yang jelas.
- Jalankan formatter dan test sebelum meminta review.

Contoh branch:

```txt
feature/onboarding-source-channel
feature/project-create-page
feature/agent-command-model
fix/deployment-status-badge
```

---

## Contribution Notes

Project ini sedang berada di fase awal pengembangan. Fokus utama saat ini adalah membangun fondasi MVP yang stabil dan mudah dipahami contributor.

Area kontribusi awal yang cocok:

- dokumentasi,
- UI component,
- onboarding flow,
- project CRUD,
- deployment simulation,
- admin validation dashboard,
- test coverage,
- issue reproduction,
- local setup improvement.

Untuk kontribusi yang menyentuh agent, deployment engine, command execution, atau security-sensitive area, diskusikan terlebih dahulu melalui issue atau maintainer.

---

## Status Project

Sakala Dashboard masih berada pada fase awal MVP.

Target awal:

- auth dan GitHub login,
- onboarding singkat,
- project CRUD,
- deployment record,
- deployment simulation,
- realtime deployment progress,
- admin validation pulse,
- agent command API foundation.

Belum menjadi target awal:

- production SLA,
- billing,
- managed database,
- custom domain penuh,
- multi-node scheduling kompleks,
- serverless runtime,
- microVM isolation.

---

## License

Apache License, Version 2.0. See [LICENSE](./LICENSE) for details.

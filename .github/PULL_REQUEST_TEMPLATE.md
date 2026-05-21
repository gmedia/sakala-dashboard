## Ringkasan

Jelaskan perubahan utama pada pull request ini secara singkat.

## Jenis Perubahan

Pilih yang sesuai:

- [ ] `feat` - fitur baru
- [ ] `fix` - perbaikan bug
- [ ] `docs` - dokumentasi
- [ ] `style` - formatting / style tanpa mengubah logic
- [ ] `refactor` - perubahan struktur kode tanpa mengubah behavior
- [ ] `perf` - peningkatan performa
- [ ] `test` - penambahan/perubahan test
- [ ] `build` - dependency/build system
- [ ] `ci` - CI/CD
- [ ] `chore` - maintenance
- [ ] `revert` - revert perubahan

## Perubahan yang Dilakukan

-
-
-

## Area yang Terdampak

Pilih yang sesuai:

- [ ] Auth / GitHub Login
- [ ] Onboarding
- [ ] Dashboard UI
- [ ] Projects
- [ ] Deployments
- [ ] Agent API
- [ ] Webhooks
- [ ] Admin / Validation Pulse
- [ ] Documentation
- [ ] Tests
- [ ] Other:

## Screenshot / Rekaman

Tambahkan screenshot atau rekaman jika PR mengubah tampilan UI.

| Sebelum | Sesudah |
| --- | --- |
|  |  |

## Cara Menguji

Tuliskan langkah manual untuk menguji perubahan ini.

1.
2.
3.

## Checklist

- [ ] Commit sudah mengikuti Conventional Commits (`feat(scope): message`)
- [ ] Perubahan fokus pada satu issue/task
- [ ] Tidak ada secret, token, credential, atau data sensitif yang ikut ter-commit
- [ ] UI tetap menggunakan copy Bahasa Indonesia untuk user-facing text
- [ ] Advanced setting tidak ditampilkan ke user kecuali memang dibutuhkan
- [ ] Dashboard tidak menjalankan Docker/Caddy/Railpack secara langsung
- [ ] Dokumentasi sudah diupdate jika ada perubahan flow/setup/arsitektur
- [ ] Screenshot ditambahkan jika ada perubahan UI

## Quality Check

Centang yang sudah dijalankan:

- [ ] `php artisan test --compact`
- [ ] `vendor/bin/pint`
- [ ] `./vendor/bin/phpstan analyse`
- [ ] `npm run build`
- [ ] `npx prettier --check resources/js`

## Related Issue

Closes #

# 06 — Konfigurasi Autopilot, Watchdog & Monitoring

## 6.1 Mengaktifkan autopilot per akun

Setelah token tersimpan (bab 03) dan AI+storage siap (bab 05), daftarkan akun
di CMS dan aktifkan autopilot. Konfigurasi kemarin untuk `@mayorihomebake`:

| Setting | Nilai kemarin | Arti |
|---------|---------------|------|
| `enabled` | `1` | Autopilot ON (0 = mati, spt. `@borneo_queenshop`) |
| `posts_per_day` | `2` | Maksimal 2 post/hari (anti-spam) |
| `language` | `id` | Bahasa Indonesia |
| `niches` | home & living, dapur, peralatan rumah tangga, kitchen hacks | Topik generate AI |
| `preferred_hours` | 8, 12, 18, 21 | Slot jam (dipilih acak 2, waktunya diacak anti-pola) |
| `parts` | 2–4 | Jumlah part per thread (auto-split kalau >500 char/part) |

## 6.2 Cara kerja scheduler (tiap 30 detik)

1. **Publish**: ada `scheduled` post yang `scheduled_at <= now` → kirim ke
   Threads API → status jadi `posted`, simpan post ID. Kalau gagal → catat
   error, status `failed` (tidak di-retry membabi-buta).
2. **Generate**: slot hari ini belum penuh & autopilot ON → panggil AI dengan
   persona akun → simpan sebagai `scheduled` di jam slot berikutnya.
   Cek **dedup** (topik sudah pernah dipakai?) dan **min-gap** (jarak antar
   post cukup?) sebelum menjadwalkan.
3. **Auto-reply**: ada balasan baru di post → kirim soft-sell reply
   (link produk di **reply**, bukan di thread utama).

## 6.3 Watchdog — server mati, nyala sendiri

Scheduler butuh proses 24/7. Kemarin **server sempat mati** (connection refused
port 8455) dan post 21:49 gagal — watchdog yang menyelamatkan:

- Cron **`threads-cms-watchdog`** tiap **30 menit**: GET `/api/mini-stats`;
  kalau tidak merespons → restart server otomatis.
- Terbukti: 5 run terakhir sukses, server kembali sehat tanpa intervensi manual.

Untuk produksi serius: jalankan di **VPS** (bukan laptop) + systemd/launchd
`KeepAlive` (contoh di `deploy/`).

## 6.4 Monitoring

- **Mini dashboard** `/mini` + JSON `/api/mini-stats`:
  `scheduler_ok`, `ai_status`, `storage_status`, `posted_total`, `queued`, `failed`.
- **Dashboard web full** (kemarin dibuat terpisah): fetch data CMS real-time,
  auto-refresh tiap 2 menit.
- **Aturan main**: angka health (`queued`, `failed`) dicek **langsung dari API**,
  bukan dari ingatan — kemarin verifikasi ulang via curl dilakukan tepat
  sebelum setiap laporan status ke user.

## 6.5 Pelajaran insiden kemarin

| Kejadian | Pelajaran |
|----------|-----------|
| Server mati → post 21:49 gagal | Watchdog wajib; jangan andalkan proses manual |
| Post gagal tidak auto-retry | Untuk kasus kritis, republish manual + catat; lalu perbaiki watchdog |
| Token 60 hari | Pasang pengingat/cron refresh **sebelum** 15 Des 2026 |

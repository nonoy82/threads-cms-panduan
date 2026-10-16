# 08 — Troubleshooting: Semua Error Kemarin + Solusinya

Tabel konsolidasi dari 15–16 Okt 2026. Kalau mengulang proses dan mentok,
cari gejala di sini dulu.

## Meta App & OAuth (bagian tersulit)

| # | Gejala | Penyebab | Solusi |
|---|--------|----------|--------|
| 1 | Graph API Explorer: **"Fitur Tidak Tersedia"** saat Generate Access Token | Provisioning aplikasi baru di sisi Meta (ditunggu >3 jam tak sembuh) | **Lewati Explorer** — pakai alur OAuth manual (bab 03) |
| 2 | Error **1349168** "URL Blocked: redirect URI is not whitelisted" | `redirect_uri` tidak sama persis dengan yang terdaftar | Samakan persis; daftarkan URL `https` publik sampai jadi **chip** (Enter) |
| 3 | Kolom "URL Callback Alihkan" menolak disimpan | Diisi `localhost` — Meta tidak mengizinkan localhost untuk Threads API | Pakai URL `https://` publik (halaman callback penangkap `?code=`) |
| 4 | Permission tidak aktif / token tidak bisa publish | Menambah permission via "Tambahkan ke Tinjauan Aplikasi" (jalur app review) | Tambah via tombol **"+ Tambahkan"** di tiap baris permission (bab 2.4) |
| 5 | Akun Threads tidak bisa login OAuth | Ditambah via dialog "Add People" umum (itu untuk role FB) | Tambah via **"Add or Remove Threads Testers"** di pengaturan use-case, pakai **username** |
| 6 | Invite tester "tidak terjadi apa-apa" | Undangan belum diterima di sisi akun Threads | Buka aplikasi Threads → **Website permissions → Invites** → Accept |
| 7 | Token short-lived kedaluwarsa cepat | Itu normal (~1 jam) — lupa exchange | Selalu exchange ke long-lived 60 hari segera setelah dapat code |
| 8 | Token mati total setelah 60 hari | Tidak ada refresh otomatis | Cron/pengingat refresh **sebelum** hari ke-60 (bab 3.5) |

## Server & Scheduler

| # | Gejala | Penyebab | Solusi |
|---|--------|----------|--------|
| 9 | `connection refused` port 8455 | Server mati (kemarin: 16 Okt) | Watchdog cron 30 menit auto-restart (bab 6.3); untuk permanen pakai VPS + systemd/launchd KeepAlive |
| 10 | Post terjadwal tidak terbit | Server mati saat jadwal tiba | Republish manual sekali, lalu pastikan watchdog jalan |
| 11 | `AI_API_KEY belum di-set` di log | `.env` belum diisi | Isi `AI_BASE_URL`/`AI_API_KEY`/`AI_MODEL` — server tetap jalan tanpa AI (graceful degrade) |
| 12 | Gambar tidak tampil di post | Bucket tidak PUBLIC / `S3_PUBLIC_BASE` salah | Set bucket PUBLIC; uji upload→GET→hapus (bab 5.2) |

## Prinsip debug umum

1. **Cek `/api/mini-stats` dulu** — satu endpoint menjawab: scheduler hidup?
   AI siap? Storage siap? Ada yang gagal?
2. **Baca error persis seperti tertulis** (kode error Meta seperti 1349168
   bisa di-search) — jangan menebak.
3. **Ubah satu variabel per percobaan** — kemarin kegagalan berulang terjadi
   karena beberapa setting diubah bersamaan.
4. **Pisahkan masalah Meta vs masalah server** — kalau API Meta yang menolak
   (error code), perbaikinya di dashboard Meta, bukan di kode.

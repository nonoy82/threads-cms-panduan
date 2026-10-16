# Panduan Duplikasi: Threads CMS Autopilot (100% AUTO)

Dokumentasi langkah-demi-langkah membangun ulang **automation Threads autopilot**
yang kemarin berhasil jalan — dari nol sampai autopilot 100% AUTO menerbitkan
postingan AI ke Threads tanpa sentuhan manual.

Ditulis berdasarkan build nyata tanggal **15–16 Oktober 2026** (repo acuan:
`Faresapn/threads-cms-v2`), termasuk **semua kegagalan yang dialami dan cara
memperbaikinya** — terutama di bagian pembuatan aplikasi Meta & generate token
Threads, yang sempat gagal berulang kali.

## Untuk siapa panduan ini

- Kamu yang mau **belajar ulang** prosesnya dan **mengulang di tempat lain**
  (VPS sendiri, laptop lain, dsb).
- Kamu yang mau **menduplikasi pembuatannya pakai AI coding tools**
  (Trae AI, Cursor, dsb) — kasih repo ini + repo kode sebagai konteks.
- Kamu yang mau menjadikannya **SaaS yang bisa dijual** (lihat
  `07-duplikasi-jadi-saas.md`).

## Urutan baca (sesuai urutan pengerjaan kemarin)

| # | File | Isi |
|---|------|-----|
| 1 | `01-arsitektur.md` | Gambaran besar: komponen & alur data automation |
| 2 | `02-meta-app-dan-koneksi.md` | **(DETAIL)** Membuat aplikasi di Meta Developers + menghubungkan dengan benar |
| 3 | `03-token-oauth-threads.md` | **(DETAIL)** Alur OAuth & generate token Threads yang sempat gagal berulang |
| 4 | `04-setup-server-lokal.md` | Clone repo, venv, database, jalankan server |
| 5 | `05-storage-dan-ai.md` | Object storage (Supabase) + endpoint AI (Gemini) |
| 6 | `06-autopilot-scheduler-watchdog.md` | Konfigurasi autopilot, anti-spam, watchdog, monitoring |
| 7 | `07-duplikasi-jadi-saas.md` | Checklist menjadikan ini SaaS yang bisa dijual |
| 8 | `08-troubleshooting.md` | Semua error kemarin + solusinya, dalam satu tabel |

## Hasil akhir yang didapat kemarin

- Autopilot **100% AUTO** untuk akun `@mayorihomebake`: AI generate thread
  (niche home & living / dapur), schedule, auto-publish, + auto soft-sell reply.
- 2 postingan perdana terbit otomatis (18:31 & 21:49 WIB, 16 Okt 2026).
- Watchdog cron tiap 30 menit (restart server kalau mati) + mini dashboard monitoring.
- Token Threads long-lived 60 hari (perlu refresh sebelum ~15 Des 2026).

## Aturan keamanan (penting!)

Dokumen ini **tidak berisi secret asli** (app_secret, token, API key).
Semua credential ditulis sebagai placeholder seperti `<ISI_DI_SINI>`.
Credential asli hanya hidup di file lokal: `~/.threads-bot/app.json`
dan `~/.threads-bot/tokens.json` (keduanya `chmod 600`), serta `.env`
(di-`gitignore`).

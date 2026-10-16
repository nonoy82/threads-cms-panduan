# 07 — Dari Script Pribadi ke SaaS yang Bisa Dijual

Kamu bertanya: bagaimana supaya ini bisa diduplikasi (Trae AI / Cursor) dan
dijadikan **SaaS sendiri**. Berikut peta jalannya.

## 7.1 Cara duplikasi pakai AI coding tools (sekarang juga bisa)

1. Siapkan repo kode (`threads-cms-v2`) + repo panduan ini dalam satu folder.
2. Di Cursor/Trae, jadikan keduanya konteks, lalu prompt contoh:
   > "Pelajari repo threads-cms-v2 dan panduan di threads-cms-panduan/.
   > Buatkan ulang aplikasi yang sama dengan stack [Next.js + Supabase /
   > Python + Postgres — pilih satu], pertahankan alur: OAuth Meta →
   > simpan token → scheduler generate+publish → dashboard."
3. AI akan meniru arsitektur bab 01 — yang penting kamu sudah paham **alur
   OAuth (bab 02–03)** karena itu bagian tersulit dan tidak bisa ditebak AI
   dari nol.

## 7.2 Yang harus diubah dari versi pribadi ke SaaS

| Aspek | Versi kemarin (pribadi) | Versi SaaS |
|-------|-------------------------|------------|
| Token | 1 file `tokens.json` lokal | Tabel `user_tokens` di DB, terenkripsi per-user |
| Meta App | 1 app milik sendiri, tester manual | **1 app untuk semua user** → wajib lolos **App Review** Meta untuk permission publish |
| Scheduler | 1 thread untuk 1–2 akun | Job queue per-user (mis. Celery/BullMQ), quota per paket |
| Storage | 1 bucket Supabase | Prefix/folder per-user dalam 1 bucket |
| Secret | File `chmod 600` | Secret manager (env terenkripsi / Vault), tidak pernah di-log |
| Multi-user | Tidak ada | Auth (login), isolasi data per tenant |
| Billing | Tidak ada | Langganan (mis. Midtrans/Stripe/Xendit untuk ID) |

## 7.3 Jebakan Meta untuk SaaS publik

- **App Review wajib**: aplikasi yang dipakai orang lain (bukan tester)
  membutuhkan review & persetujuan Meta untuk scope `threads_content_publish`
  dkk. Siapkan: privacy policy, video demo penggunaan, alasan bisnis.
- **Token refresh massal**: puluhan/ratusan token 60-harian → butuh cron
  refresh otomatis yang robust (versi kemarin belum otomatis!).
- **Rate limit**: 250 post/hari/AKUN — aman per user, tapi pantau limit
  per-app juga.
- **Kebijakan platform**: spam = banned. Pertahankan anti-spam bab 01
  (cap harian, min-gap, dedup) sebagai fitur, bukan sekadar etika.

## 7.4 Urutan build SaaS yang disarankan

1. **OAuth multi-user dulu** (bab 02–03 + tabel token terenkripsi) — ini fondasi.
2. Scheduler per-user + dashboard per-user.
3. App Review Meta (ajukan lebih awal — prosesnya berminggu-minggu).
4. Billing + landing page.
5. Baru scale: queue, monitoring, backup.

## 7.5 Estimasi jujur

- Duplikasi versi pribadi di stack baru pakai AI: **hitungan hari**.
- Versi SaaS siap jual (dengan App Review lolos): **hitungan minggu–bulan**,
  dengan App Review sebagai bottleneck utama yang tidak bisa dipercepat.

Mulai dari yang kecil: buat versi pribadi berjalan dulu di VPS (bab 04–06),
baru naikkan ke SaaS.

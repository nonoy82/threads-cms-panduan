# 02 — Membuat Aplikasi di Meta & Menghubungkannya dengan Benar (DETAIL)

Ini bagian yang kemarin **paling sering gagal**. Ikuti urutannya persis —
setiap sub-bab di bawah adalah satu titik kegagalan nyata yang dialami
tanggal 16 Okt 2026, beserta solusinya.

---

## 2.1 Akun Meta Developer + verifikasi

1. Buka <https://developers.facebook.com> → login dengan akun Facebook.
2. Daftar sebagai developer (**Get Started**). Meta akan meminta **verifikasi
   identitas** (KTP/paspor) — selesaikan dulu; tanpa ini kamu tidak bisa
   membuat aplikasi.
3. Hasil kemarin: verifikasi selesai, baru bisa lanjut.

## 2.2 Membuat aplikasi

1. **My Apps → Create App.**
2. Pilih tipe/kasus penggunaan **Threads** (use case akses Threads API).
   Kemarin nama aplikasinya: **"Threads CMS"**.
3. Catat dua nilai ini — simpan di file lokal, JANGAN di repo:
   - **App ID** (`client_id`) — boleh dicantumkan di URL OAuth (tidak rahasia).
   - **App Secret** (`client_secret`) — RAHASIA. Hanya dipakai server-side
     saat exchange token.

> Template penyimpanan lokal (`~/.threads-bot/app.json`, `chmod 600`):
> ```json
> {
>   "app_id": "<APP_ID_KAMU>",
>   "app_secret": "<APP_SECRET_KAMU>",
>   "redirect_uri": "<CALLBACK_HTTPS_KAMU>",
>   "scope": "threads_basic,threads_content_publish,threads_manage_replies,threads_read_replies,threads_manage_insights"
> }
> ```

## 2.3 Pengaturan dasar aplikasi (wajib diisi)

Di **App Settings → Basic**:

- **Privacy Policy URL** — WAJIB ada, Meta menolak aplikasi tanpa ini.
  Kemarin solusinya: buat satu halaman statis sederhana berisi kebijakan
  privasi (boleh di-host di mana saja yang publik, mis. `https://muse.ai/s/...`),
  lalu tempel URL-nya dan klik **Simpan**.
- App Icon, nama tampilan, dsb — isi sewajarnya.

## 2.4 Tab "Izin dan Fitur" — tambah 5 permission (titik gagal #1)

Di pengaturan use-case Threads, buka tab **Permissions / Izin dan Fitur**.
Tambahkan 5 scope ini:

| Scope | Fungsi |
|-------|--------|
| `threads_basic` | Baca profil & post dasar |
| `threads_content_publish` | Menerbitkan post |
| `threads_manage_replies` | Mengelola balasan (auto-reply) |
| `threads_read_replies` | Membaca balasan |
| `threads_manage_insights` | Baca views/likes (analytics) |

⚠️ **Cara yang BENAR:** klik tombol **"+ Tambahkan"** (plus) **di tiap baris
permission** satu per satu.

❌ **Cara yang SALAH (kemarin gagal):** lewat tombol **"Tambahkan ke Tinjauan
Aplikasi"** / app review — itu jalur untuk aplikasi publik yang butuh review
Meta, bukan untuk menambahkan permission ke aplikasi sendiri. Hasilnya
permission tidak aktif dan token tidak punya scope yang cukup.

## 2.5 Redirect URI — harus HTTPS publik (titik gagal #2)

Di pengaturan **Threads API → URL Callback Alihkan** (Redirect URIs):

- ❌ **Jangan pakai `localhost`** — Meta MENOLAK dengan error **1349168**
  ("URL Blocked: This redirect failed because the redirect URI is not
  whitelisted in the app's Client OAuth Settings"). Kolomnya bahkan tidak bisa
  menyimpan URL localhost.
- ✅ Pakai **URL `https://` publik**, mis. halaman callback buatan sendiri
  (`https://domain-kamu.com/callback`) yang tugasnya cuma satu: **menangkap
  dan menampilkan `?code=`** dari URL.
- Setelah mengetik URL, tekan **Enter** sampai URL berubah menjadi **chip**
  (label) — itu tanda Meta sudah mengonfirmasi/menyimpannya. Kalau belum jadi
  chip, berarti belum tersimpan.

> Kemarin solusi yang dipakai: halaman callback publik sementara yang
> menampilkan authorization code, lalu code dikirim ke server untuk di-exchange.

## 2.6 Menambah Threads Tester (titik gagal #3)

Akun Threads yang mau dipakai (`@mayorihomebake`) harus didaftarkan sebagai
**tester** aplikasi — kalau tidak, login OAuth akan ditolak.

⚠️ **Cara yang BENAR:** di **pengaturan use-case Threads**, cari
**"Add or Remove Threads Testers"** → masukkan **USERNAME Threads**
(mis. `mayorihomebake`, tanpa @).

❌ **Cara yang SALAH (kemarin gagal):** lewat dialog **"Add People"** umum di
Roles — itu untuk peran aplikasi Facebook (admin/developer), bukan untuk
Threads tester. Invite lewat jalur ini tidak berlaku untuk login Threads.

## 2.7 Tester menerima undangan (titik gagal #4)

Undangan tidak otomatis aktif. Pemilik akun Threads harus:

1. Buka **aplikasi Threads** (HP).
2. Masuk ke **Website permissions → Invites**.
3. **Terima (Accept)** undangan dari aplikasi "Threads CMS".

Tanpa langkah ini, authorize tetap gagal walau semua setting sudah benar.

## 2.8 Peran (Roles) di aplikasi Meta

Struktur peran yang benar:

| Peran | Diatur di | Fungsi |
|-------|-----------|--------|
| **Admin / Developer** | App Dashboard → Roles → Add People (umum) | Kelola aplikasi Meta |
| **Threads Tester** | Pengaturan use-case → Add or Remove Threads Testers | Akun Threads yang boleh login OAuth |

Jangan tertukar: kemarin kegagalan berulang terjadi karena mencoba
menambahkan akun Threads lewat dialog Roles umum.

## 2.9 Graph API Explorer "Fitur Tidak Tersedia" (titik gagal #5)

Setelah aplikasi baru dibuat, tombol **Generate Access Token** di Graph API
Explorer menampilkan **"Fitur Tidak Tersedia"** ("Login Facebook saat ini
tidak tersedia untuk aplikasi ini, karena kami sedang memperbarui detail
tambahan untuk aplikasi ini. Silakan coba lagi nanti.")

- Ini masalah **provisioning di sisi Meta** untuk aplikasi baru — kemarin
  ditunggu >3 jam tidak sembuh-sembuh.
- **Solusi: jangan tunggu. Pakai alur OAuth manual** (dibahas lengkap di
  `03-token-oauth-threads.md`): buat authorize link sendiri → user login di
  `threads.net` → dapat `code` → exchange via API.

## 2.10 Checklist sebelum lanjut ke token

- [ ] Developer terverifikasi, aplikasi "Threads CMS" dibuat
- [ ] Privacy Policy URL terisi & tersimpan
- [ ] 5 permission ditambah via tombol + per baris
- [ ] Redirect URI = `https://` publik, sudah jadi chip
- [ ] Username Threads ditambah via "Add or Remove Threads Testers"
- [ ] Undangan diterima di Threads app → Website permissions → Invites
- [ ] `app.json` lokal terisi (app_id, app_secret, redirect_uri, scope), `chmod 600`

Kalau semua centang, lanjut ke `03-token-oauth-threads.md`.

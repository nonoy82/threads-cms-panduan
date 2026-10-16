# 03 — Alur OAuth & Generate Token Threads (DETAIL)

Token adalah **satu-satunya blocker** kemarin: dari pagi sampai siang gagal
terus, sore berhasil lewat **OAuth manual**. Bagian ini menjelaskan alurnya
persis seperti yang jalan di kode (`lib/threads_api.py`).

## 3.1 Peta alur (4 langkah)

```
1. AUTHORIZE   : user dibuka ke https://threads.net/oauth/authorize?...&response_type=code
                    → user login sebagai @akun-threads → Meta redirect ke
                      <redirect_uri>?code=<AUTH_CODE>
2. CODE→TOKEN  : server POST https://graph.threads.net/oauth/access_token
                    {client_id, client_secret, grant_type=authorization_code,
                     redirect_uri, code}
                    → dapat SHORT-LIVED token (~1 jam)
3. EXCHANGE    : GET https://graph.threads.net/access_token
                    ?grant_type=th_exchange_token&client_secret=...&access_token=<short>
                    → dapat LONG-LIVED token (60 hari)
4. VALIDASI    : GET https://graph.threads.net/v1.0/me?fields=id,username&access_token=<long>
                    → dapat user_id + username → simpan token
```

## 3.2 Langkah 1 — Bangun authorize URL

Parameter wajib:

```
https://threads.net/oauth/authorize?
  client_id=<APP_ID>
  &redirect_uri=<URL_CALLBACK_HTTPS_YANG_SUDAH_JADI_CHIP>
  &scope=threads_basic,threads_content_publish,threads_manage_replies,threads_read_replies,threads_manage_insights
  &response_type=code
  &state=<RANDOM_STRING_UNTUK_CEGAH_CSRF>
```

- `redirect_uri` **harus identik** dengan yang terdaftar di aplikasi
  (lihat 2.5) — beda satu karakter saja → error 1349168.
- Buka URL ini di browser → login sebagai akun Threads yang sudah jadi tester
  (lihat 2.6–2.7).

## 3.3 Langkah 2–3 — Tukarkan code jadi token

Di kode, ini fungsi `oauth_exchange_code(code, redirect_uri)`:

```python
# step 1: code -> short-lived token
POST https://graph.threads.net/oauth/access_token
  client_id, client_secret, grant_type="authorization_code",
  redirect_uri, code
→ {"access_token": "<SHORT>", ...}

# step 2: short -> long-lived (60 hari)
GET https://graph.threads.net/access_token
  ?grant_type=th_exchange_token
  &client_secret=<APP_SECRET>
  &access_token=<SHORT>
→ {"access_token": "<LONG_60_HARI>", ...}
```

Catatan:
- `client_secret` hanya dipakai **server-side** — jangan pernah taruh di
  frontend/URL yang bisa dilihat user.
- Kalau exchange gagal (mis. token ternyata sudah long-lived), pakai token
  yang ada dan lanjut — jangan hard-fail.

## 3.4 Langkah 4 — Validasi & simpan

```python
GET https://graph.threads.net/v1.0/me?fields=id,username&access_token=<LONG>
→ {"id": "27916086028064296", "username": "mayorihomebake"}
```

Simpan ke `~/.threads-bot/tokens.json` (`chmod 600`):

```json
{
  "mayorihomebake": {
    "access_token": "<LONG_LIVED_TOKEN>",
    "user_id": "27916086028064296",
    "obtained_at": "2026-10-16T..."
  }
}
```

Verifikasi kuota (opsional tapi disarankan):

```
GET https://graph.threads.net/v1.0/<USER_ID>/threads_publishing_limit
    ?fields=quota_usage,quota_total
→ kemarin: quota_total=250/hari, quota_usage=0
```

## 3.5 Refresh token (jangan lupa!)

- Long-lived token berlaku **60 hari**. Kemarin diperoleh 16 Okt 2026 →
  **wajib refresh sebelum ~15 Des 2026**.
- Di kode sudah ada `refresh_token()` / `exchange_long_lived()` — tapi **belum
  otomatis**. Untuk versi serius: pasang cron mingguan yang cek umur token
  dan refresh otomatis sebelum kedaluwarsa.

## 3.6 Semua kegagalan token kemarin (ringkas)

| Gejala | Penyebab | Solusi |
|--------|----------|--------|
| Graph API Explorer: "Fitur Tidak Tersedia" | Provisioning app baru di Meta (>3 jam) | Lewati; pakai OAuth manual (3.1–3.4) |
| Error 1349168 "URL Blocked" | `redirect_uri` tidak terdaftar / pakai localhost | Daftarkan URL `https` publik, konfirmasi jadi chip |
| Kolom callback menolak localhost | Meta memang tidak mengizinkan localhost untuk Threads API | Pakai halaman callback publik |
| Login ditolak padahal setting benar | Akun belum jadi Threads Tester / invite belum di-accept | 2.6 + 2.7 |
| Token tidak bisa publish | Scope kurang (permission salah jalur) | 2.4 — tambah via tombol + per baris |
| "Add People" tidak mempan untuk akun Threads | Jalur salah (itu untuk role FB) | Pakai "Add or Remove Threads Testers" |

## 3.7 Prinsip yang bisa dibawa ke SaaS

Alur di atas adalah **OAuth 2.0 Authorization Code standar** — pola yang sama
berlaku untuk Instagram Graph API, TikTok API, YouTube API, dsb. Yang berubah
hanya: nama endpoint, nama scope, dan umur token. Kalau kamu paham pola
"authorize → code → exchange → long-lived → refresh", kamu bisa
menghubungkan **API apapun**.

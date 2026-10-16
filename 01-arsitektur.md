# 01 — Arsitektur: Apa yang Sebenarnya Dibangun

## Ide besar (satu kalimat)

Sebuah **server kecil yang nyala 24/7**, tiap interval mengecek "apa yang harus
dikerjakan sekarang": generate thread pakai AI → simpan sebagai draft terjadwal
→ publish ke Threads API tepat waktu → balas otomatis dengan soft-sell.

## Komponen (repo `threads-cms-v2`)

```
threads-cms-v2/
├── server.py          # HTTP server (Python stdlib, TANPA framework) + scheduler thread
├── lib/
│   ├── threads_api.py # OAuth, publish, insights, auto-reply (Threads Graph API)
│   ├── ai.py          # Generate thread via LLM OpenAI-compatible + persona per akun
│   ├── db.py          # SQLite (db/cms.db): posts, accounts, media
│   ├── r2.py          # Upload gambar ke object storage S3-compatible (Supabase/R2/B2)
│   ├── scraper.py     # Fallback ambil gaya tulisan dari link Threads
│   └── hooks.py       # Hook library (10 tipe hook stop-scrolling)
├── web/               # Dashboard (index.html + app.js + app.css)
├── scripts/migrate.py # Migrasi DB
├── deploy/            # Autostart (launchd macOS / cron watchdog)
└── .env               # Credential storage & AI (TIDAK di-commit)
```

## Alur data autopilot (100% AUTO)

```
[Scheduler thread: cek tiap 30 detik]
        │
        ├─► Ada post terjadwal yang waktunya tiba? ──► publish via Threads API
        │                                              (text ≤500 char/part, auto-split)
        │
        ├─► Slot kosong & autopilot ON? ──► AI generate thread (persona akun)
        │                                  ──► simpan sebagai scheduled post
        │
        └─► Ada balasan baru di post? ──► auto soft-sell reply
                                           (link produk di REPLY, bukan di thread)
```

## Aturan anti-spam (penting untuk akun sehat)

- **Cap 2 post/hari per akun** (kemarin: jam 8/12/18/21 WIB dipilih acak 2 slot).
- **Min-gap antar post** — scheduler menolak menjadwalkan terlalu rapat.
- **Anti-dedup** — topik yang sudah pernah diposting tidak di-generate ulang.
- **Jam acak anti-pola** — waktu publish diacak dalam slot, tidak monoton.

## Penyimpanan & secret

| Secret | Lokasi | Izin file |
|--------|--------|-----------|
| Meta `app_id` + `app_secret` | `~/.threads-bot/app.json` | `600` |
| Token Threads long-lived | `~/.threads-bot/tokens.json` | `600` |
| Storage + AI key | `threads-cms-v2/.env` | `600`, di-`gitignore` |

Prinsip: **secret tidak pernah masuk repo, tidak pernah masuk log, tidak pernah
dicetak ke layar.**

## Endpoint penting

- `http://127.0.0.1:8455` — dashboard utama
- `http://127.0.0.1:8455/mini` — mini dashboard monitoring
- `http://127.0.0.1:8455/api/mini-stats` — JSON status
  (`scheduler_ok`, `ai_status`, `storage_status`, `posted_total`, `queued`, `failed`)

## Kenapa desainnya begini (keputusan yang bisa kamu tiru)

1. **Python stdlib tanpa framework** → deploy di mana saja, dependensi minim
   (cuma `boto3` untuk storage).
2. **SQLite** → cukup untuk 1 user / 1 server; naik ke Postgres kalau jadi SaaS.
3. **Threads API hanya terima gambar via URL publik** → wajib object storage +
   CDN, tidak bisa upload binary langsung.
4. **Scheduler butuh proses 24/7** → serverless (Vercel/Cloudflare Workers)
   tidak cocok; pakai VPS / komputer yang selalu nyala + watchdog.

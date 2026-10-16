# 04 — Setup Server Lokal (Clone → Jalan)

> Prasyarat: Python 3.10+, `pip`, akses internet. Diuji di Linux; macOS mirip
> (lihat `deploy/` untuk autostart launchd).

## 4.1 Clone & dependensi

```bash
git clone https://github.com/<USER>/<REPO>.git
cd <REPO>

# Satu-satunya dependensi eksternal: boto3 (untuk object storage)
python3 -m venv venv
./venv/bin/pip install boto3
```

Kenapa cuma boto3: server HTTP-nya pakai **Python stdlib** (`http.server`),
database pakai **SQLite** (stdlib), request HTTP pakai `urllib` (stdlib).
Makin sedikit dependensi → makin gampang diduplikasi di Trae/Cursor/VPS.

## 4.2 File environment

```bash
cp .env.example .env
chmod 600 .env
# lalu isi credential storage + AI (lihat 05-storage-dan-ai.md)
```

`.env` **wajib masuk `.gitignore`** — kemarin file ini memang tidak di-commit.

## 4.3 Inisialisasi database

```bash
./venv/bin/python3 lib/db.py        # buat db/cms.db + tabel
./venv/bin/python3 scripts/migrate.py  # migrasi akun & queue
```

Isi DB: tabel `posts` (draft/scheduled/posted), `accounts`, `media`,
`autoreply_config`, dsb. (lihat `db/schema*.sql`).

## 4.4 Jalankan

```bash
./run.sh
# → http://127.0.0.1:8455
```

`run.sh` memilih Python dari venv (yang punya boto3), lalu `exec server.py`.
Server menjalankan **dua hal sekaligus**:

1. **HTTP server** di port `8455` (dashboard + API).
2. **Scheduler thread** — loop tiap 30 detik: cek post yang jatuh tempo →
   publish; cek slot kosong → generate via AI; cek balasan → auto-reply.

## 4.5 Verifikasi instalasi

```bash
curl -s http://127.0.0.1:8455/api/mini-stats
```

Harus kembali HTTP 200 dengan JSON berisi `scheduler_ok`, `ai_status`,
`storage_status`. Kalau `ai_status` / `storage_status` belum "siap", server
tetap jalan (graceful degrade — scheduler hanya log peringatan, tidak crash),
tapi autopilot belum bisa generate/publish. Lanjut ke bab 05.

## 4.6 Endpoint yang ditambahkan kemarin

- `/mini` — mini dashboard monitoring (HTML ringan).
- `/api/mini-stats` — JSON status untuk watchdog & dashboard eksternal.

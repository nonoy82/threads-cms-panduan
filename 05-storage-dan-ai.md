# 05 — Object Storage (Supabase) + Endpoint AI (Gemini)

## 5.1 Kenapa butuh object storage

Threads API **tidak menerima upload gambar binary langsung** — gambar harus
diserahkan sebagai **URL publik**. Jadi alurnya: user drag-drop gambar di
dashboard → server upload ke object storage → dapat URL publik → URL itu yang
dikirim ke Threads API.

Kode (`lib/r2.py`) digeneralisasi ke **S3-compatible apa saja** via env:

```
S3_ENDPOINT_URL, S3_REGION, S3_ACCESS_KEY_ID, S3_SECRET_ACCESS_KEY,
S3_BUCKET, S3_PUBLIC_BASE, S3_PREFIX
```

## 5.2 Setup Supabase (yang dipakai kemarin)

Awalnya rencana pakai Cloudflare R2, tapi kemarin user memilih **Supabase**
(verifikasi email Backblaze bermasalah):

1. Buat project Supabase → catat **project ref** & **region**
   (kemarin: region `ap-southeast-1`).
2. **Storage → New Bucket** → nama mis. `cms-media` → set **PUBLIC**
   (gambar harus bisa diakses publik tanpa auth).
3. **Storage → Configuration → S3** → **Enable S3 protocol** →
   **Generate access key** → dapat Access Key ID + Secret.
4. Isi ke `.env`:
   ```
   S3_ENDPOINT_URL=https://<PROJECT_REF>.supabase.co/storage/v1/s3
   S3_REGION=ap-southeast-1
   S3_ACCESS_KEY_ID=<ISI>
   S3_SECRET_ACCESS_KEY=<ISI>
   S3_BUCKET=cms-media
   S3_PUBLIC_BASE=https://<PROJECT_REF>.supabase.co/storage/v1/object/public/cms-media
   S3_PREFIX=threads
   ```
5. **Uji end-to-end** (kemarin lolos): ping OK → upload file test → GET URL
   publik 200 dengan byte identik → hapus file test.

> Alternatif di `.env.example`: Cloudflare R2 (10 GB gratis/bulan) atau
> Backblaze B2 — ganti env-nya saja, kode tidak berubah.

## 5.3 Endpoint AI (Gemini, OpenAI-compatible)

Generate thread memakai LLM **OpenAI-compatible** — artinya endpoint apa saja
yang meniru API OpenAI bisa dipakai (Gemini, OpenAI, Groq, Ollama lokal, dll):

```
AI_BASE_URL=<URL_ENDPOINT>/v1
AI_API_KEY=<ISI>
AI_MODEL=<NAMA_MODEL>
```

Kemarin: **Gemini** via endpoint OpenAI-compatible-nya.

Konsep **persona per akun** (`lib/ai.py`):
- Setiap akun punya: nama persona, instruksi gaya, bahasa, + link Threads referensi.
- "Belajar gaya": fetch post dari link referensi (Graph API kalau akun sendiri,
  fallback scrape) → AI rangkum ciri gaya → disimpan → dipakai saat generate.
- Playbook: **tanpa em-dash**, soft-sell (link produk di reply, bukan di thread).

## 5.4 Graceful degrade

Kalau `AI_API_KEY` belum di-set, scheduler **tidak crash** — hanya log
peringatan dan melewati generate. Ini penting: server tetap sehat walau satu
komponen belum dikonfigurasi (terbukti kemarin saat AI/storage belum siap).

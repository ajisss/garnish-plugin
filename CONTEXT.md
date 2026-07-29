# Konteks Proyek — GARNISH

*"The final touch before you serve it to the client."*

**Baca file ini duluan sebelum ngerjain apapun.** Ini rangkuman semua
keputusan yang sudah diambil di luar sesi ini — jangan tanya ulang hal yang
sudah dijawab di sini, tapi tetap tanya kalau ada detail teknis baru yang
belum diputuskan.

## Latar Belakang

Ini dibuat buat hackathon internal (tema: "AI Agents — Automate the Work We
Do Every Day"), dikerjakan **solo** dalam **2 hari**. User adalah UI/UX
designer di agency (Etalas) yang sebelumnya sudah membangun plugin
`design-agent` (lihat bagian "Reuse dari design-agent" di bawah).

## Problem Statement

> Kita sering harus audit landing page/screen klien secara manual sebelum
> ngasih rekomendasi redesign — buang waktu berjam-jam buat identifikasi
> masalah desain & konten satu-satu. Kita mau bikin agent yang bisa audit
> otomatis (desain + konten), kasih rekomendasi konkret, dan langsung
> generate revisi (copy + UI konsisten pakai component library internal),
> supaya proses evaluasi-ke-proposal yang biasanya makan waktu berhari-hari
> jadi hitungan menit.

## Defining the Agent

| Field | Isian |
|---|---|
| **User** | Tim internal Etalas (sales buat pitch, PM/designer buat deliverable klien) |
| **Input** | URL landing page ATAU screen aplikasi apapun (bukan cuma landing page — lihat "Broadened Framing" di bawah) |
| **Agent Actions** | Screenshot & ekstrak halaman → deteksi gejala fatal → checkpoint tanya user mau fix apa → eksekusi fix |
| **Output** | Laporan temuan fatal (prioritized) + revisi konten dan/atau desain |
| **Success Criteria** | Dikasih 1 URL bermasalah, agent nemuin minimal 2-3 gejala fatal valid, user pilih mana yang mau difix, hasil revisi keluar dengan laporan before/after |

## Keputusan Desain Paling Penting: "Fatal-Only", Bukan Audit Lengkap

Ini keputusan sengaja — GARNISH BUKAN checklist audit menyeluruh (beda dari
tools kayak Unbounce/HubSpot Grader yang udah ada). Fokus cuma ke gejala
yang benar-benar mengganggu/fatal, biar laporan pendek dan actionable,
bukan overwhelming.

### Gejala fatal & cara deteksinya (PENTING — bedakan yang terukur vs judgment)

| Gejala | Terukur objektif? | Cara deteksi |
|---|---|---|
| Kontras teks gak kebaca | ✅ Ya | Hitung rasio kontras WCAG dari computed CSS (via `extract-styles.py`) |
| Style tombol/komponen gak konsisten dalam 1 halaman | ✅ Ya | Baca computed CSS semua tombol, bandingkan (via `extract-styles.py`) |
| CTA gak keliatan di atas fold | ✅ Ya | Cek posisi bbox section relatif ke viewport height |
| Placeholder ketinggalan ("Lorem ipsum", "TODO", dll) | ✅ Ya | Cari pola teks di HTML mentah |
| Layout rusak/overlapping element | ⚠️ Sebagian | Deteksi visual dari screenshot, lebih rewel tapi masih bisa dicoba |
| Value proposition gak jelas di headline | ❌ Judgment | Penilaian Claude — WAJIB dilabel sebagai penilaian, bukan fakta |
| Gak ada trust signal/social proof | ❌ Judgment | Penilaian Claude — WAJIB dilabel sebagai penilaian |

**Aturan wajib:** urutan prioritas pengerjaan adalah gejala TERUKUR dulu
(paling reliable, bisa dibuktikan dengan angka), baru gejala JUDGMENT
sebagai pelengkap. Setiap temuan kategori judgment HARUS dilabel eksplisit
ke user sebagai "penilaian AI", bukan disamarkan seolah fakta pasti — ini
prinsip yang sama dengan confidence marker di plugin `design-agent`.

## Reuse dari `design-agent` (plugin lain yang sudah ada, dependency)

GARNISH TIDAK membangun ulang pipeline design-to-code dari nol, dan TIDAK
membundel copy `design-agent` di repo ini. GARNISH depend ke plugin
`design-agent` yang sudah ada terpisah (`ajisss/design-agent-plugin`,
v1.2.0) — install `garnish` idealnya ikut resolve dependency ini, tapi
kalau marketplace berbeda tidak bisa auto-resolve, `design-fix` mengecek
sendiri di runtime dan kasih instruksi install manual kalau belum ada
(lihat `skills/design-fix/SKILL.md`).

Untuk bagian "fix desain", GARNISH memanggil skill `design-agent`:
- `/design-agent:inspo` — cari referensi baru yang menjawab temuan fatal
  spesifik (bukan referensi generik seluruh halaman)
- `/design-agent:select` — checkpoint pemilihan referensi tetap berlaku
- `/design-agent:spec` — ekstrak referensi terpilih jadi token, scope
  hanya komponen/section yang fatal
- `/design-agent:build` — rebuild HANYA komponen/section yang fatal, pakai
  component library yang di-scaffold `design-fix`

**Component library baru** (Button, Card, Input — cuma 3 komponen, jangan
lebih) di-scaffold oleh `design-fix` sendiri (bukan skill terpisah), kalau
project belum punya, dipakai `design-agent:build` supaya hasil fix desain
konsisten strukturnya.

Untuk "fix konten" (rewrite copy), GARNISH TIDAK pakai Superpowers — itu
kesalahpahaman yang sempat muncul dan sudah dikoreksi. Superpowers itu
metodologi coding (TDD/planning), bukan alat buat menulis ulang copy.
Content-fix itu murni instruksi tertulis-ulang oleh Claude berdasarkan
temuan audit.

## Broadened Framing — Bukan Cuma Landing Page

Awalnya scope cuma "landing page audit", tapi diputuskan diperluas jadi
generic: bisa audit screen/halaman apapun (dashboard, settings, onboarding,
dll), bukan cuma landing page marketing. Ini memperluas target klien dari
"yang punya landing page" ke "siapapun yang punya produk digital". Secara
teknis TIDAK menambah kompleksitas signifikan — screenshot & deteksi
gejala fatal itu proses yang sama untuk jenis halaman apapun, asal skill-nya
ditulis generic (jangan hardcode asumsi "pasti ada hero/CTA/social proof").
**Prioritas pengembangan & testing tetap landing page dulu** — dashboard
menyusul tanpa perlu ubah skill-nya kalau memang ditulis generic.

**Batasan yang tetap berlaku:** GARNISH audit SATU halaman per run. TIDAK
audit multi-halaman, user flow, atau navigasi antar halaman — itu di luar
scope (terlalu kompleks buat solo 2 hari), disebut sebagai "Future
Improvements" saja.

## Model Bisnis (buat konteks "kenapa ini penting", bukan tugas teknis)

1. **Sales/pitch tool** — dipakai tim sales Etalas sebelum ketemu calon
   klien, jadi bahan pitch konkret
2. **Deliverable berbayar** — "Audit Report" jadi paket terpisah sebelum
   kontrak redesign penuh
3. **Standalone tool** (jangka panjang, TIDAK jadi fokus hackathon) —
   freemium buat dijual ke luar Etalas

## Nama & Positioning

- **Nama**: GARNISH (bukan akronim, murni nama dengan metafora "sentuhan
  akhir sebelum disajikan")
- **Hubungan dengan tools lain**: kalau SANDWICH itu "dapur" yang masak
  requirement jadi spec, GARNISH itu quality check terakhir sebelum
  "hidangan" (produk/halaman) keluar ke meja klien
- **Diferensiasi dari kompetitor** (Unbounce, HubSpot Grader, dll): fokus
  fatal-only (bukan checklist lengkap) + langsung fix pakai design system
  internal sendiri, bukan cuma kasih skor

## Prinsip yang Diwarisi dari `design-agent` (tetap berlaku di sini)

- **Checkpoint adalah hard stop**, bukan formalitas — tunggu jawaban
  eksplisit user sebelum lanjut tahap berikutnya
- **Registry/state harus di file**, bukan diandalkan dari ingatan chat —
  GARNISH punya `.garnish/registry/` sendiri (`audits.json`, `journal.jsonl`),
  di-scaffold via skill `/garnish:init`, dengan stable ID per audit (`A-00X`)
  dan per temuan (`F-00X`), mengikuti pola yang sama persis dengan SANDWICH
  dan `design-agent`
- **Jujur soal ketidakpastian** — penilaian judgment vs fakta terukur harus
  dibedakan secara eksplisit ke user
- **Konten bukan bagian dari style yang direplikasi** — kalau fix desain
  reuse referensi/benchmark, jangan timpa konten yang sudah ditulis dengan
  teks dari sumber lain

## Prioritas Eksekusi (kalau waktu 2 hari mepet, urutan ini yang dipegang)

1. **Wajib solid**: skill `garnish:check` — deteksi gejala fatal (terukur
   dulu, baru judgment)
2. **Wajib solid**: checkpoint "mau dibenerin yang mana" + minimal 1 jalur
   eksekusi (content-fix)
3. **Selesai**: integrasi ke `design-agent` + component library buat fix
   desain, lewat skill `garnish:design-fix` — semua 4 skill (`init`,
   `check`, `content-fix`, `design-fix`) sudah solid, bukan lagi stretch
   goal.

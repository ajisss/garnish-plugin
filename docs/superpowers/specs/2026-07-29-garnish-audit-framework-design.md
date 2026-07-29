# GARNISH — Audit Framework Grounding & Scope Selection

Refinement kedua: bikin `/garnish:check` audit berdasarkan framework UI/UX
yang mapan (bukan heuristik ad-hoc), kasih saran perbaikan konkret per
temuan, dan tambah checkpoint pemilihan scope di awal.

## 1. Checkpoint scope — sebelum audit mulai

Langkah baru di paling awal `check` (sebelum ekstraksi halaman): tanya
user mau audit scope apa, **multi-select**:
> "Mau audit apa? Bisa pilih lebih dari satu: Konten, UI/UX, Komponen,
> WCAG, atau Semua."

Tunggu jawaban eksplisit — hard stop, bukan asumsi default "semua". Hanya
jalankan modul deteksi yang sesuai scope yang dipilih (hemat waktu &
laporan tetap fokus, sejalan prinsip fatal-only).

**Fatal-only tetap berlaku PER SCOPE** — dalam scope yang dipilih, tetap
cuma laporkan yang benar-benar mengganggu, bukan checklist lengkap semua
kemungkinan di kategori itu.

## 2. Rubric per scope (tertanam di `skills/check/SKILL.md`, bukan mekanisme subagent terpisah)

Ditulis sebagai referensi ringkas di dalam skill supaya Claude selalu
pakai kerangka yang sama tiap audit (bukan penilaian bebas yang bisa
beda-beda tiap run) — ini portable, gak butuh mekanisme custom subagent
yang belum tentu konsisten kepanggil di semua environment instalasi.

- **Konten**: prinsip CRO dasar — kejelasan value proposition, kejelasan
  & kekuatan CTA copy, ada/tidaknya social proof & urgency, friksi
  (panjang form, langkah berlebih). Existing: placeholder, value-prop,
  trust-signal. Ditambah: framing "saran" yang menjelaskan KENAPA (prinsip
  CRO yang dilanggar) dan CONTOH perbaikan umum — tanpa angka/persentase
  dampak yang dikarang.
- **UI/UX**: 10 Usability Heuristics Nielsen (versi ringkas, cuma yang
  relevan buat audit halaman statis: match real world, consistency &
  standards, recognition over recall, aesthetic & minimalist design) +
  prinsip Gestalt (proximity, alignment, similarity — buat menilai
  grouping/spacing dari data bbox yang ada). Existing: CTA position,
  layout rusak (overlap/overflow).
- **Komponen**: existing consistency check (radius/padding/warna tombol &
  card), dibingkai dengan prinsip atomic design (variant/state komponen
  harus konsisten dalam satu design system).
- **WCAG**: existing kontras. DITAMBAH (baru, disepakati sekarang, gampang
  dicek dari HTML mentah yang sudah di-fetch, gak butuh tool baru):
  - **Alt text**: `<img>` tanpa atribut `alt` atau `alt=""` pada gambar
    yang bukan dekoratif jelas = fatal.
  - **Heading hierarchy**: urutan heading loncat level (mis. h1 langsung
    ke h3 tanpa h2) = fatal. Bisa dicek dari `typography[].level` yang
    sudah ada per section di hasil `extract-styles.py`, plus urutan
    heading di HTML mentah untuk section yang datanya gak lengkap dari
    situ.

## 3. Setiap temuan wajib punya field `suggestion`

Bukan cuma "apa yang salah", tapi juga "kenapa ini masalah (dikaitkan ke
prinsip di atas) dan gimana biasanya diperbaiki" — konkret, bukan generik.
Contoh: "CTA 'Submit' kurang actionable — prinsip CRO: copy CTA yang
spesifik & berorientasi hasil (mis. 'Dapatkan Diskon 20%') biasanya lebih
efektif dari label generik. Saran: ganti ke copy yang menyebutkan
benefit/hasil konkret."

**Tidak ada estimasi angka/persentase dampak** (mis. "naikin konversi
15%") — itu klaim yang gak bisa diverifikasi tanpa data real, akan jadi
AI slop kalau dikarang. Saran tetap kualitatif & grounded ke prinsip.

## 4. Perubahan schema

Field baru per finding di `audits.json` (lihat `SCHEMA.md`):
```json
{
  "scope": "konten | ui-ux | komponen | wcag",
  "suggestion": "string — kenapa masalah + cara umum memperbaiki, TANPA angka dikarang"
}
```
`type` enum ditambah: `alt-text`, `heading-hierarchy`. `scope` field baru
ini yang dipakai buat filter mana yang dijalankan sesuai pilihan
checkpoint Langkah 1, dan buat grouping laporan.

## Yang TIDAK berubah

- Tetap fatal-only per scope, bukan audit menyeluruh.
- Tidak ada mekanisme custom subagent terpisah — rubric tertanam sebagai
  referensi di skill, dibaca & diikuti Claude tiap kali skill ini jalan.
- `content-fix`/`design-fix` dan QA loop-nya tidak berubah — cuma baca
  field `scope`/`suggestion` tambahan kalau relevan buat konteks fix,
  logic utamanya tetap sama.

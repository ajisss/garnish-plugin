# GARNISH Refinement — Auto-Init, Layout Detection, Post-Fix QA Loop

Refinement singkat setelah user coba plugin dan minta 3 perubahan ke alur v0.3.0 yang sudah ada.

## 1. Hilangkan `/garnish:init` sebagai langkah eksplisit user

`skills/check/SKILL.md` Step 0 sekarang langsung melakukan langkah init
(bikin `.garnish/registry/{audits.json,journal.jsonl,SCHEMA.md}`) sendiri
kalau belum ada, dengan 1 baris info ke user ("registry belum ada, saya
setup dulu") — bukan lagi mengarahkan user ke `/garnish:init` secara
terpisah. `skills/init/SKILL.md` tetap ada sebagai skill mandiri untuk
kasus reset manual (deskripsinya disesuaikan supaya jelas ini opsional,
bukan prasyarat wajib).

## 2. Deteksi "layout rusak" (sebelumnya dead field)

Final review sebelumnya menemukan `type: "layout-rusak"` disebut di
`design-fix` tapi tidak pernah dideteksi `check`. Sekarang diimplementasi
pakai data yang sudah tersedia dari `extract-styles.py` (tanpa perlu tool
baru): untuk tiap pasangan section berurutan, bandingkan `bbox` — kalau
`section[i].bbox.y + height` melebihi `section[i+1].bbox.y` (overlap
vertikal), atau `bbox.width` sebuah section jauh melebihi viewport (indikasi
overflow horizontal), tandai `type: "layout-rusak"`, `category: "measured"`.
Tambahkan `layout-rusak` sebagai nilai eksplisit di enum `type` pada
`SCHEMA.md` (sebelumnya cuma tertutupi `lainnya`).

## 3. QA loop setelah fix, sebelum ditampilkan ke user

Baik `content-fix` maupun `design-fix`, sebelum langkah "tampilkan
before/after ke user", menjalankan re-check: audit ulang (pakai logic
deteksi yang sama dengan `check`, cuma untuk jenis temuan yang relevan ke
fix ini) terhadap hasil yang sudah diperbaiki — untuk `design-fix`
terhadap dev server lokal (`extract-styles.py <url-dev-server> ...`),
untuk `content-fix` terhadap teks yang baru ditulis (cek ulang pola
placeholder/vague-CTA yang sama).

- Kalau bersih (temuan yang di-fix beneran hilang) → lanjut tampilkan hasil.
- Kalau masih ada sisa → ulangi langkah fix (khusus temuan yang masih
  bermasalah), lalu cek ulang lagi. Maksimal 3 putaran total.
- Putaran ke-3 masih ada sisa → tampilkan apa adanya ke user, sebutkan
  eksplisit mana yang belum bersih dan kenapa loop dihentikan — JANGAN
  klaim "selesai" kalau masih ada temuan aktif.

Update registry: field finding tetap `content-fixed`/`design-fixed` cuma
kalau re-check-nya beneran bersih. Kalau mentok 3 putaran dengan sisa
masalah, status finding tetap `open` (bukan `content-fixed`/`design-fixed`)
dan dicatat di laporan sebagai "belum berhasil diperbaiki otomatis".

## Scope tidak berubah

- WCAG di luar kontras (alt text, semantic heading, dll) — TETAP di luar
  scope untuk sekarang, sesuai keputusan sebelumnya.
- "Komponen yang digunakan" — tetap dicakup oleh deteksi konsistensi
  tombol/card yang sudah ada, tidak ada aspek baru ditambahkan.

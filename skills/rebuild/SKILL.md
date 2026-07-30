---
name: rebuild
description: Bangun ulang landing page BARU dari nol berdasarkan SEMUA temuan audit /garnish:check (full rebuild, bukan cuma bagian yang fatal seperti /garnish:design-fix). Gunakan skill ini ketika user pilih opsi "bikin landing page baru" di checkpoint setelah /garnish:check. Konten dari halaman asli dipertahankan, cuma bagian yang ditandai fatal yang ditulis ulang sesuai suggestion. Referensi desain dicari baru (full landing page, bukan component showcase sempit) lewat design-agent, dibangun ke PROJECT BARU terpisah — tidak menimpa apapun secara otomatis.
---

# /garnish:rebuild — Full Rebuild Landing Page

Skill ini orchestrator, sama seperti `/garnish:design-fix` — tidak
menulis ulang logic `design-agent`. Bedanya: scope-nya SELURUH halaman
(bukan cuma komponen/section fatal), dan output-nya project baru
terpisah (bukan project yang sedang dikerjakan user).

## Langkah

### 1. Baca SEMUA temuan dari audit
Ambil SEMUA `findings` berstatus `open` dari audit yang dimaksud di
`.garnish/registry/audits.json` — semua scope (konten, ui-ux, komponen,
wcag), bukan cuma yang user pilih spesifik (rebuild ini menyeluruh).

Pisahkan:
- **Temuan konten** (`scope: "konten"`, type `placeholder`/`value-prop`/
  `trust-signal`): dipakai Langkah 4 (tingkatkan konten).
- **Temuan desain** (`scope` lain): dipakai sebagai konteks brief
  pencarian referensi di Langkah 5.

### 2. Tanya lokasi project baru — HARD STOP
> "Rebuild ini bakal bikin project BARU terpisah (gak menimpa apapun).
> Mau disimpan di folder/nama apa?"

Tunggu jawaban eksplisit sebelum bikin apapun.

### 3. Setup project baru
- Kalau folder yang disebutkan belum ada, buat foldernya.
- Tanya framework & styling (sama seperti Langkah 1 `/design-agent:build`)
  kalau belum jelas dari konteks.
- Jalankan `/design-agent:init` di project baru itu — jangan asumsikan
  `.design/registry/` sudah ada di project yang baru dibuat.

### 4. Tingkatkan konten (mirip `/garnish:content-fix`, tapi menyeluruh)
Untuk tiap section di halaman ASLI yang diaudit:
- Section yang temuannya ditandai fatal (`placeholder`/`value-prop`/
  `trust-signal`) → tulis ulang mengikuti `suggestion` yang ada (lensa
  AIDA/PAS kalau relevan) — kasih 2-3 opsi seperti `content-fix`, tunggu
  user pilih.
- Section yang TIDAK ada temuan fatal → **pertahankan konten ASLI**
  (headline, body copy, dll) — JANGAN ditulis ulang/diganti tanpa alasan,
  ini bukan kesempatan buat "sekalian dirapiin" konten yang sudah oke.
- Tanya konteks brand/tone dulu kalau belum ada (sama seperti
  `content-fix` Langkah 2).

### 5. Cari referensi baru — FULL landing page (BUKA lebar, beda dari `design-fix`)
Susun brief ke `/design-agent:inspo` berdasarkan jenis produk/industri
halaman yang diaudit + temuan desain dari Langkah 1. BEDA dari
`design-fix`: brief ini TIDAK meminta referensi sempit (component
showcase) — justru minta landing page LENGKAP yang jadi contoh baik buat
struktur/gaya keseluruhan. Contoh:
> "Cari referensi landing page [jenis produk/industri] yang solid —
> struktur hero-features-social proof-CTA jelas, kontras tinggi, CTA
> menonjol, gak ada masalah [sebutkan temuan desain utama dari audit]"

### 6. Checkpoint pemilihan referensi
`/design-agent:select` seperti biasa — hard stop, jangan putuskan sendiri.

### 7. Scaffold component library
Sama persis seperti `/garnish:design-fix` Langkah 6 — cek stack, cek
Button/Card/Input yang sudah ada (reuse, jangan timpa), atau bikin baru
kalau belum ada.

### 8. Ekstrak spec — FULL halaman
Panggil `/design-agent:spec` pada referensi terpilih. BEDA dari
`design-fix`: TIDAK ada instruksi pembatasan scope token — token diambil
buat SELURUH struktur halaman referensi, karena ini rebuild penuh.
Konfirmasi struktur section (`sectionsConfirmed`) berlaku normal.

### 9. Build — SELURUH halaman
Panggil `/design-agent:build` dengan spec dari Langkah 8. BEDA dari
`design-fix`: TIDAK ada pembatasan "cuma komponen fatal" — bangun semua
section sesuai `sections` di spec, pakai konten dari Langkah 4 (bukan
konten asli referensi — tetap ikuti aturan pemisahan style/konten yang
sudah ada di `design-agent`) dan component library dari Langkah 7.

### 10. QA — re-audit hasil rebuild
Setelah build selesai, jalankan ulang deteksi yang relevan dari
`/garnish:check` (semua scope yang tadinya diaudit di Langkah 1) terhadap
dev server project baru — logic sama seperti QA di `/garnish:design-fix`
Langkah 9 (`extract-styles.py` buat scope measured, `WebFetch` buat
placeholder/alt-text/heading, dst). Maksimal 3 putaran perbaikan kalau
masih ada temuan tersisa — kalau di putaran ke-3 masih ada sisa, laporkan
apa adanya, JANGAN klaim rebuild "sudah sempurna".

### 11. Update registry & laporkan
Update entry audit di `.garnish/registry/audits.json`:
```json
{ "status": "resolved", "rebuiltTo": "<path project baru>" }
```
(Set `resolved` karena rebuild mencakup SEMUA temuan, bukan sebagian —
beda dari `content-fix`/`design-fix` yang cuma resolve kalau semua
finding lain juga sudah `-fixed`.)

Append journal:
```json
{"ts":"<ISO 8601>","event":"rebuild_started","auditId":"A-00X"}
{"ts":"<ISO 8601>","event":"rebuild_completed","auditId":"A-00X","rebuiltTo":"<path>"}
```

Laporkan ke user: lokasi project baru, section apa aja yang di-generate,
referensi yang dipakai (link/ID), konten mana yang ditulis ulang vs
dipertahankan asli, dan hasil QA (lolos di putaran ke berapa, atau sisa
temuan yang belum bersih setelah 3 putaran).

## Yang TIDAK boleh dilakukan skill ini
- Menimpa project yang sedang dikerjakan user tanpa konfirmasi lokasi
  eksplisit di Langkah 2
- Menulis ulang konten section yang TIDAK ditandai fatal — pertahankan
  aslinya
- Mencari referensi sempit (component showcase) seperti `design-fix` —
  rebuild butuh referensi full landing page
- Membatasi scope `/design-agent:spec`/`build` ke komponen tertentu —
  rebuild membangun SELURUH halaman
- Menulis ulang logic `inspo`/`select`/`spec`/`build`/ekstraksi audit
  sendiri — selalu panggil skill yang sudah ada
- Skip checkpoint pemilihan referensi atau checkpoint lokasi project baru
- Menandai audit `"resolved"` sebelum lolos QA Langkah 10, atau
  melanjutkan retry lebih dari 3 putaran tanpa melapor apa adanya

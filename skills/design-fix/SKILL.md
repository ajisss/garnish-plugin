---
name: design-fix
description: Perbaiki desain (UI/UX, komponen, WCAG) berdasarkan temuan fatal dari /garnish:check. Nanya dulu mau fix SEMUA temuan desain atau yang prioritas TINGGI aja. Untuk temuan visual (kontras tombol, konsistensi komponen, posisi CTA, layout rusak, evaluasi heuristik, section landing page yang hilang) — cari referensi baru yang menjawab temuan spesifik (reuse plugin design-agent), scaffold component library kalau belum ada, lalu rebuild HANYA komponen/section yang ditandai fatal (bukan seluruh halaman). Untuk temuan struktural (alt text, heading hierarchy, target size, icon label) — edit kode langsung tanpa perlu referensi visual. Gunakan skill ini ketika user pilih "fix desain" atau "keduanya" di checkpoint setelah audit. Butuh plugin design-agent ter-install HANYA kalau ada temuan visual — cek dulu, kasih instruksi install kalau belum ada.
---

# /garnish:design-fix — Perbaikan Desain

Skill ini adalah orchestrator — tidak menulis ulang logic ekstraksi token,
pencarian referensi, atau build. Semua itu tetap dikerjakan skill
`design-agent` yang sudah ada.

## Langkah

### 1. Baca temuan dari registry
Ambil temuan (`findings`) dengan `status: "open"` dan `scope` dalam
`{"ui-ux", "komponen", "wcag"}` dari audit yang dimaksud di
`.garnish/registry/audits.json`.

### 1.5. Checkpoint — semua temuan atau prioritas tinggi aja (HARD STOP)

Hitung jumlah temuan (dari Langkah 1) yang `severity: "tinggi"` vs total
keseluruhan, lalu tanya:
> "Ada {total} temuan desain (scope UI/UX, komponen, WCAG) — {jumlah
> tinggi} di antaranya prioritas TINGGI. Mau saya benerin SEMUA temuan
> ({total}), atau yang prioritas TINGGI aja ({jumlah tinggi})?"

Tunggu jawaban eksplisit. Kalau user pilih "prioritas tinggi aja", filter
temuan yang diproses di langkah-langkah berikutnya jadi HANYA yang
`severity: "tinggi"` — temuan `severity: "sedang"` yang tidak dipilih
TETAP `status: "open"` di registry (bukan didiamkan tanpa jejak — bisa
di-fix nanti lewat run terpisah).

Pisahkan temuan yang akan diproses (baik "semua" atau "tinggi aja") jadi
dua kelompok, karena cara fix-nya beda:

- **Kelompok A — butuh referensi visual** (`type`: `contrast` [tombol],
  `consistency`, `cta-position`, `layout-rusak`, `ui-heuristic`,
  `missing-section`) → lanjut ke Langkah 2-11 seperti biasa (orchestrate
  ke `design-agent`).
- **Kelompok B — fix struktural langsung, TANPA cari referensi**
  (`type`: `alt-text`, `heading-hierarchy`, `target-size`, `icon-label`)
  → skip Langkah 4-7 (gak perlu referensi visual buat nambah atribut
  `alt`/`aria-label`, membetulkan urutan heading, atau memperbesar
  padding tombol), langsung edit kode yang relevan memakai `suggestion`
  dari finding sebagai arah, lalu lanjut ke Langkah 9 (QA) dan 10 (update
  registry) — `designRef` untuk finding kelompok B boleh `null` karena
  tidak ada referensi/spec yang dipakai.

### 2. Cek plugin `design-agent` ter-install
Kalau SEMUA finding yang mau di-fix di run ini adalah Kelompok B (tidak
ada Kelompok A sama sekali) → skip Langkah 2-3 ini, langsung proses
Kelompok B (Langkah 1.5) lalu Langkah 9 — `design-agent` tidak
dibutuhkan sama sekali untuk fix struktural murni.

Kalau ada Kelompok A, lanjut cek plugin `design-agent` seperti biasa.
Coba deteksi apakah skill `design-agent` tersedia (mis. cek lewat daftar
skill yang ke-load, atau fallback cek filesystem):
```bash
find ~/.claude/plugins/cache -name "inspo" -path "*design-agent*" -type d 2>/dev/null | head -1
```
Kalau tidak ketemu:
> "Perbaikan desain butuh plugin `design-agent` ter-install. Jalankan ini
> dulu, lalu bilang saya kalau sudah:
> ```
> /plugin marketplace add ajisss/design-agent-plugin
> /plugin install design-agent@design-agent-marketplace
> /reload-plugins
> ```"
Tunggu konfirmasi user sebelum lanjut — jangan asumsikan sudah ter-install.

### 3. Cek prasyarat `.design/registry/` di project ini
Kalau project yang diaudit belum punya `.design/registry/project.json`
(belum pernah `/design-agent:init`), beri tahu user secara transparan dan
jalankan `/design-agent:init` dulu — jangan diam-diam menganggap sudah ada.

### 4. Susun brief pencarian per temuan fatal
Kelompokkan temuan yang relevan (mis. semua yang soal tombol jadi satu
brief, yang soal posisi CTA jadi brief lain) — jangan gabung temuan yang
tidak nyambung jadi satu query membingungkan. **Brief WAJIB eksplisit minta
referensi yang scope-nya SEMPIT** — fokus ke komponen/pola yang fatal saja,
BUKAN landing page utuh dengan banyak section (component showcase, style
guide, atau potongan kecil halaman yang fokus ke satu komponen itu lebih
disukai). Ini penting supaya konfirmasi struktur section di Langkah 7 tetap
ringan (lihat catatan di Langkah 7). Contoh brief:
- Temuan `contrast`/`consistency` pada tombol → *"cari referensi POLA TOMBOL
  CTA saja — bisa berupa component showcase, style guide, atau bagian kecil
  dari halaman yang fokus ke tombol dengan kontras tinggi dan style
  konsisten [sebutkan konteks produk/industri dari halaman yang diaudit
  kalau relevan] — BUKAN landing page utuh dengan banyak section"*
- Temuan `cta-position` → *"cari referensi pola penempatan CTA yang selalu
  terlihat di atas fold — cukup bagian hero/CTA-nya saja sebagai contoh
  pola, BUKAN halaman lengkap dari atas sampai footer"*

Panggil `/design-agent:inspo` dengan brief itu untuk tiap kelompok temuan.

### 5. Checkpoint pemilihan referensi
`/design-agent:select` tetap berlaku seperti biasa (hard stop pemilihan
referensi) — jangan skip atau putuskan sendiri referensi mana yang dipakai.

### 6. Scaffold component library (kalau belum ada)
- Kalau `.design/registry/project.json` sudah punya `stack` terisi (dari
  sesi `design-agent` sebelumnya di project ini) → pakai itu, jangan tanya
  ulang.
- Kalau belum → tanya framework & styling (sama seperti Langkah 1
  `/design-agent:build`), simpan ke `project.json` lewat mekanisme yang
  sama seperti `design-agent:build` supaya konsisten dan tidak tanya ulang
  di sesi berikutnya.
- Cek apakah `Button`/`Card`/`Input` sudah ada di lokasi konvensi project
  (mis. `src/components/ui/` untuk React/Next.js, atau folder komponen
  setara sesuai stack yang terdeteksi). Kalau **sudah ada** → reuse,
  JANGAN timpa atau bikin ulang di lokasi lain.
- Kalau belum ada → bikin 3 komponen dasar (Button: variant + size prop
  minimal; Card: container dengan slot konten; Input: text input dasar
  dengan label) — style-nya BELUM diisi hardcode di sini, biarkan
  `/design-agent:build` di Langkah 8 yang mengisi dari token spec supaya
  nilainya bukan tebakan.

### 7. Ekstrak spec dari referensi terpilih (scope terbatas)
Panggil `/design-agent:spec` pada referensi yang dipilih di Langkah 5,
dengan instruksi eksplisit ke prosesnya: token yang diekstrak HANYA untuk
jenis komponen/section yang fatal (mis. cuma token tombol kalau
temuannya soal tombol) — bukan seluruh design system halaman referensi.

Catatan: `/design-agent:spec` punya hard-stop wajib (Langkah "3.5. Tentukan
& konfirmasi struktur halaman") yang meminta user konfirmasi struktur
section penuh (hero/fitur/footer/dll) sebelum spec bisa
`sectionsConfirmed: true` — ini bagian dari alur `design-agent` dan TIDAK
diubah di sini. Karena referensi yang dicari di Langkah 4 sudah sengaja
dibuat sempit (component showcase / potongan kecil halaman, bukan landing
page utuh), section yang perlu dikonfirmasi di sini otomatis cuma
1-2 section yang relevan — jadi checkpoint ini tetap cepat dan fokus,
bukan konfirmasi struktur halaman penuh yang tidak nyambung dengan
perbaikan satu komponen.

### 8. Build — HANYA komponen/section yang fatal
Panggil `/design-agent:build` pada spec dari Langkah 7, dengan instruksi
eksplisit: rebuild HANYA komponen/section yang berkaitan dengan temuan
fatal di Langkah 1, pakai component library dari Langkah 6. JANGAN sentuh
section atau konten lain yang tidak ditandai fatal — ini scope yang lebih
sempit dari perilaku default `/design-agent:build` (yang biasanya
membangun section sesuai `sections` penuh di spec), jadi sebutkan
pembatasan ini secara eksplisit saat memanggil `/design-agent:build`.

### 9. QA — re-audit hasil build sebelum ditandai selesai

Sebelum menandai finding manapun `design-fixed`, verifikasi hasilnya
beneran menyelesaikan temuan yang tadi fatal. Caranya beda untuk Kelompok
A vs Kelompok B (lihat Langkah 1.5):

**Untuk Kelompok A** (butuh computed CSS):
1. Pastikan dev server project berjalan (tanya user URL-nya kalau belum
   diketahui dari sesi ini, mis. `http://localhost:3000`).
2. Cari `extract-styles.py` (path sama seperti Langkah 2) dan jalankan ke
   dev server tersebut:
   ```bash
   python3 "$EXTRACT_STYLES" <url-dev-server> .garnish/registry/audit-<audit-id>-recheck.json .garnish/registry/screenshots/<audit-id>-recheck
   ```
3. Dari hasil JSON itu, ulangi HANYA deteksi yang relevan ke jenis temuan
   di Langkah 1 (mis. kalau tadi soal kontras tombol, hitung ulang
   kontras tombol di hasil build ini — logic deteksinya sama persis
   dengan `/garnish:check` Langkah 3, bagian yang relevan saja).

**Untuk Kelompok B** (`alt-text`, `heading-hierarchy` — butuh HTML, bukan
computed CSS):
1. Pastikan dev server project berjalan.
2. `WebFetch <url-dev-server>` buat HTML hasil build.
3. Ulangi deteksi yang sama persis dengan `/garnish:check` Langkah 3a
   (cek `<img>` tanpa `alt`, atau urutan heading yang masih loncat level)
   pada bagian kode yang barusan diedit.

**Kedua kelompok** — hasil pengecekan ulang:
4. **Kalau bersih** (temuan yang di-fix beneran tidak muncul lagi) →
   lanjut ke Langkah 10 (update registry).
5. **Kalau masih ada sisa**:
   - Kelompok A → kembali ke Langkah 7 (ekstrak spec dari referensi
     terpilih) atau Langkah 8 (build) — perbaiki spesifik yang masih
     meleset, JANGAN ulangi dari Langkah 4 (cari referensi baru lagi)
     kecuali referensi yang dipakai memang jadi penyebabnya.
   - Kelompok B → langsung edit lagi kode yang relevan (tidak ada
     referensi/spec yang perlu diulang).

   Ulangi Langkah 9 (QA) lagi setelah perbaikan.
6. Maksimal 3 putaran QA total. Kalau di putaran ke-3 masih ada sisa,
   HENTIKAN loop — JANGAN tandai finding itu `design-fixed`. Laporkan ke
   user secara eksplisit di Langkah 11: finding mana yang belum berhasil
   diperbaiki otomatis setelah 3 percobaan, dan kenapa (sebutkan hasil
   pengukuran terakhirnya).

### 10. Update registry
Untuk tiap finding yang berhasil di-fix (lolos QA Langkah 9):

Kelompok A:
```json
{
  "status": "design-fixed",
  "fixedAt": "<ISO 8601>",
  "designRef": {
    "refIds": ["R-00X dari .design/registry/references.json project ini"],
    "specId": "S-00X dari .design/registry/specs.json project ini"
  }
}
```

Kelompok B (tidak ada referensi/spec yang dipakai):
```json
{ "status": "design-fixed", "fixedAt": "<ISO 8601>", "designRef": null }
```
Append journal (di awal Langkah 4, sebelum mulai cari referensi):
```json
{"ts":"<ISO 8601>","event":"design_fix_started","auditId":"A-00X","findingIds":["F-00X"]}
```
Dan (di akhir, per finding yang selesai):
```json
{"ts":"<ISO 8601>","event":"design_fixed","auditId":"A-00X","findingId":"F-00X","specId":"S-00X"}
```
Kalau semua finding desain di audit ini sudah `design-fixed` (dan tidak
ada temuan `open` lain yang tersisa dari kategori konten), update
`audits.json` → `status: "resolved"`. Kalau user tadi pilih "prioritas
tinggi aja" di Langkah 1.5, finding `severity: "sedang"` yang sengaja
tidak diproses akan tetap `status: "open"` — JANGAN update audit jadi
`resolved` selama itu masih ada, walau semua finding `tinggi` sudah
`design-fixed`.

### 11. Laporkan before/after ke user
Tampilkan ringkasan: komponen/section mana yang di-fix, referensi apa yang
dipakai (link/ID), hasil QA Langkah 9 (lolos di putaran ke berapa, atau
finding mana yang masih belum lolos setelah 3 putaran), dan screenshot
atau deskripsi hasil akhir — jangan cuma bilang "sudah selesai" tanpa
bukti konkret.

## Yang TIDAK boleh dilakukan skill ini
- Menulis ulang logic `inspo`/`select`/`spec`/`build` sendiri — selalu
  panggil skill `design-agent` yang sudah ada
- Rebuild seluruh halaman kalau cuma sebagian yang ditandai fatal
- Skip checkpoint Langkah 1.5 (semua vs prioritas tinggi) atau
  memproses finding `severity: "sedang"` padahal user sudah pilih
  "prioritas tinggi aja"
- Menandai audit `resolved` kalau ada finding `severity: "sedang"` yang
  sengaja tidak diproses (masih `status: "open"`) karena user pilih
  "prioritas tinggi aja"
- Menimpa component library yang sudah ada di lokasi konvensi project
- Melanjutkan tanpa plugin `design-agent` ter-install atau tanpa
  `.design/registry/` ter-init di project yang diaudit
- Skip checkpoint pemilihan referensi (`/design-agent:select`)
- Menandai finding `"design-fixed"` sebelum lolos QA Langkah 9, atau
  melanjutkan retry lebih dari 3 putaran tanpa melapor apa adanya ke user

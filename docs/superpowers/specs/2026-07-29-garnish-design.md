# GARNISH — Design Spec

*"The final touch before you serve it to the client."*

## Masalah

Audit desain & konten landing page/screen sebelum kasih rekomendasi
redesign ke klien selama ini dikerjakan manual — buang waktu berjam-jam
buat identifikasi masalah satu-satu, dan seringnya jadi checklist umum yang
gak fokus ke yang paling urgent. Dibutuhkan agent yang bisa audit otomatis
(fatal-only, bukan audit lengkap), kasih rekomendasi konkret, dan langsung
generate revisi (konten dan/atau desain).

Lihat `garnish-planning.md` (bahan hackathon internal Etalas) untuk latar
belakang bisnis lengkap — dokumen ini fokus ke desain teknis.

## Solusi & Sumber

Sebagian besar desain ini sudah dituliskan konkret lewat dua iterasi
scaffold terpisah (`garnish-repo.zip` dan `garnish-repo (1).zip`, dibuat di
sesi Claude lain) dan direkonsiliasi di sini. Yang diadopsi APA ADANYA dari
iterasi terbaru (`garnish-repo (1).zip`) karena sudah lebih matang dari
rancangan awal:

- `garnish/skills/init/SKILL.md` — scaffold `.garnish/registry/` +
  `SCHEMA.md`, mengikuti pola persis `design-agent:init`.
- `garnish/skills/check/SKILL.md` — audit gejala fatal, tulis ke registry
  dengan ID stabil (`A-00X`/`F-00X`), **reconciliation-aware** (audit ulang
  URL yang sama → tampilkan riwayat sebelumnya dulu, tanya mau ulang dari
  nol atau lihat status lama).
- `garnish/skills/content-fix/SKILL.md` — rewrite copy berdasarkan temuan
  di registry, update `status` temuan jadi `content-fixed` setelah user
  pilih opsi.
- `garnish/CLAUDE.md` / `CONTEXT.md` — aturan wajib + latar belakang,
  termasuk aturan baru "registry adalah satu-satunya sumber kebenaran".
- `garnish/.claude-plugin/plugin.json` — sudah punya
  `"dependencies": ["design-agent"]`, versi `0.2.0`.

**Yang TIDAK diadopsi**: copy `design-agent/` yang ikut dibundel di kedua
zip itu — itu versi lama/sederhana (109 baris `extract-styles.py`, tanpa
`compare-tokens.py`/loop QA token/`SCHEMA.md`/tests). GARNISH depend ke
`design-agent` yang di `/Users/ihsanaziz/design-agent-plugin` (v1.2.0,
sudah dipolish lewat `2026-07-27-precise-replication-design.md`).

**Yang masih perlu ditulis baru** (belum ada di kedua zip): skill
`garnish:design-fix` — orchestrator yang menghubungkan temuan fatal ke
pipeline `design-agent`. Ini fokus utama dokumen ini.

## Komponen

### 1. Repo & dependency

```
garnish-plugin/                    (repo baru, terpisah dari design-agent-plugin)
  .claude-plugin/
    plugin.json        ← dependencies: ["design-agent"], version 0.2.0
    marketplace.json
  CLAUDE.md / CONTEXT.md
  skills/
    init/SKILL.md          ← dari garnish-repo (1).zip, dipakai apa adanya
    check/SKILL.md         ← dari garnish-repo (1).zip, dipakai apa adanya
    content-fix/SKILL.md   ← dari garnish-repo (1).zip, dipakai apa adanya
    design-fix/SKILL.md    ← BARU, ditulis di sini
```

`design-agent` TIDAK di-bundle di repo ini — user install terpisah (atau
otomatis lewat dependency resolution plugin marketplace), mengarah ke
`ajisss/design-agent-plugin` yang sudah ada di GitHub.

### 2. Schema `audits.json` — extend, bukan redesign

Field per-finding tambahan (opsional, cuma diisi kalau finding itu di-fix
lewat jalur desain):
```json
{
  "id": "F-001",
  "category": "measured | judgment",
  "type": "contrast | consistency | cta-position | placeholder | value-prop | trust-signal | lainnya",
  "title": "string",
  "description": "string",
  "status": "open | content-fixed | design-fixed | dismissed",
  "fixedAt": "ISO 8601 | null",
  "designRef": {
    "refIds": ["string — ID dari .design/registry/references.json project yang sama"],
    "specId": "string — ID dari .design/registry/specs.json project yang sama"
  }
}
```

`designRef` adalah cross-reference murni (cuma nyimpen ID) ke registry
`design-agent` di project yang sama — GARNISH tidak menduplikasi data
token/spec, cukup nunjuk ke sumbernya biar bisa ditelusuri balik.

Journal event baru (tambahan dari yang sudah ada di `garnish-repo (1).zip`:
`audit_created`, `finding_detected`, `fix_selected`, `content_fixed`,
`finding_dismissed`):
```json
{"ts":"...","event":"design_fix_started","auditId":"A-00X","findingIds":["F-00X"]}
{"ts":"...","event":"design_fixed","auditId":"A-00X","findingId":"F-00X","specId":"S-00X"}
```
(`design_fixed` sudah disebut di daftar event `garnish-repo (1).zip`, cuma
belum ada skill yang benar-benar menulisnya — `design-fix` yang mengisi
gap ini.)

### 3. `/garnish:design-fix` (skill baru)

Dipanggil setelah user pilih "desain" atau "keduanya" di checkpoint
`/garnish:check`. Scope: **hanya rebuild komponen/section yang ditandai
fatal** — section/konten lain yang tidak fatal tidak disentuh (konsisten
dengan aturan CLAUDE.md GARNISH #7 soal konten tidak boleh ditimpa
asal-asalan, diperluas ke bagian desain juga).

Langkah:

1. **Baca temuan dari registry** — ambil finding dengan `status: "open"`
   dan kategori/`type` yang relevan ke desain (kontras, konsistensi
   komponen, posisi CTA, layout) dari audit yang dimaksud.
2. **Cek prasyarat `design-agent`** — kalau project yang diaudit belum
   punya `.design/registry/` (belum pernah `/design-agent:init`), jalankan
   itu dulu (beri tahu user secara transparan, bukan diam-diam).
3. **Susun brief pencarian per temuan fatal** — bukan satu query besar
   untuk seluruh halaman, tapi brief spesifik per jenis temuan. Contoh:
   - Temuan `contrast`/`consistency` pada tombol → brief ke
     `/design-agent:inspo`: *"cari referensi tombol CTA dengan kontras
     tinggi dan style konsisten"*.
   - Temuan `cta-position` → brief soal pola penempatan CTA yang selalu
     terlihat di atas fold.
   Kalau ada beberapa temuan desain sekaligus, boleh digabung jadi satu
   brief `/design-agent:inspo` kalau memang berhubungan (mis. sama-sama
   soal tombol), tapi jangan gabung temuan yang gak nyambung jadi satu
   query yang membingungkan.
4. Panggil `/design-agent:inspo` dengan brief itu → `/design-agent:select`
   seperti biasa (checkpoint pemilihan referensi tetap berlaku, bukan
   diputuskan otomatis oleh `design-fix`).
5. **Scaffold component library** kalau project belum punya
   `Button`/`Card`/`Input` di lokasi konvensi (deteksi dulu, jangan
   asumsikan lokasi):
   - Kalau `.design/registry/project.json` sudah punya `stack` terisi
     (dari sesi `design-agent` sebelumnya) → pakai itu, jangan tanya ulang.
   - Kalau belum → tanya framework/styling (sama seperti langkah 1
     `/design-agent:build`).
   - Cek apakah `Button`/`Card`/`Input` sudah ada di lokasi konvensi
     project (mis. `src/components/ui/` untuk React, atau setara sesuai
     stack) — kalau **sudah ada**, reuse, JANGAN timpa/bikin ulang.
   - Kalau belum ada → bikin 3 komponen dasar (props minimal: variant/size
     untuk Button, dst) dengan style dari token yang akan divalidasi di
     Langkah 6 — bukan hardcode nilai baru sebelum spec ada.
6. Panggil `/design-agent:spec` pada referensi terpilih, dengan scope
   token **hanya untuk jenis komponen/section yang fatal** (mis. cuma
   token tombol kalau temuannya soal tombol) — bukan seluruh design
   system halaman.
7. Panggil `/design-agent:build`, dengan instruksi eksplisit ke proses
   build: **HANYA rebuild komponen/section yang ditandai fatal**, pakai
   component library dari Langkah 5, dan **jangan sentuh** section/konten
   lain yang tidak fatal.
8. **Update registry**: untuk tiap finding yang berhasil di-fix, set
   `status: "design-fixed"`, `fixedAt`, dan isi `designRef.refIds`/`specId`
   dari ID yang dihasilkan `/design-agent:inspo`/`select`/`spec` di Langkah
   4 & 6. Append journal `design_fix_started` (di awal) dan `design_fixed`
   per finding (di akhir).
9. Tampilkan laporan before/after ke user (screenshot atau deskripsi
   perubahan per komponen yang di-fix).

## Fallback & error handling

- Project belum punya `design-agent` ter-install sama sekali (bukan cuma
  belum di-init) → beri tahu user terus terang, kasih instruksi install
  (`/plugin install design-agent@...`), tanya lanjut atau batal — sama
  seperti pola existing di `design-agent:build` Jalur Superpowers.
- `/design-agent:inspo` tidak menghasilkan referensi yang relevan (semua
  ditolak user di `/design-agent:select`) → laporkan ke user, tawarkan
  brief ulang dengan arahan berbeda, jangan lanjut ke build dengan
  referensi seadanya.
- `/design-agent:spec` blocked (`confidence` rendah/`sectionsConfirmed:
  false`) → ikuti aturan blocking yang sudah ada di `design-agent:spec`,
  jangan paksa lanjut ke build.
- Component library sudah ada tapi strukturnya beda dari yang diharapkan
  (mis. `Button` ada tapi bukan di lokasi konvensi) → tanya user dulu
  lokasi yang benar, jangan bikin duplikat di tempat lain.

## Testing

- Tidak ada script baru untuk skill ini (murni instruksi prompt,
  orchestrating skill lain) — verifikasi lewat dry-run manual: audit satu
  landing page nyata yang punya masalah kontras/konsistensi tombol jelas,
  pilih "desain" di checkpoint, ikuti sampai `/design-agent:build` selesai,
  cek hasilnya di kode (cuma komponen yang fatal yang berubah) dan di
  registry (`status: "design-fixed"`, `designRef` terisi).
- Verifikasi reuse component library: jalankan `design-fix` dua kali di
  project yang sama — pastikan run kedua tidak bikin ulang
  `Button`/`Card`/`Input` yang sudah ada dari run pertama.

## Batasan yang disadari

- Sama seperti `design-agent`, kualitas replikasi desain tetap bergantung
  pada ketersediaan referensi yang relevan & aset visual (lihat
  `2026-07-28-visual-asset-sourcing-design.md` di `design-agent-plugin`
  untuk keterbatasan soal ini).
- `design-fix` menambah cukup banyak checkpoint berantai (`inspo` →
  `select` → `spec` → `build`, masing-masing punya hard stop sendiri) —
  ini pilihan sadar (lebih thorough, sesuai keputusan user) tapi berarti
  alur fix desain lebih lambat dibanding kalau langsung pakai halaman yang
  diaudit sebagai referensi tunggal.
- Scope `design-fix` sengaja dibatasi ke komponen/section yang fatal —
  tidak melakukan redesign holistik satu halaman penuh dalam satu run.

# GARNISH

*"The final touch before you serve it to the client."*

Audit gejala fatal desain & konten (bukan checklist lengkap), lalu
benerin kontennya dan/atau desainnya (reuse plugin `design-agent`) — atau
bikin landing page BARU dari nol berdasarkan hasil audit.

## Install

```
/plugin marketplace add ajisss/garnish-plugin
/plugin install garnish@garnish-marketplace
/reload-plugins
```

## Dependency

Butuh plugin `design-agent` sudah ter-install (dipakai `/garnish:design-fix`
buat cari referensi, ekstrak token, dan rebuild komponen). Kalau belum ada:
```
/plugin marketplace add ajisss/design-agent-plugin
/plugin install design-agent@design-agent-marketplace
/reload-plugins
```
Lihat `CONTEXT.md` untuk detail relasinya. `design-fix` mengecek sendiri
apakah dependency ini tersedia sebelum jalan, dan kasih instruksi di atas
kalau belum ada.

## Cara pakai

Skill dipanggil otomatis lewat **natural language** (kalimat biasa),
BUKAN diketik sebagai slash command manual. Ketik `/garnish:check` di
prompt akan muncul "No commands match" — itu **normal**, bukan tanda
gagal install (sama seperti plugin `design-agent`).

Test yang benar: ketik kalimat biasa, mis.:
> "audit landing page ini: https://contoh.com"

**Skill(check)** yang otomatis ke-trigger dari kalimat itu. Tidak perlu
jalankan `/garnish:init` dulu — `check` auto-setup registry sendiri kalau
belum ada.

Sebelum audit mulai, `check` nanya scope dulu (bisa pilih beberapa):
**Konten, UI/UX, Komponen, WCAG**. Tiap temuan digroundkan ke framework
UI/UX yang mapan (Nielsen Usability Heuristics, Gestalt, Atomic Design,
subset WCAG AA, prinsip CRO — lihat `CONTEXT.md`) dan disertai saran
perbaikan konkret, bukan cuma "ini salah".

## Struktur
```
CONTEXT.md              ← WAJIB dibaca duluan, semua latar belakang & keputusan
CLAUDE.md                ← aturan permanen
skills/
  ├── init/SKILL.md           ← reset registry manual (opsional, bukan prasyarat)
  ├── check/SKILL.md          ← checkpoint scope + audit gejala fatal per scope, auto-setup registry
  ├── content-fix/SKILL.md    ← rewrite copy + QA, update status temuan di registry
  ├── design-fix/SKILL.md     ← orchestrator ke design-agent (fix bertarget: komponen/section fatal) + QA
  ├── rebuild/SKILL.md        ← orchestrator ke design-agent (full rebuild ke project baru) + QA
  └── monitor/SKILL.md        ← re-audit URL yang sama, delta report (regresi/baru/open/bersih) + artifact HTML
```

## Status
Lengkap untuk skenario "wajib solid" + stretch goal dari planning awal,
plus refinement audit framework & opsi rebuild:
- `check` — checkpoint scope (Konten/UI-UX/Komponen/WCAG), deteksi per
  scope digroundkan ke rubric UI/UX yang mapan (Nielsen, Gestalt, Laws of
  UX, Progressive Disclosure, Cognitive Load, AIDA/PAS), tiap temuan ada
  saran perbaikan (kadang diperkuat rujukan kredibel dari web search
  terbatas). Fitur tambahan:
  - **Severity P0–P3** per temuan, Health Score 0–100 per scope + overall
  - **`wcagLevel` (A / AA / AAA)** untuk tiap temuan scope WCAG
  - **Metrik terukur** per temuan: kontras, target-size, alt-text,
    heading-hierarchy, `line-length` (threshold 75ch), `cramped-padding`
    (threshold 8px vertikal / 12px horizontal)
  - **Brand context** — isi `.garnish/brand.md` (nama produk, audience,
    tone, warna utama) agar audit konten & UI lebih presisi; diisi saat
    `/garnish:init` atau manual kapan saja
  - **Laporan HTML** background putih + screenshot halaman yang diaudit
    ditampilkan di bawah Health Score
- `content-fix` — rewrite copy + QA sebelum ditandai selesai
- `design-fix` — fix BERTARGET (cuma komponen/section fatal): integrasi
  `design-agent` + scaffold component library buat temuan visual, fix
  struktural langsung (tanpa referensi) buat alt-text/heading-hierarchy,
  dengan QA loop (maks 3 putaran). Checkpoint dimension selection:
  pilih dimensi spesifik yang mau di-fix (Kontras / Tipografi / Spacing /
  Struktur / Komponen / WCAG struktural)
- `rebuild` — full rebuild SELURUH landing page ke project baru terpisah,
  konten asli dipertahankan kecuali bagian fatal, dengan QA loop juga

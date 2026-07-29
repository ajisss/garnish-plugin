# GARNISH

*"The final touch before you serve it to the client."*

Audit gejala fatal desain & konten (bukan checklist lengkap), lalu
benerin kontennya dan/atau desainnya (reuse plugin `design-agent`).

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
  └── design-fix/SKILL.md     ← orchestrator ke design-agent (inspo/select/spec/build) + fix struktural WCAG + QA
```

## Status
Lengkap untuk skenario "wajib solid" + stretch goal dari planning awal,
plus refinement audit framework:
- `check` — checkpoint scope (Konten/UI-UX/Komponen/WCAG), deteksi per
  scope digroundkan ke rubric UI/UX yang mapan, tiap temuan ada saran
  perbaikan
- `content-fix` — rewrite copy + QA sebelum ditandai selesai
- `design-fix` — integrasi `design-agent` + scaffold component library
  buat temuan visual, fix struktural langsung (tanpa referensi) buat
  alt-text/heading-hierarchy, keduanya dengan QA loop (maks 3 putaran)
  sebelum hasil ditampilkan ke user

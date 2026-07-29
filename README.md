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

## Struktur
```
CONTEXT.md              ← WAJIB dibaca duluan, semua latar belakang & keputusan
CLAUDE.md                ← aturan permanen
skills/
  ├── init/SKILL.md           ← scaffold registry (.garnish/registry/), sekali per project
  ├── check/SKILL.md          ← audit gejala fatal, simpan ke registry
  ├── content-fix/SKILL.md    ← rewrite copy, update status temuan di registry
  └── design-fix/SKILL.md     ← orchestrator ke design-agent (inspo/select/spec/build)
```

## Status
Lengkap untuk skenario "wajib solid" + stretch goal dari planning awal:
`check` (deteksi terukur + judgment), checkpoint, `content-fix`, dan
`design-fix` (integrasi `design-agent` + scaffold component library).

---
name: init
description: Scaffold registry (.garnish/registry/) yang dibutuhkan skill check dan content-fix ke project/folder saat ini. Gunakan skill ini ketika user pertama kali pakai garnish di sebuah folder dan belum ada .garnish/registry/, atau user eksplisit minta "setup garnish". Jalankan SEBELUM /garnish:check kalau registry belum ada.
---

# /garnish:init — Setup Registry

## Langkah

### 1. Cek apakah sudah pernah di-init
Cek apakah `.garnish/registry/audits.json` sudah ada.
- Kalau **sudah ada** → tanya user apakah mau reset, atau batalkan.
- Kalau **belum ada** → lanjut.

### 2. Buat struktur folder & file
```bash
mkdir -p .garnish/registry/screenshots
```

```bash
cat > .garnish/registry/audits.json << 'EOF'
{ "audits": [] }
EOF
touch .garnish/registry/journal.jsonl
```

### 3. Tulis `SCHEMA.md`
```markdown
# Garnish Registry Schema

## audits.json
\`\`\`json
{
  "audits": [
    {
      "id": "A-001",
      "url": "string",
      "screenshotPath": "string | null",
      "capturedAt": "ISO 8601",
      "findings": [
        {
          "id": "F-001",
          "category": "measured | judgment",
          "type": "contrast | consistency | cta-position | placeholder | value-prop | trust-signal | lainnya",
          "title": "string singkat",
          "description": "string",
          "status": "open | content-fixed | design-fixed | dismissed",
          "fixedAt": "ISO 8601 | null",
          "designRef": {
            "refIds": ["string — ID dari .design/registry/references.json project ini"],
            "specId": "string — ID dari .design/registry/specs.json project ini"
          }
        }
      ],
      "status": "audited | in-progress | resolved"
    }
  ]
}
\`\`\`

`category: "measured"` = terukur objektif (kontras, konsistensi, posisi CTA,
placeholder) — WAJIB dari perhitungan nyata, bukan tebakan.
`category: "judgment"` = penilaian AI (value prop, trust signal) — WAJIB
dilabel eksplisit ke user sebagai penilaian, bukan fakta pasti.

`designRef` cuma diisi untuk finding yang di-fix lewat `/garnish:design-fix`
— cross-reference murni ke registry `design-agent` di project yang sama
(TIDAK menduplikasi data token/spec di sini).

ID (`A-00X`, `F-00X`) lanjut dari ID terakhir yang ada — jangan mulai dari
1 lagi kalau sudah ada audit sebelumnya.

## journal.jsonl
Append-only. Event yang dipakai: `audit_created`, `finding_detected`,
`fix_selected`, `content_fixed`, `design_fix_started`, `design_fixed`,
`finding_dismissed`.

\`\`\`json
{"ts":"ISO 8601","event":"audit_created","auditId":"A-001","url":"..."}
{"ts":"ISO 8601","event":"finding_detected","auditId":"A-001","findingId":"F-001","category":"measured","type":"contrast"}
{"ts":"ISO 8601","event":"fix_selected","auditId":"A-001","findingId":"F-001","fixType":"content|design"}
{"ts":"ISO 8601","event":"content_fixed","auditId":"A-001","findingId":"F-001"}
{"ts":"ISO 8601","event":"design_fix_started","auditId":"A-001","findingIds":["F-001"]}
{"ts":"ISO 8601","event":"design_fixed","auditId":"A-001","findingId":"F-001","specId":"S-001"}
{"ts":"ISO 8601","event":"finding_dismissed","auditId":"A-001","findingId":"F-001","reason":"..."}
\`\`\`
```
(Tulis konten di atas persis ke file `.garnish/registry/SCHEMA.md`.)

### 4. Konfirmasi ke user
> "Registry Garnish sudah siap di `.garnish/registry/`. Coba audit halaman
> pertama dengan minta 'audit [url]'."

## Yang TIDAK boleh dilakukan skill ini
- Menimpa `.garnish/registry/` yang sudah ada tanpa konfirmasi eksplisit
- Menjalankan skill `check`/`content-fix`/`design-fix` sebagai bagian dari
  init — ini cuma setup

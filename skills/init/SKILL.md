---
name: init
description: Reset registry (.garnish/registry/) di project/folder saat ini. TIDAK WAJIB dijalankan manual — /garnish:check sudah auto-setup registry sendiri kalau belum ada. Gunakan skill ini HANYA kalau user eksplisit minta "reset garnish" atau "setup ulang garnish" (mis. mau hapus semua riwayat audit dan mulai dari nol).
---

# /garnish:init — Reset Registry (Manual, Opsional)

`/garnish:check` sudah auto-setup registry sendiri di background kalau
belum ada — skill ini HANYA untuk kasus user eksplisit mau reset/hapus
riwayat audit yang sudah ada.

## Langkah

### 1. Konfirmasi reset
Cek apakah `.garnish/registry/audits.json` sudah ada.
- Kalau **sudah ada** → user memanggil skill ini artinya mau reset,
  konfirmasi dulu: "Ini bakal hapus semua riwayat audit yang ada di
  `.garnish/registry/`. Yakin mau reset dari nol?" Tunggu jawaban eksplisit
  sebelum lanjut.
- Kalau **belum ada** → tidak ada yang perlu direset, tapi tetap lanjut
  bikin strukturnya (setara pertama kali setup).

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
      "pageType": "landing-page | non-landing-page-forced",
      "scopeAudited": ["konten | ui-ux | komponen | wcag, ..."],
      "screenshotPath": "string | null",
      "capturedAt": "ISO 8601",
      "findings": [
        {
          "id": "F-001",
          "scope": "konten | ui-ux | komponen | wcag",
          "category": "measured | judgment",
          "type": "contrast | consistency | cta-position | placeholder | layout-rusak | alt-text | heading-hierarchy | target-size | icon-label | value-prop | trust-signal | ui-heuristic | missing-section | lainnya",
          "title": "string singkat",
          "description": "string",
          "suggestion": "string — kenapa masalah (rujuk prinsip UI/UX/CRO/WCAG) + cara umum memperbaiki, TANPA angka dampak dikarang",
          "sourceRef": "string — URL sumber kredibel pendukung suggestion, dari pengayaan opsional (Langkah 3.5 check) | null",
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

`category: "measured"` = terukur objektif (kontras tombol, konsistensi,
posisi CTA, placeholder, layout rusak, alt text, heading hierarchy,
target size) — WAJIB dari perhitungan nyata, bukan tebakan.
`category: "judgment"` = penilaian AI (kontras teks dari screenshot,
value prop, trust signal, evaluasi heuristik & Laws of UX) — WAJIB
dilabel eksplisit ke user sebagai penilaian, bukan fakta pasti.

`scope` mengelompokkan tiap temuan ke salah satu dari 4 kategori audit
yang bisa dipilih user di checkpoint `/garnish:check` (`konten`, `ui-ux`,
`komponen`, `wcag`) — lihat rubric lengkap (Nielsen heuristics, Gestalt,
atomic design, WCAG subset, prinsip CRO, struktur landing page standar)
di `skills/check/SKILL.md`. `scopeAudited` di level audit mencatat scope
mana saja yang DIPILIH user saat audit itu dijalankan (bukan semua scope
yang ada).

`pageType: "landing-page"` = default, halaman kelihatan seperti landing
page marketing. `"non-landing-page-forced"` = user tetap lanjut audit
walau `/garnish:check` mendeteksi & memperingatkan halaman ini kelihatan
bukan landing page (dashboard/aplikasi) — hasil audit di kasus ini
tercatat kurang presisi karena rubric-nya dioptimasi buat landing page.

`suggestion` WAJIB ada di tiap finding — saran perbaikan konkret yang
merujuk balik ke rubric, TANPA estimasi angka/persentase dampak yang
tidak bisa diverifikasi. `sourceRef` OPSIONAL — cuma diisi kalau
`/garnish:check` berhasil menemukan sumber kredibel pendukung lewat
pengayaan best-effort (dibatasi ke domain UX/design tepercaya, bukan
sumber baru buat kriteria — rubric tetap satu-satunya penentu kriteria).

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
{"ts":"ISO 8601","event":"audit_created","auditId":"A-001","url":"...","scopeAudited":["ui-ux","wcag"]}
{"ts":"ISO 8601","event":"finding_detected","auditId":"A-001","findingId":"F-001","scope":"wcag","category":"measured","type":"contrast"}
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

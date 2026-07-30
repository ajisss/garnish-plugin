---
name: monitor
description: Re-audit URL yang sebelumnya sudah pernah di-audit Garnish, bandingkan temuan baru vs audit terakhir, tampilkan delta (regresi temuan yang sudah di-fix, masalah baru, temuan lama yang masih open, temuan yang bersih sendiri). Gunakan skill ini ketika user bilang "re-audit [url]", "cek ulang [url]", atau "monitor [url]".
---

# /garnish:monitor — Re-Audit & Delta Comparison

Skill ini menjalankan ulang audit untuk URL yang sudah pernah diaudit,
lalu membandingkan temuan baru dengan audit terakhir untuk mendeteksi
regresi dan perubahan.

## Langkah

### 1. Cari audit terakhir di registry

Baca `.garnish/registry/audits.json`. Cari entry dengan `url` yang cocok
(normalisasi trailing slash & case), ambil yang paling baru
(`capturedAt` terbesar).

- Kalau **tidak ada** audit sebelumnya untuk URL ini → hentikan dan
  arahkan user:
  > "URL ini belum pernah diaudit sebelumnya. Mulai dengan 'audit [url]'
  > dulu biar ada baseline-nya."
- Kalau **ada** → catat sebagai `baselineAudit` (simpan ID, findings,
  scopeAudited-nya).

### 2. Checkpoint scope

Tanya scope yang mau di-re-audit:
> "Mau re-audit scope apa? (sama seperti audit sebelumnya: {scopeAudited
> dari baseline}, atau pilih scope lain)"

Tunggu jawaban. Default: pakai scope yang sama dengan `baselineAudit`.

### 3. Ekstrak data halaman saat ini

Jalankan ekstraksi yang sama persis seperti `/garnish:check` Langkah 2
(screenshot + extract-styles.py kalau tersedia, WebFetch HTML sebagai
fallback). Gunakan scope yang dipilih di Langkah 2.

### 4. Deteksi temuan baru

Jalankan deteksi yang sama persis seperti `/garnish:check` Langkah 3
(sesuai scope), menghasilkan daftar `currentFindings` (raw — belum ada
ID registry).

### 5. Bandingkan dengan baseline

Petakan `currentFindings` vs `baselineAudit.findings` berdasarkan
kesamaan `type` + `scope` (bukan ID — ID itu unik per finding, bukan
per jenis masalah):

**Regresi** — finding di baseline yang `status: "design-fixed"` atau
`"content-fixed"`, tapi terdeteksi lagi di `currentFindings` dengan
`type` + `scope` yang sama. Ini berarti fix sebelumnya tidak bertahan
(kemungkinan code overwrite, merge conflict, dll).

**Masalah baru** — finding di `currentFindings` yang tidak ada padanannya
di `baselineAudit.findings` (tidak ada entry dengan `type` + `scope`
yang sama). Ini regresi dari perspektif berbeda — ada masalah baru yang
sebelumnya tidak ada.

**Masih open** — finding di baseline yang `status: "open"` dan masih
terdeteksi di `currentFindings` (type + scope sama). Tidak ada perubahan
— belum disentuh dan masih bermasalah.

**Bersih sendiri** — finding di baseline yang `status: "open"` tapi TIDAK
terdeteksi lagi di `currentFindings`. Kemungkinan ada code change lain
yang tidak lewat Garnish yang menyelesaikan masalah ini, atau false
positive di audit sebelumnya.

### 6. Tulis audit baru ke registry

Buat entry audit baru (ID lanjut dari yang terakhir, mis. `A-004`) dengan
struktur yang sama seperti audit biasa, PLUS field tambahan:

```json
{
  "id": "A-004",
  "url": "...",
  "pageType": "landing-page",
  "scopeAudited": ["..."],
  "screenshotPath": "...",
  "capturedAt": "<ISO 8601>",
  "compareWith": "A-001",
  "findings": [
    {
      "id": "F-020",
      "scope": "...",
      "category": "measured | judgment",
      "severity": "tinggi | sedang",
      "type": "...",
      "title": "...",
      "description": "...",
      "suggestion": "...",
      "sourceRef": null,
      "metric": null,
      "deltaStatus": "regresi | baru | open | bersih",
      "baselineFindingId": "F-00X | null",
      "status": "open",
      "fixedAt": null,
      "designRef": null
    }
  ],
  "healthScore": {
    "byScope": { "wcag": 63 },
    "overall": 63
  },
  "status": "audited",
  "rebuiltTo": null
}
```

`compareWith` = ID dari `baselineAudit`.
`deltaStatus` per finding:
- `"regresi"` → finding terdeteksi lagi padahal sudah di-fix
- `"baru"` → tidak ada di baseline
- `"open"` → lama, masih open, masih bermasalah
- `"bersih"` → lama, open, tapi sudah tidak terdeteksi

`baselineFindingId` = ID finding di baseline yang jadi padanan (null
untuk `deltaStatus: "baru"`).

Append ke journal:
```json
{"ts":"<ISO 8601>","event":"monitor_run","auditId":"A-004","compareWith":"A-001","regresiCount":0,"baruCount":0,"openCount":0,"bersihCount":0}
```

### 7. Tampilkan laporan delta

Tampilkan ringkasan berbasis delta, bukan daftar flat:

```
Re-Audit: [URL]
Dibandingkan dengan audit [tanggal baseline] (A-00X)
Scope: [scope yang diaudit]

━━ HEALTH SCORE ━━
Overall: {baseline.overall} → {current.overall} ({+/- diff})
[per scope kalau ada perubahan, mis: WCAG: 48 → 78 (+30)]

━━ REGRESI ({N} temuan) ━━
Fix yang tidak bertahan — perlu perhatian segera.
[list temuan dengan severity badge]

━━ MASALAH BARU ({N} temuan) ━━
Tidak ada di audit sebelumnya.
[list temuan]

━━ MASIH OPEN ({N} temuan) ━━
Belum disentuh sejak audit [tanggal baseline].
[list temuan]

━━ BERSIH SENDIRI ({N} temuan) ━━
Tidak terdeteksi lagi — kemungkinan sudah tertangani.
[list temuan]
```

Kalau salah satu kategori kosong, skip bagian itu.

Format tiap temuan: `[TINGGI/SEDANG] F-00X — title (type)`

### 8. Generate artifact HTML delta report

Buat HTML artifact (sama seperti `/garnish:check` Langkah 7) dengan
layout delta:

- Header: URL + tanggal re-audit + "dibandingkan dengan audit [tanggal
  baseline]"
- Health Score delta card: "{baseline.overall} → {current.overall}"
  dengan panah naik (hijau) atau turun (merah) + diff absolut, background
  warna sesuai skor terbaru (merah 0–40, amber 41–70, hijau 71–100)
- Stat cards: REGRESI (merah), BARU (oranye), MASIH OPEN (kuning),
  BERSIH (hijau)
- 4 section terpisah (satu per delta status), urutan: Regresi → Baru →
  Masih Open → Bersih Sendiri
- Card tiap temuan sama seperti artifact audit biasa (badge severity +
  kategori + ID + deskripsi + saran + sourceRef link kalau ada) +
  badge delta status dengan warna sesuai
- **Print/PDF CSS — WAJIB disertakan di `<style>`:**
  ```css
  @media print {
    body { margin: 0; }
    .header { -webkit-print-color-adjust: exact; print-color-adjust: exact; }
    .stat-cards { display: flex !important; }
    .finding-card { break-inside: avoid; page-break-inside: avoid; box-shadow: none; border: 1px solid #e2e8f0; }
    .badge { -webkit-print-color-adjust: exact; print-color-adjust: exact; }
    @page { margin: 1.5cm 2cm; }
  }
  ```

### 9. Tawarin next action

> "Re-audit selesai. Ada {N regresi} regresi dan {N baru} masalah baru.
> Mau saya:
> 1. Fix regresi dulu (design-fix / content-fix)
> 2. Lihat detail temuan tertentu
> 3. Tidak ada tindakan sekarang"

Tunggu jawaban. Kalau user pilih fix → panggil skill yang sesuai
(`/garnish:content-fix` atau `/garnish:design-fix`) dengan konteks audit
baru (A-004, bukan baseline).

## Yang TIDAK boleh dilakukan skill ini
- Overwrite atau modify entry audit baseline — selalu buat entry BARU
- Menganggap "bersih sendiri" = "sudah resolved" di registry baseline —
  cukup catat di `deltaStatus` finding baru, status baseline tidak
  berubah kecuali user eksplisit minta dismiss
- Membandingkan dengan audit yang bukan untuk URL yang sama
- Skip pembuatan artifact HTML (Langkah 8) — artifact wajib dibuat tiap
  re-audit selesai

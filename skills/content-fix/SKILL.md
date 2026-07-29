---
name: content-fix
description: Menulis ulang konten (headline, CTA, copy) berdasarkan temuan dari /garnish:check yang tersimpan di registry (.garnish/registry/audits.json). Gunakan skill ini ketika user pilih "fix konten" di checkpoint setelah audit. Tidak menggunakan Superpowers — ini murni rewrite teks oleh Claude. Selalu kasih 2-3 opsi rewrite, bukan 1 versi final, dan tanya konteks brand/tone dulu kalau belum ada. Update status temuan di registry setelah user pilih.
---

# /garnish:content-fix — Perbaikan Konten

## Langkah

### 1. Baca temuan dari registry
Ambil temuan (`findings`) dengan `status: "open"` dan kategori/type yang
relevan ke konten (bukan desain) dari audit yang dimaksud di
`.garnish/registry/audits.json`.

### 2. Cek konteks brand/tone
Kalau belum ada info soal tone/brand voice yang diinginkan, tanya dulu:
> "Sebelum saya tulis ulang, ini mau tone yang gimana — formal/profesional,
> santai/friendly, atau ada brand voice spesifik yang harus diikuti?"

Kalau user udah kasih konteks sebelumnya (brief awal, dll), lewati
pertanyaan ini.

### 3. Tulis ulang HANYA bagian yang ditandai fatal
Jangan rewrite seluruh halaman — cuma bagian yang temuannya spesifik
disebut di registry (headline gak jelas, CTA vague, dll).

### 4. Kasih 2-3 opsi per temuan, bukan 1 versi final
Untuk tiap bagian yang di-rewrite, kasih beberapa varian dengan pendekatan
berbeda (mis. "lebih to-the-point" vs "lebih emosional/storytelling"),
biar user pilih — jangan asumsikan 1 versi itu pasti yang terbaik.

### 5. Tampilkan before/after
Format: teks asli → temuan masalahnya apa (sebutkan ID temuan, mis. F-003)
→ opsi rewrite (2-3 varian).

### 6. Tunggu user pilih

Setelah user memilih salah satu opsi (atau minta revisi lagi dulu — ulangi
Langkah 3-5 kalau begitu), lanjut ke Langkah 6.5 SEBELUM update registry —
jangan langsung tandai `content-fixed`.

### 6.5. QA — verifikasi ulang sebelum ditandai selesai

Cek teks yang baru dipilih user terhadap kriteria yang tadinya bikin
temuan ini fatal:
- Kalau temuannya `type: "placeholder"` → pastikan teks baru TIDAK
  mengandung pola placeholder yang sama ("lorem ipsum", "TODO", dll).
- Kalau temuannya `type: "value-prop"` (judgment) → nilai ulang sendiri
  (sebagai penilaian AI juga) apakah teks baru benar-benar lebih jelas
  value proposition-nya dibanding versi asli — jangan asumsikan otomatis
  lebih baik cuma karena baru ditulis ulang.

Kalau QA ini menemukan teks yang dipilih user masih bermasalah (jarang
terjadi, tapi mungkin kalau user pilih opsi yang ternyata masih vague):
beri tahu user secara spesifik apa yang masih kurang, tawarkan opsi
rewrite baru (ulangi dari Langkah 3), maksimal 3 putaran total untuk
temuan yang sama. Kalau di putaran ke-3 masih belum lolos QA, laporkan
apa adanya ke user ("masih terasa kurang jelas, tapi ini upaya terbaik
setelah 3 percobaan — mau lanjut pakai ini atau coba arahan lain?") —
JANGAN tandai `content-fixed` kalau QA belum lolos.

### 7. Update registry (hanya setelah lolos QA Langkah 6.5)
```json
{ "status": "content-fixed", "fixedAt": "<ISO 8601>" }
```
Append journal:
```json
{"ts":"<ISO 8601>","event":"content_fixed","auditId":"A-00X","findingId":"F-00X"}
```

### 8. Cek apakah audit ini sudah selesai semua
Setelah update temuan di atas, lihat lagi entry audit yang sama
(`A-00X`) di `audits.json` — cek SEMUA `findings[]`-nya. Kalau tidak ada
lagi finding dengan `status: "open"` (artinya semua sudah
`content-fixed`, `design-fixed`, atau `dismissed`), update juga
`status` di level audit (bukan level finding) jadi `"resolved"`.

Kalau masih ada finding lain berstatus `"open"` di audit ini (mis. ada
temuan desain yang belum di-`design-fix`), JANGAN ubah status audit —
biarkan tetap `"in-progress"`.

## Yang TIDAK boleh dilakukan skill ini
- Menggunakan Superpowers atau metodologi coding apapun — ini murni
  penulisan teks
- Rewrite seluruh halaman kalau cuma sebagian yang ditandai fatal
- Kasih 1 versi final tanpa alternatif
- Mengubah copy yang TIDAK ditandai sebagai temuan fatal di registry
- Menandai temuan "content-fixed" sebelum user benar-benar memilih opsi
- Menandai "content-fixed" sebelum lolos QA Langkah 6.5, atau melanjutkan
  QA-retry lebih dari 3 putaran tanpa melapor apa adanya ke user

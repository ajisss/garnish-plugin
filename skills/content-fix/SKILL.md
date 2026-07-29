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

### 6. Tunggu user pilih, lalu update registry
Setelah user pilih salah satu opsi (atau minta revisi lagi), update
`.garnish/registry/audits.json` untuk temuan itu:
```json
{ "status": "content-fixed", "fixedAt": "<ISO 8601>" }
```
Append journal:
```json
{"ts":"<ISO 8601>","event":"content_fixed","auditId":"A-00X","findingId":"F-00X"}
```

## Yang TIDAK boleh dilakukan skill ini
- Menggunakan Superpowers atau metodologi coding apapun — ini murni
  penulisan teks
- Rewrite seluruh halaman kalau cuma sebagian yang ditandai fatal
- Kasih 1 versi final tanpa alternatif
- Mengubah copy yang TIDAK ditandai sebagai temuan fatal di registry
- Menandai temuan "content-fixed" sebelum user benar-benar memilih opsi

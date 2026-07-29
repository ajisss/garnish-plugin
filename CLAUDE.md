# GARNISH — Project Rules

**Baca `CONTEXT.md` dulu** sebelum ngerjain apapun — isinya semua keputusan
yang sudah diambil (problem, scope, prioritas, alasan di balik tiap
keputusan). Aturan di file ini adalah implementasi konkret dari prinsip di
`CONTEXT.md`.

## Aturan Wajib

1. **Registry adalah satu-satunya sumber kebenaran, bukan chat history.**
   Sebelum audit atau fix apapun, baca dulu `.garnish/registry/audits.json`.
   Semua temuan & status fix HARUS ditulis balik ke registry, bukan cuma
   disebutkan di chat. Kalau `.garnish/registry/` belum ada, jalankan
   `/garnish:init` dulu.

2. **Bedakan temuan terukur vs judgment secara eksplisit.**
   Gejala fatal yang bisa dihitung pasti (kontras, konsistensi komponen,
   posisi CTA, placeholder) dilabel sebagai **fakta**. Gejala yang butuh
   penilaian (value proposition, trust signal) WAJIB dilabel eksplisit ke
   user sebagai **"penilaian AI"**, bukan disamarkan seolah fakta pasti.

3. **Checkpoint adalah hard stop.**
   Setelah audit selesai dan temuan ditampilkan, WAJIB berhenti dan
   menunggu user memilih mana yang mau difix (konten/desain/keduanya/tidak
   ada). Jangan lanjut eksekusi fix atas asumsi sendiri.

4. **Fatal-only, bukan checklist lengkap.**
   Jangan generate laporan audit yang panjang mencakup semua kemungkinan
   masalah kecil. Fokus HANYA ke gejala yang benar-benar mengganggu
   (lihat daftar di `CONTEXT.md`). Laporan pendek dan actionable, bukan
   overwhelming.

5. **Fix desain reuse plugin `design-agent`, jangan bikin ulang.**
   Untuk fix desain, panggil `/design-agent:inspo`, `/design-agent:select`,
   `/design-agent:spec`, dan `/design-agent:build` yang sudah ada — jangan
   menulis ulang logic ekstraksi token, pencarian referensi, atau build
   dari nol di sini.

6. **Fix konten TIDAK pakai Superpowers.**
   Content-fix adalah instruksi menulis ulang teks berdasarkan temuan
   audit, dikerjakan langsung oleh Claude — bukan diserahkan ke metodologi
   coding apapun.

7. **Konten yang sudah ada tidak boleh ditimpa asal-asalan.**
   Kalau user pilih fix desain tapi TIDAK fix konten, konten asli halaman
   HARUS tetap dipakai persis — jangan ikut berubah karena efek samping
   proses rebuild desain.

8. **Satu halaman per audit run.**
   Jangan mencoba audit multi-halaman, flow, atau navigasi antar halaman —
   di luar scope. Kalau user minta itu, jelaskan itu di luar scope saat ini
   dan catat sebagai future improvement.

9. **Design-fix hanya rebuild komponen/section yang fatal.**
   Jangan rebuild seluruh halaman walau referensi baru sudah dipilih —
   cuma bagian yang ditandai fatal di `/garnish:check` yang boleh berubah.

## Struktur Project

```
CONTEXT.md          ← latar belakang lengkap, WAJIB dibaca duluan
.claude-plugin/      ← manifest plugin
.garnish/registry/   ← state permanen (dibuat via /garnish:init), di-commit ke git
skills/
  ├── init/SKILL.md          ← scaffold registry (sekali per project)
  ├── check/SKILL.md         ← audit gejala fatal
  ├── content-fix/SKILL.md   ← rewrite copy berdasarkan temuan
  └── design-fix/SKILL.md    ← orchestrator ke plugin design-agent
```

# GARNISH — Project Rules

**Baca `CONTEXT.md` dulu** sebelum ngerjain apapun — isinya semua keputusan
yang sudah diambil (problem, scope, prioritas, alasan di balik tiap
keputusan). Aturan di file ini adalah implementasi konkret dari prinsip di
`CONTEXT.md`.

## Aturan Wajib

1. **Registry adalah satu-satunya sumber kebenaran, bukan chat history.**
   Sebelum audit atau fix apapun, baca dulu `.garnish/registry/audits.json`.
   Semua temuan & status fix HARUS ditulis balik ke registry, bukan cuma
   disebutkan di chat. Kalau `.garnish/registry/` belum ada, `/garnish:check`
   auto-setup sendiri (lihat Langkah 0 di `skills/check/SKILL.md`) — tidak
   perlu jalankan skill setup terpisah. `/garnish:init` cuma dipakai kalau
   user eksplisit mau reset registry yang sudah ada.

2. **Bedakan temuan terukur vs judgment secara eksplisit.**
   Gejala fatal yang bisa dihitung pasti (kontras tombol, konsistensi
   komponen, posisi CTA, placeholder, layout rusak, alt text, heading
   hierarchy) dilabel sebagai **fakta**. Gejala yang butuh penilaian
   (kontras teks dari screenshot, value proposition, trust signal,
   evaluasi heuristik UI/UX) WAJIB dilabel eksplisit ke user sebagai
   **"penilaian AI"**, bukan disamarkan seolah fakta pasti.

2.5. **Audit selalu digroundkan ke rubric UI/UX yang mapan, konsisten
     tiap run.** Sebelum audit mulai, `/garnish:check` menanyakan scope
     (Konten/UI-UX/Komponen/WCAG, bisa pilih beberapa). Penilaian dalam
     tiap scope WAJIB merujuk ke rubric tertanam di
     `skills/check/SKILL.md` (Nielsen Usability Heuristics, Gestalt, Laws
     of UX, Progressive Disclosure, Cognitive Load, Atomic Design, subset
     WCAG AA, prinsip CRO, lensa AIDA/PAS) — bukan penilaian bebas yang
     bisa beda-beda tiap kali dijalankan. Rubric ini SATU-SATUNYA penentu
     kriteria temuan — web search (Langkah 3.5) boleh dipakai buat
     memperkuat `suggestion` dengan rujukan dari domain UX/design kredibel,
     TAPI TIDAK BOLEH menambah kriteria/jenis temuan baru di luar rubric.
     Tiap temuan WAJIB punya field `suggestion` (saran konkret merujuk
     rubric), TANPA estimasi angka/persentase dampak yang tidak bisa
     diverifikasi.

3. **Tawarin next step setelah audit, tunggu jawaban.**
   Setelah laporan + artifact keluar, `/garnish:check` WAJIB aktif tawarin
   pilihan (fix konten / fix desain / keduanya / rebuild / cukup laporan)
   dan tunggu jawaban user. JANGAN lanjut fix/rebuild tanpa pilihan
   eksplisit dari user di step ini.

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
   `design-fix` boleh nanya user "fix SEMUA temuan desain (`tinggi`+
   `sedang`) atau `tinggi` doang" (Langkah 1.5 di skill-nya) — "semua" di
   sini tetap berarti "semua temuan FATAL yang ada", BUKAN section yang
   gak punya temuan sama sekali. Pengecualian sadar satu-satunya buat
   full rebuild independen dari status fatal: skill `/garnish:rebuild`
   (aturan #11) — jangan pakai aturan ini buat membatasi `rebuild`.

10. **QA wajib sebelum temuan ditandai selesai.**
    `content-fix`, `design-fix`, DAN `rebuild` harus re-verifikasi hasilnya
    (bukan cuma asumsi proses fix-nya berhasil) sebelum status finding
    diubah jadi `content-fixed`/`design-fixed`/audit jadi `resolved` dan
    ditampilkan ke user. Maksimal 3 putaran perbaikan-ulang — kalau masih
    belum bersih di putaran ke-3, laporkan apa adanya, jangan klaim
    selesai.

11. **Rebuild adalah full rebuild ke project BARU, bukan fix bertarget.**
    `/garnish:rebuild` (dipilih user di checkpoint `/garnish:check`)
    membangun SEMUA section ke project baru terpisah, referensi dicari
    full landing page (bukan component showcase sempit seperti
    `design-fix`). Konten section yang TIDAK ditandai fatal tetap
    dipertahankan asli (aturan #7 tetap berlaku) — cuma section fatal
    yang ditulis ulang sesuai `suggestion`. TIDAK BOLEH menimpa project
    yang sedang dikerjakan user tanpa konfirmasi lokasi eksplisit.

## Struktur Project

```
CONTEXT.md          ← latar belakang lengkap, WAJIB dibaca duluan
.claude-plugin/      ← manifest plugin
.garnish/registry/   ← state permanen (auto-dibuat oleh /garnish:check), di-commit ke git
skills/
  ├── init/SKILL.md          ← reset registry manual (opsional, bukan prasyarat)
  ├── check/SKILL.md         ← audit gejala fatal, auto-setup registry
  ├── content-fix/SKILL.md   ← rewrite copy berdasarkan temuan + QA
  ├── design-fix/SKILL.md    ← orchestrator ke plugin design-agent (fix bertarget) + QA
  ├── rebuild/SKILL.md       ← full rebuild landing page baru ke project terpisah + QA
  └── monitor/SKILL.md       ← re-audit + delta comparison (regresi/baru/open/bersih) + artifact HTML
```

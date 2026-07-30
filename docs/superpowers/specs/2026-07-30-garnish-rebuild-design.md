# GARNISH — Full Landing Page Rebuild (skill baru: `rebuild`)

Refinement: setelah laporan audit `/garnish:check` ditampilkan, tawarkan
opsi ke-4 di checkpoint — bukan cuma "fix konten/desain/keduanya", tapi
juga "bikin landing page baru" berdasarkan seluruh temuan audit.

## Keputusan

1. **Scope: full page rebuild**, beda dari `design-fix` yang cuma
   memperbaiki komponen/section fatal. Semua section dibangun ulang dari
   referensi baru.
2. **Konten**: pakai konten ASLI dari halaman yang diaudit, ditingkatkan
   sesuai `suggestion` yang sudah ada di temuan (AIDA/PAS, dll) — bukan
   konten baru dari nol, bukan juga konten mentah tanpa perbaikan.
3. **Output**: project BARU terpisah (bukan menimpa project yang sedang
   dikerjakan user, karena halaman yang diaudit biasanya URL eksternal,
   bukan project lokal user).
4. **Referensi**: dicari BARU lewat `/design-agent:inspo`, scope-nya full
   landing page (bukan component showcase sempit seperti `design-fix`).

## Perubahan Checkpoint `/garnish:check` Langkah 6

Pertanyaan checkpoint jadi 4 opsi:
> "Dari temuan ini, mau dibenerin yang mana — kontennya, desainnya,
> keduanya, atau mau saya bikinin landing page baru berdasarkan semua
> temuan ini?"

Kalau user pilih opsi ke-4, panggil skill baru `/garnish:rebuild` (bukan
`content-fix`/`design-fix`).

## Skill Baru: `/garnish:rebuild`

1. **Baca SEMUA temuan** dari audit yang dimaksud (semua scope, semua
   status `open`) — bukan cuma yang user pilih spesifik, karena ini
   rebuild menyeluruh.
2. **Tanya lokasi project baru** — user tentukan folder/nama project baru
   (skill ini TIDAK menimpa apapun secara otomatis).
3. **Setup project baru**: scaffold dasar (framework/styling — tanya
   kalau belum ditentukan user), lalu `/design-agent:init` di project itu.
4. **Tingkatkan konten** (mirip logic `content-fix`, tapi diterapkan ke
   SEMUA section, tidak dibatasi cuma yang fatal): ambil konten asli per
   section dari halaman yang diaudit, tulis ulang bagian yang ditandai
   fatal sesuai `suggestion` (AIDA/PAS dll), section yang TIDAK fatal
   tetap pakai konten asli. Tetap tawarkan 2-3 opsi rewrite ke user untuk
   bagian yang diperbaiki (checkpoint, konsisten dengan prinsip
   `content-fix`).
5. **Cari referensi baru — FULL landing page** lewat `/design-agent:inspo`
   (brief berdasarkan jenis produk/industri halaman yang diaudit + prinsip
   yang perlu diperbaiki dari temuan desain, mis. "landing page SaaS
   dengan hero-features-pricing-testimonial yang solid, kontras tinggi").
6. **Checkpoint pemilihan referensi** — `/design-agent:select` seperti
   biasa.
7. **Scaffold component library** (Button/Card/Input) — sama seperti
   `design-fix` Langkah 6.
8. **Ekstrak spec** — `/design-agent:spec` pada referensi terpilih, FULL
   halaman kali ini (section confirmation berlaku normal, bukan
   dipersempit seperti `design-fix`).
9. **Build** — `/design-agent:build` bangun SELURUH halaman baru, pakai
   konten dari Langkah 4 + token dari Langkah 8 + component library dari
   Langkah 7.
10. **QA** — re-audit hasil build baru (reuse logic deteksi `check`, semua
    scope yang tadinya diaudit) sebelum ditampilkan ke user, loop maks 3
    putaran sama seperti `design-fix`/`content-fix`.
11. **Laporkan**: lokasi project baru, before/after, ringkasan section apa
    aja yang di-generate, dan hasil QA.

## Registry

`rebuild` mencatat progress di `.garnish/registry/audits.json` juga —
audit yang di-rebuild dapat `status: "resolved"` (karena SEMUA temuan
tercakup dalam rebuild, bukan sebagian), dan field baru per audit:
`rebuiltTo: "path project baru"`. Journal event baru: `rebuild_started`,
`rebuild_completed`.

## Yang TIDAK berubah

- `content-fix`/`design-fix` tetap ada apa adanya buat perbaikan
  bertarget (bukan full rebuild) — `rebuild` adalah opsi TAMBAHAN, bukan
  pengganti.
- Rubric audit yang sudah ada (Nielsen, Gestalt, Laws of UX, dll) tidak
  berubah — `rebuild` cuma konsumen dari hasil audit yang sudah ada.

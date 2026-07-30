# Konteks Proyek — GARNISH

*"The final touch before you serve it to the client."*

**Baca file ini duluan sebelum ngerjain apapun.** Ini rangkuman semua
keputusan yang sudah diambil di luar sesi ini — jangan tanya ulang hal yang
sudah dijawab di sini, tapi tetap tanya kalau ada detail teknis baru yang
belum diputuskan.

## Latar Belakang

Ini dibuat buat hackathon internal (tema: "AI Agents — Automate the Work We
Do Every Day"), dikerjakan **solo** dalam **2 hari**. User adalah UI/UX
designer di agency (Etalas) yang sebelumnya sudah membangun plugin
`design-agent` (lihat bagian "Reuse dari design-agent" di bawah).

## Problem Statement

> Kita sering harus audit landing page/screen klien secara manual sebelum
> ngasih rekomendasi redesign — buang waktu berjam-jam buat identifikasi
> masalah desain & konten satu-satu. Kita mau bikin agent yang bisa audit
> otomatis (desain + konten), kasih rekomendasi konkret, dan langsung
> generate revisi (copy + UI konsisten pakai component library internal),
> supaya proses evaluasi-ke-proposal yang biasanya makan waktu berhari-hari
> jadi hitungan menit.

## Defining the Agent

| Field | Isian |
|---|---|
| **User** | Tim internal Etalas (sales buat pitch, PM/designer buat deliverable klien) |
| **Input** | URL landing page ATAU screen aplikasi apapun (bukan cuma landing page — lihat "Broadened Framing" di bawah) |
| **Agent Actions** | Screenshot & ekstrak halaman → deteksi gejala fatal → checkpoint tanya user mau fix apa (konten/desain/keduanya/full rebuild) → eksekusi fix atau rebuild |
| **Output** | Laporan temuan fatal (prioritized) + revisi konten dan/atau desain (fix bertarget), ATAU project landing page baru penuh (kalau pilih rebuild) |
| **Success Criteria** | Dikasih 1 URL bermasalah, agent nemuin minimal 2-3 gejala fatal valid, user pilih mana yang mau difix, hasil revisi keluar dengan laporan before/after |

## Keputusan Desain Paling Penting: "Fatal-Only", Bukan Audit Lengkap

Ini keputusan sengaja — GARNISH BUKAN checklist audit menyeluruh (beda dari
tools kayak Unbounce/HubSpot Grader yang udah ada). Fokus cuma ke gejala
yang benar-benar mengganggu/fatal, biar laporan pendek dan actionable,
bukan overwhelming. Ini berlaku PER SCOPE (lihat "Checkpoint Scope Audit"
di bawah) — bukan cuma fatal-only secara global.

### Checkpoint Scope Audit (Konten / UI-UX / Komponen / WCAG)

Sebelum audit mulai, `/garnish:check` menanyakan scope (bisa pilih lebih
dari satu): **Konten**, **UI/UX**, **Komponen**, **WCAG**. Hanya modul
deteksi yang sesuai scope terpilih yang dijalankan. Rubric lengkap tiap
scope (Nielsen Usability Heuristics, prinsip Gestalt, Atomic Design,
subset kriteria WCAG AA, prinsip CRO) tertanam sebagai referensi di
`skills/check/SKILL.md` — supaya penilaian konsisten tiap audit, bukan
penilaian bebas yang bisa beda-beda tiap run. Tiap temuan WAJIB disertai
field `suggestion` (kenapa masalah + cara umum memperbaiki, TANPA
estimasi angka/persentase dampak yang tidak bisa diverifikasi).

### Gejala fatal & cara deteksinya (PENTING — bedakan yang terukur vs judgment)

| Gejala | Scope | Terukur objektif? | Cara deteksi |
|---|---|---|---|
| Kontras teks gak kebaca | WCAG | ❌ Judgment (kecuali background solid jelas) | Warna teks exact + background disampling dari screenshot |
| Kontras tombol gak kebaca | WCAG | ✅ Ya | Dua warna exact dari `extract-styles.py` |
| Style tombol/komponen gak konsisten dalam 1 halaman | Komponen | ✅ Ya | Baca computed CSS semua tombol, bandingkan (via `extract-styles.py`) |
| CTA gak keliatan di atas fold | UI/UX | ✅ Ya | Cek posisi bbox section relatif ke viewport height |
| Layout rusak (section overlap/overflow) | UI/UX | ✅ Ya | Bandingkan `bbox` antar section berurutan dari `extract-styles.py` |
| Pelanggaran heuristik Nielsen (consistency, aesthetic, dll) | UI/UX | ❌ Judgment | Penilaian Claude, digroundkan ke 4 heuristik spesifik |
| Section landing page hilang (hero/CTA/features/footer CTA) | UI/UX | ❌ Judgment | Bandingkan struktur section terhadap 6 section standar landing page |
| Placeholder ketinggalan ("Lorem ipsum", "TODO", dll) | Konten | ✅ Ya | Cari pola teks di HTML mentah |
| Value proposition/CTA copy gak jelas | Konten | ❌ Judgment | Penilaian Claude, digroundkan ke prinsip CRO "Clarity" |
| Gak ada trust signal/social proof/urgency | Konten | ❌ Judgment | Penilaian Claude, digroundkan ke prinsip CRO |
| Gambar tanpa alt text | WCAG | ✅ Ya | Cari `<img>` tanpa/kosong `alt` di HTML mentah |
| Urutan heading loncat level (h1→h3 tanpa h2) | WCAG | ✅ Ya | Urutan `typography[].level` per section (deteksi parsial) |

**Aturan wajib:** urutan prioritas pengerjaan adalah gejala TERUKUR dulu
(paling reliable, bisa dibuktikan dengan angka), baru gejala JUDGMENT
sebagai pelengkap. Setiap temuan kategori judgment HARUS dilabel eksplisit
ke user sebagai "penilaian AI", bukan disamarkan seolah fakta pasti — ini
prinsip yang sama dengan confidence marker di plugin `design-agent`.

## Reuse dari `design-agent` (plugin lain yang sudah ada, dependency)

GARNISH TIDAK membangun ulang pipeline design-to-code dari nol, dan TIDAK
membundel copy `design-agent` di repo ini. GARNISH depend ke plugin
`design-agent` yang sudah ada terpisah (`ajisss/design-agent-plugin`,
v1.2.0) — install `garnish` idealnya ikut resolve dependency ini, tapi
kalau marketplace berbeda tidak bisa auto-resolve, `design-fix` mengecek
sendiri di runtime dan kasih instruksi install manual kalau belum ada
(lihat `skills/design-fix/SKILL.md`).

Untuk bagian "fix desain" (`design-fix`, bertarget), GARNISH memanggil
skill `design-agent`:
- `/design-agent:inspo` — cari referensi baru yang menjawab temuan fatal
  spesifik (bukan referensi generik seluruh halaman)
- `/design-agent:select` — checkpoint pemilihan referensi tetap berlaku
- `/design-agent:spec` — ekstrak referensi terpilih jadi token, scope
  hanya komponen/section yang fatal
- `/design-agent:build` — rebuild HANYA komponen/section yang fatal, pakai
  component library yang di-scaffold `design-fix`

Untuk **full rebuild** (`/garnish:rebuild`, opsi ke-4 di checkpoint
`/garnish:check`), pemanggilan skill `design-agent` sama tapi scope-nya
FULL halaman, bukan bertarget: referensi yang dicari `/design-agent:inspo`
adalah landing page lengkap (bukan component showcase sempit), spec &
build tidak dibatasi ke komponen tertentu — semua section dibangun ulang.
Output-nya project BARU terpisah (bukan menimpa project yang sedang
dikerjakan user), dengan konten asli halaman yang diaudit dipertahankan
kecuali bagian yang ditandai fatal (ditulis ulang sesuai `suggestion`).

**Component library baru** (Button, Card, Input — cuma 3 komponen, jangan
lebih) di-scaffold oleh `design-fix`/`rebuild` sendiri (bukan skill
terpisah), kalau project belum punya, dipakai `design-agent:build` supaya
hasil fix/rebuild desain konsisten strukturnya.

Untuk "fix konten" (rewrite copy), GARNISH TIDAK pakai Superpowers — itu
kesalahpahaman yang sempat muncul dan sudah dikoreksi. Superpowers itu
metodologi coding (TDD/planning), bukan alat buat menulis ulang copy.
Content-fix itu murni instruksi tertulis-ulang oleh Claude berdasarkan
temuan audit.

## Spesialisasi ke Landing Page (revisi dari "Broadened Framing")

**Keputusan ini merevisi keputusan sebelumnya.** Sempat diputuskan
diperluas jadi generic (bisa audit screen/halaman apapun — dashboard,
settings, onboarding, dll), tapi dibalik lagi: GARNISH sekarang
**dispesialisasikan khusus buat landing page** dulu, supaya audit &
saran perbaikannya lebih presisi/tajam (rubric UI/UX boleh asumsikan
struktur landing page — hero, features/benefits, social proof, pricing,
FAQ, footer CTA — lihat `skills/check/SKILL.md`). Dashboard/aplikasi
ditunda ke iterasi terpisah nanti, BUKAN sekarang.

`/garnish:check` mendeteksi kalau URL yang diaudit kelihatan JELAS bukan
landing page (dashboard/aplikasi) dan memperingatkan user dulu sebelum
lanjut (bisa tetap lanjut atas keputusan eksplisit user, dicatat sebagai
`pageType: "non-landing-page-forced"` dengan catatan presisi lebih
rendah).

**Batasan yang tetap berlaku:** GARNISH audit SATU halaman per run. TIDAK
audit multi-halaman, user flow, atau navigasi antar halaman — itu di luar
scope (terlalu kompleks buat solo 2 hari), disebut sebagai "Future
Improvements" saja.

## Model Bisnis (buat konteks "kenapa ini penting", bukan tugas teknis)

1. **Sales/pitch tool** — dipakai tim sales Etalas sebelum ketemu calon
   klien, jadi bahan pitch konkret
2. **Deliverable berbayar** — "Audit Report" jadi paket terpisah sebelum
   kontrak redesign penuh
3. **Standalone tool** (jangka panjang, TIDAK jadi fokus hackathon) —
   freemium buat dijual ke luar Etalas

## Nama & Positioning

- **Nama**: GARNISH (bukan akronim, murni nama dengan metafora "sentuhan
  akhir sebelum disajikan")
- **Hubungan dengan tools lain**: kalau SANDWICH itu "dapur" yang masak
  requirement jadi spec, GARNISH itu quality check terakhir sebelum
  "hidangan" (produk/halaman) keluar ke meja klien
- **Diferensiasi dari kompetitor** (Unbounce, HubSpot Grader, dll): fokus
  fatal-only (bukan checklist lengkap) + langsung fix pakai design system
  internal sendiri, bukan cuma kasih skor

## Prinsip yang Diwarisi dari `design-agent` (tetap berlaku di sini)

- **Checkpoint adalah hard stop**, bukan formalitas — tunggu jawaban
  eksplisit user sebelum lanjut tahap berikutnya
- **Registry/state harus di file**, bukan diandalkan dari ingatan chat —
  GARNISH punya `.garnish/registry/` sendiri (`audits.json`, `journal.jsonl`),
  auto-di-scaffold oleh `/garnish:check` sendiri kalau belum ada (user tidak
  perlu jalankan skill setup terpisah — `/garnish:init` cuma tersedia buat
  reset manual eksplisit), dengan stable ID per audit (`A-00X`) dan per
  temuan (`F-00X`), mengikuti pola yang sama persis dengan SANDWICH dan
  `design-agent`
- **Jujur soal ketidakpastian** — penilaian judgment vs fakta terukur harus
  dibedakan secara eksplisit ke user
- **Konten bukan bagian dari style yang direplikasi** — kalau fix desain
  reuse referensi/benchmark, jangan timpa konten yang sudah ditulis dengan
  teks dari sumber lain
- **QA sebelum diklaim selesai** — `content-fix`, `design-fix`, DAN
  `rebuild` WAJIB re-verifikasi hasilnya (bukan cuma percaya proses fix-nya
  berhasil) sebelum menandai temuan/audit selesai dan menampilkannya ke
  user, mirip prinsip loop QA berbasis token di `design-agent:build`.
  Maksimal 3 putaran perbaikan-ulang — kalau masih belum bersih di putaran
  ke-3, laporkan apa adanya, jangan klaim selesai

## Prioritas Eksekusi (kalau waktu 2 hari mepet, urutan ini yang dipegang)

1. **Wajib solid**: skill `garnish:check` — deteksi gejala fatal (terukur
   dulu, baru judgment)
2. **Wajib solid**: checkpoint "mau dibenerin yang mana" + minimal 1 jalur
   eksekusi (content-fix)
3. **Selesai**: integrasi ke `design-agent` + component library buat fix
   desain, lewat skill `garnish:design-fix` — semua 4 skill (`init`,
   `check`, `content-fix`, `design-fix`) sudah solid, bukan lagi stretch
   goal.
4. **Selesai**: opsi full rebuild landing page baru lewat skill
   `garnish:rebuild` (skill ke-5) — checkpoint `/garnish:check` sekarang
   punya 4 jalur (konten/desain/keduanya/rebuild), bukan cuma 3.

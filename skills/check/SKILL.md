---
name: check
description: Audit sebuah landing page (dispesialisasikan buat landing page, BUKAN dashboard/aplikasi untuk sekarang) untuk mendeteksi gejala fatal desain & konten — bukan checklist lengkap, hanya masalah yang benar-benar mengganggu, digroundkan ke framework UI/UX & product design (Nielsen heuristics, Gestalt, Laws of UX — Fitts's/Hick's/Von Restorff/Serial Position/Aesthetic-Usability/Peak-End, Atomic Design, WCAG, prinsip CRO, lensa copywriting AIDA/PAS, struktur landing page standar) biar konsisten tiap audit, bukan penilaian bebas. Gunakan skill ini ketika user minta "audit", "cek", "garnish", atau kasih URL dan minta dievaluasi. Kasih warning kalau URL yang diaudit kelihatan bukan landing page. Menanyakan scope audit dulu (konten/UI-UX/komponen/WCAG, bisa pilih beberapa) sebelum mulai. Checkpoint di akhir sekarang juga nawarin opsi full rebuild landing page baru (panggil /garnish:rebuild), bukan cuma fix konten/desain bertarget. Tiap temuan disertai saran perbaikan konkret, kadang diperkuat rujukan dari domain UX/design kredibel (bukan pengganti rubric tetap). Skill ini otomatis setup registry (.garnish/registry/) sendiri kalau belum ada. Setelah temuan ditampilkan, WAJIB berhenti dan tanya user mau fix konten/desain/keduanya sebelum lanjut ke perbaikan apapun.
---

# /garnish:check — Audit Gejala Fatal

Baca `CONTEXT.md` dulu kalau belum — ada tabel gejala fatal dan mana yang
terukur vs judgment.

## Rubric Referensi (dipakai konsisten tiap audit — jangan menilai bebas di luar ini)

Ini kerangka yang WAJIB jadi acuan penilaian, bukan sekadar bacaan. Tiap
temuan di scope terkait harus bisa dikaitkan balik ke salah satu poin di
bawah ini.

### UI/UX — Usability Heuristics (Nielsen, versi ringkas relevan buat halaman statis)
- **Match between system & real world**: istilah/pola yang dipakai harus
  familiar buat target user, bukan jargon internal.
- **Consistency & standards**: elemen sejenis (tombol, link, ikon)
  terlihat/berperilaku sama di seluruh halaman.
- **Recognition over recall**: opsi/navigasi/CTA harus terlihat jelas,
  user gak perlu mengingat-ingat di mana sesuatu berada.
- **Aesthetic & minimalist design**: tiap elemen di layar harus punya
  tujuan; elemen berlebih/tidak relevan menambah noise visual.

### UI/UX — Prinsip Gestalt
- **Proximity**: elemen yang terkait (label+input, ikon+teks, judul+CTA)
  harus berdekatan; elemen tak terkait diberi jarak jelas.
- **Alignment**: elemen sejajar rapi mengikuti grid, bukan berantakan.
- **Similarity**: elemen dengan fungsi sama harus terlihat sama
  (warna/bentuk/ukuran konsisten).

### UI/UX — Laws of UX

6 hukum psikologi/perilaku yang sering dipakai product designer buat
menilai landing page, di luar heuristik Nielsen dan Gestalt di atas:
- **Fitts's Law**: target klik (tombol/CTA) harus cukup besar & mudah
  dijangkau — makin kecil/jauh, makin susah diklik. Ini SATU-SATUNYA
  dari 6 hukum ini yang bisa diukur objektif (lihat WCAG di bawah, target
  size), sisanya di bawah ini judgment.
- **Hick's Law**: makin banyak pilihan/CTA yang bersaing di satu section,
  makin lama & susah user memutuskan — terlalu banyak CTA setara
  prioritasnya bikin bingung.
- **Von Restorff Effect** (Isolation Effect): CTA utama harus menonjol
  beda dari elemen sekitarnya (warna/bentuk kontras dari palet dominan
  halaman) — kalau CTA nyaru sama elemen lain, gampang terlewat.
- **Serial Position Effect**: informasi di awal (hero) dan akhir (footer
  CTA) halaman paling diingat user — info penting jangan dikubur di
  tengah halaman.
- **Aesthetic-Usability Effect**: desain yang estetis dipersepsikan lebih
  mudah dipakai (walau belum tentu beneran lebih usable) — halaman yang
  berantakan visual bikin user meragukan kredibilitas produk juga.
- **Peak-End Rule**: kesan di titik paling intens & di akhir interaksi
  paling nempel di ingatan — pengalaman di footer/CTA penutup penting
  buat kesan akhir yang baik.

### UI/UX — Progressive Disclosure & Cognitive Load (NN/g, lawsofux.com)

- **Progressive Disclosure** (Nielsen Norman Group): jangan tampilkan
  semua info/opsi sekaligus — tampilkan yang paling penting/inti dulu,
  sediakan cara jelas buat "lihat lebih lanjut" (mis. FAQ accordion,
  "baca selengkapnya") buat detail sekunder. Lebih dari 2 level
  pengungkapan biasanya jadi masalah usability tersendiri.
- **Reducing Cognitive Load** (lawsofux.com) — 7 teknik, 5 di antaranya
  SUDAH tercakup prinsip lain di atas (jangan diulang sebagai temuan
  terpisah): "avoid unnecessary elements" = Aesthetic & Minimalist
  (Nielsen), "leverage common patterns" = Match Real World/Consistency
  (Nielsen), "eliminate unnecessary tasks" = Reduce Friction (CRO),
  "minimize choices" = Hick's Law, "readability" = sudah implisit di
  kontras & hierarki heading. 2 teknik BARU yang belum tercakup:
  - **Icon dengan label teks**: ikon tanpa teks pendamping menambah beban
    kognitif (user harus menerka arti ikonnya) — WAJIB ada teks atau
    `aria-label` yang jelas.
  - **Display choices as a group**: pilihan sejenis (mis. tier
    pricing, kategori fitur) harus ditampilkan berkelompok jelas
    (grid/baris sejajar), bukan tersebar acak di halaman.

### Komponen — Atomic Design
- Komponen sejenis (button, card, input) harus konsisten variant &
  state-nya di seluruh halaman — bukan di-reinvent beda-beda tiap section
  tanpa alasan hierarki visual yang jelas.

### WCAG (subset kriteria AA yang paling gampang diverifikasi objektif — BUKAN audit aksesibilitas lengkap)
- Kontras teks vs background (rasio WCAG).
- Alt text pada gambar non-dekoratif.
- Urutan heading semantik tidak boleh loncat level (h1→h2→h3, TIDAK
  boleh h1 langsung ke h3).
- **Target size** (WCAG 2.5.5, juga Fitts's Law di atas): target
  klik (tombol) idealnya minimal ~44x44px area efektif (padding + ukuran
  konten teks). Target yang jauh lebih kecil dari itu = fatal.
- **Icon tanpa label** (WCAG 4.1.2 Name/Role/Value, juga cognitive load
  di atas): elemen interaktif (tombol/link) yang cuma berisi ikon
  (SVG/font-icon) tanpa teks pendamping DAN tanpa `aria-label` = fatal.

### UI/UX — Struktur Landing Page yang Diharapkan

Garnish saat ini dispesialisasikan buat **landing page** (bukan generic
"screen apapun" — dashboard/aplikasi ditunda buat iterasi lain). 6 section
yang biasa ada di landing page:
- **Hero**: headline (value prop) + subheadline + CTA utama, semuanya di
  atas fold.
- **Features/Benefits**: penjelasan fitur/manfaat produk.
- **Social Proof**: testimoni/rating/logo klien — CATATAN: ini overlap
  sama pengecekan `trust-signal` di scope Konten, JANGAN dilaporkan dua
  kali. Kalau sudah dilaporkan sebagai `trust-signal`, jangan diulang di
  sini sebagai `missing-section`.
- **Pricing**: kondisional — cuma relevan kalau halaman jelas jualan
  produk/jasa berbayar. JANGAN tandai fatal kalau jenis landing page-nya
  gak butuh pricing (mis. landing page event/webinar/pendaftaran gratis).
- **FAQ**: kondisional — sama kayak pricing, gak semua landing page perlu.
- **Footer CTA**: CTA penutup di bagian akhir halaman (selain CTA di hero).

### Konten — Prinsip CRO (Conversion Rate Optimization) dasar
- **Clarity**: value proposition & CTA harus bisa dipahami dalam beberapa
  detik pertama, tanpa perlu mikir.
- **Reduce friction**: makin sedikit langkah/input yang diminta sebelum
  konversi (isi form, klik CTA), makin baik.
- **Social proof & urgency**: testimoni/rating/jumlah user, atau elemen
  urgency (limited time/stock) umumnya menaikkan konversi — bukan wajib
  ada di semua jenis halaman, tapi ketidakhadirannya di halaman yang
  jualan sesuatu (produk/jasa/pendaftaran) layak dicatat.

### Konten — Lensa Copywriting (AIDA & PAS)

Dipakai buat MENGANALISIS pengecekan "Kejelasan value proposition & CTA
copy" yang sudah ada (bukan finding/type baru terpisah) — bikin
`suggestion`-nya lebih terstruktur, bukan cuma "kurang jelas".
- **AIDA** (Attention → Interest → Desire → Action): headline harus
  menarik perhatian, body copy membangun minat & keinginan, CTA
  mendorong aksi konkret. Kalau salah satu tahap lemah/hilang, sebutkan
  tahap mana secara eksplisit di `suggestion` (mis. "kuat di Attention
  tapi Desire-nya lemah karena gak ada penjelasan manfaat konkret").
- **PAS** (Problem → Agitate → Solve): copy yang efektif biasanya
  menyebut masalah yang dialami target user, memperjelas dampak masalah
  itu, baru menawarkan solusi (produk/jasa). Relevan buat menilai apakah
  copy langsung "jualan fitur" tanpa membangun konteks masalah dulu.

## Langkah

### 0. Cek input & registry (auto-setup, bukan langkah terpisah)

**Deteksi mode input:**

Kalau user **attach gambar/screenshot** (bukan URL):
- Set `inputMode: "screenshot"` — seluruh audit dijalankan dalam
  **visual-only mode**
- Skip `extract-styles.py` sepenuhnya (tidak ada live DOM)
- Skip `WebFetch` (tidak ada URL untuk di-fetch)
- Semua finding otomatis `category: "judgment"` — tidak ada yang bisa
  diukur pasti dari screenshot
- Langkah 2 (screenshot + extract-styles) digantikan dengan: analisis
  gambar yang di-attach langsung sebagai sumber visual utama
- Langkah 2.5 (deteksi bukan landing page) tetap berlaku — nilai dari
  tampilan visual
- Di laporan dan artifact, tambahkan note eksplisit:
  > "⚠️ Audit ini berdasarkan screenshot — semua temuan adalah penilaian
  > AI, tidak ada yang terukur pasti. Untuk audit lebih akurat, berikan
  > URL live."
- `url` di registry diisi `"screenshot:<nama-file atau deskripsi singkat>"`

Kalau user kasih **URL** (default) → `inputMode: "url"`, lanjut normal.

**Session-fresh: wipe registry di awal session baru**

Sebelum setup/baca registry, cek apakah ada output `/garnish:check` atau
`/garnish:monitor` sebelumnya di conversation history sesi ini (yaitu:
apakah ada laporan audit yang sudah ditampilkan di chat ini).

- **Tidak ada** (ini invokasi audit pertama di session ini) → wipe registry:
  ```bash
  rm -rf .garnish/registry/
  ```
  Lalu setup ulang dari nol (langkah di bawah). Tidak perlu kasih tau user
  secara eksplisit — ini perilaku yang diharapkan.

- **Sudah ada** (sudah ada audit sebelumnya di session ini) → skip wipe,
  langsung baca registry yang ada.

**Baca brand context (kalau ada):**

Setelah registry dicek, baca `.garnish/brand.md` kalau file itu ada:
```bash
cat .garnish/brand.md 2>/dev/null
```
Simpan isinya sebagai `brandContext` — dipakai Langkah 3 saat audit scope
Konten dan UI-UX untuk menilai kesesuaian tone, warna, dan audience. Kalau
file tidak ada atau kosong, lanjut audit tanpa brand context (tidak perlu
kasih tau user).

**Setup registry:**

Kalau `.garnish/registry/audits.json` belum ada (setelah wipe atau memang
belum pernah), setup sendiri di sini — JANGAN minta user jalankan skill
lain dulu:
```bash
mkdir -p .garnish/registry/screenshots
cat > .garnish/registry/audits.json << 'EOF'
{ "audits": [] }
EOF
touch .garnish/registry/journal.jsonl
```
Tulis juga `.garnish/registry/SCHEMA.md` — isinya sama persis dengan yang
didokumentasikan di `skills/init/SKILL.md` Langkah 3 (baca file itu untuk
konten lengkapnya, salin verbatim).

Kasih tau user singkat (1 baris, gak usah bertele-tele): "Registry belum
ada, saya setup dulu di `.garnish/registry/`." — lalu lanjut ke Langkah 1
di pesan yang sama, jangan berhenti nunggu konfirmasi buat langkah setup
ini (ini bukan checkpoint, cuma info).

Kalau `.garnish/registry/audits.json` SUDAH ada (dan tidak di-wipe), langsung
lanjut tanpa basa-basi setup.

Cek juga apakah URL yang sama pernah diaudit sebelumnya (`audits.json`).
Kalau ada entry sebelumnya, **deteksi intent dari cara user memanggil skill ini**:

**Intent "audit ulang"** (user sudah jelas mau deteksi baru) — langsung
supersede tanpa tanya, kalau user bilang salah satu dari:
- "audit ulang [url]" / "audit lagi [url]"
- "coba audit lagi" / "re-audit"
- "reset dan audit ulang"
- atau kalimat apapun yang punya kata "ulang", "lagi", "re-", "fresh"
→ Ubah status entry lama jadi `"superseded"` di `audits.json`, lalu
lanjut ke Langkah 1 dan jalankan **deteksi penuh dari awal** (BUKAN baca
dari registry lama). Entry audit baru (A-00X baru) akan dibuat di Langkah 4.

**Intent ambigu** (user cuma kasih URL atau tidak ada sinyal re-audit) —
baru tanya:
> "URL ini pernah diaudit tanggal X, ada Y temuan, status Z.
> Mau audit ulang dari nol, atau lihat status temuan yang lama dulu?"
- Jawab **"lihat status lama"** → tampilkan temuan dari entry lama,
  tawarin pilihan Langkah 6 berdasarkan temuan yang masih `open`.
  JANGAN jalankan deteksi baru.
- Jawab **"audit ulang"** → sama seperti intent "audit ulang" di atas.

### 0.5. Discover semua halaman situs (kalau audit dimulai dari 1 URL)

Sebelum checkpoint scope (Langkah 1), tanya dulu cakupan audit — HARD STOP:

> "Mau audit halaman mana?
> 1. **Semua halaman** — saya discover dulu via sitemap, lalu audit tiap halaman
> 2. **Homepage aja** — cuma audit URL yang dikasih"

Kalau user tidak spesifik (cuma kasih URL tanpa konteks lain), default
tawarin kedua pilihan ini. Tunggu jawaban.

- Pilih **"Semua halaman"** → lanjut ke discovery di bawah
- Pilih **"Homepage aja"** → skip discovery, langsung ke Langkah 1 dengan
  URL yang dikasih user saja

Kalau user sudah spesifik di awal (mis. "audit semua halaman [url]" atau
"audit homepage [url]") → skip pertanyaan ini, langsung eksekusi sesuai
intent.

**Discovery (hanya kalau pilih semua):**

Cari tau halaman APA SAJA yang ada di situs ini — jangan cuma audit URL
yang dikasih user, karena biasanya itu cuma pintu masuk (homepage).

1. **Cek `/sitemap.xml` dari domain yang sama** (`curl` atau `WebFetch`
   ke `<origin>/sitemap.xml`). Ini sumber discovery UTAMA — jangan cuma
   andalkan link di nav, karena nav bisa miss halaman yang sengaja/tidak
   sengaja gak ditaut di menu utama (pernah kejadian: halaman landing
   campaign terpisah cuma ketemu lewat sitemap, sama sekali gak ada di
   nav homepage).
2. Kalau `sitemap.xml` gak ada (404) atau gagal di-fetch, fallback ke
   scan link `<nav>`/`<footer>` di HTML mentah halaman yang dikasih user.
3. Dari daftar URL yang ketemu, **saring jadi 3 kelompok**:
   - **Halaman nyata berbeda** (path berbeda, mis. `/`, `/products`,
     `/pricing`, `/mvp`) → semuanya masuk antrian audit.
   - **Anchor link ke section di halaman yang sama** (`/#work`,
     `/#faq`, dst — fragment `#...` di path yang sama) → BUKAN halaman
     terpisah, skip, jangan diaudit lagi (section-nya sudah ter-cover
     saat audit halaman itu).
   - **Blog/artikel individual** (pola URL `/blog/judul-artikel`,
     `/news/...`, `/artikel/...` — halaman detail di bawah satu index
     listing) → **default-nya SKIP, cuma audit index/listing-nya**
     (mis. `/blog`), JANGAN audit satu-satu tiap artikel. Ini
     berlaku otomatis, TIDAK perlu tanya user dulu — artikel individual
     nyaris selalu berbagi template yang sama dengan index, jadi kalau
     ada masalah struktural, index atau satu sample sudah cukup
     mewakili.
   - Kalau ada pola sub-halaman berulang LAIN yang bukan blog (mis.
     katalog produk dengan halaman detail per produk `/products/nama`),
     TETAP tanya user sekali: audit semua/sample beberapa/skip. Simpan
     jawabannya dan terapkan konsisten kalau situs yang sama diaudit
     ulang nanti — jangan tanya ulang tiap kali.
4. Tampilkan singkat ke user daftar halaman nyata yang bakal diaudit
   (1 baris, bukan checkpoint — info aja, lanjut ke Langkah 1 di pesan
   yang sama) — kecuali ada keputusan yang butuh dikonfirmasi (poin 3,
   sub-halaman berulang non-blog).
5. Untuk tiap halaman nyata di antrian, jalankan Langkah 1–7 di bawah
   ini SATU PER SATU (tiap halaman = 1 audit id `A-00X` baru sendiri di
   registry, sesuai Langkah 4) — bukan digabung jadi satu ekstraksi.
   Checkpoint scope (Langkah 1) cukup ditanya SEKALI di awal dan
   diterapkan ke semua halaman dalam antrian ini, kecuali user secara
   eksplisit minta scope berbeda per halaman.
6. Kalau temuan yang PERSIS SAMA (biasanya karena komponen bersama
   seperti header/footer/navbar) muncul di lebih dari satu halaman dalam
   antrian ini, JANGAN tulis sebagai temuan terpisah per halaman —
   catat SEKALI dengan referensi ke semua ID audit/halaman yang
   terdampak (lihat format di Langkah 4 dan Langkah 7).

Kalau user cuma minta audit 1 URL spesifik secara eksplisit (mis. "audit
cuma halaman ini aja, jangan yang lain") — hormati itu, skip Langkah 0.5
ini sepenuhnya dan audit persis 1 URL yang diminta.

### 1. Checkpoint — pilih scope audit (HARD STOP)
Sebelum ekstraksi/deteksi apapun, tanya:
> "Mau audit apa? Bisa pilih lebih dari satu — **Konten**, **UI/UX**,
> **Komponen**, **WCAG**, atau **Semua**."

Tunggu jawaban eksplisit. JANGAN asumsikan "Semua" sebagai default kalau
user belum jawab. Kalau user pilih "Semua", proses sebagai pilih semua 4
scope. Simpan pilihan ini (dipakai Langkah 3 buat nentuin modul deteksi
mana yang jalan) dan catat di entry audit (`scopeAudited`, Langkah 4).

**Fatal-only tetap berlaku per scope** — dalam scope yang dipilih, cuma
laporkan yang benar-benar mengganggu, bukan semua kemungkinan temuan di
kategori itu.

### 2. Ekstrak halaman (reuse tool dari `design-agent`, jangan tulis ulang)
Kalau scope yang dipilih HANYA "Konten" (tidak termasuk UI/UX, Komponen,
atau WCAG) → skip langkah `extract-styles.py` ini sepenuhnya, langsung ke
`WebFetch <url>` buat HTML mentah (dipakai deteksi konten). Untuk scope
lain (UI/UX, Komponen, WCAG, atau kombinasi apapun yang menyertakannya),
tetap jalankan:
```bash
EXTRACT_STYLES=$(find ~/.claude/plugins/cache -name "extract-styles.py" -path "*design-agent*" 2>/dev/null | head -1)
if [ -n "$EXTRACT_STYLES" ]; then
  python3 "$EXTRACT_STYLES" <url> .garnish/registry/audit-<audit-id>.json .garnish/registry/screenshots/<audit-id>-sections
else
  echo "extract-styles.py tidak ditemukan — kemungkinan plugin design-agent belum ter-install."
fi
```
Kalau `$EXTRACT_STYLES` kosong (blok `else` di atas jalan, `python3` TIDAK
pernah dipanggil dengan path kosong): beri tahu user terus terang bahwa
`extract-styles.py` tidak ditemukan, dan lanjutkan HANYA dengan deteksi
yang tidak butuh computed CSS untuk scope yang dipilih (placeholder/alt
text/heading-hierarchy dari HTML mentah, dan judgment berdasar screenshot
manual/deskripsi user) — jangan klaim deteksi kontras/konsistensi/
posisi-CTA/layout-rusak kalau datanya tidak ada.

Kalau `$EXTRACT_STYLES` ketemu, ini menghasilkan JSON terstruktur (warna
dominan, tipografi per section, tombol, container, bbox per section) DAN
screenshot full-page di
`.garnish/registry/screenshots/<audit-id>-sections/_full-page.png` — pakai
file itu sebagai `screenshotPath` di registry (Langkah 4), jangan cari
script screenshot terpisah.

Catatan: JSON hasilnya juga punya `sections[].screenshot_path`, tapi itu
path screenshot CROP per section (`section-<index>.png`), bukan full-page
— jangan tertukar dengan `_full-page.png` di atas.

Kalau `extract-styles.py` gagal jalan (exit code 1 — situs block bot,
butuh login, atau Playwright/Pillow belum terinstall): beri tahu user
terus terang, dan lanjutkan HANYA dengan deteksi yang tidak butuh
computed CSS untuk scope yang dipilih.

Ambil juga HTML mentah halaman (`WebFetch <url>`) — WAJIB dipakai sebagai
sumber data utama di samping screenshot, bukan cuma buat placeholder/alt
text/heading hierarchy. Semua penilaian yang butuh tau ISI/STRUKTUR
halaman (guard Langkah 2.5, evaluasi heuristik & missing-section di
Langkah 3c, kejelasan value-prop/trust-signal di Langkah 3d) HARUS
merujuk ke teks & tag semantik dari HTML mentah ini (heading, `<nav>`,
`<form>`, `<section>`, class/id yang menyebut "pricing"/"faq"/"testimonial"
dll, isi teks sebenarnya) — screenshot cuma pelengkap buat verifikasi
visual (tata letak, kontras, dll), BUKAN satu-satunya sumber buat
menyimpulkan konten/struktur halaman.

**Kalau deteksi section dari `extract-styles.py` kelihatan tidak
lengkap** (mis. jumlah section yang terdeteksi jauh lebih sedikit dari
yang kelihatan di screenshot — bisa terjadi karena heuristik deteksinya
terbatas, atau section pendek/dengan struktur tidak umum ke-skip):
JANGAN hentikan audit. Lanjutkan pakai data yang berhasil dideteksi, dan
lengkapi dari HTML mentah secara manual (cari tag `<section>`/`<header>`/
`<footer>`/heading tambahan yang mungkin tidak masuk hasil ekstraksi) buat
menutup gap-nya sebisa mungkin. Sebutkan di laporan akhir (Langkah 5)
kalau deteksi section-nya tidak lengkap, supaya user tau keterbatasannya
— tapi tetap sajikan hasil audit dari data yang ada, jangan menolak audit
sepenuhnya hanya karena deteksinya sebagian.

### 2.5. Guard — pastikan ini landing page (HARD STOP kalau meragukan)

Garnish saat ini dispesialisasikan buat landing page (lihat Rubric
Referensi "UI/UX — Struktur Landing Page"). Sebelum deteksi mulai, nilai
DARI DUA SUMBER — screenshot full-page DAN HTML mentah (Langkah 2) —
apakah ini kelihatan seperti landing page marketing (single scrolling
page tentang produk/value proposition, section-section seperti
hero/fitur/CTA, teks/CTA yang mengarah ke konversi), ATAU kelihatan
seperti aplikasi/dashboard (butuh login — cek ada `<form>` login/`<input
type="password">`, sidebar navigasi dengan banyak link internal app,
tabel data, banyak kontrol interaktif kompleks). Jangan simpulkan cuma
dari tampilan visual — screenshot bisa menipu (mis. halaman marketing
untuk produk SaaS yang menampilkan screenshot dashboard sebagai bagian
dari konten, itu TETAP landing page, bukan dashboard sungguhan).

Kalau kelihatan JELAS bukan landing page (mis. dashboard admin, halaman
settings aplikasi, halaman yang butuh login buat diakses): STOP, tanya
user:
> "Halaman ini kelihatan bukan landing page (lebih mirip [dashboard/
> aplikasi]) — garnish saat ini dioptimasi khusus buat landing page, jadi
> rubric UI/UX & saran yang diberikan mungkin kurang presisi buat jenis
> halaman ini. Tetap mau lanjut audit (dengan catatan presisi lebih
> rendah), atau batal?"

Tunggu jawaban eksplisit. Kalau user pilih lanjut, catat di entry audit
(`pageType: "non-landing-page-forced"`) dan tetap jalankan deteksi
seperti biasa. Kalau ragu-ragu (bukan jelas landing page, tapi juga bukan
jelas dashboard/app) → jangan STOP, anggap layak diaudit seperti biasa
tanpa warning (guard ini cuma buat kasus yang JELAS beda kategori, bukan
setiap ambiguitas kecil).

### 3. Deteksi — HANYA modul yang sesuai scope yang dipilih Langkah 1

Urutan tetap: TERUKUR dulu (paling reliable) baru JUDGMENT. Setiap temuan
WAJIB diisi field `suggestion` (lihat format di Langkah 4) — kenapa ini
melanggar prinsip di Rubric Referensi di atas, dan gimana biasanya
diperbaiki. JANGAN kasih estimasi angka/persentase dampak (mis. "naikin
konversi 15%") — itu klaim yang gak bisa diverifikasi, jadi kualitatif
saja, grounded ke rubric.

#### 3a. Scope "WCAG" (`category: "measured"` semua, kecuali disebutkan lain)

**`wcagLevel` — wajib diisi untuk tiap temuan di scope WCAG:**
Tiap finding WCAG harus punya field `wcagLevel` yang menyatakan level
compliance WCAG yang dilanggar. Tentukan berdasarkan jenis temuan:

| type | wcagLevel | Kriteria |
|------|-----------|---------|
| `contrast` (normal text < 4.5:1) | `"AA"` | WCAG 1.4.3 |
| `contrast` (large text < 3:1) | `"AA"` | WCAG 1.4.3 |
| `alt-text` | `"A"` | WCAG 1.1.1 |
| `heading-hierarchy` | `"A"` | WCAG 1.3.1 |
| `target-size` | `"AA"` | WCAG 2.5.5 |
| `icon-label` | `"A"` | WCAG 4.1.2 |

Kalau temuan di scope WCAG tidak punya padanan di tabel ini → `wcagLevel: null`.
Temuan di scope LAIN (konten/ui-ux/komponen) → `wcagLevel: null` selalu.

- **Alt text** (`type: "alt-text"`):
  1. Dari HTML mentah, extract SEMUA tag `<img>` — buat list lengkap
     (src + alt value).
  2. Filter: tandai setiap `<img>` yang (a) tidak punya atribut `alt`
     sama sekali, atau (b) punya `alt=""` tapi gambarnya jelas bukan
     dekoratif (bukan spacer/divider, ada konten visual nyata).
  3. Hitung total: berapa `<img>` keseluruhan, berapa yang missing/empty alt.
  4. Laporkan angkanya secara eksplisit: "Ditemukan X gambar tanpa alt
     text dari total Y gambar di halaman."
  Batasan: tidak menilai apakah teks alt yang ADA sudah deskriptif —
  cuma cek keberadaannya.

- **Heading hierarchy** (`type: "heading-hierarchy"`):
  1. Dari HTML mentah, extract SEMUA tag `<h1>`–`<h6>` secara berurutan
     sesuai posisinya di DOM — buat sequence level, mis. [1, 2, 2, 3, 2, 4].
  2. Telusuri sequence itu: tiap kali level NAIK (angka level makin besar)
     lebih dari 1 tingkat sekaligus (mis. h1 → h3, atau h2 → h4) = loncat
     level = fatal.
  3. Laporkan di mana tepatnya loncat: "h1 'Gaia Digital Agency' langsung
     diikuti h3 'Our Work' tanpa h2 di antaranya."
  CATATAN: `extract-styles.py` hanya tangkap maks 3 heading per section —
  wajib gunakan HTML mentah sebagai sumber utama untuk cek ini.

- **Kontras teks** (`type: "contrast"`):
  JSON tidak punya `background_color` di level section — background harus
  diestimasi dari screenshot. Untuk tiap `typography[]` entry:
  pakai `color`-nya (exact hex) sebagai warna teks, visual-sample warna
  background di belakang teks itu dari crop section. Hitung rasio WCAG:
  - Teks normal: wajib ≥ 4.5:1
  - Teks besar (≥24px atau ≥18.5px bold): wajib ≥ 3:1
  Kontras teks = `category: "judgment"` karena background estimasi visual.
  Kontras tombol = `category: "measured"` karena `buttons[].color` vs
  `buttons[].background_color` keduanya field JSON exact.

- **Target size** (`type: "target-size"`, Fitts's Law + WCAG 2.5.5):
  1. Dari `buttons[]`, untuk tiap tombol: ambil `padding` (mis. "12px 24px"
     = vertikal 12px, horizontal 24px).
  2. Estimasi tinggi efektif = (2 × padding-vertikal) + ~16px tinggi font.
  3. Kalau hasil < 32px = fatal (jelas di bawah ambang 44px WCAG 2.5.5).
  Tetap `category: "measured"` — angka dari field JSON exact.

- **Icon tanpa label** (`type: "icon-label"`, WCAG 4.1.2 + cognitive load):
  1. Dari HTML mentah, cari SEMUA elemen `<button>` dan `<a>`.
  2. Filter: elemen yang isinya HANYA `<svg>` atau font-icon class
     (`fa-*`, `icon-*`, `bi-*`, `material-icons`, dll) TANPA teks visible
     di dalamnya DAN tanpa `aria-label`/`aria-labelledby`/`title`.
  3. Buat list semua yang ditemukan — laporkan satu per satu.
  Kalau ada teks visible (walau kecil) ATAU `aria-label` terisi → skip.

#### 3b. Scope "Komponen" (`category: "measured"`)
- **Konsistensi komponen**:
  1. Dari JSON, kumpulkan SEMUA `buttons[]` di seluruh section — buat
     tabel: section index | button text | background_color | border_radius
     | padding.
  2. Kumpulkan SEMUA `containers[]` di seluruh section — buat tabel:
     section index | border_radius | padding.
  3. Untuk `border_radius` tombol: cek apakah nilai uniknya > 2 nilai
     berbeda tanpa pola hierarki jelas (mis. 4px, 8px, 16px sekaligus
     tanpa ada beda konteks primary/secondary) = fatal.
  4. Untuk `background_color` tombol: cek apakah ada > 2 warna berbeda
     signifikan (bukan shade gelap/terang dari warna yang sama) tanpa
     pola hierarki jelas = fatal.
  5. Untuk `padding` container: cek apakah variasinya ekstrem (mis.
     "8px" vs "64px" pada container yang fungsinya sama) = fatal.
  6. Laporkan NILAI KONKRET yang bertentangan, bukan hanya "tidak
     konsisten" — mis. "Tombol di section 1: radius 4px, section 3: 16px,
     section 5: 0px."
  Rujuk ke prinsip Atomic Design di Rubric Referensi saat menyusun
  `suggestion`.

- **Line length terlalu panjang** (`type: "line-length"`, `category: "measured"`):
  1. Dari JSON, cari semua text node di `paragraphs[]` atau body copy
     (bukan heading). Estimasi karakter per baris: `bbox.width` ÷
     `font_size` × 0.45 (faktor rata-rata untuk font proporsi normal).
  2. Kalau estimasi > 80 karakter per baris pada blok teks utama = fatal
     (threshold optimal: 60–75ch, max 80ch).
  3. Laporkan: "Estimasi ~Xch per baris di section Y (threshold: 75ch)."
  Rujuk ke prinsip Typography readability di `suggestion`.

- **Cramped padding** (`type: "cramped-padding"`, `category: "measured"`):
  1. Dari JSON, cek `padding` di semua `buttons[]` dan `containers[]`.
  2. Untuk tombol: kalau padding vertikal < 8px atau horizontal < 12px = fatal.
  3. Untuk container/card: kalau padding di semua sisi < 12px = fatal.
  4. Laporkan nilai konkret: "Tombol di section X: padding 4px 8px
     (minimum: 8px 12px)."
  Rujuk ke prinsip Whitespace dan Cognitive Load di `suggestion`.

#### 3c. Scope "UI/UX"
**Kalau `brandContext` tersedia:** gunakan `warna utama` dan `tone of voice`
sebagai referensi saat menilai konsistensi visual dan kesesuaian mood desain.
Contoh: kalau brand context menyebut "minimalis" tapi halaman penuh animasi
dan warna ramai → flag sebagai temuan judgment UI-UX.

- **Posisi CTA** (`category: "measured"`):
  1. Dari JSON, ambil section index 0 (hero). Cek `bbox.height`-nya.
  2. Cek apakah ada `buttons[]` di section 0. Kalau ada, estimasi posisi
     bawah tombol = `bbox.top` section 0 + `bbox.height` section 0.
  3. Kalau estimasi posisi > 900px, ATAU tidak ada `buttons[]` sama
     sekali di section 0 = fatal.
  4. Laporkan angkanya: "Hero section tingginya Xpx, tombol CTA pertama
     baru muncul di ~Ypx dari atas halaman."
  Kalau hero
  section sudah lebih tinggi dari 900px DAN tidak ada tombol di dalamnya
  = fatal (CTA kemungkinan di bawah fold). Rujuk ke "Recognition over
  recall" di `suggestion`.
- **Layout rusak** (`category: "measured"`): dari `sections[].bbox`
  (terurut by index), untuk tiap pasangan section berurutan: kalau
  `section[i].bbox.y + height` LEBIH BESAR dari `section[i+1].bbox.y` =
  overlap vertikal = fatal. Kalau `bbox.width` jauh lebih besar dari
  viewport (1440px default) = indikasi overflow horizontal = fatal. Rujuk
  ke prinsip Alignment/Similarity (Gestalt) di `suggestion`.
- **Evaluasi heuristik & Laws of UX** (`category: "judgment"`, WAJIB
  label eksplisit ke user): nilai halaman terhadap 4 heuristik Nielsen,
  5 dari 6 Laws of UX (Hick's, Von Restorff, Serial Position,
  Aesthetic-Usability, Peak-End — Fitts's Law sudah dicek terpisah secara
  measured lewat `target-size` di scope WCAG, jangan diulang di sini), DAN
  2 prinsip tambahan di Rubric Referensi (Progressive Disclosure, Display
  choices as a group) — semuanya berdasarkan KOMBINASI screenshot
  full-page DAN HTML mentah (Langkah 2). Contoh penerapan:
  - "Match real world"/"Recognition over recall": baca teks navigasi/CTA
    yang sebenarnya dari HTML, bukan cuma nebak dari tampilan.
  - "Hick's Law": hitung berapa CTA/tombol dengan bobot visual setara
    dalam satu section (dari `buttons[]`) — kalau ada 3+ tombol primer
    yang bersaing sama kuat di satu section tanpa hierarki jelas, itu
    fatal (user bingung mana yang harus diklik).
  - "Von Restorff Effect": bandingkan `background_color` tombol CTA
    utama dengan `colors.dominant` halaman — kalau warnanya SAMA/mirip
    dengan warna paling dominan (jadi CTA nyaru, gak menonjol) = fatal.
  - "Serial Position Effect": info paling penting (value prop utama)
    HARUS ada di hero (section pertama) — kalau baru muncul di
    tengah/akhir halaman = fatal.
  - "Aesthetic-Usability"/"Peak-End": penilaian lebih kualitatif dari
    screenshot — cuma laporkan kalau pelanggarannya jelas terlihat (mis.
    section terakhir terasa "ditinggal" tanpa CTA/penutup yang layak),
    jangan penilaian estetika bebas di luar konteks 2 hukum ini.
  - "Progressive Disclosure": halaman menampilkan SEMUA detail sekaligus
    (mis. semua FAQ terbuka penuh tanpa accordion, atau hero penuh teks
    panjang) padahal seharusnya bisa diringkas dulu dengan opsi
    "lihat lebih lanjut" = layak dicatat.
  - "Display choices as a group": pilihan sejenis (mis. tier pricing,
    kategori fitur) tersebar gak jelas pengelompokannya (bukan grid/baris
    sejajar) = fatal.

  Hanya laporkan kalau benar-benar ada pelanggaran jelas terhadap salah
  satu poin di atas (fatal-only) — bukan penilaian estetika bebas di luar
  11 poin itu (4 Nielsen + 5 Laws of UX + Progressive Disclosure + Display
  choices as a group; Fitts's Law dihitung terpisah sebagai `target-size`).
  Sebutkan nama heuristik/hukum/prinsip yang dilanggar secara eksplisit di
  judul temuan.
- **Kelengkapan struktur landing page** (`type: "missing-section"`,
  `category: "judgment"`): bandingkan section yang ada dengan 6 section
  di Rubric Referensi "UI/UX — Struktur Landing Page". WAJIB cross-check
  ke HTML mentah dulu (cari heading/teks/class/id yang menyebut
  fitur/harga/FAQ/testimoni secara eksplisit) sebelum menyimpulkan sebuah
  section "hilang" — jangan simpulkan cuma dari jumlah section di JSON
  hasil `extract-styles.py` atau dari screenshot, karena section itu bisa
  saja ada tapi tidak lengkap terdeteksi (lihat catatan resiliensi
  deteksi section di Langkah 2). Fatal-only — cuma laporkan section yang
  BENERAN HILANG (sudah dicek dari HTML, bukan cuma dari data yang parsial)
  dan kehilangannya benar-benar masuk akal jadi masalah:
  - Hero tanpa CTA sama sekali, atau Footer CTA gak ada di section
    terakhir → hampir selalu layak dilaporkan (CTA di awal DAN akhir itu
    pola umum yang kuat).
  - Features/Benefits sama sekali gak ada → layak dilaporkan (jarang ada
    landing page produk tanpa penjelasan fitur/manfaat).
  - Pricing/FAQ hilang → HANYA laporkan kalau dari konteks halaman jelas
    ini jualan produk/jasa berbayar (ada harga disebut di tempat lain,
    kata "beli"/"subscribe"/"upgrade", dll) — kalau halamannya event
    gratis/pendaftaran/portofolio, JANGAN tandai ini sebagai masalah.
  - Social Proof hilang → JANGAN dilaporkan di sini kalau sudah
    dilaporkan sebagai `trust-signal` di scope Konten (hindari duplikat).

#### 3d. Scope "Konten"
**Kalau `brandContext` tersedia:** gunakan `target audience`, `tone of voice`,
dan `nama produk` sebagai konteks saat menilai value prop, trust signal, dan
copy. Contoh: kalau audience "enterprise B2B" tapi copy terlalu casual atau
tidak ada social proof → flag lebih kuat dari tanpa brand context. Sebut
eksplisit di `suggestion` kalau temuan berkaitan dengan brand context.

- **Placeholder ketinggalan** (`category: "measured"`):
  1. Dari HTML mentah, scan SEMUA text node untuk pola berikut (case
     insensitive): "lorem ipsum", "lorem", "TODO", "[placeholder]",
     "your text here", "insert text", "coming soon", "untitled",
     "sample text", "dummy".
  2. Buat list setiap kemunculan: tag HTML + teks aslinya.
  3. Laporkan dengan konteks: "Ditemukan di `<h2>`: 'Lorem ipsum dolor...'
     di section Features."

- **Kejelasan value proposition** (`category: "judgment"`):
  1. Kutip H1 dan kalimat pertama body copy di hero section secara
     verbatim dari HTML.
  2. Terapkan lensa AIDA: apakah H1 menangkap Attention (spesifik,
     benefit-driven, bukan deskripsi kategori umum)? Apakah ada kalimat
     Interest/Desire sebelum CTA?
  3. Terapkan lensa PAS: apakah ada Problem yang diartikulasikan? Agitate
     (kenapa masalah ini menyakitkan)? Solve (solusi konkret)?
  4. Kalau H1-nya generic (hanya nama brand/kategori tanpa benefit
     konkret, mis. "Gaia Digital Marketing Agency" tanpa diferensiasi) =
     fatal.
  5. `suggestion` WAJIB sebutkan: tahap AIDA yang lemah + kutip teks
     aslinya + arah perbaikan spesifik. Contoh: "H1 'Gaia Digital Agency'
     hanya menyebut nama — gagal di tahap Attention karena tidak ada
     benefit. Ubah ke format '[Outcome] untuk [target audience]'."

- **Social proof & urgency** (`category: "judgment"`):
  1. Dari HTML, cari secara aktif: tag `<blockquote>`, class/id dengan
     kata "testimonial"/"review"/"rating"/"client"/"logo"/"trust", elemen
     bintang (★/☆/`fa-star`), angka seperti "4.8/5" atau "500+ clients".
  2. Cari urgency: kata "limited", "only X left", "ends", "deadline",
     countdown timer element.
  3. Kalau halaman jelas jualan produk/jasa/tiket berbayar DAN tidak
     ditemukan SATU PUN elemen dari poin 1 = fatal.
  4. Laporkan: "Tidak ditemukan testimonial, rating, atau logo klien
     di seluruh halaman — meskipun ada X klien disebutkan di portfolio."

**WAJIB**: setiap temuan `category: "judgment"` harus dilabel eksplisit
`(penilaian AI)` di laporan — jangan disamarkan seolah fakta pasti.

### 3.5. Pengayaan sumber (opsional, best-effort — bukan pengganti rubric)

Rubric Referensi di atas TETAP jadi acuan utama & satu-satunya penentu
KRITERIA temuan — langkah ini TIDAK menambah kriteria baru di luar
rubric, cuma memperkuat `suggestion` dengan kutipan/rujukan dari sumber
kredibel biar lebih "berbobot" dan bisa ditelusuri user sendiri.

Untuk finding `category: "judgment"` yang paling signifikan — pilih
2-3 finding **untuk di-search** (jangan search buat setiap finding,
ini best-effort pelengkap bukan wajib menyeluruh; jumlah finding total
boleh lebih dari ini): `WebSearch` dengan query spesifik ke
prinsip yang dilanggar (mis. "Hick's Law too many CTA options landing
page", "value proposition clarity above the fold best practice"), DIBATASI
ke domain kredibel lewat `allowed_domains`:
```
nngroup.com, lawsofux.com, baymard.com, smashingmagazine.com,
uxdesign.cc, interaction-design.org, alistapart.com, uxplanet.org,
webaim.org, uxmatters.com
```

Kalau ketemu artikel/sumber yang relevan & mendukung: tambahkan rujukan
singkat di akhir `suggestion` (mis. "(rujukan: Nielsen Norman Group —
<url>)") dan isi field `sourceRef` di finding (Langkah 4) dengan URL-nya.
Kalau gak ketemu yang relevan, atau search gagal: LEWATI saja, jangan
paksa cari terus atau menunda audit — ini pelengkap opsional, bukan
prasyarat.

**Batasan khusus langkah 3.5 ini** (selain daftar umum di akhir file):
- Memakai hasil pencarian buat MENAMBAH kriteria/jenis temuan baru di
  luar Rubric Referensi yang sudah ada — TIDAK BOLEH
- Mencari dari domain di luar daftar kredibel di atas — TIDAK BOLEH
- Search lebih dari 2-3 kali per audit, atau menahan/menunda laporan
  cuma buat nunggu hasil pencarian tambahan — TIDAK BOLEH

### 3.6. Tentukan `severity` tiap temuan

4 tier, makin tinggi angka makin kritis. Dipakai oleh `/garnish:design-fix`
buat nawarin pilihan "fix semua" vs "fix P0/P1 aja".

- **`severity: "P0"`** — menghalangi user SEPENUHNYA menyelesaikan aksi
  utama atau mengakses konten inti. Fix segera, tidak ada workaround.
  Contoh: CTA tidak ada/tidak bisa diklik, layout rusak total, value
  proposition tidak ada/tidak terbaca sama sekali, WCAG blocking (kontras
  teks utama < 3:1).

- **`severity: "P1"`** — menurunkan pengalaman secara signifikan atau
  melanggar WCAG AA, tapi user masih bisa selesaikan aksi dengan susah
  payah. Fix sebelum launch.
  Contoh: kontras tombol CTA 3:1–4.5:1, alt text hilang di gambar produk
  utama, heading hierarchy loncat di section kritis, target size < 44px
  pada elemen primer.

- **`severity: "P2"`** — friction yang terasa tapi ada workaround / tidak
  blok konversi. Fix di iterasi berikutnya.
  Contoh: inkonsistensi radius tombol, icon tanpa label pada elemen
  sekunder, heading loncat di section non-kritis, target size kecil pada
  elemen tersier.

- **`severity: "P3"`** — nice-to-have, polish. Tidak menghalangi konversi,
  tapi memperbaikinya meningkatkan kesan profesional/kepercayaan.
  Contoh: teks tombol CTA generik ("Submit"), gambar tanpa `loading="lazy"`,
  kurang white-space antarsection, font weight kurang kontras untuk
  subheading, animasi tidak konsisten, tidak ada favicon, meta description
  lemah.

  **Temuan `P3` WAJIB dideteksi dan disimpan ke registry, tapi TIDAK
  ditampilkan di laporan utama** — tersedia lewat pilihan "Lihat temuan
  minor" di Langkah 6.

Kalau ragu antara dua level, pilih yang lebih rendah (angka lebih besar).

### 3.7. Isi field `metric` untuk temuan terukur

Untuk setiap temuan dengan `category: "measured"`, isi field `metric`
dengan angka konkret hasil deteksi. Ini membuat laporan punya data, bukan
hanya narasi.

Mapping `type` → `metric`:

| type | value | unit | threshold | passing |
|------|-------|------|-----------|---------|
| `contrast` | rasio aktual, mis. `2.1` | `"ratio"` | `4.5` (normal text) atau `3.0` (large text ≥18px) | `false` kalau di bawah threshold |
| `target-size` | estimasi px dari CSS, mis. `28` | `"px"` | `44` | `false` kalau < 44 |
| `alt-text` | jumlah `<img>` tanpa alt / total img, mis. `"3/7"` | `"missing/total"` | `"0/total"` | `false` kalau ada yang missing |
| `heading-hierarchy` | jumlah loncat level, mis. `2` (h1→h3 dihitung 1, h2→h4 dihitung 1) | `"jumps"` | `0` | `false` kalau > 0 |
| `icon-label` | jumlah elemen interaktif tanpa label, mis. `4` | `"count"` | `0` | `false` kalau > 0 |
| `consistency` | jumlah variasi berbeda (mis. `"3 nilai border-radius berbeda"`) | `"variants"` | `1` | `false` kalau > 1 |
| `placeholder` | jumlah kemunculan teks placeholder, mis. `2` | `"count"` | `0` | `false` kalau > 0 |
| `line-length` | estimasi karakter per baris, mis. `95` | `"ch"` | `75` | `false` kalau > 80 |
| `cramped-padding` | nilai padding terkecil yang ditemukan, mis. `"4px 8px"` | `"px"` | `"8px 12px"` | `false` kalau di bawah minimum |

Untuk `category: "judgment"` (value-prop, trust-signal, ui-heuristic,
missing-section, cta-position) → `metric: null`.

Untuk `contrast`: kalau rasio aktual tidak bisa dihitung persis dari data
yang tersedia (screenshot-only), tetap isi dengan estimasi terbaik + catat
di `description` bahwa ini estimasi visual, bukan pengukuran CSS.

### 3.8. Hitung Health Score per scope dan overall

**Formula:**
- Base: 100 per scope
- Setiap finding `P0` yang open: −20
- Setiap finding `P1` yang open: −10
- Setiap finding `P2` yang open: −5
- Setiap finding `P3` yang open: −2
- Floor: 0 (tidak bisa negatif)

**Hitung per scope** yang diaudit (konten / ui-ux / komponen / wcag),
lalu hitung **overall** = rata-rata dari semua scope yang diaudit.

Contoh: scope wcag punya 1 temuan P0, 1 P1, 2 P2 →
`100 - 20 - 10 - (2×5) = 60`.

Simpan hasil di entry audit (field `healthScore`, lihat Langkah 4).
Angka ini BUKAN nilai absolut kualitas desain — ini metrik internal
Garnish untuk tracking progres antar-audit. Jangan interpret sebagai
"skor desain 63/100".

### 4. Tulis ke registry

**WAJIB selesaikan 3.7 dan 3.8 dulu sebelum nulis ke sini** — field
`metric` tiap finding dan `healthScore` audit HARUS sudah dihitung,
baru tulis ke registry. Jangan skip dengan alasan apapun.

Buat entry audit baru di `.garnish/registry/audits.json` (ID lanjut dari
yang terakhir, format `A-00X`), dengan tiap temuan dapat ID sendiri
(`F-00X`, lanjut dari terakhir juga — jangan reset per audit):

```json
{
  "id": "A-00X",
  "url": "...",
  "pageType": "landing-page | non-landing-page-forced",
  "scopeAudited": ["konten", "ui-ux", "komponen", "wcag"],
  "screenshotPath": ".garnish/registry/screenshots/A-00X-sections/_full-page.png",
  "capturedAt": "<ISO 8601>",
  "findings": [
    {
      "id": "F-00X",
      "scope": "konten | ui-ux | komponen | wcag",
      "category": "measured | judgment",
      "severity": "P0 | P1 | P2 | P3",
      "type": "contrast | consistency | cta-position | placeholder | layout-rusak | alt-text | heading-hierarchy | target-size | icon-label | line-length | cramped-padding | value-prop | trust-signal | ui-heuristic | missing-section",
      "title": "...",
      "description": "...",
      "suggestion": "...",
      "sourceRef": "URL sumber kredibel (Langkah 3.5) | null — kalau ada",
      "metric": null,
      "wcagLevel": "A | AA | AAA | null — diisi untuk scope:wcag, null untuk scope lain",
      "status": "open",
      "fixedAt": null,
      "designRef": null,
      "affectedAuditIds": ["A-00Y", "A-00Z"]
    }
  ],
  "healthScore": {
    "byScope": {
      "konten": 85,
      "ui-ux": 63,
      "komponen": 78,
      "wcag": 48
    },
    "overall": 69
  },
  "status": "audited"
}
```

`scopeAudited` cuma catat scope yang DIPILIH user di Langkah 1 (bukan
semua scope yang ada) — dipakai kalau nanti user mau audit ulang scope
yang belum pernah dicek.

`affectedAuditIds` (opsional, `null`/tidak ada field ini kalau temuan
spesifik ke 1 halaman) — dipakai KHUSUS untuk temuan yang identik persis
di beberapa halaman dalam antrian audit multi-halaman (Langkah 0.5),
biasanya karena komponen bersama (header/footer/navbar). Tulis temuan
itu SEKALI di audit halaman pertama yang ketemu, isi `affectedAuditIds`
dengan ID audit halaman lain yang punya temuan identik — JANGAN duplikat
entry finding yang sama di tiap audit halaman terdampak.

Append journal:
```json
{"ts":"<ISO 8601>","event":"audit_created","auditId":"A-00X","url":"...","scopeAudited":["ui-ux","wcag"]}
```
Satu baris `finding_detected` per temuan. Kalau audit ini bagian dari
antrian multi-halaman (Langkah 0.5), tambahkan juga sekali di awal:
```json
{"ts":"<ISO 8601>","event":"multi_page_audit_started","pages":["/","/products","/blog"],"skippedBlogPosts":25}
```

### 5. Susun laporan — pendek, prioritized, actionable, dikelompokkan per scope
Format per temuan:
```
[<severity: P0/P1/P2>] [WCAG <level>] <judul singkat> — <scope> — <kategori: terukur/penilaian AI> (F-00X)
                         ↑ hanya tampilkan badge [WCAG X] kalau wcagLevel != null
📏 {metric.value} {metric.unit} (threshold: {metric.threshold})  ← hanya kalau category: "measured"
<1-2 kalimat penjelasan + kenapa ini masalah>
Saran: <isi suggestion> [+ rujukan URL kalau sourceRef terisi]
```
Contoh: `[P1] [WCAG AA] Kontras tombol CTA tidak cukup — wcag — terukur (F-003)`
Urutkan: P0 → P1 → P2. Kelompokkan per scope kalau user pilih lebih dari
satu. Laporkan SEMUA temuan P0, P1, dan P2 — jangan dikurangi. Temuan
`P3` TIDAK ditampilkan di sini (tersedia di pilihan Langkah 6).

**Positive findings — WAJIB disertakan** setelah daftar temuan:

Pilih 2–3 hal konkret yang sudah bagus di halaman ini (struktur, copy,
komponen, konsistensi, dll) dan sebut secara spesifik — bukan pujian
generik. Ini bukan basa-basi: ini penanda apa yang harus dipertahankan
saat fix. Format:

```
━━ YANG SUDAH BAGUS ━━
✓ <hal konkret yang bagus dan kenapa>
✓ <hal konkret lainnya>
```

**WAJIB tampilkan Health Score** (dari `healthScore` yang sudah dihitung di
Langkah 3.8) setelah positive findings:

```
━━ HEALTH SCORE ━━
Konten: {skor}/100  |  UI-UX: {skor}/100  |  Komponen: {skor}/100  |  WCAG: {skor}/100
Overall: {skor}/100
```
Hanya tampilkan scope yang diaudit. Skor 0–40 = 🔴 kritis, 41–70 = 🟡
perlu perhatian, 71–100 = 🟢 baik. Tambahkan emoji sesuai range di
depan tiap angka. Beri catatan pendek: "_(Health Score = metrik Garnish
untuk tracking progres, bukan rating desain absolut)_".

Di akhir laporan, tambahkan satu baris:
> "_(+ {N} temuan minor tersembunyi — pilih opsi 6 untuk lihat)_"
Kalau tidak ada temuan `P3`, skip baris ini.

### 6. Tawarin next step (aktif, tunggu jawaban)

Setelah laporan dan artifact ditampilkan, langsung tawarin pilihan — jangan
tunggu user ngetik duluan:

> "Mau lanjut ngapain?
> 1. **Fix konten** — rewrite copy berdasarkan temuan
> 2. **Fix desain** — rebuild komponen/section fatal pakai design system
> 3. **Keduanya** — konten + desain sekaligus
> 4. **Rebuild** — bikin landing page baru dari nol berdasarkan semua temuan
> 5. **Cukup laporan** — tidak perlu fix sekarang
> 6. **Lihat temuan minor** — tampilkan {N} temuan P3 yang tersembunyi"
> _(Opsi 6 hanya tampil kalau ada temuan `P3` di registry)_

Tunggu jawaban. Proses sesuai pilihan:
- **1 / 2 / 3** → append journal `fix_selected` per temuan terkait, update
  `audits.json` → `status: "in-progress"`, panggil skill yang sesuai.
- **4** → panggil `/garnish:rebuild` dengan ID audit ini.
- **5** → konfirmasi singkat: "Oke, temuan tersimpan. Bisa re-audit kapanpun
  setelah ada update dengan bilang 're-audit [url]'." Selesai.
- **6** → tampilkan semua temuan `severity: "P3"` dari audit ini dalam
  format yang sama (badge MINOR hijau/teal, urutan per scope), lalu tawarin
  pilihan 1-5 lagi (tanpa opsi 6). Temuan minor ini bisa juga di-fix lewat
  `design-fix` atau `content-fix` — user cukup bilang "fix yang minor juga"
  setelah lihat daftarnya.

### 7. Generate Laporan HTML — gaya deliverable UX agency (otomatis, jalan SEBELUM tunggu jawaban Langkah 6)

Segera setelah laporan teks ditampilkan di Langkah 5 (SEBELUM menunggu
jawaban user di Langkah 6), buat SATU file HTML report yang bisa
dijadikan deliverable ke klien. Baca data dari entry audit yang baru
saja ditulis ke `.garnish/registry/audits.json` (bukan dari memori —
baca file-nya). **Kalau ini bagian dari antrian multi-halaman (Langkah
0.5), buat SATU laporan gabungan untuk SEMUA halaman dalam antrian itu**
— bukan satu file terpisah per halaman.

**Cara tampil — buka langsung, bukan cuma link.** Tulis file HTML ke
direktori scratchpad lokal, lalu jalankan `open <path>` (macOS) supaya
langsung terbuka di browser default user tanpa perlu klik link. Kalau
user secara eksplisit minta versi yang bisa di-share/di-hosting (bukan
cuma review lokal), baru pakai Artifact tool sebagai tambahan — bukan
pengganti `open`.

**Struktur laporan — gaya deliverable UX agency, bukan dump temuan
mentah:**

1. **Cover** — judul situs yang diaudit, tanggal, cakupan halaman
   ("N halaman dari M URL total di sitemap" kalau multi-halaman), scope
   yang diaudit, "Disusun oleh Garnish".
2. **Ringkasan Eksekutif** — satu paragraf simpulan kondisi keseluruhan
   (jujur, bukan generik — sebutkan angka konkret: total temuan,
   breakdown severity, ada/tidaknya bug sistemik lintas-halaman) + row
   stat card (jumlah halaman diaudit, total temuan, P0 count, P1 count,
   P2 count).
3. **Health Score** — baris card score dengan warna:
   - Overall score ditampilkan besar di tengah, warna background
     sesuai range: merah (#FEE2E2) kalau 0–40, amber (#FEF3C7) kalau
     41–70, hijau (#D1FAE5) kalau 71–100
   - Baris kecil di bawahnya: per-scope score (`Konten: 85 | UI-UX: 63 |
     ...`) hanya untuk scope yang diaudit
   - Label kecil: "Health Score — metrik progres Garnish, bukan rating
     desain absolut"
4. **Scorecard per kategori** — satu baris per scope yang diaudit
   (Konten/UI-UX/Komponen/WCAG), status "Bersih" (hijau) atau "N temuan"
   (amber/merah tergantung severity tertinggi di scope itu).
5. **Positive findings** — card terpisah dengan background hijau muda,
   judul "Yang Sudah Bagus ✓", list 2–3 hal konkret yang sudah benar.
   Wajib ada, bukan opsional.
4. **Rekomendasi Prioritas** — daftar bernomor, diurutkan berdasarkan
   DAMPAK bukan cuma severity: temuan yang berasal dari komponen bersama
   (berdampak ke banyak halaman sekaligus lewat `affectedAuditIds`) naik
   ke atas karena satu perbaikan menyelesaikan banyak temuan sekaligus.
   Tiap item: judul aksi konkret, 1-2 kalimat kenapa, referensi ID
   temuan.
5. **Detail Temuan & Observasi**, dikelompokkan per halaman (atau per
   "Komponen Bersama" untuk temuan lintas-halaman — tampilkan SEKALI di
   sini dengan daftar semua halaman terdampak, JANGAN diulang per
   halaman). Untuk tiap temuan `severity: "sedang"` atau `"tinggi"`:
   - Badge severity + badge WCAG level (kalau `wcagLevel` != null, mis. pill biru/navy "WCAG A" / "WCAG AA" / "WCAG AAA") + kategori (terukur/penilaian AI) + ID temuan
   - Judul, deskripsi, saran (`suggestion`)
   - Kalau `metric` tidak null, tampilkan bar kecil "📏 {value} {unit}
     (threshold: {threshold})" dengan warna merah kalau `passing: false`
   - **Screenshot di-crop ketat ke area yang relevan** (pakai
     `screenshot_path` per section dari hasil `extract-styles.py`, crop
     lebih jauh kalau perlu) — JANGAN tempel screenshot full-page atau
     full-section utuh kalau cuma sebagian kecil yang jadi masalah.
   - Temuan `severity: "P3"` TIDAK ditampilkan di sini (tetap
     ikuti Langkah 6 soal itu).
   - **Kalau satu halaman NOL temuan `tinggi`/`sedang`** (bersih):
     JANGAN tampilkan galeri screenshot per section. Sebagai gantinya,
     pilih SATU crop yang menonjolkan praktik desain/konten yang bagus
     di halaman itu (positive highlight — beri badge "Praktik Baik" dan
     1-2 kalimat kenapa ini contoh yang layak dipertahankan/dicontoh).

**Print CSS — tetap WAJIB** disertakan di `<style>` (sama seperti
sebelumnya) supaya laporan bisa di-print / Save as PDF dari browser
dengan layout benar (warna badge tetap muncul, card tidak terpotong di
tengah halaman):
```css
@media print {
  body { margin: 0; }
  .header, .cover { -webkit-print-color-adjust: exact; print-color-adjust: exact; }
  .stat-cards, .stat-row { display: flex !important; }
  .finding-card, .finding, .positive, .priority-item { break-inside: avoid; page-break-inside: avoid; box-shadow: none; }
  .badge, .pill { -webkit-print-color-adjust: exact; print-color-adjust: exact; }
  @page { margin: 1.5cm 2cm; }
}
```

**Visual style dasar** (bisa dikembangkan, minimal harus ada):
- Font system-ui/sans-serif, cover pakai background gelap kontras teks
  putih, body report background terang
- Card per temuan: border-left tebal sesuai severity (merah=tinggi,
  amber=sedang), border-left hijau khusus untuk card "Praktik Baik"
- Badge inline pill: TINGGI=merah, SEDANG=amber, TERUKUR=biru,
  PENILAIAN AI=abu-abu, PRAKTIK BAIK=hijau
- Footer: "Disusun oleh Garnish · garnish-plugin"
- Responsive (max-width ~880px, centered)

**Tidak perlu:**
- Tombol aksi / navigasi interaktif di dalam laporan
- Data apapun di luar temuan audit dalam antrian ini (jangan include
  data dari audit lama/tidak terkait)

Setelah laporan terbuka, baru tampilkan pertanyaan Langkah 6 di pesan
yang sama (atau pesan berikutnya).

## Yang TIDAK boleh dilakukan skill ini
- Membuat laporan audit lengkap/checklist panjang — fokus fatal-only per
  scope yang dipilih
- Mengandalkan web search (Langkah 3.5) sebagai sumber kriteria — rubric
  tertanam TETAP satu-satunya penentu apa yang dianggap temuan
- Menjalankan modul deteksi di luar scope yang dipilih user di Langkah 1
- Melabel penilaian judgment (value prop, trust signal, evaluasi
  heuristik) sebagai fakta pasti
- Menilai UI/UX di luar 4 heuristik Nielsen, 3 prinsip Gestalt, 6 Laws
  of UX, Progressive Disclosure, dan Display-choices-as-a-group yang ada
  di Rubric Referensi — kalau ada masalah lain di luar itu, bukan bagian
  dari deteksi terstandar skill ini
- Melaporkan 5 teknik cognitive-load yang sudah tercakup prinsip lain
  (avoid unnecessary elements, leverage common patterns, eliminate
  unnecessary tasks, minimize choices, readability) sebagai temuan
  terpisah/dobel dari prinsip yang sudah mewakilinya
- Menilai kejelasan konten di luar lensa AIDA/PAS yang ada di Rubric
  Referensi, atau membuat `type`/finding terpisah untuk AIDA/PAS (itu
  lensa analisis buat `value-prop`, bukan finding baru)
- Menyertakan estimasi angka/persentase dampak (mis. "naikin konversi
  X%") di `suggestion` — klaim itu tidak bisa diverifikasi, jaga tetap
  kualitatif
- Lanjut ke fix konten/desain tanpa checkpoint eksplisit
- Melakukan lebih dari 1 ekstraksi/deteksi (Langkah 2-4) dalam satu
  entry audit `A-00X` — tiap URL tetap harus jadi entry audit sendiri
  walau dikerjakan berurutan dalam satu sesi multi-halaman (Langkah 0.5)
- Audit artikel blog individual satu-satu (`/blog/judul-artikel`) kalau
  index/listing-nya (`/blog`) sudah diaudit — cukup index-nya, kecuali
  user eksplisit minta artikel tertentu
- Mensimulasikan user flow/navigasi ANTAR halaman (klik tombol A pindah
  ke halaman B, menilai transisinya) — ini beda dari sekadar audit tiap
  halaman terpisah, tetap di luar scope
- Menampilkan screenshot full-page/full-section utuh di laporan untuk
  temuan yang cuma butuh crop area kecil, atau menampilkan galeri
  screenshot section-by-section untuk halaman yang nol temuan fatal
  (harusnya pilih satu positive-highlight crop, bukan wall of screenshots)
- Menduplikasi temuan yang sama persis (biasanya bug komponen bersama)
  sebagai entry terpisah di tiap halaman — harus dikonsolidasi lewat
  `affectedAuditIds`
- Mengklaim deteksi kontras/konsistensi/posisi-CTA/layout-rusak kalau
  `extract-styles.py` gagal jalan dan datanya tidak ada
- Menimpa `.garnish/registry/` yang sudah ada saat auto-setup Langkah 0
  (auto-setup HANYA jalan kalau registry belum ada sama sekali)
- Melanjutkan audit ke halaman yang JELAS bukan landing page (dashboard/
  aplikasi) tanpa warning & konfirmasi eksplisit dari Langkah 2.5
- Menandai `missing-section` fatal untuk Pricing/FAQ di halaman yang
  jelas bukan jualan produk/jasa berbayar, atau melaporkan Social Proof
  dua kali (sebagai `trust-signal` DAN `missing-section`)
- Menandai `missing-section` HANYA dari data `extract-styles.py`/screenshot
  tanpa cross-check ke HTML mentah dulu
- Menolak/menghentikan audit total hanya karena deteksi section dari
  `extract-styles.py` tidak lengkap — tetap audit pakai data yang ada,
  sebutkan keterbatasannya di laporan
- Menyimpulkan penilaian judgment (heuristik, missing-section, kejelasan
  konten) HANYA dari screenshot tanpa merujuk HTML mentah yang sudah
  di-fetch di Langkah 2

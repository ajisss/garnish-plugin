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

### 0. Cek registry (auto-setup, bukan langkah terpisah)
Kalau `.garnish/registry/audits.json` belum ada, setup sendiri di sini —
JANGAN minta user jalankan skill lain dulu:
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

Kalau `.garnish/registry/audits.json` SUDAH ada, langsung lanjut tanpa
basa-basi setup.

Cek juga apakah URL yang sama pernah diaudit sebelumnya (`audits.json`).
Kalau ada dan statusnya `"resolved"` atau `"in-progress"`, informasikan ke
user: "URL ini pernah diaudit tanggal X, ada Y temuan, status Z" — dan
tanya apakah mau audit ulang dari nol atau lihat status yang lama dulu.

### 1. Checkpoint — pilih scope audit (HARD STOP)
Sebelum ekstraksi/deteksi apapun, tanya:
> "Mau audit apa? Bisa pilih lebih dari satu — Konten, UI/UX, Komponen,
> WCAG, atau Semua."

Tunggu jawaban eksplisit. JANGAN asumsikan "Semua" sebagai default kalau
user belum jawab. Simpan pilihan ini (dipakai Langkah 3 buat nentuin modul
deteksi mana yang jalan) dan catat di entry audit (`scopeAudited`,
Langkah 4).

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
- **Kontras teks**: JSON tidak punya field `background_color` di level
  section (`aggregate_extraction()` cuma keluarin `index`, `bbox`,
  `typography`, `buttons`, `containers` per section) — background harus
  diambil dari screenshot, bukan JSON. Untuk tiap `typography[]` entry:
  pakai `color`-nya (exact) sebagai warna teks, buka crop
  `sections[].screenshot_path` section yang bersangkutan, sample warna
  background di belakang/dekat teks itu secara visual. Hitung rasio
  kontras WCAG. Di bawah 4.5:1 untuk teks normal (atau 3:1 untuk teks
  besar ≥24px/18.5px bold) = fatal.

  **Default kategori untuk kontras teks (bukan tombol) adalah
  `category: "judgment"`** — karena background-nya estimasi visual, bukan
  field JSON exact. Boleh naik jadi `measured` HANYA kalau background di
  crop jelas tidak ambigu (satu warna solid flat penuh, tanpa
  gradient/gambar/pattern). Ragu → pilih `judgment`.

  Untuk tombol, kontras tetap `category: "measured"` tanpa pengecualian —
  `buttons[].color` vs `buttons[].background_color` sama-sama field JSON
  exact.
- **Alt text** (`type: "alt-text"`): dari HTML mentah (Langkah 2), cari
  semua tag `<img>` — kalau tidak ada atribut `alt` sama sekali, atau
  `alt=""` pada gambar yang jelas bukan dekoratif (bukan spacer/ikon
  kosong), tandai fatal. Batasan: skill ini tidak menilai apakah teks alt
  yang ADA sudah deskriptif dengan baik — cuma cek keberadaannya.
- **Heading hierarchy** (`type: "heading-hierarchy"`): susun urutan
  heading dari `typography[].level` di semua section (urut by index
  section, lalu urut dalam array) — CATATAN: `extract-styles.py` cuma
  menangkap maksimal 3 heading per section, jadi ini deteksi parsial,
  bukan lengkap semua heading di halaman. Telusuri urutan level: kalau
  suatu heading levelnya lebih dari 1 tingkat di atas level terdalam yang
  sudah ditemukan sejauh ini (mis. baru ketemu h1, lalu langsung ketemu
  h3 tanpa h2 di antaranya) = fatal (loncat level).
- **Target size** (`type: "target-size"`, rujuk Fitts's Law + WCAG 2.5.5
  di Rubric Referensi): dari `buttons[].padding` (format CSS, mis.
  "12px 24px" = vertikal 12px, horizontal 24px), estimasi tinggi area
  klik efektif = 2×padding-vertikal + tinggi font teks tombol (ambil dari
  `typography[]` terkait kalau ada, atau asumsikan ~16px kalau tidak
  diketahui). Kalau hasil estimasi jelas jauh di bawah ~44px (mis. di
  bawah ~32px) = fatal. Ini estimasi kasar dari CSS, bukan pengukuran
  piksel layar sungguhan — tetap `category: "measured"` karena
  angka-angkanya dari field JSON exact, bukan tebakan visual.
- **Icon tanpa label** (`type: "icon-label"`, rujuk WCAG 4.1.2 + cognitive
  load di Rubric Referensi): dari HTML mentah (Langkah 2), cari elemen
  `<button>`/`<a>` yang isinya HANYA `<svg>`/elemen font-icon (mis. class
  `icon-*`/`fa-*`) TANPA teks visible di dalamnya DAN tanpa atribut
  `aria-label`/`aria-labelledby`/`title` — tandai fatal. Kalau ada teks
  visible (walau kecil) atau `aria-label` terisi, JANGAN tandai fatal.

#### 3b. Scope "Komponen" (`category: "measured"`)
- **Konsistensi komponen**: bandingkan `buttons[]` dan `containers[]` di
  seluruh section — kalau `border_radius`/`padding` (field yang ada di
  keduanya) beda-beda tanpa pola jelas, atau `background_color` pada
  `buttons[]` beda-beda tanpa pola (`containers[]` tidak punya field
  warna) — mis. 3 tombol dengan radius 4px, 8px, 16px tanpa alasan
  hierarki visual = fatal. Rujuk ke prinsip Atomic Design di Rubric
  Referensi saat menyusun `suggestion`.

#### 3c. Scope "UI/UX"
- **Posisi CTA** (`category: "measured"`): cek apakah section pertama
  (index 0, biasanya hero) punya `bbox.height` yang membuatnya (atau
  tombol di dalamnya) berada dalam viewport ~900px pertama. Kalau hero
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
- **Placeholder ketinggalan** (`category: "measured"`): dari HTML mentah
  (Langkah 2), cari pola teks "lorem ipsum", "TODO", "[placeholder]",
  "your text here", "lorem", dll.
- **Kejelasan value proposition & CTA copy** (`category: "judgment"`):
  nilai apakah headline & CTA copy jelas dalam beberapa detik pertama,
  merujuk prinsip "Clarity" di Rubric Referensi, DIANALISIS pakai lensa
  AIDA dan/atau PAS (Rubric Referensi "Konten — Lensa Copywriting") —
  `suggestion` WAJIB sebutkan tahap AIDA yang lemah (Attention/Interest/
  Desire/Action) atau elemen PAS yang hilang (Problem/Agitate/Solve),
  bukan cuma bilang "kurang jelas" secara umum. Tetap konkret soal arah
  perbaikan, tapi rewrite beneran tetap tugas `/garnish:content-fix`,
  bukan skill ini.
- **Social proof & urgency** (`category: "judgment"`): cek ada/tidaknya
  testimoni/rating/logo klien/elemen urgency — cuma jadi temuan fatal
  kalau halamannya jelas jualan sesuatu (produk/jasa/pendaftaran/tiket)
  dan sama sekali tidak ada elemen ini, merujuk prinsip "Social proof &
  urgency".

**WAJIB**: setiap temuan `category: "judgment"` harus dilabel eksplisit
`(penilaian AI)` di laporan — jangan disamarkan seolah fakta pasti.

### 3.5. Pengayaan sumber (opsional, best-effort — bukan pengganti rubric)

Rubric Referensi di atas TETAP jadi acuan utama & satu-satunya penentu
KRITERIA temuan — langkah ini TIDAK menambah kriteria baru di luar
rubric, cuma memperkuat `suggestion` dengan kutipan/rujukan dari sumber
kredibel biar lebih "berbobot" dan bisa ditelusuri user sendiri.

Untuk finding `category: "judgment"` yang paling signifikan (maksimal 2-3
finding per audit — jangan search buat setiap finding, ini best-effort
pelengkap bukan wajib menyeluruh): `WebSearch` dengan query spesifik ke
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

### 4. Tulis ke registry
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
      "type": "contrast | consistency | cta-position | placeholder | layout-rusak | alt-text | heading-hierarchy | target-size | icon-label | value-prop | trust-signal | ui-heuristic | missing-section",
      "title": "...",
      "description": "...",
      "suggestion": "...",
      "sourceRef": "URL sumber kredibel (Langkah 3.5) | null — kalau ada",
      "status": "open",
      "fixedAt": null,
      "designRef": null
    }
  ],
  "status": "audited"
}
```

`scopeAudited` cuma catat scope yang DIPILIH user di Langkah 1 (bukan
semua scope yang ada) — dipakai kalau nanti user mau audit ulang scope
yang belum pernah dicek.

Append journal:
```json
{"ts":"<ISO 8601>","event":"audit_created","auditId":"A-00X","url":"...","scopeAudited":["ui-ux","wcag"]}
```
Satu baris `finding_detected` per temuan.

### 5. Susun laporan — pendek, prioritized, actionable, dikelompokkan per scope
Format per temuan:
```
[FATAL] <judul singkat> — <scope> — <kategori: terukur/penilaian AI> (F-00X)
<1-2 kalimat penjelasan + kenapa ini masalah>
Saran: <isi suggestion> [+ rujukan URL kalau sourceRef terisi]
```
Kelompokkan per scope kalau user pilih lebih dari satu. Maksimal tampilkan
gejala yang benar-benar fatal per scope — jangan generate daftar panjang.
Kalau cuma nemu 2-3 masalah per scope, itu cukup (jangan dipaksa nambah
biar keliatan "lengkap").

### 6. Checkpoint — HARD STOP
Setelah laporan ditampilkan, tanya:
> "Dari temuan ini, mau dibenerin yang mana — kontennya, desainnya,
> keduanya, atau mau saya bikinin landing page baru berdasarkan semua
> temuan ini (full rebuild, bukan cuma bagian yang fatal)? Atau mau
> lihat detail salah satu temuan dulu?"

Tunggu jawaban eksplisit. JANGAN lanjut ke fix/rebuild apapun tanpa
konfirmasi ini.

- Kalau user pilih konten/desain/keduanya → append journal `fix_selected`
  per temuan yang dipilih, update `audits.json` → `status: "in-progress"`,
  lanjut ke `/garnish:content-fix` dan/atau `/garnish:design-fix`.
- Kalau user pilih **bikin landing page baru** → JANGAN proses lewat
  `content-fix`/`design-fix`. Panggil skill `/garnish:rebuild` sebagai
  gantinya, sertakan ID audit ini sebagai konteksnya (rebuild baca ulang
  SEMUA temuan dari audit yang sama, bukan cuma yang user "pilih" di sini
  — jadi tidak perlu append `fix_selected` per finding untuk jalur ini).

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
- Audit lebih dari 1 halaman dalam satu run
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

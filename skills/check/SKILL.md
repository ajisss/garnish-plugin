---
name: check
description: Audit sebuah landing page atau screen aplikasi (URL apapun) untuk mendeteksi gejala fatal desain & konten — bukan checklist lengkap, hanya masalah yang benar-benar mengganggu, digroundkan ke framework UI/UX (Nielsen heuristics, Gestalt, atomic design, WCAG, prinsip CRO) biar konsisten tiap audit, bukan penilaian bebas. Gunakan skill ini ketika user minta "audit", "cek", "garnish", atau kasih URL dan minta dievaluasi. Menanyakan scope audit dulu (konten/UI-UX/komponen/WCAG, bisa pilih beberapa) sebelum mulai. Tiap temuan disertai saran perbaikan konkret. Skill ini otomatis setup registry (.garnish/registry/) sendiri kalau belum ada. Setelah temuan ditampilkan, WAJIB berhenti dan tanya user mau fix konten/desain/keduanya sebelum lanjut ke perbaikan apapun.
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

### Komponen — Atomic Design
- Komponen sejenis (button, card, input) harus konsisten variant &
  state-nya di seluruh halaman — bukan di-reinvent beda-beda tiap section
  tanpa alasan hierarki visual yang jelas.

### WCAG (subset kriteria AA yang paling gampang diverifikasi objektif — BUKAN audit aksesibilitas lengkap)
- Kontras teks vs background (rasio WCAG).
- Alt text pada gambar non-dekoratif.
- Urutan heading semantik tidak boleh loncat level (h1→h2→h3, TIDAK
  boleh h1 langsung ke h3).

### Konten — Prinsip CRO (Conversion Rate Optimization) dasar
- **Clarity**: value proposition & CTA harus bisa dipahami dalam beberapa
  detik pertama, tanpa perlu mikir.
- **Reduce friction**: makin sedikit langkah/input yang diminta sebelum
  konversi (isi form, klik CTA), makin baik.
- **Social proof & urgency**: testimoni/rating/jumlah user, atau elemen
  urgency (limited time/stock) umumnya menaikkan konversi — bukan wajib
  ada di semua jenis halaman, tapi ketidakhadirannya di halaman yang
  jualan sesuatu (produk/jasa/pendaftaran) layak dicatat.

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

Ambil juga HTML mentah halaman (`WebFetch <url>`) — dipakai buat deteksi
placeholder, alt text, dan heading hierarchy (Langkah 3), karena
`extract-styles.py` tidak mengembalikan teks konten atau atribut HTML,
cuma computed style.

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
- **Evaluasi heuristik** (`category: "judgment"`, WAJIB label eksplisit
  ke user): nilai halaman terhadap 4 heuristik Nielsen di Rubric Referensi
  (match real world, consistency & standards, recognition over recall,
  aesthetic & minimalist) berdasarkan screenshot full-page dan struktur
  section yang ada. Hanya laporkan kalau benar-benar ada pelanggaran
  jelas terhadap salah satu heuristik itu (fatal-only) — bukan penilaian
  estetika bebas di luar ke-4 poin itu. Sebutkan heuristik mana yang
  dilanggar secara eksplisit di judul temuan.

#### 3d. Scope "Konten"
- **Placeholder ketinggalan** (`category: "measured"`): dari HTML mentah
  (Langkah 2), cari pola teks "lorem ipsum", "TODO", "[placeholder]",
  "your text here", "lorem", dll.
- **Kejelasan value proposition & CTA copy** (`category: "judgment"`):
  nilai apakah headline & CTA copy jelas dalam beberapa detik pertama,
  merujuk prinsip "Clarity" di Rubric Referensi. `suggestion` harus
  konkret (mis. arah rewrite seperti apa), tapi rewrite beneran tetap
  tugas `/garnish:content-fix`, bukan skill ini.
- **Social proof & urgency** (`category: "judgment"`): cek ada/tidaknya
  testimoni/rating/logo klien/elemen urgency — cuma jadi temuan fatal
  kalau halamannya jelas jualan sesuatu (produk/jasa/pendaftaran/tiket)
  dan sama sekali tidak ada elemen ini, merujuk prinsip "Social proof &
  urgency".

**WAJIB**: setiap temuan `category: "judgment"` harus dilabel eksplisit
`(penilaian AI)` di laporan — jangan disamarkan seolah fakta pasti.

### 4. Tulis ke registry
Buat entry audit baru di `.garnish/registry/audits.json` (ID lanjut dari
yang terakhir, format `A-00X`), dengan tiap temuan dapat ID sendiri
(`F-00X`, lanjut dari terakhir juga — jangan reset per audit):

```json
{
  "id": "A-00X",
  "url": "...",
  "scopeAudited": ["konten", "ui-ux", "komponen", "wcag"],
  "screenshotPath": ".garnish/registry/screenshots/A-00X-sections/_full-page.png",
  "capturedAt": "<ISO 8601>",
  "findings": [
    {
      "id": "F-00X",
      "scope": "konten | ui-ux | komponen | wcag",
      "category": "measured | judgment",
      "type": "contrast | consistency | cta-position | placeholder | layout-rusak | alt-text | heading-hierarchy | value-prop | trust-signal | ui-heuristic",
      "title": "...",
      "description": "...",
      "suggestion": "...",
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
Saran: <isi suggestion>
```
Kelompokkan per scope kalau user pilih lebih dari satu. Maksimal tampilkan
gejala yang benar-benar fatal per scope — jangan generate daftar panjang.
Kalau cuma nemu 2-3 masalah per scope, itu cukup (jangan dipaksa nambah
biar keliatan "lengkap").

### 6. Checkpoint — HARD STOP
Setelah laporan ditampilkan, tanya:
> "Dari temuan ini, mau dibenerin yang mana — kontennya, desainnya, atau
> dua-duanya? Atau mau lihat detail salah satu temuan dulu?"

Tunggu jawaban eksplisit. JANGAN lanjut ke fix apapun tanpa konfirmasi ini.
Setelah user pilih, append journal `fix_selected` per temuan yang dipilih
dan update `audits.json` → `status: "in-progress"`.

## Yang TIDAK boleh dilakukan skill ini
- Membuat laporan audit lengkap/checklist panjang — fokus fatal-only per
  scope yang dipilih
- Menjalankan modul deteksi di luar scope yang dipilih user di Langkah 1
- Melabel penilaian judgment (value prop, trust signal, evaluasi
  heuristik) sebagai fakta pasti
- Menilai UI/UX di luar 4 heuristik Nielsen dan 3 prinsip Gestalt yang
  ada di Rubric Referensi — kalau ada masalah lain di luar itu, bukan
  bagian dari deteksi terstandar skill ini
- Menyertakan estimasi angka/persentase dampak (mis. "naikin konversi
  X%") di `suggestion` — klaim itu tidak bisa diverifikasi, jaga tetap
  kualitatif
- Lanjut ke fix konten/desain tanpa checkpoint eksplisit
- Audit lebih dari 1 halaman dalam satu run
- Mengklaim deteksi kontras/konsistensi/posisi-CTA/layout-rusak kalau
  `extract-styles.py` gagal jalan dan datanya tidak ada
- Menimpa `.garnish/registry/` yang sudah ada saat auto-setup Langkah 0
  (auto-setup HANYA jalan kalau registry belum ada sama sekali)

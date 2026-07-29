---
name: check
description: Audit sebuah landing page atau screen aplikasi (URL apapun) untuk mendeteksi gejala fatal desain & konten — bukan checklist lengkap, hanya masalah yang benar-benar mengganggu. Gunakan skill ini ketika user minta "audit", "cek", "garnish", atau kasih URL dan minta dievaluasi. Skill ini otomatis setup registry (.garnish/registry/) sendiri kalau belum ada — user tidak perlu jalankan skill setup terpisah. Hasil temuan disimpan ke registry dengan ID stabil. Setelah temuan ditampilkan, WAJIB berhenti dan tanya user mau fix konten/desain/keduanya sebelum lanjut ke perbaikan apapun.
---

# /garnish:check — Audit Gejala Fatal

Baca `CONTEXT.md` dulu kalau belum — ada tabel lengkap gejala fatal yang
harus dideteksi dan mana yang terukur vs judgment.

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

### 1. Ekstrak halaman (reuse tool dari `design-agent`, jangan tulis ulang)
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
`extract-styles.py` tidak ditemukan, dan langsung lompat ke perilaku
fallback yang sama seperti kalau `extract-styles.py` gagal jalan
(dijelaskan di bawah) — lanjutkan HANYA dengan deteksi yang tidak butuh
computed CSS, lalu skip ke Langkah 2.

Kalau `$EXTRACT_STYLES` ketemu (blok `if` di atas jalan), lanjut baca hasilnya:

Ini menghasilkan JSON terstruktur (warna dominan, tipografi per section,
tombol, container, bbox per section) DAN screenshot full-page di
`.garnish/registry/screenshots/<audit-id>-sections/_full-page.png` — pakai
file itu sebagai `screenshotPath` di registry (Langkah 4), jangan cari
script screenshot terpisah.

Catatan: JSON hasilnya juga punya `sections[].screenshot_path`, tapi itu
path screenshot CROP per section (`section-<index>.png`), bukan full-page
— jangan tertukar dengan `_full-page.png` di atas.

Kalau `extract-styles.py` gagal jalan (exit code 1 — situs block bot,
butuh login, atau Playwright/Pillow belum terinstall) ATAU kalau tadi
`$EXTRACT_STYLES` kosong: beri tahu user terus terang, dan lanjutkan
HANYA dengan deteksi yang tidak butuh computed CSS (placeholder text
dari Langkah 2 bagian terakhir, dan judgment dari Langkah 3 berdasar
screenshot manual/deskripsi user) — jangan klaim deteksi
kontras/konsistensi/posisi-CTA kalau datanya tidak ada.

Ambil juga HTML mentah halaman (`WebFetch <url>`) untuk deteksi placeholder
teks di Langkah 2 — ini terpisah dari `extract-styles.py` karena tool itu
tidak mengembalikan teks konten, cuma computed style.

### 2. Deteksi gejala TERUKUR dulu (prioritas utama)
Dari JSON hasil `extract-styles.py`:
- **Kontras teks**: JSON tidak punya field `background_color` di level
  section (`aggregate_extraction()` cuma keluarin `index`, `bbox`,
  `typography`, `buttons`, `containers` per section) — jadi background
  harus diambil dari screenshot, bukan JSON. Untuk tiap `typography[]`
  entry: pakai `color`-nya (exact, dari computed CSS) sebagai warna teks,
  lalu buka crop `sections[].screenshot_path` section yang bersangkutan
  dan sample warna background yang terlihat langsung di belakang/dekat
  teks itu secara visual. Hitung rasio kontras WCAG dari dua warna itu. Di
  bawah 4.5:1 untuk teks normal (atau 3:1 untuk teks besar ≥24px/18.5px
  bold) = fatal.

  **Default kategori untuk temuan ini adalah `category: "judgment"`** —
  karena warna backgroundnya hasil estimasi visual dari screenshot, bukan
  field JSON exact, walaupun rumus WCAG-nya sendiri presisi. Boleh naik
  jadi `category: "measured"` HANYA kalau background di crop itu jelas
  tidak ambigu — mis. section itu jelas-jelas satu warna solid flat yang
  keliatan penuh di belakang teks, tanpa gradient/gambar/pattern yang
  ikut menutupi area itu. Kalau ragu antara dua kategori ini, pilih
  `judgment` — jangan default ke `measured` hanya karena rumus
  kontrasnya matematis.

  Ini KHUSUS untuk teks heading/body. Untuk tombol, kontrasnya tetap
  `category: "measured"` tanpa pengecualian — dua warnanya sama-sama
  field JSON exact: `buttons[].color` vs `buttons[].background_color`,
  tidak ada estimasi visual sama sekali.
- **Konsistensi komponen**: bandingkan `buttons[]` dan `containers[]` di
  seluruh section — kalau `border_radius`/`padding` (field yang ada di
  keduanya) beda-beda tanpa pola jelas, atau `background_color` pada
  `buttons[]` beda-beda tanpa pola (`containers[]` tidak punya field
  warna, cuma `border_radius`/`padding`/`box_shadow`/`gap`) — mis. 3
  tombol dengan radius 4px, 8px, 16px tanpa alasan hierarki visual = fatal.
- **Posisi CTA**: cek apakah section pertama (index 0, biasanya hero)
  punya `bbox.height` yang membuatnya (atau tombol di dalamnya) berada
  dalam viewport ~900px pertama. Kalau hero section sudah lebih tinggi
  dari 900px DAN tidak ada tombol di dalamnya = fatal (CTA kemungkinan di
  bawah fold).
- **Placeholder ketinggalan**: dari HTML mentah (Langkah 1), cari pola
  teks "lorem ipsum", "TODO", "[placeholder]", "your text here", "lorem",
  dll.
- **Layout rusak**: dari `sections[].bbox` (sudah terurut by index), untuk
  tiap pasangan section berurutan (`section[i]`, `section[i+1]`): kalau
  `section[i].bbox.y + section[i].bbox.height` LEBIH BESAR dari
  `section[i+1].bbox.y`, berarti kedua section itu overlap secara
  vertikal = fatal (section saling menimpa). Kalau `bbox.width` sebuah
  section jauh lebih besar dari viewport yang dipakai `extract-styles.py`
  (1440px default) = indikasi overflow horizontal = fatal. Ini
  `category: "measured"` (murni perbandingan angka bbox, tidak ada
  estimasi visual).

### 3. Deteksi gejala JUDGMENT (pelengkap, label eksplisit)
- Kejelasan value proposition di headline
- Ada/tidaknya trust signal (testimoni, logo klien, rating, dll)

**WAJIB**: setiap temuan dari kategori ini harus dilabel eksplisit
`(penilaian AI)` di laporan — jangan disamarkan seolah fakta pasti kayak
temuan terukur.

### 4. Tulis ke registry
Buat entry audit baru di `.garnish/registry/audits.json` (ID lanjut dari
yang terakhir, format `A-00X`), dengan tiap temuan dapat ID sendiri
(`F-00X`, lanjut dari terakhir juga — jangan reset per audit):

```json
{
  "id": "A-00X",
  "url": "...",
  "screenshotPath": ".garnish/registry/screenshots/A-00X-sections/_full-page.png",
  "capturedAt": "<ISO 8601>",
  "findings": [
    {
      "id": "F-00X",
      "category": "measured | judgment",
      "type": "contrast | consistency | cta-position | placeholder | layout-rusak | value-prop | trust-signal",
      "title": "...",
      "description": "...",
      "status": "open",
      "fixedAt": null,
      "designRef": null
    }
  ],
  "status": "audited"
}
```

Append journal:
```json
{"ts":"<ISO 8601>","event":"audit_created","auditId":"A-00X","url":"..."}
```
Satu baris `finding_detected` per temuan.

### 5. Susun laporan — pendek, prioritized, actionable
Format per temuan:
```
[FATAL] <judul singkat> — <kategori: terukur/penilaian AI> (F-00X)
<1-2 kalimat penjelasan + kenapa ini masalah>
```
Maksimal tampilkan gejala yang benar-benar fatal — jangan generate daftar
panjang. Kalau cuma nemu 2-3 masalah, itu cukup (jangan dipaksa nambah
biar keliatan "lengkap").

### 6. Checkpoint — HARD STOP
Setelah laporan ditampilkan, tanya:
> "Dari temuan ini, mau dibenerin yang mana — kontennya, desainnya, atau
> dua-duanya? Atau mau lihat detail salah satu temuan dulu?"

Tunggu jawaban eksplisit. JANGAN lanjut ke fix apapun tanpa konfirmasi ini.
Setelah user pilih, append journal `fix_selected` per temuan yang dipilih
dan update `audits.json` → `status: "in-progress"`.

## Yang TIDAK boleh dilakukan skill ini
- Membuat laporan audit lengkap/checklist panjang — fokus fatal-only
- Melabel penilaian judgment (value prop, trust signal) sebagai fakta pasti
- Lanjut ke fix konten/desain tanpa checkpoint eksplisit
- Audit lebih dari 1 halaman dalam satu run
- Mengklaim deteksi kontras/konsistensi/posisi-CTA/layout-rusak kalau
  `extract-styles.py` gagal jalan dan datanya tidak ada
- Menimpa `.garnish/registry/` yang sudah ada saat auto-setup Langkah 0
  (auto-setup HANYA jalan kalau registry belum ada sama sekali)

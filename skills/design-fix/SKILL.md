---
name: design-fix
description: Perbaiki desain (UI/UX, komponen, WCAG) berdasarkan temuan fatal dari /garnish:check. Nanya dulu mau fix SEMUA temuan desain atau yang prioritas TINGGI aja. Untuk temuan visual (kontras tombol, konsistensi komponen, posisi CTA, layout rusak, evaluasi heuristik, section landing page yang hilang) — cari referensi baru yang menjawab temuan spesifik (reuse plugin design-agent), scaffold component library kalau belum ada, lalu rebuild HANYA komponen/section yang ditandai fatal (bukan seluruh halaman). Untuk temuan struktural (alt text, heading hierarchy, target size, icon label) — edit kode langsung tanpa perlu referensi visual. Gunakan skill ini ketika user pilih "fix desain" atau "keduanya" di checkpoint setelah audit. Butuh plugin design-agent ter-install HANYA kalau ada temuan visual — cek dulu, kasih instruksi install kalau belum ada.
---

# /garnish:design-fix — Perbaikan Desain

Skill ini adalah orchestrator — tidak menulis ulang logic ekstraksi token,
pencarian referensi, atau build. Semua itu tetap dikerjakan skill
`design-agent` yang sudah ada.

## Langkah

### 1. Baca temuan dari registry
Ambil temuan (`findings`) dengan `status: "open"` dan `scope` dalam
`{"ui-ux", "komponen", "wcag"}` dari audit yang dimaksud di
`.garnish/registry/audits.json`.

### 1.5. Checkpoint — semua temuan atau prioritas tinggi aja (HARD STOP)


Hitung jumlah temuan (dari Langkah 1) yang `severity: "tinggi"` vs total
keseluruhan, lalu tanya:
> "Ada {total} temuan desain (scope UI/UX, komponen, WCAG) — {jumlah
> tinggi} di antaranya prioritas TINGGI. Mau saya benerin SEMUA temuan
> ({total}), atau yang prioritas TINGGI aja ({jumlah tinggi})?"

Tunggu jawaban eksplisit. Kalau user pilih "prioritas tinggi aja", filter
temuan yang diproses di langkah-langkah berikutnya jadi HANYA yang
`severity: "tinggi"` — temuan `severity: "sedang"` yang tidak dipilih
TETAP `status: "open"` di registry (bukan didiamkan tanpa jejak — bisa
di-fix nanti lewat run terpisah).

Pisahkan temuan yang akan diproses (baik "semua" atau "tinggi aja") jadi
dua kelompok, karena cara fix-nya beda:

- **Kelompok A — butuh referensi visual** (`type`: `contrast` [tombol],
  `consistency`, `cta-position`, `layout-rusak`, `ui-heuristic`,
  `missing-section`) → lanjut ke Langkah 2-11 seperti biasa (orchestrate
  ke `design-agent`).
- **Kelompok B — fix struktural langsung, TANPA cari referensi**
  (`type`: `alt-text`, `heading-hierarchy`, `target-size`, `icon-label`)
  → skip Langkah 4-7 (gak perlu referensi visual buat nambah atribut
  `alt`/`aria-label`, membetulkan urutan heading, atau memperbesar
  padding tombol), langsung edit kode yang relevan memakai `suggestion`
  dari finding sebagai arah, lalu lanjut ke Langkah 9 (QA) dan 10 (update
  registry) — `designRef` untuk finding kelompok B boleh `null` karena
  tidak ada referensi/spec yang dipakai.

### 1.6. Checkpoint — pilih design system (HARD STOP)

Tanya user design system mana yang mau dipakai sebagai fondasi komponen:

> "Mau pakai design system apa sebagai fondasi? Pilihannya:
>
> **React-based (Tailwind):**
> 1. **shadcn/ui** — copy-paste components, Tailwind CSS Variables, kustomisasi penuh
> 2. **Bag UI** — shadcn blocks siap pakai (hero, CTA, pricing, navbar, footer)
> 3. **Flowbite** — komponen Tailwind siap pakai, ada React library-nya
> 4. **DaisyUI** — Tailwind plugin, semantic class names, theme system built-in
>
> **React-based (CSS-in-JS / styled):**
> 5. **Chakra UI** — accessible, theme-able, dark mode built-in
> 6. **Mantine** — full-featured, banyak komponen, hooks library included
> 7. **Material UI (MUI)** — Google Material Design, React ready
> 8. **Ant Design** — enterprise-grade, komponen lengkap
>
> **Headless (bawa styling sendiri):**
> 9. **Radix UI** — primitives tanpa styling, kontrol penuh
> 10. **Headless UI** (by Tailwind team) — accessible, minimal
>
> **Figma-first / multi-framework:**
> 11. **Untitled UI** — Figma-first, clean minimal, ada Tailwind export
>
> 12. Lainnya (sebutkan nama + link kalau bisa)
>
> Design system ini menentukan STRUKTUR komponen — token warna/spacing/radius dari referensi yang kita cari akan di-apply sebagai override di atasnya."

Tunggu jawaban eksplisit. Simpan pilihan sebagai `designSystem` — dipakai
di Langkah 4, 6, dan 8.

**Kenapa ini penting:** DS yang mapan punya struktur, spacing, accessibility,
dan behavior yang sudah benar — ini yang mencegah "AI slop" (komponen yang
kelihatan random karena dikarang dari nol). Token dari referensi (warna,
radius, shadow) cukup di-apply sebagai CSS variable override di atas
komponen DS, bukan menggantikan struktur komponennya.

### 2. Cek plugin `design-agent` ter-install
Kalau SEMUA finding yang mau di-fix di run ini adalah Kelompok B (tidak
ada Kelompok A sama sekali) → skip Langkah 2-3 ini, langsung proses
Kelompok B (Langkah 1.5) lalu Langkah 9 — `design-agent` tidak
dibutuhkan sama sekali untuk fix struktural murni.

Kalau ada Kelompok A, lanjut cek plugin `design-agent` seperti biasa.
Coba deteksi apakah skill `design-agent` tersedia (mis. cek lewat daftar
skill yang ke-load, atau fallback cek filesystem):
```bash
find ~/.claude/plugins/cache -name "inspo" -path "*design-agent*" -type d 2>/dev/null | head -1
```
Kalau tidak ketemu:
> "Perbaikan desain butuh plugin `design-agent` ter-install. Jalankan ini
> dulu, lalu bilang saya kalau sudah:
> ```
> /plugin marketplace add ajisss/design-agent-plugin
> /plugin install design-agent@design-agent-marketplace
> /reload-plugins
> ```"
Tunggu konfirmasi user sebelum lanjut — jangan asumsikan sudah ter-install.

### 3. Cek prasyarat `.design/registry/` di project ini
Kalau project yang diaudit belum punya `.design/registry/project.json`
(belum pernah `/design-agent:init`), beri tahu user secara transparan dan
jalankan `/design-agent:init` dulu — jangan diam-diam menganggap sudah ada.

### 4. Susun brief pencarian per temuan fatal
Kelompokkan temuan yang relevan (mis. semua yang soal tombol jadi satu
brief, yang soal posisi CTA jadi brief lain) — jangan gabung temuan yang
tidak nyambung jadi satu query membingungkan.

**Karena struktur komponen sudah ditentukan DS (Langkah 1.6), brief di sini
HANYA mencari TOKEN VISUAL** — warna, spacing, radius, shadow, typography
scale — bukan mencari komponen atau struktur baru. Framing brief yang benar:

> *"Cari referensi untuk TOKEN VISUAL [konteks produk/industri] yang mau
> dipakai sebagai override di atas komponen {designSystem} yang sudah ada:
> - Palet warna utama + aksen (untuk tombol CTA, highlight)
> - Radius border (sharp/rounded/pill — mana yang cocok dengan karakter brand)
> - Typography scale (heading size, weight, line-height)
> - Spacing rhythm (padding komponen, gap antar section)
> Cukup style guide, landing page satu-halaman, atau component showcase
> yang kaya variasi visual — BUKAN landing page multi-section utuh"*

Brief yang salah: *"cari referensi tombol CTA"* → ini akan menghasilkan
referensi yang menggoda untuk rebuild struktur komponen (AI slop).
Brief yang benar: *"cari referensi token warna/radius untuk CTA yang
akan di-apply ke komponen Button shadcn/ui"*.

Panggil `/design-agent:inspo` dengan brief itu untuk tiap kelompok temuan.

### 5. Checkpoint pemilihan referensi
`/design-agent:select` tetap berlaku seperti biasa (hard stop pemilihan
referensi) — jangan skip atau putuskan sendiri referensi mana yang dipakai.

### 6. Scaffold dari design system yang dipilih (Langkah 1.6)

Prinsip utama: **import komponen dari DS, jangan bikin dari nol**. Struktur,
spacing internal, dan behavior sudah benar di DS — tugas kita hanya apply
token visual dari spec sebagai override.

**Setup DS (kalau belum ada di project):**

- **shadcn/ui** →
  ```bash
  npx shadcn@latest init
  npx shadcn@latest add button card input
  ```
- **Bag UI** → clone repo, copy komponen/section yang dibutuhkan saja:
  ```bash
  git clone https://github.com/anelkabag/bag-ui /tmp/bag-ui
  cp -r /tmp/bag-ui/components/[nama-komponen] src/components/ui/
  ```
- **Flowbite** →
  ```bash
  npm install flowbite flowbite-react
  ```
  Import komponen: `import { Button } from 'flowbite-react'`
- **DaisyUI** →
  ```bash
  npm install daisyui
  ```
  Tambah ke `tailwind.config.js`: `plugins: [require('daisyui')]`
- **Chakra UI** →
  ```bash
  npm install @chakra-ui/react @emotion/react @emotion/styled framer-motion
  ```
  Wrap app dengan `<ChakraProvider theme={theme}>`
- **Mantine** →
  ```bash
  npm install @mantine/core @mantine/hooks
  ```
  Wrap app dengan `<MantineProvider theme={theme}>`
- **Material UI (MUI)** →
  ```bash
  npm install @mui/material @emotion/react @emotion/styled
  ```
- **Ant Design** →
  ```bash
  npm install antd
  ```
- **Radix UI** → install primitives yang relevan:
  ```bash
  npm install @radix-ui/react-slot @radix-ui/react-dialog  # dll sesuai kebutuhan
  ```
  Buat wrapper component lokal di `src/components/ui/` dengan styling Tailwind.
- **Headless UI** →
  ```bash
  npm install @headlessui/react
  ```
  Buat wrapper component lokal dengan styling Tailwind.
- **Untitled UI** → copy komponen dari https://www.untitledui.com/components
  ke `src/components/ui/` sesuai instruksi website.

Kalau komponen DS sudah ada di project (`src/components/ui/` atau setara) →
**reuse, jangan timpa atau duplikasi**.

Setelah DS ada, jangan isi token hardcode di sini — biarkan `/design-agent:build`
Langkah 8 yang mengisi dari spec.

### 7. Ekstrak spec dari referensi terpilih (scope terbatas)
Panggil `/design-agent:spec` pada referensi yang dipilih di Langkah 5,
dengan instruksi eksplisit ke prosesnya: token yang diekstrak HANYA untuk
jenis komponen/section yang fatal (mis. cuma token tombol kalau
temuannya soal tombol) — bukan seluruh design system halaman referensi.

Catatan: `/design-agent:spec` punya hard-stop wajib (Langkah "3.5. Tentukan
& konfirmasi struktur halaman") yang meminta user konfirmasi struktur
section penuh (hero/fitur/footer/dll) sebelum spec bisa
`sectionsConfirmed: true` — ini bagian dari alur `design-agent` dan TIDAK
diubah di sini. Karena referensi yang dicari di Langkah 4 sudah sengaja
dibuat sempit (component showcase / potongan kecil halaman, bukan landing
page utuh), section yang perlu dikonfirmasi di sini otomatis cuma
1-2 section yang relevan — jadi checkpoint ini tetap cepat dan fokus,
bukan konfirmasi struktur halaman penuh yang tidak nyambung dengan
perbaikan satu komponen.

### 8. Build — HANYA komponen/section yang fatal, token sebagai override DS

Panggil `/design-agent:build` pada spec dari Langkah 7, dengan dua
instruksi eksplisit:

1. **Scope sempit** — rebuild HANYA komponen/section yang berkaitan dengan
   temuan fatal di Langkah 1. JANGAN sentuh section atau konten lain.

2. **DS-first, token sebagai override** — JANGAN replace komponen DS dengan
   komponen baru dari nol. Cara yang benar:
   - Gunakan komponen DS yang sudah di-scaffold di Langkah 6 (`Button`,
     `Card`, dll dari shadcn/radix/MUI/dll)
   - Apply token warna/radius/spacing dari spec sebagai **CSS variable
     override** atau **Tailwind config extension**, bukan inline style
     atau komponen baru yang menggantikan DS:
     ```css
     /* globals.css — override CSS variables shadcn/ui */
     :root {
       --primary: <warna dari spec>;
       --primary-foreground: <warna dari spec>;
       --radius: <radius dari spec>;
     }
     ```
     atau untuk Tailwind:
     ```js
     // tailwind.config.js
     theme: { extend: { colors: { primary: '<dari spec>' } } }
     ```
   - Untuk **MUI**: pakai `createTheme({ palette: { primary: { main: '...' } } })`
   - Untuk **Chakra UI**: pakai `extendTheme({ colors: { brand: { 500: '...' } } })`
   - Untuk **Mantine**: pakai `createTheme({ primaryColor: '...', colors: { brand: [...] } })`
   - Untuk **Ant Design**: pakai `ConfigProvider theme={{ token: { colorPrimary: '...' } }}`
   - Untuk **DaisyUI**: override via CSS variables atau `daisyui.themes` di tailwind config
   - Untuk **Flowbite**: custom theme via `flowbite-react` theme prop atau CSS variables
   - Untuk **Radix / Headless UI**: apply token via CSS variables di wrapper component

   **Hasilnya:** struktur komponen dari DS (spacing internal, variant,
   accessibility sudah benar), visual character dari token referensi
   (warna, radius, shadow, typography). Ini yang menghindari AI slop —
   tidak ada komponen random yang dikarang dari nol.

### 9. QA — re-audit hasil build sebelum ditandai selesai

Sebelum menandai finding manapun `design-fixed`, verifikasi hasilnya
beneran menyelesaikan temuan yang tadi fatal. Caranya beda untuk Kelompok
A vs Kelompok B (lihat Langkah 1.5):

**Untuk Kelompok A** (butuh computed CSS):
1. Pastikan dev server project berjalan (tanya user URL-nya kalau belum
   diketahui dari sesi ini, mis. `http://localhost:3000`).
2. Cari `extract-styles.py` (path sama seperti Langkah 2) dan jalankan ke
   dev server tersebut:
   ```bash
   python3 "$EXTRACT_STYLES" <url-dev-server> .garnish/registry/audit-<audit-id>-recheck.json .garnish/registry/screenshots/<audit-id>-recheck
   ```
3. Dari hasil JSON itu, ulangi HANYA deteksi yang relevan ke jenis temuan
   di Langkah 1 (mis. kalau tadi soal kontras tombol, hitung ulang
   kontras tombol di hasil build ini — logic deteksinya sama persis
   dengan `/garnish:check` Langkah 3, bagian yang relevan saja).

**Untuk Kelompok B** (`alt-text`, `heading-hierarchy` — butuh HTML, bukan
computed CSS):
1. Pastikan dev server project berjalan.
2. `WebFetch <url-dev-server>` buat HTML hasil build.
3. Ulangi deteksi yang sama persis dengan `/garnish:check` Langkah 3a
   (cek `<img>` tanpa `alt`, atau urutan heading yang masih loncat level)
   pada bagian kode yang barusan diedit.

**Kedua kelompok** — hasil pengecekan ulang:
4. **Kalau bersih** (temuan yang di-fix beneran tidak muncul lagi) →
   lanjut ke Langkah 10 (update registry).
5. **Kalau masih ada sisa**:
   - Kelompok A → kembali ke Langkah 7 (ekstrak spec dari referensi
     terpilih) atau Langkah 8 (build) — perbaiki spesifik yang masih
     meleset, JANGAN ulangi dari Langkah 4 (cari referensi baru lagi)
     kecuali referensi yang dipakai memang jadi penyebabnya.
   - Kelompok B → langsung edit lagi kode yang relevan (tidak ada
     referensi/spec yang perlu diulang).

   Ulangi Langkah 9 (QA) lagi setelah perbaikan.
6. Maksimal 3 putaran QA total. Kalau di putaran ke-3 masih ada sisa,
   HENTIKAN loop — JANGAN tandai finding itu `design-fixed`. Laporkan ke
   user secara eksplisit di Langkah 11: finding mana yang belum berhasil
   diperbaiki otomatis setelah 3 percobaan, dan kenapa (sebutkan hasil
   pengukuran terakhirnya).

### 10. Update registry
Untuk tiap finding yang berhasil di-fix (lolos QA Langkah 9):

Kelompok A:
```json
{
  "status": "design-fixed",
  "fixedAt": "<ISO 8601>",
  "designRef": {
    "refIds": ["R-00X dari .design/registry/references.json project ini"],
    "specId": "S-00X dari .design/registry/specs.json project ini"
  }
}
```

Kelompok B (tidak ada referensi/spec yang dipakai):
```json
{ "status": "design-fixed", "fixedAt": "<ISO 8601>", "designRef": null }
```
Append journal (di awal Langkah 4, sebelum mulai cari referensi):
```json
{"ts":"<ISO 8601>","event":"design_fix_started","auditId":"A-00X","findingIds":["F-00X"]}
```
Dan (di akhir, per finding yang selesai):
```json
{"ts":"<ISO 8601>","event":"design_fixed","auditId":"A-00X","findingId":"F-00X","specId":"S-00X"}
```
Kalau semua finding desain di audit ini sudah `design-fixed` (dan tidak
ada temuan `open` lain yang tersisa dari kategori konten), update
`audits.json` → `status: "resolved"`. Kalau user tadi pilih "prioritas
tinggi aja" di Langkah 1.5, finding `severity: "sedang"` yang sengaja
tidak diproses akan tetap `status: "open"` — JANGAN update audit jadi
`resolved` selama itu masih ada, walau semua finding `tinggi` sudah
`design-fixed`.

### 11. Laporkan before/after ke user
Tampilkan ringkasan: komponen/section mana yang di-fix, referensi apa yang
dipakai (link/ID), hasil QA Langkah 9 (lolos di putaran ke berapa, atau
finding mana yang masih belum lolos setelah 3 putaran), dan screenshot
atau deskripsi hasil akhir — jangan cuma bilang "sudah selesai" tanpa
bukti konkret.

## Yang TIDAK boleh dilakukan skill ini
- Menulis ulang logic `inspo`/`select`/`spec`/`build` sendiri — selalu
  panggil skill `design-agent` yang sudah ada
- Rebuild seluruh halaman kalau cuma sebagian yang ditandai fatal
- Skip checkpoint Langkah 1.6 (pilih design system) — wajib tanya user
- Bikin komponen dari nol kalau komponen DS sudah tersedia — selalu pakai
  DS yang dipilih di Langkah 1.6 sebagai fondasi
- Apply token sebagai hardcode values di dalam komponen — selalu via CSS
  variable override atau theme config supaya konsisten dan maintainable
- Skip checkpoint Langkah 1.5 (semua vs prioritas tinggi) atau
  memproses finding `severity: "sedang"` padahal user sudah pilih
  "prioritas tinggi aja"
- Menandai audit `resolved` kalau ada finding `severity: "sedang"` yang
  sengaja tidak diproses (masih `status: "open"`) karena user pilih
  "prioritas tinggi aja"
- Menimpa component library yang sudah ada di lokasi konvensi project
- Melanjutkan tanpa plugin `design-agent` ter-install atau tanpa
  `.design/registry/` ter-init di project yang diaudit
- Skip checkpoint pemilihan referensi (`/design-agent:select`)
- Menandai finding `"design-fixed"` sebelum lolos QA Langkah 9, atau
  melanjutkan retry lebih dari 3 putaran tanpa melapor apa adanya ke user

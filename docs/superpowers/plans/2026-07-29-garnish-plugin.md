# GARNISH Plugin Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Scaffold the `garnish` Claude Code plugin (audit → checkpoint → content-fix and/or design-fix) as its own repo, reconciling the two prior draft iterations (`garnish-repo.zip`, `garnish-repo (1).zip`) and writing the one missing piece: `skills/design-fix/SKILL.md`.

**Architecture:** `garnish` is a standalone plugin repo at `/Users/ihsanaziz/garnish-plugin` with its own `.garnish/registry/` state (mirroring `design-agent`'s registry pattern). It depends on the `design-agent` plugin (already built at `/Users/ihsanaziz/design-agent-plugin`, v1.2.0) for the design-fix path — `garnish` never re-implements token extraction, reference search, or component building itself.

**Tech Stack:** Markdown skill files (Claude Code plugin format) + JSON registry files. No Python/code in this plugin itself — `check`'s technical detection reuses `design-agent`'s existing `hooks/extract-styles.py` (not the `capture-reference.py` referenced in the old zip drafts, which does not exist in the `design-agent` dependency this plan targets).

## Global Constraints

- `design-agent` dependency is `/Users/ihsanaziz/design-agent-plugin` (v1.2.0) — NOT the simpler bundled copy that was in either `garnish-repo.zip` draft. Any reference to a `design-agent` script must match what actually exists there: `hooks/extract-styles.py`, `hooks/compare-tokens.py`, `hooks/visual-diff.py`, `hooks/validate-tokens.py`. There is no `capture-reference.py` — do not reference it anywhere in this plugin.
- Registry principle: `.garnish/registry/` is the single source of truth. Every skill reads it before acting and writes state back to it — never rely on chat memory alone.
- Checkpoints (audit → fix choice; design-fix's own use of `/design-agent:select`) are hard stops — wait for explicit user answer, never assume.
- `category: "measured"` findings must come from actual computed values (contrast ratio math, computed-style comparison) — never a guess dressed up as measured. `category: "judgment"` findings must always be labeled `(penilaian AI)` to the user, never presented as fact.
- `design-fix` only rebuilds components/sections marked fatal — never a full-page rebuild, and never touches content not marked fatal (inherited from `design-agent`'s own style/content separation rule).
- Bahasa Indonesia for all skill prose, matching the existing style in both draft zips and `design-agent-plugin`.

---

## File Structure

```
garnish-plugin/
  .claude-plugin/
    plugin.json          (Task 1)
    marketplace.json      (Task 1)
  CLAUDE.md               (Task 1, then amended Task 5)
  CONTEXT.md              (Task 1, then amended Task 5)
  README.md               (Task 1, then amended Task 5)
  skills/
    init/SKILL.md         (Task 1 — copied as-is, no dependency-script issues)
    check/SKILL.md        (Task 2 — adapted to use extract-styles.py, not capture-reference.py)
    content-fix/SKILL.md  (Task 3 — copied as-is)
    design-fix/SKILL.md   (Task 4 — new)
```

---

### Task 1: Scaffold repo — manifest, docs, and `init` skill

**Files:**
- Create: `.claude-plugin/plugin.json`
- Create: `.claude-plugin/marketplace.json`
- Create: `CLAUDE.md`
- Create: `CONTEXT.md`
- Create: `README.md`
- Create: `skills/init/SKILL.md`

- [ ] **Step 1: Write `.claude-plugin/plugin.json`**

```json
{
  "name": "garnish",
  "description": "Audit desain & konten fatal untuk landing page/screen apapun, lalu benerin kontennya (rewrite copy) dan/atau desainnya (reuse design-agent + component library). Quality check terakhir sebelum produk disajikan ke klien.",
  "version": "0.2.0",
  "dependencies": ["design-agent"]
}
```

- [ ] **Step 2: Write `.claude-plugin/marketplace.json`**

```json
{
  "name": "garnish-marketplace",
  "owner": {
    "name": "ajisss"
  },
  "plugins": [
    {
      "name": "garnish",
      "source": "./",
      "description": "Audit gejala fatal desain & konten, lalu fix konten/desain otomatis"
    }
  ]
}
```

- [ ] **Step 3: Write `CLAUDE.md`**

```markdown
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
```

- [ ] **Step 4: Write `CONTEXT.md`**

```markdown
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
| **Agent Actions** | Screenshot & ekstrak halaman → deteksi gejala fatal → checkpoint tanya user mau fix apa → eksekusi fix |
| **Output** | Laporan temuan fatal (prioritized) + revisi konten dan/atau desain |
| **Success Criteria** | Dikasih 1 URL bermasalah, agent nemuin minimal 2-3 gejala fatal valid, user pilih mana yang mau difix, hasil revisi keluar dengan laporan before/after |

## Keputusan Desain Paling Penting: "Fatal-Only", Bukan Audit Lengkap

Ini keputusan sengaja — GARNISH BUKAN checklist audit menyeluruh (beda dari
tools kayak Unbounce/HubSpot Grader yang udah ada). Fokus cuma ke gejala
yang benar-benar mengganggu/fatal, biar laporan pendek dan actionable,
bukan overwhelming.

### Gejala fatal & cara deteksinya (PENTING — bedakan yang terukur vs judgment)

| Gejala | Terukur objektif? | Cara deteksi |
|---|---|---|
| Kontras teks gak kebaca | ✅ Ya | Hitung rasio kontras WCAG dari computed CSS (via `extract-styles.py`) |
| Style tombol/komponen gak konsisten dalam 1 halaman | ✅ Ya | Baca computed CSS semua tombol, bandingkan (via `extract-styles.py`) |
| CTA gak keliatan di atas fold | ✅ Ya | Cek posisi bbox section relatif ke viewport height |
| Placeholder ketinggalan ("Lorem ipsum", "TODO", dll) | ✅ Ya | Cari pola teks di HTML mentah |
| Layout rusak/overlapping element | ⚠️ Sebagian | Deteksi visual dari screenshot, lebih rewel tapi masih bisa dicoba |
| Value proposition gak jelas di headline | ❌ Judgment | Penilaian Claude — WAJIB dilabel sebagai penilaian, bukan fakta |
| Gak ada trust signal/social proof | ❌ Judgment | Penilaian Claude — WAJIB dilabel sebagai penilaian |

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

Untuk bagian "fix desain", GARNISH memanggil skill `design-agent`:
- `/design-agent:inspo` — cari referensi baru yang menjawab temuan fatal
  spesifik (bukan referensi generik seluruh halaman)
- `/design-agent:select` — checkpoint pemilihan referensi tetap berlaku
- `/design-agent:spec` — ekstrak referensi terpilih jadi token, scope
  hanya komponen/section yang fatal
- `/design-agent:build` — rebuild HANYA komponen/section yang fatal, pakai
  component library yang di-scaffold `design-fix`

**Component library baru** (Button, Card, Input — cuma 3 komponen, jangan
lebih) di-scaffold oleh `design-fix` sendiri (bukan skill terpisah), kalau
project belum punya, dipakai `design-agent:build` supaya hasil fix desain
konsisten strukturnya.

Untuk "fix konten" (rewrite copy), GARNISH TIDAK pakai Superpowers — itu
kesalahpahaman yang sempat muncul dan sudah dikoreksi. Superpowers itu
metodologi coding (TDD/planning), bukan alat buat menulis ulang copy.
Content-fix itu murni instruksi tertulis-ulang oleh Claude berdasarkan
temuan audit.

## Broadened Framing — Bukan Cuma Landing Page

Awalnya scope cuma "landing page audit", tapi diputuskan diperluas jadi
generic: bisa audit screen/halaman apapun (dashboard, settings, onboarding,
dll), bukan cuma landing page marketing. Ini memperluas target klien dari
"yang punya landing page" ke "siapapun yang punya produk digital". Secara
teknis TIDAK menambah kompleksitas signifikan — screenshot & deteksi
gejala fatal itu proses yang sama untuk jenis halaman apapun, asal skill-nya
ditulis generic (jangan hardcode asumsi "pasti ada hero/CTA/social proof").
**Prioritas pengembangan & testing tetap landing page dulu** — dashboard
menyusul tanpa perlu ubah skill-nya kalau memang ditulis generic.

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
  di-scaffold via skill `/garnish:init`, dengan stable ID per audit (`A-00X`)
  dan per temuan (`F-00X`), mengikuti pola yang sama persis dengan SANDWICH
  dan `design-agent`
- **Jujur soal ketidakpastian** — penilaian judgment vs fakta terukur harus
  dibedakan secara eksplisit ke user
- **Konten bukan bagian dari style yang direplikasi** — kalau fix desain
  reuse referensi/benchmark, jangan timpa konten yang sudah ditulis dengan
  teks dari sumber lain
```

- [ ] **Step 5: Write `README.md`**

```markdown
# GARNISH

*"The final touch before you serve it to the client."*

Audit gejala fatal desain & konten (bukan checklist lengkap), lalu
benerin kontennya dan/atau desainnya (reuse plugin `design-agent`).

## Install

```
/plugin marketplace add ajisss/garnish
/plugin install garnish@garnish-marketplace
/reload-plugins
```

## Dependency

Butuh plugin `design-agent` sudah ter-install (dipakai `/garnish:design-fix`
buat cari referensi, ekstrak token, dan rebuild komponen). Kalau belum ada:
```
/plugin marketplace add ajisss/design-agent-plugin
/plugin install design-agent@design-agent-marketplace
/reload-plugins
```
Lihat `CONTEXT.md` untuk detail relasinya. `design-fix` mengecek sendiri
apakah dependency ini tersedia sebelum jalan, dan kasih instruksi di atas
kalau belum ada.

## Struktur
```
CONTEXT.md              ← WAJIB dibaca duluan, semua latar belakang & keputusan
CLAUDE.md                ← aturan permanen
skills/
  ├── init/SKILL.md           ← scaffold registry (.garnish/registry/), sekali per project
  ├── check/SKILL.md          ← audit gejala fatal, simpan ke registry
  ├── content-fix/SKILL.md    ← rewrite copy, update status temuan di registry
  └── design-fix/SKILL.md     ← orchestrator ke design-agent (inspo/select/spec/build)
```

## Status
Lengkap untuk skenario "wajib solid" + stretch goal dari planning awal:
`check` (deteksi terukur + judgment), checkpoint, `content-fix`, dan
`design-fix` (integrasi `design-agent` + scaffold component library).
```

- [ ] **Step 6: Write `skills/init/SKILL.md`**

```markdown
---
name: init
description: Scaffold registry (.garnish/registry/) yang dibutuhkan skill check dan content-fix ke project/folder saat ini. Gunakan skill ini ketika user pertama kali pakai garnish di sebuah folder dan belum ada .garnish/registry/, atau user eksplisit minta "setup garnish". Jalankan SEBELUM /garnish:check kalau registry belum ada.
---

# /garnish:init — Setup Registry

## Langkah

### 1. Cek apakah sudah pernah di-init
Cek apakah `.garnish/registry/audits.json` sudah ada.
- Kalau **sudah ada** → tanya user apakah mau reset, atau batalkan.
- Kalau **belum ada** → lanjut.

### 2. Buat struktur folder & file
```bash
mkdir -p .garnish/registry/screenshots
```

```bash
cat > .garnish/registry/audits.json << 'EOF'
{ "audits": [] }
EOF
touch .garnish/registry/journal.jsonl
```

### 3. Tulis `SCHEMA.md`
```markdown
# Garnish Registry Schema

## audits.json
\`\`\`json
{
  "audits": [
    {
      "id": "A-001",
      "url": "string",
      "screenshotPath": "string | null",
      "capturedAt": "ISO 8601",
      "findings": [
        {
          "id": "F-001",
          "category": "measured | judgment",
          "type": "contrast | consistency | cta-position | placeholder | value-prop | trust-signal | lainnya",
          "title": "string singkat",
          "description": "string",
          "status": "open | content-fixed | design-fixed | dismissed",
          "fixedAt": "ISO 8601 | null",
          "designRef": {
            "refIds": ["string — ID dari .design/registry/references.json project ini"],
            "specId": "string — ID dari .design/registry/specs.json project ini"
          }
        }
      ],
      "status": "audited | in-progress | resolved"
    }
  ]
}
\`\`\`

`category: "measured"` = terukur objektif (kontras, konsistensi, posisi CTA,
placeholder) — WAJIB dari perhitungan nyata, bukan tebakan.
`category: "judgment"` = penilaian AI (value prop, trust signal) — WAJIB
dilabel eksplisit ke user sebagai penilaian, bukan fakta pasti.

`designRef` cuma diisi untuk finding yang di-fix lewat `/garnish:design-fix`
— cross-reference murni ke registry `design-agent` di project yang sama
(TIDAK menduplikasi data token/spec di sini).

ID (`A-00X`, `F-00X`) lanjut dari ID terakhir yang ada — jangan mulai dari
1 lagi kalau sudah ada audit sebelumnya.

## journal.jsonl
Append-only. Event yang dipakai: `audit_created`, `finding_detected`,
`fix_selected`, `content_fixed`, `design_fix_started`, `design_fixed`,
`finding_dismissed`.

\`\`\`json
{"ts":"ISO 8601","event":"audit_created","auditId":"A-001","url":"..."}
{"ts":"ISO 8601","event":"finding_detected","auditId":"A-001","findingId":"F-001","category":"measured","type":"contrast"}
{"ts":"ISO 8601","event":"fix_selected","auditId":"A-001","findingId":"F-001","fixType":"content|design"}
{"ts":"ISO 8601","event":"content_fixed","auditId":"A-001","findingId":"F-001"}
{"ts":"ISO 8601","event":"design_fix_started","auditId":"A-001","findingIds":["F-001"]}
{"ts":"ISO 8601","event":"design_fixed","auditId":"A-001","findingId":"F-001","specId":"S-001"}
{"ts":"ISO 8601","event":"finding_dismissed","auditId":"A-001","findingId":"F-001","reason":"..."}
\`\`\`
```
(Tulis konten di atas persis ke file `.garnish/registry/SCHEMA.md`.)

### 4. Konfirmasi ke user
> "Registry Garnish sudah siap di `.garnish/registry/`. Coba audit halaman
> pertama dengan minta 'audit [url]'."

## Yang TIDAK boleh dilakukan skill ini
- Menimpa `.garnish/registry/` yang sudah ada tanpa konfirmasi eksplisit
- Menjalankan skill `check`/`content-fix`/`design-fix` sebagai bagian dari
  init — ini cuma setup
```

- [ ] **Step 7: Verify structure**

Run: `find /Users/ihsanaziz/garnish-plugin -type f -not -path "*/.git/*" | sort`
Expected: exactly the 7 files listed above (plugin.json, marketplace.json,
CLAUDE.md, CONTEXT.md, README.md, init/SKILL.md) plus the pre-existing
`docs/superpowers/specs/2026-07-29-garnish-design.md` and this plan file.

- [ ] **Step 8: Commit**

```bash
cd /Users/ihsanaziz/garnish-plugin
git add .claude-plugin CLAUDE.md CONTEXT.md README.md skills/init
git commit -m "feat: scaffold garnish plugin (manifest, docs, init skill)"
```

---

### Task 2: `skills/check/SKILL.md` — adapted to real `design-agent` dependency

**Files:**
- Create: `skills/check/SKILL.md`

**Interfaces:**
- Consumes (from `design-agent` v1.2.0, NOT bundled here — must exist at the path the `find` command below resolves): `hooks/extract-styles.py <url> <output.json> [screenshots_dir]`, exit 0 on success. Output JSON shape: `{"colors": {"dominant": [hex,...]}, "sections": [{"index", "bbox": {"y","width","height"}, "typography": [{"level","font_size","font_weight","line_height","font_family","color"}], "buttons": [{"background_color","color","border_radius","padding","box_shadow"}], "containers": [...], "screenshot_path"}]}` — this is the ACTUAL current output shape of `design-agent`'s `extract-styles.py` (verify by reading `/Users/ihsanaziz/design-agent-plugin/hooks/extract-styles.py` before writing this skill if anything here seems inconsistent with that file).
- Produces: entries in `.garnish/registry/audits.json` per the schema written in Task 1 Step 6.

- [ ] **Step 1: Write `skills/check/SKILL.md`**

```markdown
---
name: check
description: Audit sebuah landing page atau screen aplikasi (URL apapun) untuk mendeteksi gejala fatal desain & konten — bukan checklist lengkap, hanya masalah yang benar-benar mengganggu. Gunakan skill ini ketika user minta "audit", "cek", "garnish", atau kasih URL dan minta dievaluasi. Hasil temuan disimpan ke registry (.garnish/registry/) dengan ID stabil. Setelah temuan ditampilkan, WAJIB berhenti dan tanya user mau fix konten/desain/keduanya sebelum lanjut ke perbaikan apapun. Jalankan /garnish:init dulu kalau registry belum ada.
---

# /garnish:check — Audit Gejala Fatal

Baca `CONTEXT.md` dulu kalau belum — ada tabel lengkap gejala fatal yang
harus dideteksi dan mana yang terukur vs judgment.

## Langkah

### 0. Cek registry
Kalau `.garnish/registry/audits.json` belum ada, jalankan `/garnish:init`
dulu sebelum lanjut.

Cek juga apakah URL yang sama pernah diaudit sebelumnya (`audits.json`).
Kalau ada dan statusnya `"resolved"` atau `"in-progress"`, informasikan ke
user: "URL ini pernah diaudit tanggal X, ada Y temuan, status Z" — dan
tanya apakah mau audit ulang dari nol atau lihat status yang lama dulu.

### 1. Ekstrak halaman (reuse tool dari `design-agent`, jangan tulis ulang)
```bash
EXTRACT_STYLES=$(find ~/.claude/plugins/cache -name "extract-styles.py" -path "*design-agent*" 2>/dev/null | head -1)
if [ -z "$EXTRACT_STYLES" ]; then
  echo "extract-styles.py tidak ditemukan — pastikan plugin design-agent ter-install."
fi
python3 "$EXTRACT_STYLES" <url> .garnish/registry/audit-<audit-id>.json .garnish/registry/screenshots/<audit-id>-sections
```
Ini menghasilkan JSON terstruktur (warna dominan, tipografi per section,
tombol, container, bbox per section) DAN screenshot full-page di
`.garnish/registry/screenshots/<audit-id>-sections/_full-page.png` — pakai
file itu sebagai `screenshotPath` di registry (Langkah 4), jangan cari
script screenshot terpisah.

Kalau `extract-styles.py` gagal (exit code 1 — situs block bot, butuh
login, atau Playwright/Pillow belum terinstall): beri tahu user terus
terang, dan lanjutkan HANYA dengan deteksi yang tidak butuh computed CSS
(placeholder text dari Langkah 2 bagian terakhir, dan judgment dari
Langkah 3 berdasar screenshot manual/deskripsi user) — jangan klaim
deteksi kontras/konsistensi/posisi-CTA kalau datanya tidak ada.

Ambil juga HTML mentah halaman (`WebFetch <url>`) untuk deteksi placeholder
teks di Langkah 2 — ini terpisah dari `extract-styles.py` karena tool itu
tidak mengembalikan teks konten, cuma computed style.

### 2. Deteksi gejala TERUKUR dulu (prioritas utama)
Dari JSON hasil `extract-styles.py`:
- **Kontras teks**: untuk tiap `typography[]` entry, hitung rasio kontras
  WCAG antara `color` dan `background_color` section induknya. Di bawah
  4.5:1 untuk teks normal (atau 3:1 untuk teks besar ≥24px/18.5px bold) =
  fatal. Lakukan hal sama untuk `buttons[].color` vs
  `buttons[].background_color`.
- **Konsistensi komponen**: bandingkan `buttons[]` dan `containers[]` di
  seluruh section — kalau `border_radius`/`padding`/`background_color`
  beda-beda tanpa pola jelas (mis. 3 tombol dengan radius 4px, 8px, 16px
  tanpa alasan hierarki visual) = fatal.
- **Posisi CTA**: cek apakah section pertama (index 0, biasanya hero)
  punya `bbox.height` yang membuatnya (atau tombol di dalamnya) berada
  dalam viewport ~900px pertama. Kalau hero section sudah lebih tinggi
  dari 900px DAN tidak ada tombol di dalamnya = fatal (CTA kemungkinan di
  bawah fold).
- **Placeholder ketinggalan**: dari HTML mentah (Langkah 1), cari pola
  teks "lorem ipsum", "TODO", "[placeholder]", "your text here", "lorem",
  dll.

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
      "type": "contrast | consistency | cta-position | placeholder | value-prop | trust-signal",
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
- Menulis ke registry tanpa jalanin `/garnish:init` dulu kalau belum ada
- Mengklaim deteksi kontras/konsistensi/posisi-CTA kalau `extract-styles.py`
  gagal jalan dan datanya tidak ada
```

- [ ] **Step 2: Verify no stale dependency references**

Run: `grep -rn "capture-reference" /Users/ihsanaziz/garnish-plugin/skills/check/SKILL.md`
Expected: no output (no matches) — confirms the stale script reference
from the old zip drafts was not carried over.

- [ ] **Step 3: Cross-check against the actual `extract-styles.py`**

Read `/Users/ihsanaziz/design-agent-plugin/hooks/extract-styles.py` in
full. Confirm the JSON field names referenced in the skill above
(`sections[].bbox`, `.typography[].color`, `.typography[].font_size`,
`.buttons[].background_color`, `.buttons[].border_radius`,
`.containers[]`, `.screenshot_path` per section, top-level
`_full-page.png` in the screenshots dir) match exactly what the script's
`aggregate_extraction`/`run()` functions actually produce. If any name
differs, fix `skills/check/SKILL.md` to match the real field names before
committing — do not leave a mismatch.

- [ ] **Step 4: Commit**

```bash
cd /Users/ihsanaziz/garnish-plugin
git add skills/check
git commit -m "feat: tulis skill check, adaptasi ke extract-styles.py (bukan capture-reference.py)"
```

---

### Task 3: `skills/content-fix/SKILL.md`

**Files:**
- Create: `skills/content-fix/SKILL.md`

- [ ] **Step 1: Write `skills/content-fix/SKILL.md`**

```markdown
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
```

- [ ] **Step 2: Verify**

Read the file back, confirm it matches the block above exactly (this is a
straight copy from the reconciled draft, no adaptation needed since it has
no `design-agent`-script dependency).

- [ ] **Step 3: Commit**

```bash
cd /Users/ihsanaziz/garnish-plugin
git add skills/content-fix
git commit -m "feat: tulis skill content-fix"
```

---

### Task 4: `skills/design-fix/SKILL.md` (new)

**Files:**
- Create: `skills/design-fix/SKILL.md`

**Interfaces:**
- Consumes: `.garnish/registry/audits.json` findings (Task 2's schema, `designRef` field from Task 1 Step 6's schema), and calls `/design-agent:inspo`, `/design-agent:select`, `/design-agent:spec`, `/design-agent:build` (all pre-existing skills in the `design-agent` dependency — do not reference any skill name that doesn't exist there; verify against `/Users/ihsanaziz/design-agent-plugin/skills/*/SKILL.md` frontmatter `name:` fields if unsure).
- Produces: updates to `.garnish/registry/audits.json` (`status: "design-fixed"`, `designRef.refIds`/`specId` filled), journal events `design_fix_started`/`design_fixed`.

- [ ] **Step 1: Write `skills/design-fix/SKILL.md`**

```markdown
---
name: design-fix
description: Perbaiki desain berdasarkan temuan fatal dari /garnish:check — cari referensi baru yang menjawab temuan spesifik (reuse plugin design-agent), scaffold component library kalau belum ada, lalu rebuild HANYA komponen/section yang ditandai fatal. Gunakan skill ini ketika user pilih "fix desain" atau "keduanya" di checkpoint setelah audit. Butuh plugin design-agent ter-install — cek dulu, kasih instruksi install kalau belum ada.
---

# /garnish:design-fix — Perbaikan Desain

Skill ini adalah orchestrator — tidak menulis ulang logic ekstraksi token,
pencarian referensi, atau build. Semua itu tetap dikerjakan skill
`design-agent` yang sudah ada.

## Langkah

### 1. Baca temuan dari registry
Ambil temuan (`findings`) dengan `status: "open"` yang relevan ke desain
(`type`: `contrast`, `consistency`, `cta-position`, atau `layout-rusak`
kalau ada) dari audit yang dimaksud di `.garnish/registry/audits.json`.

### 2. Cek plugin `design-agent` ter-install
Coba deteksi apakah skill `design-agent` tersedia (mis. cek lewat daftar
skill yang ke-load, atau coba baca
`~/.claude/plugins/cache/**/design-agent/**/skills/inspo/SKILL.md`). Kalau
tidak ketemu:
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
tidak nyambung jadi satu query membingungkan. Contoh brief:
- Temuan `contrast`/`consistency` pada tombol → *"cari referensi tombol
  CTA dengan kontras tinggi dan style konsisten [sebutkan konteks
  produk/industri dari halaman yang diaudit kalau relevan]"*
- Temuan `cta-position` → *"cari referensi pola penempatan CTA yang selalu
  terlihat di atas fold"*

Panggil `/design-agent:inspo` dengan brief itu untuk tiap kelompok temuan.

### 5. Checkpoint pemilihan referensi
`/design-agent:select` tetap berlaku seperti biasa (hard stop pemilihan
referensi) — jangan skip atau putuskan sendiri referensi mana yang dipakai.

### 6. Scaffold component library (kalau belum ada)
- Kalau `.design/registry/project.json` sudah punya `stack` terisi (dari
  sesi `design-agent` sebelumnya di project ini) → pakai itu, jangan tanya
  ulang.
- Kalau belum → tanya framework & styling (sama seperti Langkah 1
  `/design-agent:build`), simpan ke `project.json` lewat mekanisme yang
  sama seperti `design-agent:build` supaya konsisten dan tidak tanya ulang
  di sesi berikutnya.
- Cek apakah `Button`/`Card`/`Input` sudah ada di lokasi konvensi project
  (mis. `src/components/ui/` untuk React/Next.js, atau folder komponen
  setara sesuai stack yang terdeteksi). Kalau **sudah ada** → reuse,
  JANGAN timpa atau bikin ulang di lokasi lain.
- Kalau belum ada → bikin 3 komponen dasar (Button: variant + size prop
  minimal; Card: container dengan slot konten; Input: text input dasar
  dengan label) — style-nya BELUM diisi hardcode di sini, biarkan
  `/design-agent:build` di Langkah 8 yang mengisi dari token spec supaya
  nilainya bukan tebakan.

### 7. Ekstrak spec dari referensi terpilih (scope terbatas)
Panggil `/design-agent:spec` pada referensi yang dipilih di Langkah 5,
dengan instruksi eksplisit ke prosesnya: token yang diekstrak HANYA untuk
jenis komponen/section yang fatal (mis. cuma token tombol kalau
temuannya soal tombol) — bukan seluruh design system halaman referensi.

### 8. Build — HANYA komponen/section yang fatal
Panggil `/design-agent:build` pada spec dari Langkah 7, dengan instruksi
eksplisit: rebuild HANYA komponen/section yang berkaitan dengan temuan
fatal di Langkah 1, pakai component library dari Langkah 6. JANGAN sentuh
section atau konten lain yang tidak ditandai fatal — ini scope yang lebih
sempit dari perilaku default `/design-agent:build` (yang biasanya
membangun section sesuai `sections` penuh di spec), jadi sebutkan
pembatasan ini secara eksplisit saat memanggil `/design-agent:build`.

### 9. Update registry
Untuk tiap finding yang berhasil di-fix:
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
`audits.json` → `status: "resolved"`.

### 10. Laporkan before/after ke user
Tampilkan ringkasan: komponen/section mana yang di-fix, referensi apa yang
dipakai (link/ID), dan screenshot atau deskripsi hasil akhir — jangan cuma
bilang "sudah selesai" tanpa bukti konkret.

## Yang TIDAK boleh dilakukan skill ini
- Menulis ulang logic `inspo`/`select`/`spec`/`build` sendiri — selalu
  panggil skill `design-agent` yang sudah ada
- Rebuild seluruh halaman kalau cuma sebagian yang ditandai fatal
- Menimpa component library yang sudah ada di lokasi konvensi project
- Melanjutkan tanpa plugin `design-agent` ter-install atau tanpa
  `.design/registry/` ter-init di project yang diaudit
- Skip checkpoint pemilihan referensi (`/design-agent:select`)
- Menandai finding `"design-fixed"` sebelum build benar-benar selesai
```

- [ ] **Step 2: Cross-check skill names against the real dependency**

Run: `grep -h "^name:" /Users/ihsanaziz/design-agent-plugin/skills/*/SKILL.md`
Expected output includes exactly: `name: init`, `name: inspo`,
`name: select`, `name: spec`, `name: build`. Confirm every
`/design-agent:*` reference in `skills/design-fix/SKILL.md` matches one of
these five names — fix any mismatch before committing.

- [ ] **Step 3: Commit**

```bash
cd /Users/ihsanaziz/garnish-plugin
git add skills/design-fix
git commit -m "feat: tulis skill design-fix, orchestrator ke pipeline design-agent"
```

---

### Task 5: Reconcile top-level docs with the completed skill set

**Files:**
- Modify: `README.md`
- Modify: `CLAUDE.md`
- Modify: `CONTEXT.md`
- Modify: `.claude-plugin/plugin.json`

- [ ] **Step 1: Update `README.md` "Status" section**

The version written in Task 1 Step 5 already reflects the completed state
(no "belum ada" language) — read the file back and confirm the "Status"
section says all four skills (`init`, `check`, `content-fix`, `design-fix`)
are present. No edit needed if Task 1's content was written exactly as
specified; this step is a verification, not a new edit.

- [ ] **Step 2: Update `CLAUDE.md` struktur project block**

Already written correctly in Task 1 Step 3 (includes `design-fix/SKILL.md`
in the tree, and rule #9 about design-fix scope). Read back and confirm —
verification step, no new edit expected.

- [ ] **Step 3: Update `CONTEXT.md` "Prioritas Eksekusi" section**

The version from Task 1 Step 4 still says "Stretch goal kalau waktu masih
ada: integrasi ke design-agent + component library buat fix desain" —
this is now DONE, not a stretch goal pending. Edit this section:

Find this block in `CONTEXT.md`:
```markdown
## Prioritas Eksekusi (kalau waktu 2 hari mepet, urutan ini yang dipegang)

1. **Wajib solid**: skill `garnish:check` — deteksi gejala fatal (terukur
   dulu, baru judgment)
2. **Wajib solid**: checkpoint "mau dibenerin yang mana" + minimal 1 jalur
   eksekusi (content-fix)
3. **Stretch goal kalau waktu masih ada**: integrasi ke `design-agent` +
   component library buat fix desain
```

Replace with:
```markdown
## Prioritas Eksekusi (kalau waktu 2 hari mepet, urutan ini yang dipegang)

1. **Wajib solid**: skill `garnish:check` — deteksi gejala fatal (terukur
   dulu, baru judgment)
2. **Wajib solid**: checkpoint "mau dibenerin yang mana" + minimal 1 jalur
   eksekusi (content-fix)
3. **Selesai**: integrasi ke `design-agent` + component library buat fix
   desain, lewat skill `garnish:design-fix` — semua 4 skill (`init`,
   `check`, `content-fix`, `design-fix`) sudah solid, bukan lagi stretch
   goal.
```

- [ ] **Step 4: Bump plugin version**

In `.claude-plugin/plugin.json`, change:
```json
  "version": "0.2.0",
```
to:
```json
  "version": "0.3.0",
```
(minor bump — `design-fix` is the last planned feature from the original
scope, matching semver convention used in `design-agent-plugin`.)

- [ ] **Step 5: Verify**

```bash
cd /Users/ihsanaziz/garnish-plugin
grep -c "design-fix" CONTEXT.md README.md CLAUDE.md
cat .claude-plugin/plugin.json
```
Expected: each file has at least one non-stale mention of `design-fix`;
`plugin.json` shows `"version": "0.3.0"`.

- [ ] **Step 6: Commit**

```bash
cd /Users/ihsanaziz/garnish-plugin
git add CONTEXT.md .claude-plugin/plugin.json
git commit -m "docs: tandai design-fix selesai (bukan stretch goal lagi), bump versi ke 0.3.0"
```

---

## Self-Review Notes

- **Spec coverage**: Task 1 = repo scaffold + `init` (component 1 of design
  spec). Task 2 = `check`, adapted to the REAL `design-agent` dependency
  (catches the stale `capture-reference.py` reference from the old zip
  drafts before it ships). Task 3 = `content-fix` (copied as-is, no
  dependency issues). Task 4 = `design-fix` (the new orchestrator, the
  design spec's main focus, with the `designRef` schema extension wired
  in). Task 5 = doc reconciliation + version bump.
- **Placeholder scan**: no `TBD`/`TODO` in any skill file content above —
  every step has complete Markdown/JSON to write verbatim.
- **Type/name consistency**: `designRef` field shape is identical between
  Task 1's `SCHEMA.md` content, Task 2's `check` (writes `designRef: null`
  on creation), and Task 4's `design-fix` (fills it in). Journal event
  names (`design_fix_started`, `design_fixed`) match between the `SCHEMA.md`
  written in Task 1 and the events `design-fix` actually appends in Task 4.
  `/design-agent:*` skill names referenced in Task 4 are verified against
  the real dependency in Task 4 Step 2, not assumed from the old zip drafts.

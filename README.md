# Belajar JavaScript & Node.js

Materi belajar JavaScript dan Node.js untuk pemula, ditulis dalam Bahasa Indonesia dan dibangun dengan [MkDocs](https://www.mkdocs.org/) + [Material for MkDocs](https://squidfunk.github.io/mkdocs-material/). Setiap bab dimulai dari konsep paling sederhana, dilengkapi contoh kode yang dijalankan dengan Node.js, output terminal, kesalahan umum, ringkasan, dan latihan beserta jawabannya.

Proyek ini dikembangkan dari [TopekoX/mkdocs-template](https://github.com/TopekoX/mkdocs-template).

## Isi materi

Materi dibagi dua bagian: **JavaScript Dasar** (bab 1 sampai 14) dan **Node.js** (bab 15 sampai 23). Pada sidebar, keduanya adalah dua grup yang bisa dibuka-tutup.

### JavaScript Dasar

| Bab | Topik | Level |
|---|---|---|
| 1 | Variables | Beginner |
| 2 | Operators & Expressions | Beginner |
| 3 | Strings & Numbers | Beginner |
| 4 | Conditionals | Beginner |
| 5 | Loops | Beginner |
| 6 | Functions | Beginner |
| 7 | Scope & Hoisting | Intermediate |
| 8 | Arrays | Intermediate |
| 9 | Objects | Intermediate |
| 10 | Error Handling & Debugging | Intermediate |
| 11 | Modules | Intermediate |
| 12 | Asynchronous JavaScript | Intermediate |
| 13 | Classes & OOP dasar | Intermediate |
| 14 | Proyek akhir (CLI pencatat pengeluaran) | Advanced |

Semua bab JavaScript Dasar sudah tersedia.

### Node.js

| Bab | Topik | Status |
|---|---|---|
| 15 | Pengenalan Node.js | Tersedia |
| 16 | npm dan `package.json` | Tersedia |
| 17 | Modul bawaan inti | Tersedia |
| 18 | Proses dan lingkungan | Tersedia |
| 19 | Events dan Streams | Tersedia |
| 20 | Server HTTP dasar | Tersedia |
| 21 | Express | Tersedia |
| 22 | REST API dan pengujian | Tersedia |
| 23 | Proyek akhir | Tersedia |

Semua bab Node.js sudah tersedia.

Bab dibaca berurutan: setiap bab hanya memakai konsep dari bab sebelumnya. Bab 1 sampai 10 dijalankan sebagai file tunggal (`node nama-file.js`). Bab 11 sampai 14 memakai folder proyek dengan `package.json` bertipe `module` (ES Modules). Bagian Node.js memakai ES Modules juga: folder `node-belajar` (bab 15, 17 sampai 20) dan proyek terpisah untuk bab yang memasang paket (`proyek-npm`, `belajar-express`, `api-tugas`, dan `api-pengeluaran`). Bab 16 dan 21 sampai 23 membutuhkan koneksi internet untuk `npm install`.

## Prasyarat

| Kebutuhan | Untuk apa | Versi yang diuji |
|---|---|---|
| Python 3 | Membangun situs dengan MkDocs | 3.14 |
| Node.js | Menjalankan contoh kode di materi | 24.11.1 |

Contoh kode ditulis dengan sintaks modern dan diharapkan jalan di Node.js versi LTS terbaru. Beberapa pesan error dan perilaku (misalnya deteksi otomatis ES Module di bab 11) dapat berbeda pada versi lama.

## Instalasi

Windows PowerShell:

```powershell
python -m venv .venv
.venv\Scripts\Activate.ps1
pip install -r requirements.txt
```

macOS/Linux: `python3 -m venv .venv && source .venv/bin/activate && pip install -r requirements.txt`.

## Menjalankan situs

```powershell
mkdocs serve              # http://127.0.0.1:8000 dengan live reload
mkdocs build --strict     # membangun ke site/; gagal jika ada tautan rusak atau peringatan
```

Material 9.x menampilkan kotak pemberitahuan "MkDocs 2.0" pada setiap build. Itu pengumuman dari upstream, bukan peringatan proyek, dan tidak menggagalkan `--strict`. `requirements.txt` membatasi `mkdocs<2`.

## Struktur proyek

```
mkdocs.yml                  identitas situs, tema, nav, ekstensi, aset
requirements.txt
docs/
  index.md                  halaman depan
  javascript/               JavaScript Dasar
    index.md                daftar bab
    01-variables.md ... 14-proyek-akhir.md
  nodejs/                   Node.js
    index.md                daftar bab
    15-pengenalan.md ...
  stylesheets/extra.css     penyesuaian kecil yang mengikuti tema
  javascripts/              konfigurasi MathJax dan Mermaid
  assets/vendor/mermaid/    Mermaid yang di-vendor (dengan LICENSE dan VERSION.txt)
  getting-started/, curriculum/, authoring/, examples/
                            halaman referensi bawaan template
examples/                   sumber contoh bawaan template, dipakai halaman referensi
```

Halaman referensi bawaan template (`getting-started/`, `curriculum/`, `authoring/`, `examples/`) tetap dibangun dan bisa dibuka lewat tautan, tetapi disembunyikan dari sidebar lewat `not_in_nav` di `mkdocs.yml`. Berkas `PRD.md`, `TECHNICAL_DESIGN.md`, `VALIDATION_REPORT.md`, dan `PROMPT_MKDOCS_TEMPLATE.md` di root adalah dokumen bawaan template, bukan bagian materi.

## Menambah atau mengubah bab

1. Buat `docs/javascript/NN-nama.md`. Judul halaman diambil dari `nav:`, sehingga bab **tidak** memakai heading `#`. Heading tertinggi adalah `##`, lalu `###` dan `####`.
2. Daftarkan di `nav:` pada `mkdocs.yml`, pada grup yang sesuai (`JavaScript Dasar` atau `Node.js`). Bab Node.js disimpan di `docs/nodejs/`. File yang tidak ada di `nav:` dan tidak masuk `not_in_nav` akan memicu peringatan, yang menggagalkan `mkdocs build --strict`.
3. Perbarui tabel di indeks bagiannya (`docs/javascript/index.md` atau `docs/nodejs/index.md`) dan tabel pada README ini.
4. Jalankan `mkdocs build --strict`.

Setiap bab memuat: tujuan pembelajaran, pendahuluan dan motivasi, penjelasan konsep, sintaks, contoh kode, penjelasan komponen kode, output yang diharapkan, penjelasan langkah eksekusi, contoh praktis, kesalahan umum, latihan, dan ringkasan.

### Aturan kode di materi

- **Semua contoh harus dijalankan** dengan Node.js sebelum dimasukkan, dan output yang tertulis harus sama dengan hasil nyata.
- Jangan menampilkan output yang tidak diverifikasi. Jika output dipersingkat (misalnya jejak stack), tandai dengan `Output (dipersingkat)` dan `...`.
- Jangan memakai konsep yang belum diajarkan di bab sebelumnya. Jika perlu menyinggungnya, beri tahu bab mana yang membahasnya.
- Contoh proyek multi-file (bab 11 sampai 15 dan seterusnya) memakai blok kode berjudul, misalnya ` ```javascript title="data.js" `.
- Gunakan Bahasa Indonesia yang sederhana. Istilah teknis tetap dalam bahasa Inggris dan dijelaskan saat pertama muncul.

### Fitur Markdown yang dipakai

| Fitur | Contoh |
|---|---|
| Blok kode dengan judul | ` ```javascript title="app.js" ` |
| Kotak catatan | `!!! tip "Judul"` lalu isi dengan indentasi 4 spasi |
| Jawaban yang bisa dilipat | `??? success "Jawaban"` |
| Diagram | ` ```mermaid ` (flowchart, classDiagram, dan lainnya) |

Penjelasan lengkap tersedia di halaman referensi `docs/authoring/` (buka lewat `/authoring/` di situs yang sedang berjalan).

## Tema

Tombol di header mengganti **Sistem → Terang → Gelap** (palet bawaan Material, diatur di `mkdocs.yml` pada `theme.palette`). Pilihan diingat di browser. Antarmuka situs berbahasa Indonesia (`theme.language: id`). CSS kustom harus memakai variabel `var(--md-*)`.

## Deploy ke GitHub Pages

Belum pernah dideploy. Untuk men-deploy:

1. Buat repositori GitHub dan push proyek ini ke `main`.
2. Pastikan `site_url` di `mkdocs.yml` sesuai alamat situsmu. Saat ini bernilai `https://topekox.my.id/mkdocs-template/` (domain kustom GitHub Pages). Tanpa domain kustom, bentuknya `https://<user>.github.io/<repo>/`. Nilai ini dipakai untuk sitemap dan tautan kanonik. Jika ingin tombol repositori di header kembali, tambahkan juga `repo_url` dan `repo_name`.
3. Jalankan `mkdocs gh-deploy --strict`. Perintah ini membangun situs dan mendorongnya ke branch `gh-pages`.
4. Di GitHub: **Settings → Pages → Build and deployment → Deploy from a branch → `gh-pages` / root**.

## Ketergantungan jaringan saat situs dibuka

| Ketergantungan | Kapan | Catatan |
|---|---|---|
| `cdn.jsdelivr.net` (MathJax 3.2.2) | setiap halaman | Dimuat oleh `extra_javascript`. Materi JavaScript tidak memakai rumus, jadi baris ini bisa dihapus dari `mkdocs.yml` jika tidak ingin ada permintaan ke CDN |
| Google Fonts | tidak pernah | `theme.font: false` memakai font sistem |
| Mermaid | tidak pernah | sudah di-vendor di `docs/assets/vendor/mermaid/` |

Contoh kode di materi yang memakai jaringan (misalnya `fetch`) hanya dijelaskan dan tidak dijalankan.

## Lisensi

Dirilis di bawah [MIT License](LICENSE). Pustaka Mermaid yang di-vendor di `docs/assets/vendor/mermaid/` juga berlisensi MIT dan menyimpan berkas `LICENSE` sendiri. Perbarui pemegang hak cipta di `LICENSE` sesuai kebutuhan.

## Tujuan pembelajaran

Setelah mempelajari bab ini, kamu dapat:

- Menjelaskan apa itu npm, paket, dan registry.
- Membuat dan membaca `package.json`.
- Memasang, memakai, dan menghapus paket.
- Membaca nomor versi (*semantic versioning*) dan simbol `^` serta `~`.
- Membedakan `dependencies` dan `devDependencies`.
- Menjelaskan fungsi `node_modules` dan `package-lock.json`, serta apa yang boleh masuk ke Git.
- Membuat dan menjalankan skrip dengan `npm run`, serta memakai `npx`.
- Menilai sebuah paket sebelum memakainya.

Bab ini melanjutkan bab 15 dan memakai ES Modules (bab 11). Semua perintah `npm` membutuhkan koneksi internet.

## Pendahuluan: tidak perlu menulis semuanya sendiri

Bayangkan kamu membangun rumah. Kamu tidak membuat paku, kabel, dan keran sendiri, melainkan membelinya di toko bahan bangunan dan fokus pada rumahnya. Dunia pemrograman bekerja serupa: untuk urusan umum seperti mengolah tanggal, membuat server web, atau membaca file konfigurasi, sudah ada kode siap pakai buatan orang lain.

Kumpulan kode siap pakai itu disebut **paket** (*package*). Node.js punya "toko" terbesar di dunia untuk paket JavaScript. Tiga hal yang berbeda sering disebut dengan nama yang sama, jadi bedakan dulu:

| Istilah | Artinya |
|---|---|
| **npm registry** | Gudang online (npmjs.com) yang menyimpan jutaan paket |
| **npm** (alat) | Program baris perintah `npm` yang ikut terpasang bersama Node.js, untuk mengunduh dan mengelola paket |
| **package.json** | Berkas di proyekmu yang mencatat identitas proyek dan paket apa saja yang dibutuhkan |

## `package.json`: kartu identitas proyek

Buat folder proyek baru dan inisialisasi dengan `npm init -y`. Opsi `-y` (*yes*) menjawab semua pertanyaan dengan nilai bawaan.

```bash
mkdir proyek-npm
cd proyek-npm
npm init -y
```

Output (contoh, alamat foldermu akan berbeda):

```text
Wrote to C:\Users\kamu\proyek-npm\package.json:

{
  "name": "proyek-npm",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "keywords": [],
  "author": "",
  "license": "ISC",
  "type": "commonjs"
}
```

Berkas `package.json` sekarang ada di folder itu. Isinya JSON (bab 9) dengan field-field berikut:

| Field | Fungsi |
|---|---|
| `name` | Nama proyek. Diambil dari nama folder |
| `version` | Versi proyekmu, mengikuti aturan *semantic versioning* (dibahas di bawah) |
| `description`, `keywords`, `author`, `license` | Informasi tentang proyek |
| `main` | File utama jika proyek ini dipakai sebagai paket |
| `scripts` | Perintah pintasan yang dijalankan dengan `npm run` |
| `type` | `"module"` untuk ES Modules atau `"commonjs"` untuk CommonJS (bab 11) |
| `dependencies` | Paket yang dibutuhkan agar program berjalan (muncul setelah kamu memasang paket) |
| `devDependencies` | Paket yang hanya dibutuhkan saat pengembangan |

Nilai bawaan `type` adalah `"commonjs"`. Karena kita memakai ES Modules, ubah menjadi `"module"`. Perintah `npm pkg set` mengubah isi `package.json` tanpa perlu membukanya:

```bash
npm pkg set type=module
npm pkg get type
```

Output:

```text
"module"
```

Satu pengaturan penting lain untuk proyek aplikasi (bukan paket yang akan diterbitkan): `"private": true`, yang mencegah proyek tak sengaja diunggah ke registry.

```bash
npm pkg set private=true --json
```

Opsi `--json` penting di sini: tanpa opsi itu npm menulis `true` sebagai teks (`"true"`), bukan sebagai nilai boolean.

## Memasang dan memakai paket

### Gagal sebelum dipasang

Kita akan memakai paket **dayjs**, pustaka kecil untuk mengolah tanggal. Buat file `app.js` yang memakainya, sebelum paketnya dipasang:

```javascript title="proyek-npm/app.js"
import dayjs from "dayjs";

const tanggal = dayjs("2025-03-01");

console.log(tanggal.format("DD/MM/YYYY"));
console.log(tanggal.add(10, "day").format("YYYY-MM-DD"));
console.log(tanggal.diff(dayjs("2025-02-01"), "day"));
```

```bash
node app.js
```

Output (dipersingkat):

```text
Error [ERR_MODULE_NOT_FOUND]: Cannot find package 'dayjs'
```

Perhatikan perbedaan dengan bab 11: alamat modul milik sendiri diawali `./`, sedangkan nama tanpa `./` (seperti `"dayjs"`) berarti **paket**, yang dicari Node.js di folder `node_modules`. Karena belum ada, error muncul.

### Memasang

Pasang dayjs. Kita menyebut versinya (`@1.11.23`) agar hasil di bab ini sama dengan milikmu:

```bash
npm install dayjs@1.11.23
```

Output (contoh, waktunya berbeda-beda):

```text
added 1 package, and audited 2 packages in 742ms

found 0 vulnerabilities
```

Tiga hal terjadi:

1. Folder **`node_modules`** dibuat, berisi kode dayjs yang diunduh dari registry.
2. Field **`dependencies`** di `package.json` bertambah, mencatat paket dan versinya.
3. Berkas **`package-lock.json`** dibuat, mencatat versi persis dari setiap paket yang terpasang.

Sekarang program bisa dijalankan:

```bash
node app.js
```

Output:

```text
01/03/2025
2025-03-11
28
```

Isi `package.json` sekarang kira-kira:

```json
{
  "name": "proyek-npm",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "keywords": [],
  "author": "",
  "license": "ISC",
  "type": "module",
  "private": true,
  "dependencies": {
    "dayjs": "^1.11.23"
  }
}
```

Perintah `npm install` memasang satu atau lebih paket. Beberapa bentuk yang umum:

| Perintah | Fungsi |
|---|---|
| `npm install nama-paket` | Memasang versi terbaru |
| `npm install nama-paket@1.2.3` | Memasang versi tertentu |
| `npm install` | Memasang **semua** paket yang tercatat di `package.json` |
| `npm uninstall nama-paket` | Menghapus paket |

## Memahami nomor versi

Pada `"dayjs": "^1.11.23"`, ada dua hal: nomor versi `1.11.23` dan awalan `^`.

### Semantic versioning

Hampir semua paket memakai tiga angka **MAJOR.MINOR.PATCH**, misalnya `1.11.23`:

| Bagian | Naik ketika | Contoh |
|---|---|---|
| **MAJOR** (`1`) | Ada perubahan yang **bisa merusak** kode lama | `1.x.x` menjadi `2.0.0` |
| **MINOR** (`11`) | Ada fitur baru yang **tetap kompatibel** | `1.11.x` menjadi `1.12.0` |
| **PATCH** (`23`) | Hanya perbaikan bug | `1.11.23` menjadi `1.11.24` |

### Simbol `^` dan `~`

Di `package.json`, awalan di depan versi menentukan versi mana saja yang boleh dipasang saat `npm install` dijalankan:

| Penulisan | Artinya | Contoh `^1.2.3` / `~1.2.3` / `1.2.3` |
|---|---|---|
| `^1.2.3` | Boleh naik MINOR dan PATCH, **tidak boleh** naik MAJOR | `1.2.3` sampai di bawah `2.0.0` |
| `~1.2.3` | Boleh naik PATCH saja | `1.2.3` sampai di bawah `1.3.0` |
| `1.2.3` | Harus persis | hanya `1.2.3` |

`npm install` menulis `^` secara bawaan. Kamu bisa mencobanya langsung dengan `npx` dan paket **semver**, yang dipakai npm sendiri untuk menghitung ini:

```bash
npx --yes semver@7.8.5 -r "^1.2.3" 1.2.2 1.2.9 1.9.0 2.0.0
```

Output:

```text
1.2.9
1.9.0
```

Perintah itu menampilkan versi yang memenuhi rentang `^1.2.3`. `1.2.2` terlalu rendah dan `2.0.0` naik MAJOR, jadi keduanya tidak ikut.

## `npx`: menjalankan alat tanpa memasangnya

`npx` (bagian dari npm) mengunduh sebuah paket yang punya perintah, menjalankannya, lalu selesai. Cocok untuk alat yang hanya dipakai sesekali. Opsi `--yes` menjawab "ya" pada pertanyaan konfirmasi unduhan. Contoh di atas dengan `semver` memakainya.

Jika paket sudah terpasang di proyek, `npx` memakai yang ada di `node_modules` lebih dulu. Itu cara menjalankan alat pengembangan seperti `npx prettier` (lihat bagian berikut).

## `devDependencies`: paket untuk pengembangan

Sebagian paket hanya berguna saat kamu menulis kode, bukan saat program berjalan di server, misalnya alat perapi kode. Pasang dengan `--save-dev` (singkatnya `-D`):

```bash
npm install --save-dev prettier@3.9.10
```

Output (contoh):

```text
added 1 package, and audited 3 packages in 660ms

found 0 vulnerabilities
```

Sekarang `package.json` punya dua kelompok:

```json
  "dependencies": {
    "dayjs": "^1.11.23"
  },
  "devDependencies": {
    "prettier": "^3.9.10"
  }
```

| | `dependencies` | `devDependencies` |
|---|---|---|
| Dibutuhkan saat | Program berjalan | Pengembangan, pengujian, perapian kode |
| Contoh | Express, dayjs | Prettier, alat pengujian |
| Dipasang di server produksi | Ya | Tidak, dengan `npm install --omit=dev` |

Daftar semua paket yang terpasang dilihat dengan `npm ls`. Opsi `--depth=0` hanya menampilkan paket yang kamu pasang langsung, tanpa paket-paket yang dibutuhkan paket itu:

```bash
npm ls --depth=0
```

Output (contoh):

```text
proyek-npm@1.0.0 C:\Users\kamu\proyek-npm
├── dayjs@1.11.23
└── prettier@3.9.10
```

## `node_modules` dan `package-lock.json`

| Berkas atau folder | Isi | Masuk Git? |
|---|---|---|
| `package.json` | Daftar paket yang kamu butuhkan, dengan rentang versi | **Ya** |
| `package-lock.json` | Versi **persis** dari setiap paket, termasuk paket di dalam paket | **Ya** |
| `node_modules/` | Hasil unduhan, bisa berukuran ratusan megabita | **Tidak** |

`node_modules` bisa selalu dibuat ulang dari `package.json` dan `package-lock.json`, jadi tidak perlu disimpan di Git. Daftarkan di berkas `.gitignore`:

```text title="proyek-npm/.gitignore"
node_modules/
```

`package-lock.json` memastikan semua orang (dan server) memasang versi yang **sama persis**, walaupun `package.json` hanya menulis rentang seperti `^1.11.23`. Tanpa berkas itu, dua orang yang menjalankan `npm install` di hari berbeda bisa mendapat versi yang berbeda.

Buktikan bahwa `node_modules` bisa dibuat ulang. Hapus foldernya, lalu pasang lagi dari `package.json`:

```bash
rm -rf node_modules
npm install
node app.js
```

Output (dipersingkat):

```text
01/03/2025
2025-03-11
28
```

Perintah `rm -rf` berlaku di Git Bash, macOS, dan Linux. Di PowerShell gunakan `Remove-Item -Recurse -Force node_modules`. Setelah `npm install`, program berjalan lagi seperti sebelumnya. Inilah yang kamu lakukan setiap kali mengunduh proyek orang lain: jalankan `npm install` dulu.

## Skrip: pintasan perintah

Field `scripts` menyimpan perintah yang diberi nama, dijalankan dengan `npm run nama`. Tambahkan tiga skrip:

```bash
npm pkg set scripts.start="node app.js"
npm pkg set scripts.dev="node --watch app.js"
npm pkg set scripts.format="prettier --write ."
```

Hasilnya di `package.json`:

```json
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "start": "node app.js",
    "dev": "node --watch app.js",
    "format": "prettier --write ."
  },
```

Jalankan dengan:

```bash
npm start
```

Output:

```text
> proyek-npm@1.0.0 start
> node app.js

01/03/2025
2025-03-11
28
```

Dua baris pertama dicetak oleh npm untuk menunjukkan perintah apa yang dijalankan. Tiga nama skrip bersifat khusus dan tidak memerlukan kata `run`: `npm start`, `npm test`, dan `npm stop`. Skrip lain dijalankan dengan `npm run nama`, misalnya `npm run dev`. Perintah `npm run` tanpa nama menampilkan daftar skrip yang ada.

Skrip `format` memakai Prettier yang terpasang di `node_modules`. Di dalam `scripts`, npm otomatis mencari perintah di `node_modules`, jadi `prettier` bisa ditulis tanpa `npx`:

```bash
npm run format
```

Output (contoh, waktunya berbeda-beda):

```text
> proyek-npm@1.0.0 format
> prettier --write .

app.js 24ms (unchanged)
package-lock.json 2ms (unchanged)
package.json 1ms (unchanged)
```

Manfaat skrip: perintah panjang cukup dihafal sekali dengan nama pendek, dan siapa pun yang membuka proyek bisa melihat cara menjalankannya di `package.json`.

## Memperbarui dan menghapus paket

| Perintah | Fungsi |
|---|---|
| `npm outdated` | Menampilkan paket yang punya versi lebih baru |
| `npm update` | Memperbarui paket dalam batas rentang di `package.json` |
| `npm uninstall nama-paket` | Menghapus paket dan mencoret dari `package.json` |
| `npm audit` | Memeriksa paket yang terpasang terhadap daftar celah keamanan yang diketahui |

Hapus Prettier dengan:

```bash
npm uninstall prettier
```

Output (contoh):

```text
removed 1 package, and audited 2 packages in 1s

found 0 vulnerabilities
```

Setelah ini, field `devDependencies` hilang dari `package.json` karena tidak ada lagi isinya.

## Memilih paket dengan bijak

Siapa pun boleh mengunggah paket ke registry, sehingga kualitasnya beragam. Kode dari paket berjalan dengan hak yang sama seperti kodemu, jadi memasang paket sembarangan sama dengan mempercayakan komputermu pada orang asing. Sebelum memasang, periksa:

- **Apakah memang perlu?** Banyak kebutuhan sudah dilayani Node.js sendiri (bab 17). Jangan memasang paket hanya untuk satu fungsi sederhana.
- **Seberapa banyak dipakai?** Lihat jumlah unduhan mingguan di halaman paket di npmjs.com.
- **Apakah dirawat?** Periksa tanggal rilis terakhir dan apakah masalah (*issues*) ditanggapi.
- **Seberapa besar?** Paket yang membawa banyak paket lain memperbesar risiko dan ukuran `node_modules`.
- **Lisensinya apa?** Pastikan cocok dengan penggunaanmu.
- **Namanya benar?** Penyerang kadang menerbitkan paket dengan nama mirip (`dayjs` vs `dayis`) berharap kamu salah ketik.

## Kesalahan umum

### 1. Lupa `npm install` setelah mengunduh proyek

Folder `node_modules` tidak ikut di Git, jadi setiap proyek yang baru diunduh menghasilkan `Cannot find package` sampai kamu menjalankan `npm install`.

### 2. Memasang di folder yang salah

`npm` bekerja di folder tempat perintahnya dijalankan. Pastikan terminalmu berada di folder yang berisi `package.json` proyekmu.

### 3. Memasukkan `node_modules` ke Git

Folder ini besar dan bisa dibuat ulang. Selalu tulis `node_modules/` di `.gitignore`.

### 4. Menghapus atau mengedit `package-lock.json` manual

Berkas itu dikelola oleh npm. Menghapusnya membuat versi paket bisa berubah tanpa disadari. Biarkan npm yang memperbaruinya, dan masukkan ke Git.

### 5. Salah mengetik nama skrip

```bash
npm run satrt
```

Output (dipersingkat):

```text
npm error Missing script: "satrt"
```

Pesan itu juga menyarankan `npm run` untuk melihat daftar skrip yang tersedia.

### 6. Mengira `^` selalu aman

Pengembang paket bisa keliru dan merusak sesuatu dalam rilis MINOR. Karena itu `package-lock.json` penting, dan versi baru sebaiknya diuji setelah `npm update`.

### 7. Memasang paket global tanpa perlu

Opsi `-g` memasang paket untuk seluruh komputer, bukan per proyek. Itu membuat proyek bergantung pada komputer tertentu. Pasang per proyek, dan jalankan lewat skrip atau `npx`.

### 8. Memasang paket hanya untuk hal sepele

Satu fungsi kecil jarang layak menambah paket baru. Tulis sendiri, atau cari di API bawaan Node.js dulu.

## Ringkasan

- **npm** adalah alat dan sekaligus registry paket JavaScript. Paket dipasang dengan `npm install`.
- `package.json` mencatat identitas proyek, skrip, dan daftar paket. Buat dengan `npm init -y`, ubah dengan `npm pkg set`.
- Paket yang namanya tanpa `./` dicari di `node_modules`. Alamat dengan `./` adalah modul milik sendiri.
- Versi berbentuk `MAJOR.MINOR.PATCH`. `^` mengizinkan naik MINOR dan PATCH, `~` hanya PATCH.
- `dependencies` dibutuhkan saat program berjalan, `devDependencies` hanya saat pengembangan (`--save-dev`).
- Masukkan `package.json` dan `package-lock.json` ke Git. Jangan masukkan `node_modules`.
- `npm install` tanpa nama memasang ulang semua paket dari `package.json`.
- `scripts` + `npm run` membuat pintasan perintah. `start`, `test`, dan `stop` tidak memerlukan `run`.
- `npx` menjalankan perintah dari paket tanpa memasangnya permanen.
- Pilih paket dengan hati-hati, dan utamakan fitur bawaan Node.js.

## Latihan

Kerjakan sendiri dulu sebelum melihat jawaban. Latihan memakai folder `proyek-npm` dari bab ini.

### Latihan 1: Rentang versi

Dengan `npx semver`, tentukan versi mana dari `1.4.0`, `1.4.9`, `1.5.0`, `2.0.0` yang memenuhi `~1.4.0`. Tebak dulu hasilnya.

??? success "Jawaban"
    ```bash
    npx --yes semver@7.8.5 -r "~1.4.0" 1.4.0 1.4.9 1.5.0 2.0.0
    ```

    ```text
    1.4.0
    1.4.9
    ```

    `~1.4.0` hanya mengizinkan naik PATCH, jadi `1.5.0` dan `2.0.0` tidak memenuhi.

### Latihan 2: Hari dari sebuah tanggal

Buat file `hari.js` yang memakai dayjs untuk menampilkan nama hari dari tanggal `2025-12-25` (format `dddd`).

??? success "Contoh jawaban"
    ```javascript title="proyek-npm/hari.js"
    import dayjs from "dayjs";

    console.log(dayjs("2025-12-25").format("dddd"));
    ```

    ```bash
    node hari.js
    ```

    ```text
    Thursday
    ```

### Latihan 3: Skrip sendiri

Tambahkan skrip bernama `hari` yang menjalankan `hari.js`, lalu jalankan dengan `npm run`.

??? success "Contoh jawaban"
    ```bash
    npm pkg set scripts.hari="node hari.js"
    npm run hari
    ```

    ```text
    > proyek-npm@1.0.0 hari
    > node hari.js

    Thursday
    ```

### Latihan 4: Apa yang masuk Git?

Dari berkas dan folder berikut di `proyek-npm`, mana yang sebaiknya dimasukkan ke Git dan mana yang tidak: `app.js`, `package.json`, `package-lock.json`, `node_modules/`, `.gitignore`, `hari.js`. Jelaskan alasannya.

??? success "Jawaban"
    - **Masuk Git:** `app.js`, `hari.js`, `package.json`, `package-lock.json`, dan `.gitignore`. Semuanya adalah sumber proyek, atau pencatat versi yang dibutuhkan agar orang lain bisa membangun ulang.
    - **Tidak masuk Git:** `node_modules/`. Folder ini hasil unduhan yang besar dan bisa dibuat ulang dengan `npm install`.

### Latihan 5: Perbaiki error

Kamu mengunduh proyek dari temanmu, lalu `node app.js` menghasilkan `Cannot find package 'dayjs'`. Apa penyebabnya dan apa perbaikannya?

??? success "Jawaban"
    Folder `node_modules` tidak ikut di Git, sehingga paket dayjs belum terpasang di komputermu. Jalankan `npm install` di folder proyek yang berisi `package.json`, lalu jalankan `node app.js` lagi.

### Latihan 6: Paket pengembangan

Pasang kembali `prettier@3.9.10` sebagai `devDependencies`, lalu tampilkan isi `devDependencies` dengan `npm pkg get`.

??? success "Jawaban"
    ```bash
    npm install --save-dev prettier@3.9.10
    npm pkg get devDependencies
    ```

    ```text
    {
      "prettier": "^3.9.10"
    }
    ```

### Latihan 7: Menilai paket

Kamu menemukan dua paket untuk keperluan yang sama. Paket A diunduh 20 juta kali seminggu, dirilis terakhir dua minggu lalu, dan hanya membawa 2 paket lain. Paket B diunduh 300 kali seminggu, tidak diperbarui selama empat tahun, dan membawa 40 paket lain. Mana yang kamu pilih, dan faktor apa saja yang kamu pertimbangkan?

??? success "Jawaban"
    Pilih **paket A**. Ia banyak dipakai (lebih banyak mata yang menemukan masalah), masih dirawat (celah keamanan cepat diperbaiki), dan punya sedikit paket tambahan (risiko dan ukuran lebih kecil). Paket B berisiko: jarang dipakai, tidak dirawat, dan membawa banyak paket lain yang masing-masing bisa menjadi sumber masalah.

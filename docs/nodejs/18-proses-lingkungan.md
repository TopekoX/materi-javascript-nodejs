## Tujuan pembelajaran

Setelah mempelajari bab ini, kamu dapat:

- Membaca konfigurasi dari variabel lingkungan (`process.env`) dan berkas `.env`.
- Menjelaskan bahwa semua nilai variabel lingkungan berupa teks, dan mengubahnya dengan benar.
- Membedakan *stdin*, *stdout*, dan *stderr*, serta mengalihkannya dari terminal.
- Membaca masukan pengguna dengan `node:readline`, baik interaktif maupun dari pipe.
- Menangani sinyal seperti `SIGINT` (Ctrl + C) untuk menutup program dengan rapi.
- Memakai event `exit` dan `beforeExit`, serta penangkap error global.
- Menulis alat baris perintah yang bersikap baik terhadap program lain (output bersih, exit code benar).

Bab ini memakai bab 12 (async), bab 14 (argumen dan exit code), bab 15 (`process`), dan bab 17 (`parseArgs`). Semua contoh dijalankan dari folder `node-belajar`.

## Pendahuluan: program hidup di dalam lingkungan

Sebuah program tidak berjalan sendirian. Ia dijalankan oleh seseorang atau program lain, di komputer dengan pengaturan tertentu, dan harus berkomunikasi dengan sekitarnya. Bayangkan seorang pegawai di kantor:

| Di kantor | Di program Node.js |
|---|---|
| Memo pengaturan yang ditempel di dinding (alamat server, mode kerja) | **Variabel lingkungan** (`process.env`) |
| Kotak surat masuk | **stdin** (masukan standar) |
| Kotak surat keluar | **stdout** (keluaran standar) |
| Kotak pengaduan | **stderr** (keluaran error) |
| Alarm kebakaran yang menyuruh semua orang keluar dengan tertib | **Sinyal** (`SIGINT`, `SIGTERM`) |
| Laporan akhir hari: "beres" atau "ada masalah" | **Exit code** |

Alat baris perintah yang baik memanfaatkan semuanya. Bab 14 sudah menyentuh sebagian (argumen dan exit code). Bab ini melengkapinya.

## Variabel lingkungan

**Variabel lingkungan** adalah pasangan nama dan nilai yang disimpan sistem operasi dan diwariskan ke setiap program yang dijalankan. Contohnya `PATH` (daftar folder tempat program dicari). Di Node.js, semuanya ada di objek **`process.env`**. Simpan sebagai `config.js`:

```javascript title="config.js"
const PORT = Number(process.env.PORT) || 3000;
const DEBUG = process.env.DEBUG === "true";
const LINGKUNGAN = process.env.NODE_ENV || "development";

console.log({ PORT, DEBUG, LINGKUNGAN });
console.log(typeof process.env.PATH, process.env.TIDAK_ADA);
```

```bash
node config.js
```

Output:

```text
{ PORT: 3000, DEBUG: false, LINGKUNGAN: 'development' }
string undefined
```

Fungsi program ini adalah membaca **konfigurasi dari luar kode**: nomor port, mode debug, dan jenis lingkungan. Kode yang sama bisa berjalan di laptopmu dan di server, dengan pengaturan berbeda, tanpa mengubah satu baris pun.

Hal-hal yang perlu dipahami tentang `process.env`:

| Sifat | Penjelasan |
|---|---|
| Nilainya selalu **teks** | `PORT=4000` menjadi `"4000"`, bukan angka `4000`. Ubah dengan `Number(...)` |
| Yang tidak ada bernilai `undefined` | Karena itu pola `Number(process.env.PORT) \|\| 3000` memberi nilai bawaan |
| Tidak ada tipe boolean | `DEBUG=true` menjadi `"true"`. Bandingkan dengan `=== "true"` |

Pola `Number(x) || 3000` memanfaatkan `Number(undefined)` yang menghasilkan `NaN` (falsy), sehingga `||` memilih 3000 (bab 3 dan 4).

### Mengatur variabel lingkungan

Cara memberi nilai saat menjalankan program berbeda di tiap terminal:

| Terminal | Perintah |
|---|---|
| Git Bash, macOS, Linux | `PORT=4000 node config.js` |
| PowerShell | `$env:PORT = "4000"; node config.js` |
| Command Prompt (cmd) | `set PORT=4000` lalu `node config.js` |

Karena rumit dan berbeda-beda, cara yang lebih nyaman adalah berkas `.env` (dibahas berikut).

## Berkas `.env`

Berkas `.env` menyimpan variabel lingkungan untuk proyekmu, satu per baris dengan format `NAMA=nilai`. Node.js bisa memuatnya langsung dengan opsi **`--env-file`**. Buat berkas `.env` di folder `node-belajar`:

```text title=".env"
# Pengaturan aplikasi
PORT=4000
DEBUG=true
NODE_ENV=production
KUNCI="rahasia dengan spasi"
```

Jalankan dengan opsi `--env-file`:

```bash
node --env-file=.env config.js
```

Output:

```text
{ PORT: 4000, DEBUG: true, LINGKUNGAN: 'production' }
string undefined
```

Aturan penulisan `.env`:

- Satu pasangan per baris: `NAMA=nilai`. Tanpa spasi di sekitar tanda `=`.
- Baris yang diawali `#` adalah komentar.
- Nilai yang mengandung spasi ditulis dengan tanda kutip.

Beberapa variasi yang berguna:

| Perintah | Fungsi |
|---|---|
| `node --env-file=.env app.js` | Memuat `.env`. Gagal dengan exit code 9 jika berkasnya tidak ada |
| `node --env-file-if-exists=.env app.js` | Memuat jika ada, melanjutkan tanpa error jika tidak ada |
| `process.loadEnvFile(".env")` | Memuat dari dalam kode, di tengah program |
| `util.parseEnv(teks)` | Mengurai teks berformat `.env` menjadi objek, tanpa mengubah `process.env` |

!!! warning "Jangan masukkan `.env` ke Git"
    Berkas `.env` biasanya berisi rahasia: kata sandi database, kunci API, token. Tulis `.env` di `.gitignore`. Sebagai gantinya, simpan berkas `.env.example` di Git yang berisi nama variabelnya saja (tanpa nilai rahasia) supaya orang lain tahu apa yang harus mereka isi.

## stdin, stdout, dan stderr

Setiap program Node.js punya tiga "saluran" standar:

| Saluran | Objek | Arah | Fungsi |
|---|---|---|---|
| **stdin** | `process.stdin` | masuk | Masukan, dari keyboard atau dari program lain lewat pipe |
| **stdout** | `process.stdout` | keluar | Hasil normal program |
| **stderr** | `process.stderr` | keluar | Pesan error dan peringatan |

Kamu sudah memakainya: `console.log` menulis ke stdout, dan `console.error` menulis ke stderr. Di terminal keduanya tampak di layar yang sama, tetapi sebenarnya dua jalur terpisah. Pemisahan itu berguna karena keluaran bisa dialihkan sendiri-sendiri. Simpan sebagai `dua-saluran.js`:

```javascript title="dua-saluran.js"
console.log("hasil normal");
console.error("pesan error");
process.stdout.write("tanpa baris baru");
process.stdout.write("\n");
```

```bash
node dua-saluran.js
```

Output:

```text
hasil normal
pesan error
tanpa baris baru
```

Sekarang alihkan stdout ke berkas dengan `>`, dan stderr dengan `2>`:

```bash
node dua-saluran.js > hasil.txt 2> galat.txt
```

Perintah itu tidak menampilkan apa pun di layar. Isi kedua berkas:

```bash
cat hasil.txt
cat galat.txt
```

Output:

```text
hasil normal
tanpa baris baru
pesan error
```

`hasil.txt` hanya berisi keluaran normal, dan `galat.txt` hanya pesan error. (Untuk melihat isi berkas: `cat` di Git Bash, macOS, Linux, `Get-Content` di PowerShell, dan `type` di cmd.)

`process.stdout.write(teks)` mirip `console.log`, tetapi **tidak menambahkan baris baru**. Cocok untuk menulis di baris yang sama, misalnya indikator kemajuan.

!!! tip "Aturan penting untuk alat baris perintah"
    Hasil yang akan dipakai program lain harus ke **stdout**, sedangkan pesan untuk manusia (peringatan, kesalahan, log) ke **stderr**. Dengan begitu `node alat.js > hasil.txt` tidak mencampur pesan error ke dalam hasil.

## Membaca masukan pengguna

### Satu pertanyaan dengan `readline`

Modul `node:readline/promises` membaca masukan baris demi baris. Simpan sebagai `tanya.js`:

```javascript title="tanya.js"
import { createInterface } from "node:readline/promises";

const rl = createInterface({ input: process.stdin, output: process.stdout });

const nama = await rl.question("Nama kamu? ");
console.log(`Halo, ${nama}!`);

rl.close();
```

Jalankan secara interaktif dengan `node tanya.js`. Kamu akan melihat pertanyaan, mengetik jawaban, lalu menekan Enter:

```text
Nama kamu? Budi
Halo, Budi!
```

Untuk mencobanya tanpa mengetik, kirim jawabannya lewat **pipe** (`|`), yang menyambungkan stdout satu program ke stdin program lain:

```bash
echo Budi | node tanya.js
```

Output:

```text
Nama kamu? Halo, Budi!
```

(Pada pipe, jawaban tidak tampil di layar karena bukan diketik, sehingga `Halo, Budi!` menyambung di baris yang sama.)

Penjelasan kode:

| Kode | Artinya |
|---|---|
| `createInterface({ input, output })` | Membuat pembaca baris dari stdin yang menulis pertanyaan ke stdout |
| `await rl.question("...")` | Menampilkan pertanyaan lalu menunggu pengguna menekan Enter. Menghasilkan Promise berisi jawaban (teks) |
| `rl.close()` | Menutup pembaca. **Wajib**, agar program bisa berhenti |

Dua pertanyaan berurutan bekerja dengan baik saat diketik langsung:

```javascript title="kuis.js"
import { createInterface } from "node:readline/promises";

const rl = createInterface({ input: process.stdin, output: process.stdout });

const nama = await rl.question("Nama? ");
const umur = Number(await rl.question("Umur? "));

console.log(`${nama}, tahun depan kamu ${umur + 1} tahun.`);
rl.close();
```

Sesi interaktif (yang kamu ketik ada setelah tanda tanya):

```text
Nama? Budi
Umur? 20
Budi, tahun depan kamu 21 tahun.
```

Jawaban selalu berupa **teks**, sehingga umur diubah dengan `Number(...)` (bab 3).

### Banyak baris: `for await` dan `rl.prompt()`

`rl.question` cocok untuk satu atau dua pertanyaan. Untuk program yang membaca baris terus-menerus (menu, kalkulator, perintah berulang), gunakan pola `for await (const baris of rl)`. Simpan sebagai `menu.js`:

```javascript title="menu.js"
import { createInterface } from "node:readline";

const rl = createInterface({ input: process.stdin, output: process.stdout });
let total = 0;

rl.setPrompt("Jumlah (kosong = selesai)? ");
rl.prompt();

for await (const baris of rl) {
  const masukan = baris.trim();
  if (masukan === "") break;

  const angka = Number(masukan);
  if (Number.isNaN(angka)) {
    console.log("Bukan angka, diabaikan.");
  } else {
    total += angka;
    console.log(`Total sementara: ${total}`);
  }
  rl.prompt();
}

console.log(`Total akhir: ${total}`);
rl.close();
```

Mencobanya secara interaktif memberi tampilan seperti ini (yang kamu ketik ada setelah tanda tanya):

```text
Jumlah (kosong = selesai)? 100
Total sementara: 100
Jumlah (kosong = selesai)? abc
Bukan angka, diabaikan.
Jumlah (kosong = selesai)? 250
Total sementara: 350
Jumlah (kosong = selesai)? 
Total akhir: 350
```

Dan dengan pipe (sehingga bisa diulang persis):

```bash
printf '100\nabc\n250\n\n' | node menu.js
```

Output:

```text
Jumlah (kosong = selesai)? Total sementara: 100
Jumlah (kosong = selesai)? Bukan angka, diabaikan.
Jumlah (kosong = selesai)? Total sementara: 350
Jumlah (kosong = selesai)? Total akhir: 350
```

Penjelasan:

- `rl.setPrompt(...)` menentukan teks petunjuk, dan `rl.prompt()` menampilkannya.
- `for await (const baris of rl)` menerima setiap baris yang diketik (atau dikirim lewat pipe) satu per satu, dan berakhir ketika masukan habis atau kita `break`.
- Pola ini bekerja baik secara interaktif **maupun** lewat pipe (lihat kesalahan umum nomor 4 untuk alasan tidak memakai `rl.question` berulang).

### Membaca seluruh masukan dari pipe

Banyak alat Unix menerima data dari alat lain lewat pipe. Simpan sebagai `hitung-kata.js`:

```javascript title="hitung-kata.js"
let teks = "";

for await (const bagian of process.stdin) {
  teks += bagian;
}

const kata = teks.trim().split(" ");
console.log(`${kata.length} kata`);
```

```bash
echo satu dua tiga | node hitung-kata.js
```

Output:

```text
3 kata
```

`process.stdin` sendiri adalah aliran data (dibahas di bab 19), yang bisa dibaca dengan `for await`. Setiap potongan data yang datang ditambahkan ke `teks` sampai aliran selesai.

### Interaktif atau dari pipe?

Program sering perlu tahu apakah ada manusia di depannya. `process.stdin.isTTY` bernilai `true` jika masukan berasal dari terminal interaktif, dan `undefined` jika berasal dari pipe atau berkas. Simpan sebagai `mode.js`:

```javascript title="mode.js"
console.log(process.stdin.isTTY ? "interaktif" : "dari pipe");
```

```bash
echo x | node mode.js
```

Output:

```text
dari pipe
```

Dengan `node mode.js` langsung di terminal, hasilnya `interaktif`. Berguna misalnya untuk menampilkan pertanyaan hanya jika ada manusia, dan membaca diam-diam jika tidak.

## Sinyal dan menutup program dengan rapi

Saat kamu menekan `Ctrl + C`, sistem operasi mengirim **sinyal** `SIGINT` (*signal interrupt*) ke program. Secara bawaan Node.js langsung berhenti. Tetapi program yang punya pekerjaan setengah jalan, misalnya berkas yang sedang ditulis atau koneksi yang terbuka, sebaiknya menutup semuanya dulu. Caranya mendengarkan sinyal itu dengan `process.on`.

Dua sinyal yang paling penting:

| Sinyal | Datang dari |
|---|---|
| `SIGINT` | Pengguna menekan `Ctrl + C` di terminal |
| `SIGTERM` | Sistem atau pengelola proses meminta program berhenti baik-baik (misalnya saat server dimatikan) |

Simpan sebagai `tutup-rapi.js`:

```javascript title="tutup-rapi.js"
let tick = 0;

const timer = setInterval(() => {
  tick++;
  console.log("tick", tick);
}, 100);

function tutup(sinyal) {
  console.log(`\nMenerima ${sinyal}, membersihkan...`);
  clearInterval(timer);
  console.log("selesai rapi");
}

process.on("SIGINT", () => tutup("SIGINT"));
process.on("SIGTERM", () => tutup("SIGTERM"));

setTimeout(() => process.emit("SIGINT"), 250);
```

```bash
node tutup-rapi.js
```

Output:

```text
tick 1
tick 2

Menerima SIGINT, membersihkan...
selesai rapi
```

Penjelasan:

1. `setInterval` mencetak `tick` setiap 100 ms. Tanpa penanganan, program berjalan terus.
2. `process.on("SIGINT", fungsi)` mendaftarkan fungsi yang dipanggil ketika sinyal datang. Fungsi `tutup` membersihkan (di sini: `clearInterval`). Setelah interval dihentikan dan tidak ada pekerjaan tertunda, program berhenti sendiri (bab 15).
3. Baris terakhir (`process.emit("SIGINT")`) hanya **simulasi** supaya contoh ini berhenti tanpa kamu menekan `Ctrl + C`. Pada program sungguhan, baris itu dihapus, dan kamu yang menekan `Ctrl + C` kapan pun. Cobalah: hapus baris itu, jalankan, lalu tekan `Ctrl + C` setelah beberapa tick.

!!! note "Sinyal di Windows"
    Windows tidak mengenal sistem sinyal seperti macOS dan Linux. Pada Windows, `SIGINT` (dari `Ctrl + C`) tetap bekerja, tetapi `SIGTERM` tidak bisa dikirim dari program lain. Kode di atas tetap aman dijalankan di semua sistem.

## Event `exit` dan `beforeExit`

Node.js memberi dua kesempatan sebelum program benar-benar berhenti. Simpan sebagai `siklus.js`:

```javascript title="siklus.js"
process.on("beforeExit", (kode) => {
  console.log(`beforeExit, kode ${kode}`);
});

process.on("exit", (kode) => {
  console.log(`exit, kode ${kode}`);
});

console.log("mulai");
setTimeout(() => console.log("timer"), 20);
```

```bash
node siklus.js
```

Output:

```text
mulai
timer
beforeExit, kode 0
exit, kode 0
```

| Event | Kapan | Bisa menjadwalkan pekerjaan baru? |
|---|---|---|
| `beforeExit` | Ketika tidak ada lagi pekerjaan tertunda, sesaat sebelum berhenti | Ya. Jika kamu menjadwalkan sesuatu, program lanjut berjalan |
| `exit` | Tepat saat proses akan berhenti (termasuk setelah `process.exit()`) | **Tidak.** Hanya kode **sinkron** yang dijalankan |

Karena `exit` hanya menjalankan kode sinkron, timer atau `await` di dalamnya tidak akan sempat berjalan. Simpan sebagai `exit-async.js`:

```javascript title="exit-async.js"
process.on("exit", () => {
  console.log("handler exit");
  setTimeout(() => console.log("TIDAK PERNAH TERCETAK"), 0);
});

console.log("selesai");
```

```bash
node exit-async.js
```

Output:

```text
selesai
handler exit
```

Event `exit` cocok untuk pekerjaan beres-beres yang cepat dan sinkron, misalnya mencetak pesan penutup atau menulis log terakhir.

## Menangani error yang lolos

Di bab 10 kamu belajar `try...catch`. Tetapi error juga bisa lolos dari semua `catch`: pelemparan di dalam timer, atau Promise yang ditolak tanpa penanganan (bab 12). Node.js menyediakan dua penangkap terakhir. Simpan sebagai `penangkap.js`:

```javascript title="penangkap.js"
process.on("uncaughtException", (error) => {
  console.error("Error tidak tertangkap:", error.message);
  process.exitCode = 1;
});

process.on("unhandledRejection", (alasan) => {
  console.error("Promise ditolak tanpa penanganan:", alasan.message);
  process.exitCode = 1;
});

setTimeout(() => {
  throw new Error("ledakan di timer");
}, 10);

Promise.reject(new Error("gagal diam-diam"));
```

```bash
node penangkap.js
```

Output:

```text
Promise ditolak tanpa penanganan: gagal diam-diam
Error tidak tertangkap: ledakan di timer
```

Tanpa penangkap ini, kedua kejadian menghentikan program dengan menampilkan stack trace. Penangkap global bukan pengganti `try...catch`, melainkan jaring pengaman terakhir untuk **mencatat** apa yang terjadi dan menutup program dengan exit code yang benar. Setelah `uncaughtException`, keadaan program bisa tidak konsisten; kebiasaan yang aman adalah mencatat error lalu menghentikan program, bukan melanjutkan seolah tidak terjadi apa-apa.

## Contoh praktis: menjumlahkan angka dari masukan

Kita gabungkan variabel lingkungan, stdin, stderr, dan exit code menjadi satu alat. Program membaca angka satu per baris dari stdin, melewati baris yang bukan angka (dengan peringatan ke stderr), lalu mencetak total ke stdout:

```javascript title="jumlahkan.js"
import { createInterface } from "node:readline";

const mataUang = process.env.MATA_UANG || "Rp";
const rl = createInterface({ input: process.stdin });

let total = 0;
let valid = 0;

for await (const baris of rl) {
  const angka = Number(baris);
  if (baris.trim() === "" || Number.isNaN(angka)) {
    console.error(`Dilewati: "${baris}"`);
    continue;
  }
  total += angka;
  valid++;
}

if (valid === 0) {
  console.error("Tidak ada angka yang valid.");
  process.exitCode = 1;
} else {
  console.log(`Total ${valid} angka: ${mataUang} ${total}`);
}
```

Siapkan berkas masukan dan berkas konfigurasi:

```text title="angka.txt"
100
250
abc
50
```

```text title=".env.uang"
MATA_UANG=IDR
```

Kirim isi `angka.txt` ke program lewat pipe (`cat` di Git Bash, macOS, Linux; di PowerShell gunakan `Get-Content angka.txt | node jumlahkan.js`):

```bash
cat angka.txt | node jumlahkan.js
```

Output:

```text
Dilewati: "abc"
Total 3 angka: Rp 400
```

Sekarang ganti mata uang lewat variabel lingkungan:

```bash
cat angka.txt | node --env-file=.env.uang jumlahkan.js
```

Output (dipersingkat):

```text
Total 3 angka: IDR 400
```

(Baris `Dilewati: "abc"` tetap muncul di stderr; yang ditampilkan hanya stdout.) Terakhir, masukan tanpa satu pun angka yang valid:

```bash
echo | node jumlahkan.js
```

Output:

```text
Dilewati: ""
Tidak ada angka yang valid.
```

Program ini bersikap baik terhadap lingkungannya:

- Hasil (`Total ...`) ke stdout, peringatan dan error ke stderr, sehingga `node jumlahkan.js < angka.txt > hasil.txt` hanya menyimpan hasilnya.
- Mata uang bisa diubah tanpa mengedit kode, lewat `MATA_UANG`.
- Exit code 1 saat gagal, sehingga program lain bisa tahu bahwa masukannya bermasalah.
- Bekerja dengan masukan dari pipe maupun yang diketik langsung.

## Kesalahan umum

### 1. Mengira variabel lingkungan bukan teks

Simpan sebagai `debug-salah.js` dan buat `.env.debug`:

```text title=".env.debug"
DEBUG=false
PORT=4000
```

```javascript title="debug-salah.js"
if (process.env.DEBUG) {
  console.log("mode debug AKTIF");
} else {
  console.log("mode debug mati");
}

console.log(process.env.PORT + 1, Number(process.env.PORT) + 1);
```

```bash
node --env-file=.env.debug debug-salah.js
```

Output:

```text
mode debug AKTIF
40001 4001
```

`DEBUG=false` bernilai teks `"false"`, yang **truthy** (bab 4), sehingga mode debug justru aktif. Begitu juga `PORT + 1` menyambung teks menjadi `"40001"`. **Perbaikan:** bandingkan secara eksplisit (`process.env.DEBUG === "true"`) dan ubah angka dengan `Number(...)`.

### 2. Memasukkan `.env` ke Git

Rahasia yang sudah terlanjur masuk ke riwayat Git sulit dihapus dengan bersih. Tulis `.env` di `.gitignore` sejak awal.

### 3. Berkas `.env` tidak ditemukan

```bash
node --env-file=tidak-ada.env config.js
```

Output (dipersingkat):

```text
node.exe: tidak-ada.env: not found
```

Node.js berhenti dengan exit code 9 (nama programnya di pesan tergantung sistemmu). Gunakan `--env-file-if-exists` jika berkasnya memang opsional.

### 4. Beberapa `rl.question` dengan masukan dari pipe

Jawaban yang dikirim lewat pipe sudah masuk semua sebelum pertanyaan kedua diajukan, sehingga pertanyaan kedua menunggu selamanya. Jalankan `kuis.js` dengan hanya satu baris masukan:

```bash
echo Budi | node kuis.js
```

Output (dipersingkat):

```text
Warning: Detected unsettled top-level await
```

Program berhenti dengan exit code 13 karena menunggu jawaban kedua yang tidak pernah datang. Untuk banyak pertanyaan secara interaktif, `rl.question` aman. Untuk masukan yang bisa datang dari pipe atau berkas, pakai pola `for await (const baris of rl)`.

### 5. Lupa `rl.close()`

Tanpa `rl.close()`, stdin tetap terbuka dan program tidak berhenti sendiri di terminal interaktif. Selalu tutup `readline` setelah selesai.

### 6. Pekerjaan asinkron di dalam event `exit`

Seperti pada `exit-async.js`: yang dijalankan di `exit` hanya kode sinkron. Pekerjaan asinkron untuk beres-beres harus dilakukan sebelumnya, misalnya di handler `SIGINT` atau `beforeExit`.

### 7. Mencampur hasil dan pesan error di stdout

Jika alatmu mencetak peringatan dengan `console.log`, hasilnya tercampur saat dialihkan ke berkas atau dipakai program lain. Pesan untuk manusia ke `console.error`.

### 8. Menyimpan rahasia di dalam kode

Kata sandi atau kunci API yang ditulis langsung di file `.js` ikut ke Git dan terbaca siapa saja. Letakkan di `.env` atau variabel lingkungan.

### 9. Mengabaikan exit code

Program yang gagal tetapi keluar dengan kode 0 menipu skrip otomasi yang mengira semuanya sukses. Atur `process.exitCode = 1` untuk kegagalan.

## Ringkasan

- **`process.env`** menyimpan variabel lingkungan. Semua nilainya **teks** atau `undefined`: ubah dengan `Number(...)` dan bandingkan boolean secara eksplisit (`=== "true"`).
- Berkas **`.env`** + `node --env-file=.env` memuat konfigurasi tanpa pengaturan terminal. Jangan masukkan `.env` ke Git; sediakan `.env.example`.
- **stdout** untuk hasil, **stderr** untuk pesan error. Keduanya bisa dialihkan terpisah (`>` dan `2>`). `process.stdout.write` tidak menambah baris baru.
- **stdin** dibaca dengan `node:readline`: `rl.question` untuk satu atau dua pertanyaan interaktif, dan `for await (const baris of rl)` untuk banyak baris dari terminal maupun pipe. Selalu `rl.close()`.
- `process.stdin.isTTY` membedakan masukan interaktif dari pipe.
- **Sinyal**: dengarkan `SIGINT` (Ctrl + C) dan `SIGTERM` dengan `process.on` untuk menutup program dengan rapi.
- `exit` hanya menjalankan kode sinkron; `beforeExit` masih bisa menjadwalkan pekerjaan.
- `uncaughtException` dan `unhandledRejection` adalah jaring pengaman terakhir untuk mencatat error, bukan pengganti `try...catch`.
- Alat baris perintah yang baik: konfigurasi lewat lingkungan, hasil ke stdout, error ke stderr, exit code benar.

## Latihan

Kerjakan sendiri dulu sebelum melihat jawaban. Latihan dijalankan dari folder `node-belajar`.

### Latihan 1: Tipe variabel lingkungan

Setelah baris `process.env.JUMLAH = 5;`, apa hasil `typeof process.env.JUMLAH`? Tebak dulu.

??? success "Jawaban"
    ```javascript
    process.env.JUMLAH = 5;
    console.log(typeof process.env.JUMLAH, process.env.JUMLAH + 1);
    ```

    ```text
    string 51
    ```

    Node.js mengubah semua nilai `process.env` menjadi teks, sehingga `"5" + 1` menyambung menjadi `"51"`.

### Latihan 2: Konfigurasi dengan nilai bawaan

Tulis `salam.js` yang membaca `NAMA` dari variabel lingkungan (bawaan `"teman"`) dan mencetak `Halo, <nama>!`. Jalankan tanpa `.env`, lalu dengan berkas `.env.nama` berisi `NAMA=Sari`.

??? success "Contoh jawaban"
    ```javascript title="salam.js"
    const nama = process.env.NAMA || "teman";
    console.log(`Halo, ${nama}!`);
    ```

    ```text title=".env.nama"
    NAMA=Sari
    ```

    ```bash
    node salam.js
    node --env-file=.env.nama salam.js
    ```

    ```text
    Halo, teman!
    Halo, Sari!
    ```

### Latihan 3: Memisahkan stdout dan stderr

Dari `node -e "console.log('hasil'); console.error('galat')"`, alihkan hanya stdout ke berkas `h.txt`. Apa yang tampil di layar, dan apa isi `h.txt`?

??? success "Jawaban"
    ```bash
    node -e "console.log('hasil'); console.error('galat')" > h.txt
    cat h.txt
    ```

    ```text
    galat
    hasil
    ```

    Baris pertama (`galat`) tampil di layar karena stderr tidak dialihkan. Baris kedua (`hasil`) adalah isi `h.txt`.

### Latihan 4: Validasi umur

Buat fungsi `pesanUmur(nama, umurTeks)` yang mengembalikan `Umur harus angka.` jika `umurTeks` kosong atau bukan angka, dan selain itu kalimat `<nama>, tahun depan kamu <umur + 1> tahun.` seperti di `kuis.js`. Pisahkan logika ini dari `readline` supaya bisa diuji tanpa mengetik, lalu uji dengan `"20"`, `"dua puluh"`, dan teks kosong.

??? success "Contoh jawaban"
    Fungsinya dipisahkan dari `readline` agar mudah diuji tanpa mengetik:
    ```javascript title="umur.js"
    export function pesanUmur(nama, umurTeks) {
      const umur = Number(umurTeks);
      if (umurTeks.trim() === "" || Number.isNaN(umur)) {
        return "Umur harus angka.";
      }
      return `${nama}, tahun depan kamu ${umur + 1} tahun.`;
    }

    console.log(pesanUmur("Budi", "20"));
    console.log(pesanUmur("Budi", "dua puluh"));
    console.log(pesanUmur("Budi", ""));
    ```

    ```bash
    node umur.js
    ```

    ```text
    Budi, tahun depan kamu 21 tahun.
    Umur harus angka.
    Umur harus angka.
    ```

    Dengan logika terpisah dari `readline`, program interaktifnya tinggal memanggil `pesanUmur(nama, await rl.question("Umur? "))`.

### Latihan 5: Menghitung kata dari pipe

Pakai `hitung-kata.js` untuk menghitung kata pada teks `belajar node itu seru`.

??? success "Jawaban"
    ```bash
    echo belajar node itu seru | node hitung-kata.js
    ```

    ```text
    4 kata
    ```

### Latihan 6: Pesan penutup

Buat `pamit.js` yang mencetak `bekerja...`, lalu mencetak `sampai jumpa (kode 0)` ketika program berhenti, memakai event `exit`.

??? success "Contoh jawaban"
    ```javascript title="pamit.js"
    process.on("exit", (kode) => console.log(`sampai jumpa (kode ${kode})`));
    console.log("bekerja...");
    ```

    ```bash
    node pamit.js
    ```

    ```text
    bekerja...
    sampai jumpa (kode 0)
    ```

### Latihan 7: Hitung tick sampai Ctrl + C

Buat program yang mencetak nomor tick setiap 100 ms. Saat menerima `SIGINT`, ia menghentikan interval dan mencetak `berhenti setelah N tick`. Simulasikan `Ctrl + C` dengan `process.emit("SIGINT")` setelah 250 ms.

??? success "Contoh jawaban"
    ```javascript title="hitung-tick.js"
    let tick = 0;

    const timer = setInterval(() => {
      tick++;
      console.log("tick", tick);
    }, 100);

    process.on("SIGINT", () => {
      clearInterval(timer);
      console.log(`berhenti setelah ${tick} tick`);
    });

    setTimeout(() => process.emit("SIGINT"), 250);
    ```

    ```bash
    node hitung-tick.js
    ```

    ```text
    tick 1
    tick 2
    berhenti setelah 2 tick
    ```

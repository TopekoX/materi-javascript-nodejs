## Tujuan pembelajaran

Setelah mempelajari bab ini, kamu dapat:

- Menjelaskan apa itu modul bawaan Node.js dan mengapa ditulis dengan awalan `node:`.
- Mengolah alamat berkas dengan `node:path` dan memakai `import.meta.dirname`.
- Mengurai dan membuat alamat web dengan `URL` dan `URLSearchParams`.
- Memakai `node:util` untuk memformat teks, mengubah callback menjadi Promise, dan membaca argumen terminal dengan `parseArgs`.
- Membuat hash dan angka acak yang aman dengan `node:crypto`.
- Memahami `Buffer` dan penyandian teks (hex, base64).
- Memakai `node:timers/promises` dan `structuredClone`.
- Menemukan modul bawaan yang dibutuhkan dari dokumentasi.

Bab ini memakai materi bab 11 (modul), bab 12 (Promise), bab 14 (argumen terminal), dan bab 15 (folder `node-belajar`). Semua contoh dijalankan dari folder `node-belajar`.

## Pendahuluan: baterai sudah termasuk

Di bab 16 kamu belajar memasang paket dari luar. Tetapi sebelum mencari paket, selalu periksa dulu apa yang **sudah disediakan Node.js**. Node.js datang dengan puluhan modul bawaan untuk berkas, alamat, sistem operasi, kriptografi, jaringan, dan banyak lagi. Memakainya lebih aman (tidak ada paket tambahan yang bisa bermasalah), lebih ringan, dan tidak perlu `npm install`.

Modul bawaan diimpor dengan awalan **`node:`**:

```javascript
import path from "node:path";
import { readFile } from "node:fs/promises";
```

Awalan `node:` menandakan dengan jelas bahwa ini modul bawaan, bukan paket di `node_modules`. Tanpa awalan itu pun biasanya tetap bekerja (`import fs from "fs"`), tetapi awalan `node:` adalah kebiasaan yang disarankan, dan beberapa modul baru **hanya** bisa dipakai dengan awalan itu.

Di bab-bab sebelumnya kamu sudah memakai `node:fs/promises`, `node:path`, dan `node:assert`. Bab ini memperkenalkan modul lain yang paling sering dibutuhkan.

## `node:path`: mengolah alamat berkas

Alamat berkas ditulis berbeda di tiap sistem operasi: Windows memakai `\` (`data\laporan\x.json`), sedangkan macOS dan Linux memakai `/` (`data/laporan/x.json`). Menyambung alamat dengan tanda `+` dan `"/"` bisa rusak di salah satunya. Modul `path` mengatasi itu. Simpan sebagai `path-dasar.js`:

```javascript title="path-dasar.js"
import path from "node:path";

console.log(path.posix.join("data", "laporan", "2025", "maret.json"));
console.log(path.win32.join("data", "laporan", "2025", "maret.json"));
console.log(path.posix.join("/a/b", "../c", "./d.txt"));
console.log(path.posix.resolve("/proyek", "src", "../data", "x.json"));

const alamat = "/proyek/src/app.js";
console.log(path.posix.dirname(alamat));
console.log(path.posix.basename(alamat));
console.log(path.posix.extname(alamat));
console.log(path.posix.basename(alamat, ".js"));
console.log(path.posix.parse(alamat));
console.log(path.posix.isAbsolute("/a"), path.posix.isAbsolute("a/b"));
```

```bash
node path-dasar.js
```

Output:

```text
data/laporan/2025/maret.json
data\laporan\2025\maret.json
/a/c/d.txt
/proyek/data/x.json
/proyek/src
app.js
.js
app
{
  root: '/',
  dir: '/proyek/src',
  base: 'app.js',
  ext: '.js',
  name: 'app'
}
true false
```

`path` punya dua "dialek": `path.posix` (gaya `/`) dan `path.win32` (gaya `\`). Biasanya kamu cukup menulis `path.join(...)` dan Node.js memilih dialek sesuai sistem operasimu. Contoh di atas sengaja memakai dialek tertentu supaya hasilnya sama di semua komputer.

| Fungsi | Hasil |
|---|---|
| `join(a, b, ...)` | Menyambung bagian alamat dengan pemisah yang benar, sekaligus merapikan `..` dan `.` |
| `resolve(a, b, ...)` | Membentuk alamat **absolut**. Jika bagiannya tidak absolut, dihitung dari folder kerja |
| `dirname(alamat)` | Folder tempat berkas berada |
| `basename(alamat)` | Nama berkas (opsional: buang ekstensi dengan argumen kedua) |
| `extname(alamat)` | Ekstensi, termasuk titik |
| `parse(alamat)` | Memecah menjadi objek `root`, `dir`, `base`, `ext`, `name` |
| `isAbsolute(alamat)` | Apakah alamat dimulai dari akar (`/` atau `C:\`) |

### Alamat berkas di dalam proyek

Di bab 14 ada kesalahan umum: alamat relatif seperti `data/pengeluaran.json` dihitung dari **folder tempat perintah `node` dijalankan**, bukan dari lokasi file kodemu. Cara memperbaikinya di ES Modules adalah **`import.meta.dirname`**, yang berisi folder tempat file kode itu berada (`import.meta.filename` berisi alamat lengkap file itu sendiri). Keduanya menggantikan `__dirname` dan `__filename` milik CommonJS.

Buat folder `demo` berisi tiga file:

```json title="demo/data.json"
{ "nama": "Budi" }
```

```javascript title="demo/salah.js"
import { readFile } from "node:fs/promises";

console.log(JSON.parse(await readFile("data.json", "utf8")));
```

```javascript title="demo/benar.js"
import { readFile } from "node:fs/promises";
import { join } from "node:path";

const alamat = join(import.meta.dirname, "data.json");
console.log(JSON.parse(await readFile(alamat, "utf8")));
```

Jalankan keduanya **dari dalam** folder `demo`:

```bash
cd demo
node salah.js
node benar.js
```

Output:

```text
{ nama: 'Budi' }
{ nama: 'Budi' }
```

Keduanya berhasil, karena folder kerjanya kebetulan sama dengan lokasi file. Sekarang kembali ke folder `node-belajar` dan jalankan dari sana:

```bash
cd ..
node demo/benar.js
node demo/salah.js
```

Output (dipersingkat):

```text
{ nama: 'Budi' }
Error: ENOENT: no such file or directory, open
```

`benar.js` tetap bekerja dari folder mana pun, karena alamatnya dihitung dari lokasi file (`import.meta.dirname`). `salah.js` gagal dengan `ENOENT` karena mencari `data.json` di folder kerja. **Kebiasaan yang baik:** untuk berkas yang menjadi bagian proyek, bentuk alamatnya dengan `path.join(import.meta.dirname, ...)`.

!!! note "`import.meta.url`"
    Selain `dirname` dan `filename`, ada `import.meta.url`, yaitu alamat file ini dalam bentuk URL (`file:///...`). Fungsi `fileURLToPath` dari `node:url` mengubahnya menjadi alamat biasa. `import.meta.dirname` dan `import.meta.filename` sudah mencakup kebutuhan itu di Node.js versi terbaru.

## `URL` dan `URLSearchParams`: alamat web

Alamat web seperti `https://toko.id:8080/produk/pensil?kategori=alat#ulasan` punya banyak bagian. Memecahnya dengan `split` atau `slice` rawan salah. Kelas bawaan **`URL`** (tersedia global, tanpa `import`) melakukannya dengan benar. Simpan sebagai `url-dasar.js`:

```javascript title="url-dasar.js"
const url = new URL("https://toko.id:8080/produk/pensil?kategori=alat&urut=harga&urut=nama#ulasan");

console.log(url.protocol, url.hostname, url.port);
console.log(url.pathname, url.search, url.hash);
console.log(url.origin);

console.log(url.searchParams.get("kategori"));
console.log(url.searchParams.getAll("urut"));
console.log(url.searchParams.get("tidak-ada"));
console.log(url.searchParams.has("urut"));

url.searchParams.set("halaman", "2");
url.searchParams.delete("urut");
console.log(url.href);
```

```bash
node url-dasar.js
```

Output:

```text
https: toko.id 8080
/produk/pensil ?kategori=alat&urut=harga&urut=nama #ulasan
https://toko.id:8080
alat
[ 'harga', 'nama' ]
null
true
https://toko.id:8080/produk/pensil?kategori=alat&halaman=2#ulasan
```

| Properti | Isi |
|---|---|
| `protocol`, `hostname`, `port` | Skema, nama host, dan nomor port |
| `pathname`, `search`, `hash` | Jalur, bagian query (setelah `?`), dan bagian setelah `#` |
| `origin` | Gabungan protokol, host, dan port |
| `searchParams` | Objek `URLSearchParams` untuk membaca dan mengubah query |
| `href` | Seluruh alamat sebagai teks |

Pada `searchParams`, `get` mengambil satu nilai (atau `null` jika tidak ada), `getAll` mengambil semua nilai untuk satu nama, dan `set`, `delete`, `has` bekerja seperti namanya. Setiap perubahan langsung tercermin di `url.href`.

### Membuat query dan alamat relatif

Simpan sebagai `url-lanjut.js`:

```javascript title="url-lanjut.js"
const q = new URLSearchParams({ q: "kopi susu", min: "1000" });
console.log(q.toString());
console.log([...q]);
console.log(Object.fromEntries(q));

console.log(new URL("/api/pengguna?id=5", "http://localhost:3000").href);

try {
  new URL("bukan url");
} catch (e) {
  console.log(e.name, e.code);
}
```

```bash
node url-lanjut.js
```

Output:

```text
q=kopi+susu&min=1000
[ [ 'q', 'kopi susu' ], [ 'min', '1000' ] ]
{ q: 'kopi susu', min: '1000' }
http://localhost:3000/api/pengguna?id=5
TypeError ERR_INVALID_URL
```

- `URLSearchParams` menulis spasi sebagai `+` dan karakter khusus lainnya dengan aman (*URL encoding*). Nilainya selalu **teks**, jadi `min` tetap `"1000"` dan perlu `Number(...)` jika ingin dihitung.
- `[...q]` mengubahnya menjadi array pasangan, dan `Object.fromEntries` menjadi objek biasa (bab 9).
- `new URL(alamatRelatif, alamatDasar)` menyambung alamat relatif dengan alamat dasar.
- Alamat yang tidak valid melempar `TypeError`. Bungkus dengan `try...catch` jika alamatnya berasal dari pengguna.

Kamu akan memakai `URL` dan query ini lagi saat membuat server di bab 20.

## `node:os`: informasi sistem

Modul `os` menjawab pertanyaan tentang komputer yang menjalankan programmu. Simpan sebagai `os-dasar.js`:

```javascript title="os-dasar.js"
import os from "node:os";

console.log("Sistem:", os.type(), os.platform(), os.arch());
console.log("Jumlah inti CPU:", os.cpus().length);
console.log("Memori total (MB):", Math.round(os.totalmem() / 1024 / 1024));
console.log("Folder rumah:", os.homedir());
console.log("Folder sementara:", os.tmpdir());
console.log("Pemisah baris:", JSON.stringify(os.EOL));
```

```bash
node os-dasar.js
```

Output (contoh, berbeda di setiap komputer):

```text
Sistem: Windows_NT win32 x64
Jumlah inti CPU: 8
Memori total (MB): 16109
Folder rumah: C:\Users\kamu
Folder sementara: C:\Users\kamu\AppData\Local\Temp
Pemisah baris: "\r\n"
```

`os.EOL` adalah pemisah baris sistem: `"\r\n"` di Windows dan `"\n"` di macOS dan Linux. Kamu akan sering memakai `os.tmpdir()` untuk menaruh berkas sementara.

## `node:util`: alat bantu serba guna

### Memformat teks dan melihat isi objek

Simpan sebagai `util-dasar.js`:

```javascript title="util-dasar.js"
import { format, inspect, isDeepStrictEqual } from "node:util";

console.log(format("Halo %s, umur %d tahun, data %j", "Budi", 20, { a: 1 }));
console.log(format("%s%%", 50));

const dalam = { a: { b: { c: { d: 1 } } } };
console.log(inspect(dalam, { depth: 0 }));
console.log(inspect(dalam, { depth: null }));

console.log(isDeepStrictEqual({ a: [1, 2] }, { a: [1, 2] }));
console.log(isDeepStrictEqual([1], ["1"]));
```

```bash
node util-dasar.js
```

Output:

```text
Halo Budi, umur 20 tahun, data {"a":1}
50%
{ a: [Object] }
{
  a: { b: { c: { d: 1 } } }
}
true
false
```

| Fungsi | Kegunaan |
|---|---|
| `format(teks, ...nilai)` | Menyisipkan nilai ke dalam penanda: `%s` teks, `%d` angka, `%j` JSON, `%%` tanda persen |
| `inspect(objek, opsi)` | Mengubah objek menjadi teks seperti yang dicetak `console.log`. Opsi `depth` mengatur kedalaman objek bersarang yang ditampilkan (`null` berarti semua) |
| `isDeepStrictEqual(a, b)` | Membandingkan **isi** dua objek atau array, bukan referensinya (ingat `{} === {}` bernilai `false` di bab 9) |

`console.log` ternyata memotong objek yang bersarang lebih dari dua tingkat menjadi `[Object]`. Itulah gunanya `inspect(..., { depth: null })` saat men-debug.

### Mengubah callback menjadi Promise

Banyak kode lama memakai callback gaya Node.js (error lebih dulu, bab 12). `promisify` mengubah fungsi seperti itu menjadi fungsi yang mengembalikan Promise, sehingga bisa dipakai dengan `await`. Simpan sebagai `util-promisify.js`:

```javascript title="util-promisify.js"
import { promisify } from "node:util";
import { setTimeout as tunggu } from "node:timers/promises";

function gandakan(angka, callback) {
  setTimeout(() => {
    if (angka < 0) {
      callback(new Error("negatif"));
    } else {
      callback(null, angka * 2);
    }
  }, 10);
}

const gandakanAsync = promisify(gandakan);

console.log(await gandakanAsync(21));

try {
  await gandakanAsync(-1);
} catch (e) {
  console.log("gagal:", e.message);
}

const mulai = Date.now();
await tunggu(50);
console.log("menunggu selesai:", Date.now() - mulai >= 45);
```

```bash
node util-promisify.js
```

Output:

```text
42
gagal: negatif
menunggu selesai: true
```

Modul **`node:timers/promises`** menyediakan versi Promise dari `setTimeout` dan `setInterval`. Kita mengimpornya dengan nama `tunggu` (`setTimeout as tunggu`), sehingga `await tunggu(50)` menjeda 50 ms tanpa perlu membuat `new Promise` sendiri seperti `bantu.js` di bab 12.

### `parseArgs`: membaca argumen terminal

Di bab 14 kita menulis sendiri fungsi `parseArgumen` untuk mengurai `--tanggal=2025-03-01`. Ternyata Node.js sudah punya `parseArgs`. Simpan sebagai `util-args.js`:

```javascript title="util-args.js"
import { parseArgs } from "node:util";

const { values, positionals } = parseArgs({
  args: ["tambah", "15000", "--kategori", "makan", "-v", "--tanggal=2025-03-01"],
  options: {
    kategori: { type: "string", short: "k" },
    tanggal: { type: "string" },
    verbose: { type: "boolean", short: "v", default: false },
  },
  allowPositionals: true,
});

console.log(values);
console.log(positionals);

try {
  parseArgs({ args: ["--tidak-dikenal"], options: {} });
} catch (e) {
  console.log(e.name, e.code);
}
```

```bash
node util-args.js
```

Output:

```text
[Object: null prototype] {
  kategori: 'makan',
  verbose: true,
  tanggal: '2025-03-01'
}
[ 'tambah', '15000' ]
TypeError ERR_PARSE_ARGS_UNKNOWN_OPTION
```

Penjelasan:

- `options` mendaftarkan opsi yang dikenal, lengkap dengan jenisnya (`"string"` atau `"boolean"`), singkatan (`short`, ditulis `-v`), dan nilai bawaan.
- `allowPositionals: true` mengizinkan argumen biasa (`"tambah"`, `"15000"`), yang dikumpulkan di `positionals`.
- Opsi boleh ditulis `--tanggal=2025-03-01` maupun `--kategori makan`.
- Opsi yang tidak terdaftar melempar error `ERR_PARSE_ARGS_UNKNOWN_OPTION`, sehingga salah ketik langsung ketahuan.
- Argumen `args` biasanya tidak ditulis. Tanpanya, `parseArgs` membaca `process.argv` yang sebenarnya. Di sini kita memberi `args` agar contoh bisa dijalankan tanpa mengetik argumen.
- Nilai `values` bertipe teks. Jika perlu angka, ubah dengan `Number(...)`.

## `node:crypto`: hash dan angka acak

Modul `crypto` berisi alat kriptografi. Tiga kebutuhan paling umum: hash, ID acak, dan penyimpanan kata sandi. Simpan sebagai `crypto-dasar.js`:

```javascript title="crypto-dasar.js"
import { createHash, createHmac, randomUUID, randomInt, randomBytes } from "node:crypto";

console.log(createHash("sha256").update("halo").digest("hex"));
console.log(createHmac("sha256", "rahasia").update("pesan").digest("hex"));

const id = randomUUID();
console.log(id.length);

const dadu = randomInt(1, 7);
console.log(dadu >= 1 && dadu <= 6);

console.log(randomBytes(8).toString("hex").length);
```

```bash
node crypto-dasar.js
```

Output:

```text
a4e63bcacf6c172ad84f9f4523c8f1acaf33676fa76d3258c67b7e7bbf16d777
433b7677c62d3dcfa784d99577da42c1c330e0bdaceb371984235bc630345952
36
true
16
```

| Fungsi | Kegunaan |
|---|---|
| `createHash("sha256").update(data).digest("hex")` | **Hash**: sidik jari dari data. Data yang sama selalu menghasilkan hash yang sama, dan hash tidak bisa dibalik menjadi data asli. Dipakai untuk memeriksa keutuhan berkas |
| `createHmac(algoritma, kunci)` | Hash yang memakai kunci rahasia. Dipakai untuk membuktikan sebuah pesan tidak diubah dan berasal dari pemilik kunci |
| `randomUUID()` | Pengenal acak 36 karakter, seperti `3b241101-e2bb-4255-8caf-4136c566a962`. Hasilnya berbeda setiap kali |
| `randomInt(min, maks)` | Bilangan bulat acak dari `min` sampai **sebelum** `maks` |
| `randomBytes(n)` | `n` byte acak, untuk token atau garam (*salt*) |

!!! warning "Jangan memakai `Math.random()` untuk hal yang menyangkut keamanan"
    `Math.random()` mudah ditebak. Untuk token, ID sesi, atau kode verifikasi, gunakan `randomUUID`, `randomInt`, atau `randomBytes` dari `crypto`.

### Menyimpan kata sandi

Kata sandi **tidak boleh** disimpan sebagai teks asli, dan juga tidak cukup di-hash dengan `sha256` biasa. Gunakan fungsi hash yang sengaja lambat dan menambahkan **garam** (*salt*) acak yang berbeda untuk tiap pengguna, seperti `scrypt`. Simpan sebagai `crypto-sandi.js`:

```javascript title="crypto-sandi.js"
import { randomBytes, scryptSync, timingSafeEqual } from "node:crypto";

function buatHash(sandi) {
  const garam = randomBytes(16).toString("hex");
  const hash = scryptSync(sandi, garam, 32).toString("hex");
  return `${garam}:${hash}`;
}

function periksa(sandi, tersimpan) {
  const [garam, hash] = tersimpan.split(":");
  const hashBaru = scryptSync(sandi, garam, 32);
  return timingSafeEqual(hashBaru, Buffer.from(hash, "hex"));
}

const tersimpan = buatHash("rahasia123");
console.log(tersimpan.length);
console.log(periksa("rahasia123", tersimpan));
console.log(periksa("salah", tersimpan));
console.log(buatHash("rahasia123") === tersimpan);
```

```bash
node crypto-sandi.js
```

Output:

```text
97
true
false
false
```

Penjelasan:

1. `buatHash` membuat garam acak, menghitung `scrypt(sandi, garam)`, dan menyimpan `garam:hash` dalam satu teks (32 + 1 + 64 = 97 karakter).
2. `periksa` mengambil garam yang tersimpan, menghitung ulang hash dari sandi yang diketik pengguna, lalu membandingkannya. `timingSafeEqual` membandingkan tanpa membocorkan informasi lewat selisih waktu.
3. Karena garamnya acak, hash kata sandi yang **sama** menghasilkan teks yang berbeda setiap kali (baris terakhir `false`). Itu disengaja: dua pengguna dengan sandi sama tidak punya hash yang sama.

Di aplikasi sungguhan banyak orang memakai pustaka khusus untuk ini, tetapi prinsipnya sama: garam acak, algoritma lambat, dan perbandingan yang aman.

## `Buffer`: data biner

Komputer menyimpan semuanya sebagai byte. **`Buffer`** adalah cara Node.js memegang urutan byte, misalnya isi berkas gambar atau data jaringan. Simpan sebagai `buffer-dasar.js`:

```javascript title="buffer-dasar.js"
const b = Buffer.from("Halo");

console.log(b);
console.log(b.length);
console.log(b.toString("hex"));
console.log(b.toString("base64"));
console.log(Buffer.from("SGFsbw==", "base64").toString());

console.log(Buffer.from("é").length, "é".length);
console.log(Buffer.alloc(3));
console.log(b[0], String.fromCharCode(b[0]));

console.log(new TextEncoder().encode("Hi"));
console.log(new TextDecoder().decode(new Uint8Array([72, 105])));
console.log(btoa("Halo"), atob("SGFsbw=="));
```

```bash
node buffer-dasar.js
```

Output:

```text
<Buffer 48 61 6c 6f>
4
48616c6f
SGFsbw==
Halo
2 1
<Buffer 00 00 00>
72 H
Uint8Array(2) [ 72, 105 ]
Hi
SGFsbw== Halo
```

| Contoh | Penjelasan |
|---|---|
| `Buffer.from("Halo")` | Mengubah teks menjadi byte (UTF-8). `48 61 6c 6f` adalah kode heksadesimal untuk H, a, l, o |
| `b.toString("hex")` / `"base64"` | Mengubah byte menjadi teks dengan penyandian **hex** atau **base64**. Base64 sering dipakai untuk mengirim data biner lewat teks |
| `Buffer.from(teks, "base64")` | Kebalikannya: membaca teks base64 menjadi byte |
| `Buffer.from("é").length` | `2`, padahal `"é".length` adalah `1`: huruf `é` memakai dua byte. Panjang byte dan panjang karakter bisa berbeda untuk huruf non-ASCII |
| `Buffer.alloc(n)` | Membuat buffer berisi `n` byte nol |
| `TextEncoder` / `TextDecoder` | Pasangan standar untuk teks ke byte dan sebaliknya (juga ada di browser) |
| `btoa` / `atob` | Base64 untuk teks sederhana, juga ada di browser |

Kamu sudah memakai buffer tanpa sadar: `readFile` tanpa argumen `"utf8"` mengembalikan `Buffer`, dan memberikan `"utf8"` menyuruh Node.js mengubahnya menjadi teks.

## `structuredClone`: salinan sungguhan

Di bab 9 kamu belajar bahwa spread (`{ ...obj }`) hanya membuat salinan **dangkal**: objek di dalamnya tetap dibagi bersama. Fungsi global **`structuredClone`** membuat salinan **mendalam**. Simpan sebagai `clone.js`:

```javascript title="clone.js"
const asli = { nama: "Budi", alamat: { kota: "Bandung" }, hobi: ["baca", "main"] };

const dangkal = { ...asli };
const dalam = structuredClone(asli);

dangkal.alamat.kota = "Medan";
console.log(asli.alamat.kota);

dalam.hobi.push("masak");
console.log(asli.hobi);
console.log(dalam.hobi);

try {
  structuredClone({ sapa() {} });
} catch (e) {
  console.log(e.name);
}
```

```bash
node clone.js
```

Output:

```text
Medan
[ 'baca', 'main' ]
[ 'baca', 'main', 'masak' ]
DataCloneError
```

- Mengubah `dangkal.alamat.kota` ikut mengubah `asli`, persis seperti peringatan di bab 9.
- `structuredClone` menyalin seluruh lapisan, sehingga mengubah `dalam` tidak memengaruhi `asli`.
- Ia tidak bisa menyalin fungsi (`DataCloneError`). Ia cocok untuk data murni: objek, array, tanggal, dan sejenisnya.

## Contoh praktis: ringkasan sebuah berkas

Gabungkan beberapa modul menjadi alat baris perintah kecil. Pertama, skrip yang membuat berkas contoh dengan isi yang pasti:

```javascript title="siapkan.js"
import { writeFile } from "node:fs/promises";

await writeFile("catatan.txt", "Baris satu\nBaris dua\nBaris tiga\n");
console.log("catatan.txt dibuat");
```

```javascript title="ringkas.js"
import { parseArgs } from "node:util";
import { readFile, stat } from "node:fs/promises";
import { createHash } from "node:crypto";
import { basename, extname } from "node:path";

const { values, positionals } = parseArgs({
  options: { hash: { type: "boolean", default: false } },
  allowPositionals: true,
});

const berkas = positionals[0];

if (!berkas) {
  console.error("Penggunaan: node ringkas.js <berkas> [--hash]");
  process.exitCode = 1;
} else {
  const isi = await readFile(berkas, "utf8");
  const info = await stat(berkas);

  console.log(`Nama: ${basename(berkas)}`);
  console.log(`Ekstensi: ${extname(berkas)}`);
  console.log(`Ukuran: ${info.size} byte`);
  console.log(`Baris: ${isi.split("\n").filter(Boolean).length}`);

  if (values.hash) {
    console.log(`SHA-256: ${createHash("sha256").update(isi).digest("hex").slice(0, 12)}...`);
  }
}
```

```bash
node siapkan.js
node ringkas.js catatan.txt --hash
```

Output:

```text
catatan.txt dibuat
Nama: catatan.txt
Ekstensi: .txt
Ukuran: 32 byte
Baris: 3
SHA-256: 5bb7d91c9556...
```

Satu kali jalan memakai lima modul: `util` (membaca argumen), `fs/promises` (membaca isi dan ukuran), `crypto` (hash), `path` (nama dan ekstensi), serta async/await dari bab 12. Jika berkas tidak disebutkan:

```bash
node ringkas.js
```

Output:

```text
Penggunaan: node ringkas.js <berkas> [--hash]
```

Pesan itu dikirim lewat `console.error` dengan exit code 1, pola yang sama seperti di bab 14.

## Menemukan modul yang kamu butuhkan

Daftar lengkap modul bawaan ada di dokumentasi resmi (nodejs.org/docs/latest/api/), yang kamu kenali dari bab 15. Beberapa modul lain yang akan kamu temui nanti:

| Modul | Kegunaan |
|---|---|
| `node:fs` dan `node:fs/promises` | Berkas dan folder (sudah dipakai) |
| `node:http` | Server dan klien HTTP (bab 20) |
| `node:events` dan `node:stream` | Event dan aliran data (bab 19) |
| `node:readline` | Membaca masukan baris demi baris (bab 18) |
| `node:test` dan `node:assert` | Pengujian (bab 22) |
| `node:child_process` | Menjalankan program lain dari Node.js |
| `node:zlib` | Kompresi (misalnya gzip) |
| `node:worker_threads` | Pekerjaan berat di utas terpisah |

Aturan sederhananya: sebelum menjalankan `npm install` untuk kebutuhan umum, cari dulu di dokumentasi Node.js.

## Kesalahan umum

### 1. Menyambung alamat berkas dengan tanda `+`

`"data" + "/" + "x.json"` bekerja di satu sistem tetapi rusak di sistem lain, dan gagal merapikan `..`. Pakai `path.join`.

### 2. Memakai `__dirname` di ES Module

Simpan sebagai `salah-dirname.js`:

```javascript title="salah-dirname.js"
console.log(__dirname);
```

```bash
node salah-dirname.js
```

Output (dipersingkat):

```text
ReferenceError: __dirname is not defined in ES module scope
```

**Perbaikan:** `import.meta.dirname` (atau `import.meta.filename`).

### 3. Alamat berkas yang relatif ke folder kerja

Seperti pada `demo/salah.js`: alamat relatif dihitung dari folder tempat `node` dijalankan. Bentuk alamat proyek dengan `path.join(import.meta.dirname, ...)`.

### 4. Memecah URL secara manual

Memakai `split("?")` atau `slice` untuk alamat web mudah salah pada karakter khusus. Gunakan `new URL(...)` dan `searchParams`.

### 5. Lupa bahwa nilai query selalu teks

`url.searchParams.get("min")` mengembalikan `"1000"` (teks), bukan angka. `"1000" + 5` menjadi `"10005"`. Ubah dengan `Number(...)` (bab 3).

### 6. Memakai `Math.random()` atau hash polos untuk keamanan

Token dan ID yang sensitif harus dari `crypto`. Kata sandi disimpan dengan `scrypt` plus garam, bukan `md5` atau `sha256` polos, dan tidak boleh sebagai teks asli.

### 7. Mengira panjang `Buffer` sama dengan panjang teks

`Buffer.from("é").length` adalah 2, sedangkan `"é".length` adalah 1. Untuk ukuran dalam byte, gunakan `Buffer.byteLength(teks)`.

### 8. Menyalin objek bersarang dengan spread

Spread hanya menyalin satu lapis. Pakai `structuredClone` untuk salinan mendalam, dan ingat ia tidak bisa menyalin fungsi.

### 9. Memasang paket untuk sesuatu yang sudah bawaan

Mengolah alamat, hash, UUID, dan argumen terminal semuanya sudah tersedia. Periksa dokumentasi sebelum menambah paket.

## Ringkasan

- **Modul bawaan** dipakai tanpa `npm install` dan diimpor dengan awalan `node:`.
- `node:path`: `join`, `resolve`, `dirname`, `basename`, `extname`, `parse`. Gunakan `path.join(import.meta.dirname, ...)` agar alamat tidak bergantung folder kerja.
- `URL` dan `URLSearchParams` mengurai dan membuat alamat web. Nilai query selalu teks.
- `node:os` memberi informasi sistem (platform, CPU, memori, folder rumah, folder sementara).
- `node:util`: `format`, `inspect`, `isDeepStrictEqual`, `promisify`, dan `parseArgs` untuk argumen terminal.
- `node:timers/promises` menyediakan `setTimeout` versi Promise.
- `node:crypto`: hash (`sha256`), `randomUUID`, `randomInt`, `randomBytes`, dan `scrypt` dengan garam untuk kata sandi.
- `Buffer` memegang byte; `toString("hex" | "base64")` mengubahnya menjadi teks.
- `structuredClone` membuat salinan mendalam dari data murni.
- Periksa dokumentasi Node.js sebelum memasang paket untuk kebutuhan umum.

## Latihan

Kerjakan sendiri dulu sebelum melihat jawaban. Latihan dijalankan dari folder `node-belajar`.

### Latihan 1: Memecah alamat berkas

Dengan `path.posix.parse`, tampilkan nama (tanpa ekstensi), ekstensi, dan folder dari `/home/budi/proyek/laporan-2025.pdf`.

??? success "Contoh jawaban"
    ```javascript
    import path from "node:path";

    const { name, ext, dir } = path.posix.parse("/home/budi/proyek/laporan-2025.pdf");
    console.log(name, ext, dir);
    ```

    ```text
    laporan-2025 .pdf /home/budi/proyek
    ```

### Latihan 2: Query dari alamat

Dari `https://toko.id/cari?q=kopi&q=teh&min=500`, tampilkan semua nilai `q` sebagai array dan nilai `min` sebagai **angka** yang ditambah 100.

??? success "Contoh jawaban"
    ```javascript
    const url = new URL("https://toko.id/cari?q=kopi&q=teh&min=500");

    console.log(url.searchParams.getAll("q"));
    console.log(Number(url.searchParams.get("min")) + 100);
    ```

    ```text
    [ 'kopi', 'teh' ]
    600
    ```

### Latihan 3: Memformat dengan `util.format`

Gunakan `format` untuk menghasilkan teks `Sari punya 3 buku` dari nama `"Sari"` dan jumlah `3`.

??? success "Contoh jawaban"
    ```javascript
    import { format } from "node:util";

    console.log(format("%s punya %d buku", "Sari", 3));
    ```

    ```text
    Sari punya 3 buku
    ```

### Latihan 4: `parseArgs`

Urai argumen `["--nama", "Budi", "--umur", "20"]` dengan `parseArgs`, lalu tampilkan umur setahun lagi sebagai angka.

??? success "Contoh jawaban"
    ```javascript
    import { parseArgs } from "node:util";

    const { values } = parseArgs({
      args: ["--nama", "Budi", "--umur", "20"],
      options: {
        nama: { type: "string" },
        umur: { type: "string" },
      },
    });

    console.log(`${values.nama} tahun depan ${Number(values.umur) + 1} tahun`);
    ```

    ```text
    Budi tahun depan 21 tahun
    ```

### Latihan 5: Hash

Tampilkan 8 karakter pertama dari hash SHA-256 teks `"node"`.

??? success "Contoh jawaban"
    ```javascript
    import { createHash } from "node:crypto";

    console.log(createHash("sha256").update("node").digest("hex").slice(0, 8));
    ```

    ```text
    545ea538
    ```

### Latihan 6: Base64 bolak-balik

Ubah teks `"Belajar Node.js"` menjadi base64, lalu kembalikan lagi ke teks.

??? success "Contoh jawaban"
    ```javascript
    const kode = Buffer.from("Belajar Node.js").toString("base64");
    console.log(kode);
    console.log(Buffer.from(kode, "base64").toString());
    ```

    ```text
    QmVsYWphciBOb2RlLmpz
    Belajar Node.js
    ```

### Latihan 7: Salinan mendalam

Salin `{ daftar: [1, 2], info: { aktif: true } }` dengan `structuredClone`, ubah salinannya (tambahkan 3 ke `daftar` dan set `aktif` jadi `false`), lalu tampilkan keduanya.

??? success "Contoh jawaban"
    ```javascript
    const asli = { daftar: [1, 2], info: { aktif: true } };
    const salinan = structuredClone(asli);

    salinan.daftar.push(3);
    salinan.info.aktif = false;

    console.log(asli);
    console.log(salinan);
    ```

    ```text
    { daftar: [ 1, 2 ], info: { aktif: true } }
    { daftar: [ 1, 2, 3 ], info: { aktif: false } }
    ```

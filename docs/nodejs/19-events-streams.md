## Tujuan pembelajaran

Setelah mempelajari bab ini, kamu dapat:

- Menjelaskan pola *event-driven*: pemancar, event, dan pendengar.
- Memakai `EventEmitter`: `on`, `once`, `off`, `emit`, dan event khusus `error`.
- Membuat class sendiri yang memancarkan event.
- Menjelaskan mengapa stream dibutuhkan, dan membedakan Readable, Writable, dan Transform.
- Membaca dan menulis berkas besar dengan `createReadStream` dan `createWriteStream`.
- Menyambungkan stream dengan `pipeline` dan menangani error-nya.
- Memahami *backpressure* secara konsep.
- Memproses berkas baris demi baris tanpa memuatnya seluruhnya ke memori.

Bab ini memakai class dan `extends` (bab 13), async (bab 12), serta modul bawaan (bab 17). Semua contoh dijalankan dari folder `node-belajar`.

## Pendahuluan: dua pola yang ada di mana-mana

Hampir semua hal di Node.js dibangun di atas dua gagasan.

**Event.** Bayangkan bel pintu. Kamu tidak berdiri menatap pintu setiap detik; kamu mengerjakan hal lain, dan ketika bel berbunyi, kamu bereaksi. *Pemancar* (bel) memancarkan *event*, dan *pendengar* (kamu) bereaksi. Server web menunggu permintaan, tombol menunggu klik, proses menunggu sinyal `SIGINT` (bab 18): semuanya event.

**Stream.** Bayangkan memindahkan air dari tandon ke kolam. Kamu bisa mengangkut seluruhnya dengan satu ember raksasa (butuh ember sangat besar), atau memasang **pipa** dan membiarkan air mengalir sedikit demi sedikit. Stream adalah pipanya: data dialirkan **potongan demi potongan**, sehingga berkas sebesar berapa pun bisa diproses dengan memori kecil.

Keduanya berhubungan: **setiap stream adalah EventEmitter**. Karena itu kita mulai dari event.

## `EventEmitter`

Kelas `EventEmitter` ada di modul `node:events`. Simpan sebagai `events-dasar.js`:

```javascript title="events-dasar.js"
import { EventEmitter } from "node:events";

const pemancar = new EventEmitter();

pemancar.on("sapa", (nama) => console.log(`Halo, ${nama}!`));
pemancar.on("sapa", (nama) => console.log(`(pendengar kedua) ${nama} datang`));
pemancar.once("sekali", () => console.log("hanya sekali"));

console.log(pemancar.emit("sapa", "Budi"));
console.log(pemancar.emit("sekali"));
console.log(pemancar.emit("sekali"));
console.log(pemancar.emit("tidak-ada"));
console.log(pemancar.listenerCount("sapa"), pemancar.eventNames());

const f = () => console.log("f");
pemancar.on("x", f);
pemancar.off("x", f);
console.log(pemancar.emit("x"));
```

```bash
node events-dasar.js
```

Output:

```text
Halo, Budi!
(pendengar kedua) Budi datang
true
hanya sekali
true
false
false
2 [ 'sapa' ]
false
```

| Method | Fungsi |
|---|---|
| `on(nama, fungsi)` | Mendaftarkan pendengar yang dipanggil **setiap kali** event `nama` dipancarkan |
| `once(nama, fungsi)` | Mendaftarkan pendengar yang hanya dipanggil **sekali** |
| `off(nama, fungsi)` | Mencabut pendengar tertentu (harus fungsi yang sama) |
| `emit(nama, ...argumen)` | Memancarkan event dan meneruskan argumen ke pendengar. Mengembalikan `true` jika ada pendengar, `false` jika tidak |
| `listenerCount(nama)` | Jumlah pendengar untuk event itu |
| `eventNames()` | Daftar nama event yang punya pendengar |

Beberapa hal yang tampak dari output:

- Nama event bebas (sembarang teks). Event `"sapa"` punya dua pendengar, dan keduanya dipanggil berurutan sesuai urutan pendaftaran.
- Pendengar `once` hanya berjalan pada `emit` pertama. Sesudahnya event itu tidak punya pendengar (`false`), dan namanya hilang dari `eventNames()`.
- Event yang tidak punya pendengar tidak menyebabkan error: `emit` hanya mengembalikan `false`.

### `emit` bersifat sinkron

Saat `emit` dipanggil, semua pendengar dijalankan **saat itu juga**, sebelum baris berikutnya. Event bukan asinkron dengan sendirinya. Simpan sebagai `events-sinkron.js`:

```javascript title="events-sinkron.js"
import { EventEmitter } from "node:events";

const p = new EventEmitter();
p.on("a", () => console.log("pendengar"));

console.log("sebelum emit");
p.emit("a");
console.log("setelah emit");
```

```bash
node events-sinkron.js
```

Output:

```text
sebelum emit
pendengar
setelah emit
```

Akibatnya, pendengar harus **didaftarkan sebelum** event dipancarkan. Event yang dipancarkan sebelum ada pendengarnya hilang begitu saja.

### Event `error` itu istimewa

Event bernama `"error"` diperlakukan khusus: jika dipancarkan **tanpa pendengar**, Node.js melempar error tersebut dan program berhenti. Ini disengaja agar kegagalan tidak diam-diam diabaikan. Simpan sebagai `events-error.js`:

```javascript title="events-error.js"
import { EventEmitter } from "node:events";

const p = new EventEmitter();
p.emit("error", new Error("tak ditangani"));
```

```bash
node events-error.js
```

Output (dipersingkat):

```text
Error: tak ditangani
```

Selalu pasang pendengar `error` pada pemancar yang mungkin gagal (`p.on("error", ...)`).

### Membuat class pemancar

Class yang `extends EventEmitter` dapat memancarkan event miliknya sendiri. Pola ini dipakai di seluruh Node.js. Simpan sebagai `unduhan.js`:

```javascript title="unduhan.js"
import { EventEmitter, once } from "node:events";

class Unduhan extends EventEmitter {
  constructor(nama, total) {
    super();
    this.nama = nama;
    this.total = total;
  }

  mulai() {
    let selesai = 0;
    const timer = setInterval(() => {
      selesai += 25;
      this.emit("kemajuan", selesai, this.total);
      if (selesai >= this.total) {
        clearInterval(timer);
        this.emit("selesai", this.nama);
      }
    }, 10);
  }
}

const u = new Unduhan("laporan.pdf", 100);

u.on("kemajuan", (selesai, total) => console.log(`${Math.round((selesai / total) * 100)}%`));
u.on("selesai", (nama) => console.log(`${nama} selesai`));

u.mulai();

const [nama] = await once(u, "selesai");
console.log("await once ->", nama);
```

```bash
node unduhan.js
```

Output:

```text
25%
50%
75%
100%
laporan.pdf selesai
await once -> laporan.pdf
```

Penjelasan:

- `super()` di constructor wajib dipanggil lebih dulu (bab 13).
- `mulai()` memancarkan event `kemajuan` setiap interval, dan `selesai` di akhir. Pemakai class tidak perlu tahu cara kerja di dalamnya, cukup mendengarkan event.
- Fungsi **`once(pemancar, nama)`** dari `node:events` mengubah "tunggu satu event" menjadi **Promise**, sehingga bisa dipakai dengan `await`. Hasilnya array berisi argumen event, makanya `const [nama] = ...`.

Pola ini ada di Node.js sendiri: server HTTP (`"request"`), stream (`"data"`, `"end"`), dan `process` (`"exit"`, `"SIGINT"` dari bab 18) semuanya adalah EventEmitter.

## Mengapa stream?

Cara paling mudah membaca berkas adalah `readFile`, yang memuat **seluruh isi ke memori** sekaligus. Untuk berkas kecil itu ideal. Tetapi untuk berkas berukuran ratusan megabita atau lebih, memori bisa habis, dan program harus menunggu seluruh berkas terbaca sebelum bisa mulai bekerja.

**Stream** mengalirkan data sedikit demi sedikit. Program memproses potongan pertama selagi potongan berikutnya masih dibaca. Ada empat jenis stream:

| Jenis | Arah | Contoh |
|---|---|---|
| **Readable** | Sumber data yang bisa dibaca | `createReadStream` (berkas), `process.stdin`, respons HTTP |
| **Writable** | Tujuan data yang bisa ditulis | `createWriteStream` (berkas), `process.stdout`, permintaan HTTP |
| **Transform** | Membaca, mengubah, lalu mengeluarkan | `createGzip` (kompresi), enkripsi |
| **Duplex** | Bisa dibaca dan ditulis sekaligus | Koneksi jaringan (*socket*) |

## Stream yang bisa dibaca (Readable)

Simpan sebagai `stream-baca.js`:

```javascript title="stream-baca.js"
import { writeFile } from "node:fs/promises";
import { createReadStream } from "node:fs";

await writeFile("data.txt", "ABCDEFGHIJ");

const aliran = createReadStream("data.txt", { encoding: "utf8", highWaterMark: 4 });

aliran.on("data", (potongan) => console.log("potongan:", potongan));
aliran.on("end", () => console.log("selesai dibaca"));
aliran.on("close", () => console.log("ditutup"));
```

```bash
node stream-baca.js
```

Output:

```text
potongan: ABCD
potongan: EFGH
potongan: IJ
selesai dibaca
ditutup
```

Penjelasan:

- `highWaterMark: 4` mengatur ukuran satu potongan menjadi 4 byte. Nilai bawaannya 64 KB; angka sekecil ini hanya agar potongan terlihat jelas.
- `encoding: "utf8"` membuat potongan berupa teks. Tanpa itu potongan berupa `Buffer` (bab 17).
- Stream memancarkan event: `data` untuk setiap potongan, `end` ketika datanya habis, dan `close` ketika sumbernya ditutup. Perhatikan bahwa ini hanyalah EventEmitter.

### Membaca dengan `for await`

Cara yang lebih mudah dibaca adalah **iterasi asinkron**: stream bisa diulang dengan `for await`, seperti yang kita lakukan pada `process.stdin` di bab 18. Simpan sebagai `stream-iterasi.js`:

```javascript title="stream-iterasi.js"
import { createReadStream } from "node:fs";
import { Readable } from "node:stream";

let jumlah = 0;
let panjang = 0;

for await (const potongan of createReadStream("data.txt", { highWaterMark: 4 })) {
  jumlah++;
  panjang += potongan.length;
  console.log(potongan.toString());
}
console.log(jumlah, "potongan,", panjang, "byte");

const angka = Readable.from([1, 2, 3, 4]);
const hasil = [];
for await (const x of angka) {
  hasil.push(x * 2);
}
console.log(hasil);
```

```bash
node stream-iterasi.js
```

Output:

```text
ABCD
EFGH
IJ
3 potongan, 10 byte
[ 2, 4, 6, 8 ]
```

`Readable.from(array)` mengubah array (atau iterator apa pun) menjadi stream, berguna untuk menguji atau menyambungkan data ke stream lain.

## Stream yang bisa ditulis (Writable) dan backpressure

`createWriteStream` membuat stream untuk menulis ke berkas. Dua method pentingnya: `write(data)` untuk menambah data, dan `end()` untuk menandai bahwa tidak ada lagi data, yang akhirnya memancarkan event `finish`.

Ada satu hal penting: stream menulis ke disk lebih lambat daripada kodemu menghasilkan data. Data yang belum sempat ditulis menumpuk di **penyangga** (*buffer*) memori. Jika penyangga penuh, `write()` mengembalikan **`false`** sebagai tanda "tahan dulu". Mengabaikan tanda ini membuat memori membengkak. Mekanisme saling menyesuaikan kecepatan ini disebut ***backpressure***: seperti penjaga loket yang menyuruh antrian berhenti sebentar ketika terlalu penuh.

Cara menanggapinya: jika `write()` mengembalikan `false`, tunggu event `drain` (penyangga sudah kosong lagi) sebelum menulis berikutnya. Simpan sebagai `buat-besar.js`:

```javascript title="buat-besar.js"
import { createWriteStream } from "node:fs";
import { once } from "node:events";
import { stat } from "node:fs/promises";

const penulis = createWriteStream("besar.txt");
let sempatDitahan = 0;

for (let i = 0; i < 20000; i++) {
  const lega = penulis.write(`baris nomor ${i}\n`);
  if (!lega) {
    sempatDitahan++;
    await once(penulis, "drain");
  }
}

penulis.end();
await once(penulis, "finish");

const { size } = await stat("besar.txt");
console.log("pernah ditahan karena penuh:", sempatDitahan > 0);
console.log("ukuran:", size, "byte");
```

```bash
node buat-besar.js
```

Output:

```text
pernah ditahan karena penuh: true
ukuran: 348890 byte
```

Program menulis 20.000 baris. Beberapa kali penyangga penuh (`write` mengembalikan `false`), lalu program menunggu `drain` dengan `once` dari bagian event tadi. Dengan begitu memori tetap kecil. Kabar baiknya, `pipeline` (di bawah) mengurus semua ini otomatis.

## Menyambungkan stream dengan `pipeline`

Kekuatan stream muncul saat disambung seperti pipa: sumber, lalu pengubah, lalu tujuan. Fungsi **`pipeline`** dari `node:stream/promises` menyambungkannya, menangani backpressure, dan menutup semua stream jika terjadi error.

### Membuat Transform sendiri

Sebuah `Transform` menerima potongan, mengubahnya, lalu meneruskannya. Simpan sebagai `pipeline-besar.js`:

```javascript title="pipeline-besar.js"
import { Transform } from "node:stream";
import { pipeline } from "node:stream/promises";
import { createReadStream, createWriteStream } from "node:fs";
import { readFile, writeFile } from "node:fs/promises";

await writeFile("masuk.txt", "halo dunia\nbelajar node\nstream itu seru\n");

const hurufBesar = new Transform({
  transform(potongan, pengkodean, selesai) {
    selesai(null, potongan.toString().toUpperCase());
  },
});

await pipeline(
  createReadStream("masuk.txt"),
  hurufBesar,
  createWriteStream("keluar.txt"),
);

console.log(await readFile("keluar.txt", "utf8"));
```

```bash
node pipeline-besar.js
```

Output:

```text
HALO DUNIA
BELAJAR NODE
STREAM ITU SERU
```

Penjelasan:

- `pipeline(sumber, pengubah, tujuan)` menyambungkan ketiganya dan mengembalikan Promise yang selesai ketika seluruh data sudah tertulis, sehingga bisa di-`await`.
- Pada `Transform`, fungsi `transform(potongan, pengkodean, selesai)` dipanggil untuk setiap potongan. Panggil `selesai(error, hasil)`: `null` untuk error berarti berhasil, dan `hasil` diteruskan ke tahap berikutnya. Potongan berupa `Buffer`, sehingga kita ubah dulu dengan `toString()`.
- Seluruh berkas tidak pernah dimuat sekaligus. Cara ini tetap bekerja untuk berkas sebesar apa pun.

### Error di dalam pipeline

Jika salah satu stream gagal, `pipeline` membatalkan semuanya dan menolak Promise-nya, sehingga `try...catch` dengan `await` (bab 10 dan 12) bisa menangkapnya. Simpan sebagai `pipeline-error.js`:

```javascript title="pipeline-error.js"
import { pipeline } from "node:stream/promises";
import { createReadStream } from "node:fs";
import { Writable } from "node:stream";

const buang = new Writable({
  write(potongan, pengkodean, selesai) {
    selesai();
  },
});

try {
  await pipeline(createReadStream("tidak-ada.txt"), buang);
} catch (e) {
  console.log("gagal:", e.code);
}
```

```bash
node pipeline-error.js
```

Output:

```text
gagal: ENOENT
```

Kode `ENOENT` (berkas tidak ditemukan) sama seperti di bab 14. Cara lama menyambung stream dengan `a.pipe(b)` tidak meneruskan error dan tidak menutup stream dengan benar; gunakan `pipeline`.

## Contoh praktis: berkas besar tanpa memenuhi memori

Berkas `besar.txt` hasil `buat-besar.js` dipakai di dua pekerjaan: menghitung baris dan memampatkannya.

### Menghitung baris

Modul `readline` dari bab 18 bisa membaca stream berkas baris demi baris. Simpan sebagai `hitung-baris.js`:

```javascript title="hitung-baris.js"
import { createInterface } from "node:readline";
import { createReadStream } from "node:fs";
import { stat } from "node:fs/promises";

const rl = createInterface({
  input: createReadStream("besar.txt"),
  crlfDelay: Infinity,
});

let baris = 0;
let terpanjang = 0;

for await (const isi of rl) {
  baris++;
  if (isi.length > terpanjang) {
    terpanjang = isi.length;
  }
}

const { size } = await stat("besar.txt");
console.log(`${baris} baris, terpanjang ${terpanjang} karakter`);
console.log(`ukuran ${size} byte`);
```

```bash
node hitung-baris.js
```

Output:

```text
20000 baris, terpanjang 17 karakter
ukuran 348890 byte
```

Memori yang dipakai hanya sebesar satu baris dan satu potongan, bukan seluruh berkas. Opsi `crlfDelay: Infinity` memastikan pasangan `\r\n` (akhir baris Windows) dianggap satu pemisah baris.

### Memampatkan dengan gzip

Modul `node:zlib` menyediakan `createGzip` dan `createGunzip` sebagai stream Transform. Simpan sebagai `kompres.js`:

```javascript title="kompres.js"
import { createGzip, createGunzip } from "node:zlib";
import { pipeline } from "node:stream/promises";
import { createReadStream, createWriteStream } from "node:fs";
import { readFile, stat } from "node:fs/promises";

await pipeline(createReadStream("besar.txt"), createGzip(), createWriteStream("besar.txt.gz"));
await pipeline(createReadStream("besar.txt.gz"), createGunzip(), createWriteStream("pulih.txt"));

const asli = (await stat("besar.txt")).size;
const mampat = (await stat("besar.txt.gz")).size;

console.log("lebih kecil:", mampat < asli);
console.log("isi sama setelah dipulihkan:", (await readFile("besar.txt", "utf8")) === (await readFile("pulih.txt", "utf8")));
```

```bash
node kompres.js
```

Output:

```text
lebih kecil: true
isi sama setelah dipulihkan: true
```

Dua `pipeline` bergantian: baca berkas, mampatkan, tulis ke `.gz`, lalu baca `.gz`, ekstrak, tulis ke `pulih.txt`. Kedua pekerjaan tetap berjalan dengan memori kecil.

## Kesalahan umum

### 1. Memakai `readFile` untuk berkas yang sangat besar

`readFile` memuat semuanya ke memori. Untuk berkas besar atau yang ukurannya tidak diketahui, gunakan stream atau `readline` seperti `hitung-baris.js`.

### 2. Mendaftarkan pendengar setelah `emit`

Simpan sebagai `terlambat.js`:

```javascript title="terlambat.js"
import { EventEmitter } from "node:events";

const p = new EventEmitter();

console.log(p.emit("halo"));
p.on("halo", () => console.log("tidak pernah terdengar"));
console.log(p.emit("halo"));
```

```bash
node terlambat.js
```

Output:

```text
false
tidak pernah terdengar
true
```

Pada `emit` pertama belum ada pendengar sehingga tidak terjadi apa-apa (`false`). Pendengar hanya mendengar `emit` setelah ia didaftarkan. **Perbaikan:** daftarkan pendengar sebelum event dipancarkan.

### 3. Salah ketik nama event

`p.on("selesai", ...)` tidak pernah dipanggil jika kodemu memancarkan `"selsai"`. Tidak ada error, hanya diam. Periksa nilai kembalian `emit` (`false` berarti tidak ada yang mendengar) dan simpan nama event sebagai konstanta jika dipakai di banyak tempat.

### 4. Tidak memasang pendengar `error`

Pemancar atau stream yang mengirim `error` tanpa pendengar menghentikan seluruh program (lihat `events-error.js`). Untuk stream, gunakan `pipeline`, atau pasang `.on("error", ...)`.

### 5. Lupa memanggil `end()` pada Writable

Event `finish` baru terjadi setelah `end()` dipanggil. Tanpa `end()`, `await once(penulis, "finish")` menunggu selamanya, dan berkas mungkin tidak tertutup dengan baik.

### 6. Mengabaikan backpressure

Menulis jutaan baris dengan `write()` tanpa memeriksa nilai kembaliannya membuat memori membengkak. Periksa `false` lalu tunggu `drain`, atau serahkan pada `pipeline`.

### 7. Menyambung stream dengan `.pipe()` tanpa penanganan error

Error pada salah satu stream tidak diteruskan ke stream lain, sehingga berkas bisa tertinggal terbuka. Gunakan `pipeline`.

### 8. Memotong teks di tengah karakter

Memecah teks per potongan dan mengubah tiap `Buffer` sendiri-sendiri dengan `toString()` bisa memotong huruf non-ASCII yang memakai beberapa byte (bab 17). Beri opsi `encoding: "utf8"` pada stream, atau gunakan `readline` untuk membaca per baris.

### 9. Menambah pendengar berulang kali tanpa mencabutnya

Menambah pendengar di dalam fungsi yang dipanggil berulang kali (tanpa `off`) membuat pendengar menumpuk. Setelah lebih dari 10 pendengar untuk satu event, Node.js menampilkan peringatan `MaxListenersExceededWarning` sebagai tanda kemungkinan kebocoran memori.

## Ringkasan

- **EventEmitter** (`node:events`): `on`, `once`, `off`, `emit`. Pendengar dijalankan **sinkron** saat `emit`, dan harus didaftarkan **sebelum** event dipancarkan.
- Event `"error"` tanpa pendengar menghentikan program. Selalu siapkan pendengarnya.
- `class X extends EventEmitter` membuat objek yang memancarkan event sendiri. `once(pemancar, nama)` mengubah event menjadi Promise.
- **Stream** mengalirkan data per potongan dan menghemat memori. Jenisnya: Readable, Writable, Transform, dan Duplex. Semuanya adalah EventEmitter.
- `createReadStream` dan `createWriteStream` untuk berkas. Baca stream dengan event `data`/`end` atau `for await`.
- **Backpressure**: jika `write()` mengembalikan `false`, tunggu `drain`. `pipeline` mengurusnya otomatis.
- `pipeline(sumber, ...pengubah, tujuan)` dari `node:stream/promises` menyambung stream, menangani error, dan bisa di-`await`. Hindari `.pipe()`.
- `Transform` mengubah data di tengah pipa. `node:zlib` menyediakan `createGzip` dan `createGunzip`.
- `readline` + `createReadStream` memproses berkas baris demi baris tanpa memuat seluruhnya.

## Latihan

Kerjakan sendiri dulu sebelum melihat jawaban. Latihan dijalankan dari folder `node-belajar`.

### Latihan 1: Prediksi

Apa saja yang tercetak?

```javascript
import { EventEmitter } from "node:events";

const p = new EventEmitter();
p.once("a", () => console.log("A"));
console.log(p.emit("a"));
console.log(p.emit("a"));
```

??? success "Jawaban"
    Output: `A`, `true`, `false` (masing-masing di baris sendiri). Pendengar `once` berjalan pada `emit` pertama (mencetak `A`, lalu `emit` mengembalikan `true`). Pada `emit` kedua pendengarnya sudah dicabut, sehingga `emit` mengembalikan `false`.

### Latihan 2: Perbaiki urutan

Kode ini tidak mencetak `Halo, Budi!`. Perbaiki:

```javascript
import { EventEmitter } from "node:events";

const p = new EventEmitter();
p.emit("sapa", "Budi");
p.on("sapa", (nama) => console.log(`Halo, ${nama}!`));
```

??? success "Jawaban"
    Daftarkan pendengar sebelum memancarkan event:
    ```javascript
    import { EventEmitter } from "node:events";

    const p = new EventEmitter();
    p.on("sapa", (nama) => console.log(`Halo, ${nama}!`));
    p.emit("sapa", "Budi");
    ```

    Output: `Halo, Budi!`

### Latihan 3: Class `Hitung`

Buat class `Hitung extends EventEmitter` dengan method `mulai()` yang memancarkan event `tick` dengan angka 1, 2, 3 (setiap 20 ms), lalu event `selesai`. Cetak setiap `tick` dan pesan `selesai!`.

??? success "Contoh jawaban"
    ```javascript
    import { EventEmitter } from "node:events";

    class Hitung extends EventEmitter {
      mulai() {
        let n = 0;
        const timer = setInterval(() => {
          n++;
          this.emit("tick", n);
          if (n === 3) {
            clearInterval(timer);
            this.emit("selesai");
          }
        }, 20);
      }
    }

    const h = new Hitung();
    h.on("tick", (n) => console.log("tick", n));
    h.on("selesai", () => console.log("selesai!"));
    h.mulai();
    ```

    Output:
    ```text
    tick 1
    tick 2
    tick 3
    selesai!
    ```

### Latihan 4: Stream dari array

Ubah array `["budi", "sari", "andi"]` menjadi stream dengan `Readable.from`, lalu cetak setiap nama dalam huruf besar memakai `for await`.

??? success "Contoh jawaban"
    ```javascript
    import { Readable } from "node:stream";

    for await (const nama of Readable.from(["budi", "sari", "andi"])) {
      console.log(nama.toUpperCase());
    }
    ```

    Output:
    ```text
    BUDI
    SARI
    ANDI
    ```

### Latihan 5: Salin dengan pengubah

Buat berkas `a.txt` berisi `halo` lalu salin ke `b.txt` dengan `pipeline`, sambil mengubah semua huruf menjadi huruf besar dengan `Transform`. Cetak isi `b.txt`.

??? success "Contoh jawaban"
    ```javascript
    import { Transform } from "node:stream";
    import { pipeline } from "node:stream/promises";
    import { createReadStream, createWriteStream } from "node:fs";
    import { readFile, writeFile } from "node:fs/promises";

    await writeFile("a.txt", "halo");

    const besar = new Transform({
      transform(potongan, pengkodean, selesai) {
        selesai(null, potongan.toString().toUpperCase());
      },
    });

    await pipeline(createReadStream("a.txt"), besar, createWriteStream("b.txt"));
    console.log(await readFile("b.txt", "utf8"));
    ```

    Output: `HALO`

### Latihan 6: Menyaring baris

Dari `besar.txt` (hasil `buat-besar.js`), hitung berapa baris yang mengandung angka `999` di dalamnya (misalnya `baris nomor 999`, `baris nomor 1999`). Gunakan `readline` dan stream.

??? success "Contoh jawaban"
    ```javascript
    import { createInterface } from "node:readline";
    import { createReadStream } from "node:fs";

    const rl = createInterface({ input: createReadStream("besar.txt"), crlfDelay: Infinity });
    let cocok = 0;

    for await (const baris of rl) {
      if (baris.includes("999")) {
        cocok++;
      }
    }

    console.log(cocok);
    ```

    Output: `38`

### Latihan 7: Kompres dan bandingkan

Mampatkan `masuk.txt` dengan gzip menjadi `masuk.txt.gz`, lalu cetak `true` jika ukuran hasilnya **lebih besar** dari aslinya. Tebak dulu mengapa hasilnya bisa begitu.

??? success "Contoh jawaban"
    ```javascript
    import { createGzip } from "node:zlib";
    import { pipeline } from "node:stream/promises";
    import { createReadStream, createWriteStream } from "node:fs";
    import { stat } from "node:fs/promises";

    await pipeline(createReadStream("masuk.txt"), createGzip(), createWriteStream("masuk.txt.gz"));

    const asli = (await stat("masuk.txt")).size;
    const mampat = (await stat("masuk.txt.gz")).size;
    console.log(mampat > asli);
    ```

    Output: `true`. Berkas `masuk.txt` hanya berisi beberapa puluh byte. Format gzip menambahkan header dan penutup yang tetap, sehingga untuk data yang sangat kecil hasil kompresi justru lebih besar. Kompresi baru bermanfaat untuk data yang cukup besar dan berpola, seperti `besar.txt`.

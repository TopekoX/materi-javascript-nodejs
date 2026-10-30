## Tujuan pembelajaran

Setelah mempelajari bab ini, kamu dapat:

- Menjelaskan apa itu objek dan kapan memakainya dibanding array.
- Membuat objek, membaca, mengubah, menambah, dan menghapus propertinya.
- Menulis *method* dan memahami `this` secara dasar.
- Memakai objek bersarang dan *optional chaining* (`?.`).
- Memakai *destructuring* dan *spread* pada objek dan array.
- Memproses array berisi objek dengan `map`, `filter`, dan `reduce`.
- Menjelaskan mengapa `const` tidak membuat objek terkunci, dan cara menyalinnya.
- Mengubah objek menjadi teks JSON dan sebaliknya.

Bab ini memakai materi bab 1 sampai 8, terutama fungsi dan array. Jalankan setiap contoh dengan `node nama-file.js`.

## Pendahuluan: data yang punya nama

Array cocok untuk daftar nilai yang sejenis dan berurutan, misalnya `[80, 65, 90]`. Namun bagaimana menyimpan data **satu siswa**: namanya, umurnya, kelasnya, apakah masih aktif? Dengan array kita harus mengingat bahwa indeks 0 adalah nama, indeks 1 adalah umur, dan seterusnya. Itu mudah keliru.

Bayangkan sebuah **kartu identitas**. Setiap isian punya **label**: *Nama*, *Umur*, *Kelas*. Kita mencari data lewat labelnya, bukan lewat nomor urut.

**Objek** (*object*) bekerja seperti kartu itu: kumpulan pasangan **label dan nilai**. Label disebut **key** (kunci), nilainya disebut **value**, dan sepasang key-value disebut **properti**.

## Membuat dan mengakses objek

Objek ditulis dengan kurung kurawal `{ }`, berisi pasangan `key: value` yang dipisah koma. Simpan sebagai `objek-dasar.js`:

```javascript
const siswa = { nama: "Budi", umur: 17, aktif: true };

console.log(siswa);
console.log(siswa.nama);
console.log(siswa["umur"]);
console.log(siswa.alamat);

siswa.umur = 18;
siswa.kelas = "XII";
delete siswa.aktif;
console.log(siswa);

console.log("nama" in siswa, "aktif" in siswa);
console.log(typeof siswa, Array.isArray(siswa));
```

Output:

```text
{ nama: 'Budi', umur: 17, aktif: true }
Budi
17
undefined
{ nama: 'Budi', umur: 18, kelas: 'XII' }
true false
object false
```

Penjelasan:

| Kode | Artinya |
|---|---|
| `{ nama: "Budi", umur: 17 }` | Objek dengan dua properti. Key biasanya ditulis tanpa tanda kutip |
| `siswa.nama` | **Notasi titik**: ambil nilai properti `nama` |
| `siswa["umur"]` | **Notasi kurung siku**: sama, tetapi key ditulis sebagai string |
| `siswa.alamat` | Properti yang tidak ada menghasilkan `undefined`, bukan error |
| `siswa.umur = 18` | Mengubah nilai properti yang sudah ada |
| `siswa.kelas = "XII"` | Menambah properti baru |
| `delete siswa.aktif` | Menghapus properti |
| `"nama" in siswa` | Memeriksa apakah key ada di objek |

Objek tidak punya indeks dan tidak punya urutan nomor, sehingga tidak ada `length`.

### Kapan memakai notasi kurung siku?

Notasi kurung siku diperlukan jika key-nya disimpan di variabel atau mengandung karakter khusus. Simpan sebagai `kunci-dinamis.js`:

```javascript
const siswa = { nama: "Budi", umur: 17, "nama lengkap": "Budi Santoso" };

const kunci = "umur";
console.log(siswa[kunci]);
console.log(siswa.kunci);
console.log(siswa["nama lengkap"]);
```

Output:

```text
17
undefined
Budi Santoso
```

- `siswa[kunci]` memakai **isi variabel** `kunci` (yaitu `"umur"`), sehingga menghasilkan 17.
- `siswa.kunci` mencari properti yang bernama harfiah `kunci`, yang tidak ada.
- Key yang mengandung spasi hanya bisa diakses dengan kurung siku.

### Singkatan penulisan properti

Jika nama variabel sama dengan nama key, cukup tulis sekali. Simpan sebagai `singkatan.js`:

```javascript
const nama = "Budi";
const umur = 17;

const siswa = { nama, umur };
console.log(siswa);
```

Output:

```text
{ nama: 'Budi', umur: 17 }
```

`{ nama, umur }` sama dengan `{ nama: nama, umur: umur }`.

## Objek bersarang dan optional chaining

Nilai properti boleh berupa objek lain atau array. Simpan sebagai `bersarang.js`:

```javascript
const toko = {
  nama: "Toko Kita",
  alamat: { kota: "Bandung", kodePos: "40111" },
  jamBuka: [8, 17],
};

console.log(toko.alamat.kota);
console.log(toko.jamBuka[0]);
console.log(toko.kontak?.telepon);
console.log(toko.kontak.telepon);
```

Output (dipersingkat):

```text
Bandung
8
undefined
TypeError: Cannot read properties of undefined (reading 'telepon')
```

Penjelasan:

1. `toko.alamat.kota` membaca dua tingkat: objek `alamat` dulu, lalu `kota` di dalamnya.
2. `toko.kontak` tidak ada, nilainya `undefined`. Membaca `.telepon` dari `undefined` menyebabkan `TypeError` (baris terakhir).
3. Tanda **`?.`** (*optional chaining*) berarti "jika bagian sebelumnya ada, lanjut. Jika `undefined` atau `null`, berhenti dan hasilkan `undefined`". Jadi `toko.kontak?.telepon` aman.

Gunakan `?.` ketika data mungkin tidak lengkap, misalnya data dari pengguna atau dari server.

## Method dan `this`

Nilai properti boleh berupa fungsi. Fungsi yang menjadi properti objek disebut **method**. Simpan sebagai `method.js`:

```javascript
const kucing = {
  nama: "Mimi",
  suara() {
    return `${this.nama}: meong`;
  },
  ganti(namaBaru) {
    this.nama = namaBaru;
  },
};

console.log(kucing.suara());
kucing.ganti("Tom");
console.log(kucing.suara());
```

Output:

```text
Mimi: meong
Tom: meong
```

Penjelasan:

- `suara() { ... }` adalah cara singkat menulis method.
- **`this`** di dalam method berarti **objek yang memanggil method itu**. Saat `kucing.suara()` dipanggil, `this` adalah `kucing`, sehingga `this.nama` adalah `"Mimi"`.
- `kucing.ganti("Tom")` mengubah `this.nama` menjadi `"Tom"`, dan pemanggilan `suara()` berikutnya ikut berubah.

!!! warning "Tulis method dengan sintaks biasa"
    Di dalam objek, tulis method seperti `suara() { ... }`, bukan sebagai arrow function (`suara: () => ...`). Arrow function tidak punya `this` milik objek, sehingga `this.nama` tidak akan merujuk ke objek tersebut. Pembahasan `this` yang lebih dalam datang di bab Classes.

## Destructuring

**Destructuring** adalah cara mengambil beberapa nilai dari objek atau array sekaligus ke dalam variabel. Simpan sebagai `destructuring.js`:

```javascript
const siswa = { nama: "Sari", umur: 20, kota: "Medan" };

const { nama, umur } = siswa;
console.log(nama, umur);

const { kota: asal, kelas = "belum ada" } = siswa;
console.log(asal, kelas);

const warna = ["merah", "hijau", "biru"];
const [pertama, kedua] = warna;
console.log(pertama, kedua);

let a = "Apel";
let b = "Jeruk";
[a, b] = [b, a];
console.log(a, b);

function perkenalan({ nama, umur }) {
  return `${nama}, ${umur} tahun`;
}
console.log(perkenalan(siswa));
```

Output:

```text
Sari 20
Medan belum ada
merah hijau
Jeruk Apel
Sari, 20 tahun
```

| Bentuk | Artinya |
|---|---|
| `const { nama, umur } = siswa` | Buat variabel `nama` dan `umur` dari properti yang bernama sama |
| `{ kota: asal }` | Ambil `kota`, tetapi simpan ke variabel bernama `asal` |
| `{ kelas = "belum ada" }` | Jika properti tidak ada (`undefined`), pakai nilai default |
| `const [pertama, kedua] = warna` | Untuk array, pengambilan berdasarkan **urutan** |
| `[a, b] = [b, a]` | Menukar dua variabel tanpa variabel sementara (bandingkan latihan bab 1) |
| `function perkenalan({ nama, umur })` | Destructuring langsung pada parameter fungsi |

!!! note "Titik koma sebelum `[`"
    Baris yang diawali `[` bisa dianggap sambungan dari baris sebelumnya jika baris itu tidak diakhiri titik koma. Itu sebabnya contoh ini menulis `;` di akhir setiap baris.

## Spread

Tiga titik `...` (**spread**) "menebar" isi array atau objek ke tempat lain. Simpan sebagai `spread.js`:

```javascript
const a = [1, 2];
const b = [...a, 3, 4];
const salinan = [...a];
salinan.push(99);
console.log(a, b, salinan);

const siswa = { nama: "Budi", umur: 17 };
const baru = { ...siswa, umur: 18, kelas: "XII" };
console.log(baru);
console.log(siswa);

const bawaan = { tema: "terang", bahasa: "id" };
const pilihan = { tema: "gelap" };
console.log({ ...bawaan, ...pilihan });
```

Output:

```text
[ 1, 2 ] [ 1, 2, 3, 4 ] [ 1, 2, 99 ]
{ nama: 'Budi', umur: 18, kelas: 'XII' }
{ nama: 'Budi', umur: 17 }
{ tema: 'gelap', bahasa: 'id' }
```

Penjelasan:

- `[...a, 3, 4]` membuat array baru yang berisi isi `a` lalu 3 dan 4. `[...a]` adalah cara lain membuat salinan array (alternatif `slice()` dari bab 8).
- `{ ...siswa, umur: 18 }` menyalin semua properti `siswa` ke objek baru, lalu `umur` ditimpa. `siswa` yang asli tidak berubah.
- Jika ada key yang sama, **yang ditulis belakangan menang**. Pada `{ ...bawaan, ...pilihan }`, `tema` dari `pilihan` menimpa yang dari `bawaan`.

!!! note "Salinan dangkal"
    Spread hanya menyalin satu tingkat (*shallow copy*). Jika sebuah properti berisi objek atau array lain, salinannya masih menunjuk ke objek yang sama dengan aslinya.

## `const` dan objek

Seperti array di bab 8, `const` pada objek hanya mengunci nama variabelnya, bukan isinya. Objek juga disimpan sebagai referensi. Simpan sebagai `const-objek.js`:

```javascript
const a = { x: 1 };
a.x = 2;
a.y = 3;
console.log(a);

const b = a;
b.x = 100;
console.log(a);

const c = { ...a };
c.x = 0;
console.log(a, c);

console.log({} === {});
```

Output:

```text
{ x: 2, y: 3 }
{ x: 100, y: 3 }
{ x: 100, y: 3 } { x: 0, y: 3 }
false
```

- `a.x = 2` boleh walaupun `a` dibuat dengan `const`. Yang dilarang adalah `a = { ... }` (mengisi ulang variabelnya).
- `const b = a` bukan salinan: `b.x = 100` ikut mengubah `a`.
- `{ ...a }` membuat salinan sungguhan, sehingga mengubah `c` tidak memengaruhi `a`.
- `{} === {}` bernilai `false`, karena dua objek berbeda tidak pernah dianggap sama.

## Menelusuri isi objek

Simpan sebagai `telusuri.js`:

```javascript
const harga = { apel: 5000, jeruk: 8000, mangga: 12000 };

console.log(Object.keys(harga));
console.log(Object.values(harga));
console.log(Object.entries(harga));

for (const nama in harga) {
  console.log(`${nama}: ${harga[nama]}`);
}

for (const [nama, h] of Object.entries(harga)) {
  console.log(`${nama} -> ${h}`);
}

const total = Object.values(harga).reduce((a, b) => a + b, 0);
console.log(total);
```

Output:

```text
[ 'apel', 'jeruk', 'mangga' ]
[ 5000, 8000, 12000 ]
[ [ 'apel', 5000 ], [ 'jeruk', 8000 ], [ 'mangga', 12000 ] ]
apel: 5000
jeruk: 8000
mangga: 12000
apel -> 5000
jeruk -> 8000
mangga -> 12000
25000
```

| Perintah | Hasil |
|---|---|
| `Object.keys(obj)` | Array berisi semua key |
| `Object.values(obj)` | Array berisi semua value |
| `Object.entries(obj)` | Array berisi pasangan `[key, value]` |
| `for (const k in obj)` | Perulangan atas **key** |

`Object.entries` dipadukan dengan destructuring `[nama, h]` menghasilkan perulangan yang rapi. Setelah menjadi array, objek bisa diproses dengan `map`, `filter`, dan `reduce` dari bab 8.

## Array berisi objek

Bentuk data yang paling sering ditemui adalah array yang berisi banyak objek, seperti tabel dengan satu objek untuk tiap baris. Simpan sebagai `array-objek.js`:

```javascript
const daftar = [
  { nama: "Budi", nilai: 80 },
  { nama: "Sari", nilai: 92 },
  { nama: "Andi", nilai: 65 },
];

const namaSaja = daftar.map((s) => s.nama);
const lulus = daftar.filter((s) => s.nilai >= 70);
const andi = daftar.find((s) => s.nama === "Andi");
const rata = daftar.reduce((jumlah, s) => jumlah + s.nilai, 0) / daftar.length;
const urut = daftar.slice().sort((a, b) => b.nilai - a.nilai);

console.log(namaSaja);
console.log(lulus.map((s) => s.nama));
console.log(andi);
console.log(rata);
console.log(urut.map((s) => `${s.nama} (${s.nilai})`).join(", "));
```

Output:

```text
[ 'Budi', 'Sari', 'Andi' ]
[ 'Budi', 'Sari' ]
{ nama: 'Andi', nilai: 65 }
79
Sari (92), Budi (80), Andi (65)
```

Pola ini sama dengan bab 8, hanya elemennya sekarang berupa objek, sehingga callback membaca propertinya (`s.nilai`). Untuk mengurutkan berdasarkan properti, pembandingnya mengurangkan properti itu: `b.nilai - a.nilai` untuk urutan turun.

## JSON: objek sebagai teks

Untuk menyimpan objek ke file atau mengirimnya lewat jaringan, objek diubah menjadi teks dengan format **JSON** (*JavaScript Object Notation*). Simpan sebagai `json.js`:

```javascript
const siswa = { nama: "Budi", umur: 18, kelas: "XII" };

const teks = JSON.stringify(siswa);
console.log(teks);
console.log(typeof teks);

console.log(JSON.stringify(siswa, null, 2));

const kembali = JSON.parse(teks);
console.log(kembali.nama);
```

Output:

```text
{"nama":"Budi","umur":18,"kelas":"XII"}
string
{
  "nama": "Budi",
  "umur": 18,
  "kelas": "XII"
}
Budi
```

- `JSON.stringify(objek)` mengubah objek menjadi string JSON.
- `JSON.stringify(objek, null, 2)` menghasilkan JSON yang rapi dengan indentasi dua spasi. Dua argumen tambahan itu cukup kamu hafalkan untuk sekarang.
- `JSON.parse(teks)` mengubah string JSON kembali menjadi objek.

Di JSON, key selalu memakai tanda kutip ganda. Format ini akan sering kamu temui saat membaca file dan memanggil API di bagian Node.js.

## Contoh praktis: rekening bank

Method dan `this` sangat cocok untuk benda yang punya data sekaligus perilaku. Simpan sebagai `rekening.js`:

```javascript
const rekening = {
  pemilik: "Budi",
  saldo: 100000,
  setor(jumlah) {
    this.saldo += jumlah;
  },
  tarik(jumlah) {
    if (jumlah > this.saldo) {
      return "Saldo tidak cukup";
    }
    this.saldo -= jumlah;
    return "Berhasil";
  },
  ringkasan() {
    return `${this.pemilik}: ${this.saldo}`;
  },
};

rekening.setor(50000);
console.log(rekening.ringkasan());
console.log(rekening.tarik(200000));
console.log(rekening.tarik(30000));
console.log(rekening.ringkasan());
```

Output:

```text
Budi: 150000
Saldo tidak cukup
Berhasil
Budi: 120000
```

Proses eksekusinya:

1. `setor(50000)` menambah `this.saldo` dari 100000 menjadi 150000.
2. `tarik(200000)`: 200000 lebih besar dari saldo 150000, sehingga `return` pertama dijalankan. Saldo tidak berubah.
3. `tarik(30000)`: cukup, saldo menjadi 120000, dan hasilnya `"Berhasil"`.

## Kesalahan umum

### 1. Memakai titik untuk key yang disimpan di variabel

`siswa.kunci` mencari properti bernama `kunci`. Pakai `siswa[kunci]`. Contohnya ada di `kunci-dinamis.js`.

### 2. Membaca properti dari `undefined`

`toko.kontak.telepon` menghasilkan `TypeError` jika `kontak` tidak ada. Pakai `toko.kontak?.telepon`, atau pastikan datanya ada lebih dulu.

### 3. Salah ketik nama properti

Simpan sebagai `salah-ketik.js`:

```javascript
const produk = { nama: "Pulpen", harga: 3000 };

console.log(produk.Harga);
console.log(produk.harga * 2);
console.log(produk.Harga * 2);
```

Output:

```text
undefined
6000
NaN
```

Key bersifat *case-sensitive*. `produk.Harga` bukan `produk.harga`, dan JavaScript tidak memberi error, hanya `undefined`. Hasil perhitungannya `NaN` (dari bab 3).

### 4. Mengira `const b = a` menyalin objek

Sudah terlihat di `const-objek.js`. **Perbaikan:** `const b = { ...a };`.

### 5. Spread bukan salinan mendalam

Simpan sebagai `salinan-dangkal.js`:

```javascript
const asli = { nama: "Budi", alamat: { kota: "Bandung" } };
const salinan = { ...asli };
salinan.alamat.kota = "Medan";

console.log(asli.alamat.kota);
```

Output:

```text
Medan
```

Objek `alamat` di dalamnya masih dibagi bersama oleh `asli` dan `salinan`. Untuk mengubah properti bersarang tanpa memengaruhi aslinya, salin juga tingkat dalamnya: `{ ...asli, alamat: { ...asli.alamat, kota: "Medan" } }`.

### 6. Method ditulis sebagai arrow function

`suara: () => this.nama` tidak memakai `this` milik objek. Tulis method dengan sintaks `suara() { ... }`.

### 7. Membandingkan objek dengan `===`

`{ a: 1 } === { a: 1 }` bernilai `false`. Untuk membandingkan isi, bandingkan propertinya satu per satu, atau bandingkan hasil `JSON.stringify` (hanya jika urutan propertinya pasti sama).

## Ringkasan

- **Objek** adalah kumpulan pasangan `key: value`, ditulis `{ ... }`. Akses dengan `obj.key` atau `obj["key"]`.
- Properti bisa diubah, ditambah (`obj.baru = ...`), dan dihapus (`delete obj.key`). Properti yang tidak ada menghasilkan `undefined`.
- Gunakan kurung siku jika key ada di variabel atau mengandung spasi.
- `?.` membaca properti dengan aman dari data yang mungkin tidak ada.
- **Method** adalah fungsi di dalam objek, dan `this` merujuk ke objek yang memanggilnya.
- **Destructuring** (`const { a, b } = obj`, `const [x, y] = arr`) mengambil banyak nilai sekaligus.
- **Spread** (`{ ...obj }`, `[...arr]`) menyalin dan menggabung. Hasilnya salinan dangkal, dan key yang belakangan menimpa yang depan.
- `const` hanya mengunci nama variabel. Objek adalah referensi, jadi salin dengan spread.
- `Object.keys`, `Object.values`, dan `Object.entries` mengubah objek menjadi array agar bisa diproses.
- `JSON.stringify` mengubah objek menjadi teks, dan `JSON.parse` mengembalikannya.

## Latihan

Kerjakan sendiri dulu sebelum melihat jawaban.

### Latihan 1: Prediksi output

Apa yang tampil?

```javascript
const a = { x: 1 };
const b = a;
b.x = 5;
console.log(a.x);
```

??? success "Jawaban"
    Output: `5`. `b` hanyalah nama kedua untuk objek yang sama, sehingga mengubah `b.x` mengubah `a.x`.

### Latihan 2: Perbaiki error

Kode ini menghasilkan `TypeError`. Perbaiki agar menampilkan `undefined` tanpa error:

```javascript
const pengguna = { nama: "Ani" };
console.log(pengguna.alamat.kota);
```

??? success "Jawaban"
    ```javascript
    const pengguna = { nama: "Ani" };
    console.log(pengguna.alamat?.kota);
    ```
    Output: `undefined`. Tanda `?.` menghentikan pembacaan ketika `alamat` tidak ada.

### Latihan 3: Objek buku

Buat objek `buku` dengan properti `judul`, `penulis`, dan `tahun`, serta method `deskripsi()` yang mengembalikan teks `Judul oleh Penulis (Tahun)`.

??? success "Contoh jawaban"
    ```javascript
    const buku = {
      judul: "Laskar Pelangi",
      penulis: "Andrea Hirata",
      tahun: 2005,
      deskripsi() {
        return `${this.judul} oleh ${this.penulis} (${this.tahun})`;
      },
    };

    console.log(buku.deskripsi());
    ```
    Output: `Laskar Pelangi oleh Andrea Hirata (2005)`

### Latihan 4: Destructuring

Dari `{ nama: "Sari", umur: 20, kota: "Medan" }`, ambil `nama` dan `kota` dengan destructuring, lalu tampilkan `Sari dari Medan`.

??? success "Contoh jawaban"
    ```javascript
    const siswa = { nama: "Sari", umur: 20, kota: "Medan" };
    const { nama, kota } = siswa;

    console.log(`${nama} dari ${kota}`);
    ```
    Output: `Sari dari Medan`

### Latihan 5: Gabungkan pengaturan

Pengaturan bawaan adalah `{ tema: "terang", bahasa: "id", ukuran: 14 }`. Pengguna memilih `{ tema: "gelap" }`. Buat pengaturan akhir tanpa mengubah pengaturan bawaan.

??? success "Contoh jawaban"
    ```javascript
    const bawaan = { tema: "terang", bahasa: "id", ukuran: 14 };
    const pilihan = { tema: "gelap" };
    const akhir = { ...bawaan, ...pilihan };

    console.log(akhir);
    console.log(bawaan);
    ```
    Output:
    ```text
    { tema: 'gelap', bahasa: 'id', ukuran: 14 }
    { tema: 'terang', bahasa: 'id', ukuran: 14 }
    ```

### Latihan 6: Daftar produk

Diberikan produk berikut:

```javascript
const produk = [
  { nama: "Pensil", harga: 2000, stok: 10 },
  { nama: "Buku", harga: 8000, stok: 0 },
  { nama: "Pulpen", harga: 3000, stok: 5 },
];
```

Tampilkan nama produk yang stoknya masih ada, lalu hitung total nilai seluruh stok (jumlah harga × stok).

??? success "Contoh jawaban"
    ```javascript
    const tersedia = produk.filter((p) => p.stok > 0).map((p) => p.nama);
    const nilaiStok = produk.reduce((jumlah, p) => jumlah + p.harga * p.stok, 0);

    console.log(tersedia);
    console.log(nilaiStok);
    ```
    Output:
    ```text
    [ 'Pensil', 'Pulpen' ]
    35000
    ```
    Nilai stok: 2000 × 10 + 8000 × 0 + 3000 × 5 = 35000.

### Latihan 7: Rekening

Memakai objek `rekening` dari contoh praktis (saldo awal 100000), panggil `setor(50000)`, coba `tarik(200000)`, lalu `tarik(30000)`. Tampilkan hasil tiap `tarik` dan saldo akhir.

??? success "Jawaban"
    Output: `Saldo tidak cukup` untuk penarikan 200000, `Berhasil` untuk 30000, dan saldo akhir `120000`. Prosesnya: 100000 + 50000 = 150000, penarikan pertama ditolak, lalu 150000 - 30000 = 120000.

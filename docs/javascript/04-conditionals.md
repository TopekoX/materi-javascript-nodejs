## Tujuan pembelajaran

Setelah mempelajari bab ini, kamu dapat:

- Menjelaskan mengapa program perlu mengambil keputusan.
- Menulis `if`, `else`, dan `else if` dengan benar.
- Menggabungkan perbandingan dan operator logika di dalam kondisi.
- Menjelaskan nilai *truthy* dan *falsy*.
- Memilih antara `if`, `switch`, dan operator ternary.
- Mengenali kesalahan umum pada percabangan.

Bab ini memakai materi bab 1 sampai 3, terutama operator perbandingan dan logika dari bab 2. Jalankan setiap contoh dengan `node nama-file.js`.

## Pendahuluan: program yang mengambil keputusan

Dalam hidup sehari-hari kita terus membuat keputusan: *jika hujan, bawa payung; jika tidak, tidak perlu*. Sampai bab 3, program kita berjalan lurus dari baris pertama sampai terakhir tanpa memilih. Program yang berguna harus bisa memilih: apakah saldo cukup, apakah kata sandi benar, apakah pengguna sudah dewasa.

Struktur yang memilih jalur eksekusi berdasarkan suatu syarat disebut **percabangan** (*conditional*). Syaratnya berupa nilai `true` atau `false`, yaitu hasil operator perbandingan yang sudah kamu pelajari.

## Pernyataan `if`

### Sintaks

```text
if (kondisi) {
  // dijalankan hanya jika kondisi bernilai true
}
```

Simpan sebagai `if-dasar.js`:

```javascript
const umur = 20;

if (umur >= 17) {
  console.log("Boleh membuat KTP.");
}
console.log("Selesai.");
```

Output:

```text
Boleh membuat KTP.
Selesai.
```

Penjelasan bagian-bagiannya:

| Bagian | Artinya |
|---|---|
| `if` | Kata kunci "jika" |
| `(umur >= 17)` | **Kondisi**: ekspresi yang hasilnya `true` atau `false` |
| `{ ... }` | **Blok**: kumpulan perintah yang dijalankan jika kondisinya `true` |
| `console.log("Selesai.")` | Di luar blok, jadi **selalu** dijalankan |

Proses eksekusinya:

1. `umur` bernilai 20.
2. `umur >= 17` dihitung dan hasilnya `true`.
3. Karena `true`, isi blok dijalankan dan mencetak `Boleh membuat KTP.`.
4. Program lanjut ke baris setelah blok dan mencetak `Selesai.`.

Jika `umur` diganti menjadi `15`, kondisinya `false`, blok dilewati, dan outputnya hanya `Selesai.`.

!!! tip "Rapikan dengan indentasi"
    Isi blok digeser dua spasi ke kanan agar mudah dibaca. JavaScript tidak mewajibkannya, tetapi hampir semua programmer melakukannya.

## `else`: jalur alternatif

Jika kita ingin melakukan sesuatu yang lain ketika kondisinya `false`, tambahkan `else`. Simpan sebagai `if-else.js`:

```javascript
const nilai = 55;

if (nilai >= 60) {
  console.log("Lulus");
} else {
  console.log("Tidak lulus");
}
```

Output:

```text
Tidak lulus
```

Hanya **satu** dari dua blok yang dijalankan. Di sini `55 >= 60` bernilai `false`, jadi blok `else` yang berjalan.

## `else if`: banyak pilihan

Untuk lebih dari dua kemungkinan, rangkai dengan `else if`. Simpan sebagai `else-if.js`:

```javascript
const nilai = 82;

if (nilai >= 90) {
  console.log("Predikat A");
} else if (nilai >= 80) {
  console.log("Predikat B");
} else if (nilai >= 70) {
  console.log("Predikat C");
} else {
  console.log("Predikat D");
}
```

Output:

```text
Predikat B
```

Proses eksekusinya:

1. `82 >= 90` bernilai `false`, jadi lanjut ke pemeriksaan berikutnya.
2. `82 >= 80` bernilai `true`, jadi mencetak `Predikat B`.
3. **Sisa rangkaian dilewati**, termasuk `nilai >= 70` yang juga benar.

Kondisi diperiksa dari atas ke bawah, dan hanya blok pertama yang kondisinya `true` yang dijalankan. Karena itu **urutan penting**: tulis kondisi yang paling ketat lebih dulu. Blok `else` di akhir bersifat opsional dan menangkap semua yang tersisa.

## Kondisi yang lebih rumit

Kondisi bisa digabung dengan `&&`, `||`, dan `!` dari bab 2. Simpan sebagai `kondisi-gabungan.js`:

```javascript
const umur = 19;
const punyaKtp = true;

if (umur >= 17 && punyaKtp) {
  console.log("Boleh masuk.");
} else {
  console.log("Ditolak.");
}

const jam = 22;
if (jam < 6 || jam >= 22) {
  console.log("Waktu istirahat.");
}
```

Output:

```text
Boleh masuk.
Waktu istirahat.
```

### `if` di dalam `if`

Percabangan boleh ditaruh di dalam blok percabangan lain. Simpan sebagai `if-bersarang.js`:

```javascript
const sudahLogin = true;
const adalahAdmin = false;

if (sudahLogin) {
  if (adalahAdmin) {
    console.log("Selamat datang, admin.");
  } else {
    console.log("Selamat datang, pengguna biasa.");
  }
} else {
  console.log("Silakan login dulu.");
}
```

Output:

```text
Selamat datang, pengguna biasa.
```

Alurnya: `sudahLogin` bernilai `true`, jadi masuk ke blok luar. Di dalamnya `adalahAdmin` bernilai `false`, sehingga `else` bagian dalam yang berjalan.

!!! tip "Hindari sarang yang terlalu dalam"
    Jika `if` bersarang lebih dari dua tingkat, biasanya bisa disederhanakan dengan `&&` atau `else if`. Contoh di atas bisa juga ditulis dengan `if (sudahLogin && adalahAdmin)`.

## Truthy dan falsy

Kondisi `if` tidak harus berupa `true` atau `false` murni. JavaScript akan mengubah nilai apa pun menjadi boolean, mirip type coercion di bab 3. Nilai yang dianggap `false` disebut **falsy**, dan nilai yang dianggap `true` disebut **truthy**.

Simpan sebagai `truthy-falsy.js`:

```javascript
console.log(Boolean(false));
console.log(Boolean(0));
console.log(Boolean(""));
console.log(Boolean(null));
console.log(Boolean(undefined));
console.log(Boolean(NaN));

console.log(Boolean(1));
console.log(Boolean(-5));
console.log(Boolean("halo"));
console.log(Boolean(" "));
console.log(Boolean("0"));
```

Output:

```text
false
false
false
false
false
false
true
true
true
true
true
```

`Boolean(nilai)` menampilkan bagaimana JavaScript memperlakukan nilai tersebut di dalam kondisi.

**Hanya enam nilai falsy yang perlu dihafal:**

| Falsy | Keterangan |
|---|---|
| `false` | |
| `0` | Angka nol |
| `""` | String kosong |
| `null` | |
| `undefined` | |
| `NaN` | |

**Semua nilai lain adalah truthy**, termasuk `" "` (satu spasi) dan `"0"` (teks nol).

Kegunaan praktisnya, simpan sebagai `truthy-if.js`:

```javascript
const namaPengguna = "";

if (namaPengguna) {
  console.log(`Halo, ${namaPengguna}!`);
} else {
  console.log("Nama belum diisi.");
}
```

Output:

```text
Nama belum diisi.
```

String kosong itu falsy, jadi `else` yang berjalan. Ini cara singkat untuk memeriksa "apakah ada isinya".

## Pernyataan `switch`

Kalau satu nilai dibandingkan dengan banyak kemungkinan yang pasti, `switch` lebih rapi daripada rangkaian `else if`. Simpan sebagai `switch-dasar.js`:

```javascript
const hari = 3;
let namaHari;

switch (hari) {
  case 1:
    namaHari = "Senin";
    break;
  case 2:
    namaHari = "Selasa";
    break;
  case 3:
    namaHari = "Rabu";
    break;
  default:
    namaHari = "Hari tidak valid";
}

console.log(namaHari);
```

Output:

```text
Rabu
```

| Bagian | Artinya |
|---|---|
| `switch (hari)` | Nilai yang akan dicocokkan |
| `case 3:` | Jika nilainya sama (dengan `===`) dengan `3`, mulai jalankan dari sini |
| `break;` | Keluar dari `switch` |
| `default:` | Dijalankan jika tidak ada `case` yang cocok (opsional) |

### Menggabungkan beberapa `case`

Beberapa `case` bisa berbagi satu aksi dengan menumpuknya tanpa `break`. Simpan sebagai `switch-gabung.js`:

```javascript
const hari = 6;
let jenis;

switch (hari) {
  case 6:
  case 7:
    jenis = "Akhir pekan";
    break;
  default:
    jenis = "Hari kerja";
}

console.log(jenis);
```

Output:

```text
Akhir pekan
```

### Lupa `break`

Tanpa `break`, eksekusi **terus mengalir** ke `case` berikutnya. Simpan sebagai `switch-tanpa-break.js`:

```javascript
const buah = "apel";

switch (buah) {
  case "apel":
    console.log("Apel");
  case "jeruk":
    console.log("Jeruk");
    break;
  default:
    console.log("Lainnya");
}
```

Output:

```text
Apel
Jeruk
```

`"Jeruk"` ikut tercetak padahal `buah` adalah `"apel"`. Selalu akhiri `case` dengan `break`, kecuali kamu memang sengaja menggabungkan seperti contoh sebelumnya.

## Operator ternary

Untuk memilih antara dua nilai dengan cepat, ada versi satu baris dari `if...else`:

```text
kondisi ? nilaiJikaTrue : nilaiJikaFalse
```

Simpan sebagai `ternary.js`:

```javascript
const umur = 17;
const status = umur >= 17 ? "Boleh" : "Belum boleh";
console.log(status);
console.log(`Stok: ${0 ? "Tersedia" : "Habis"}`);
```

Output:

```text
Boleh
Stok: Habis
```

- Tanda `?` dan `:` memisahkan tiga bagian: kondisi, nilai jika `true`, nilai jika `false`.
- Ternary menghasilkan **nilai**, jadi bisa disimpan ke variabel atau disisipkan ke template literal.
- Angka `0` bersifat falsy, sehingga yang dipilih adalah `"Habis"`.

!!! warning "Jangan berlebihan"
    Pakai ternary hanya untuk pilihan sederhana. Jika harus merangkai ternary di dalam ternary, gunakan `if...else if`.

## Kapan memakai yang mana?

| Situasi | Pilihan |
|---|---|
| Satu syarat, lakukan sesuatu atau tidak | `if` |
| Dua jalur | `if...else` |
| Beberapa rentang atau syarat berbeda | `if...else if...else` |
| Satu nilai cocok dengan banyak kemungkinan pasti | `switch` |
| Memilih satu dari dua **nilai** | ternary |

## Contoh praktis

### Menghitung diskon belanja

Simpan sebagai `diskon-belanja.js`:

```javascript
const totalBelanja = 350000;
const member = true;

let diskonPersen;
if (totalBelanja >= 500000) {
  diskonPersen = 20;
} else if (totalBelanja >= 250000) {
  diskonPersen = 10;
} else {
  diskonPersen = 0;
}

if (member) {
  diskonPersen += 5;
}

const potongan = totalBelanja * diskonPersen / 100;
const bayar = totalBelanja - potongan;

console.log(`Diskon: ${diskonPersen}%`);
console.log(`Potongan: ${potongan}`);
console.log(`Bayar: ${bayar}`);
```

Output:

```text
Diskon: 15%
Potongan: 52500
Bayar: 297500
```

Proses eksekusinya:

1. `350000 >= 500000` bernilai `false`, tetapi `350000 >= 250000` bernilai `true`, sehingga `diskonPersen` menjadi 10.
2. `member` bernilai `true`, jadi diskon bertambah 5 menjadi 15.
3. `potongan` = 350000 × 15 / 100 = 52500, dan `bayar` = 350000 - 52500 = 297500.

Diskon disimpan sebagai persen bulat (bukan 0,15) untuk menghindari ekor desimal seperti yang dibahas di bab 3.

### Menentukan tahun kabisat

Aturannya: tahun kabisat jika habis dibagi 4 tetapi tidak habis dibagi 100, **atau** habis dibagi 400. Simpan sebagai `kabisat.js`:

```javascript
const tahun = 2024;

if ((tahun % 4 === 0 && tahun % 100 !== 0) || tahun % 400 === 0) {
  console.log(`${tahun} adalah tahun kabisat.`);
} else {
  console.log(`${tahun} bukan tahun kabisat.`);
}
```

Output:

```text
2024 adalah tahun kabisat.
```

Tanda kurung mengelompokkan dua syarat pertama, sama seperti urutan operasi di bab 2.

## Kesalahan umum

### 1. Memakai `=` sebagai pembanding

Simpan sebagai `salah-sama-dengan.js`:

```javascript
let umur = 10;

if (umur = 17) {
  console.log("Boleh membuat KTP.");
}
console.log(umur);
```

Output:

```text
Boleh membuat KTP.
17
```

`umur = 17` **mengisi** variabel dengan 17. Hasilnya 17 yang bersifat truthy, sehingga blok selalu berjalan, dan nilai `umur` ikut berubah. **Perbaikan:** `umur === 17`.

### 2. Urutan `else if` yang salah

Simpan sebagai `salah-urutan.js`:

```javascript
const nilai = 95;

if (nilai >= 60) {
  console.log("Predikat C");
} else if (nilai >= 90) {
  console.log("Predikat A");
}
```

Output:

```text
Predikat C
```

Nilai 95 seharusnya A, tetapi `nilai >= 60` benar lebih dulu dan sisanya dilewati. **Perbaikan:** urutkan dari syarat yang paling ketat (`>= 90`) ke yang paling longgar.

### 3. Lupa `break` di `switch`

Eksekusi mengalir ke `case` berikutnya seperti pada contoh `switch-tanpa-break.js` di atas.

### 4. Variabel di dalam blok tidak terlihat di luar

Simpan sebagai `salah-blok.js`:

```javascript
if (true) {
  const pesan = "Halo";
}
console.log(pesan);
```

Output (dipersingkat):

```text
ReferenceError: pesan is not defined
```

Variabel yang dibuat dengan `let` atau `const` di dalam `{ }` hanya dikenal di dalam blok itu. **Perbaikan:** deklarasikan variabelnya di luar blok (seperti `let diskonPersen;` pada contoh diskon), lalu isi di dalam blok. Aturan lengkapnya dibahas di bab Scope.

### 5. Memakai truthy untuk angka yang bisa bernilai 0

`if (jumlah)` dianggap `false` ketika `jumlah` adalah `0`, padahal `0` mungkin nilai yang sah. Jika yang ingin diperiksa adalah "ada nilainya", tulis perbandingan yang eksplisit, misalnya `jumlah !== undefined`.

### 6. Tanda titik koma setelah kondisi

`if (umur >= 17);` dengan titik koma di akhir membuat JavaScript menganggap `if` itu tidak punya isi. Blok setelahnya akan selalu berjalan. Jangan beri titik koma setelah kurung kondisi.

## Ringkasan

- `if (kondisi) { ... }` menjalankan blok hanya jika kondisinya `true`.
- `else` menangani kondisi `false`. `else if` menambah pilihan, dan hanya blok pertama yang cocok yang berjalan.
- Urutkan `else if` dari syarat paling ketat.
- Kondisi bisa digabung dengan `&&`, `||`, `!`.
- Nilai falsy: `false`, `0`, `""`, `null`, `undefined`, `NaN`. Selain itu truthy.
- `switch` cocok untuk satu nilai dengan banyak kemungkinan pasti. Jangan lupa `break`.
- Ternary `kondisi ? a : b` memilih satu dari dua nilai.
- Pakai `===` untuk membandingkan, bukan `=`.

## Latihan

Kerjakan sendiri dulu sebelum melihat jawaban.

### Latihan 1: Prediksi output

Apa yang tampil?

```javascript
const x = 0;
if (x) {
  console.log("A");
} else {
  console.log("B");
}
```

??? success "Jawaban"
    Output: `B`. Angka `0` bersifat falsy, jadi blok `else` yang berjalan.

### Latihan 2: Perbaiki urutan

Kode ini menampilkan `Sejuk` untuk suhu 35, padahal seharusnya `Panas`. Perbaiki:

```javascript
const suhu = 35;

if (suhu >= 20) {
  console.log("Sejuk");
} else if (suhu >= 30) {
  console.log("Panas");
} else {
  console.log("Dingin");
}
```

??? success "Jawaban"
    Tukar urutan agar syarat yang paling ketat diperiksa lebih dulu:
    ```javascript
    const suhu = 35;

    if (suhu >= 30) {
      console.log("Panas");
    } else if (suhu >= 20) {
      console.log("Sejuk");
    } else {
      console.log("Dingin");
    }
    ```
    Output: `Panas`

### Latihan 3: Genap atau ganjil

Untuk `angka = 7`, tampilkan `7 adalah ganjil` atau `7 adalah genap` memakai `if...else` dan `%`.

??? success "Contoh jawaban"
    ```javascript
    const angka = 7;

    if (angka % 2 === 0) {
      console.log(`${angka} adalah genap`);
    } else {
      console.log(`${angka} adalah ganjil`);
    }
    ```
    Output: `7 adalah ganjil`

### Latihan 4: Keterangan nilai huruf

Gunakan `switch` untuk mengubah nilai huruf menjadi keterangan: `"A"` menjadi `"Sangat baik"`, `"B"` menjadi `"Baik"`, `"C"` menjadi `"Cukup"`, selain itu `"Perlu belajar lagi"`. Coba dengan `"B"`.

??? success "Contoh jawaban"
    ```javascript
    const huruf = "B";
    let keterangan;

    switch (huruf) {
      case "A":
        keterangan = "Sangat baik";
        break;
      case "B":
        keterangan = "Baik";
        break;
      case "C":
        keterangan = "Cukup";
        break;
      default:
        keterangan = "Perlu belajar lagi";
    }

    console.log(keterangan);
    ```
    Output: `Baik`

### Latihan 5: Tahun kabisat

Ubah contoh `kabisat.js` untuk memeriksa tahun 1900 dan tahun 2000. Tebak dulu hasilnya.

??? success "Jawaban"
    - 1900 habis dibagi 100 tetapi tidak habis dibagi 400, jadi **bukan** kabisat.
    - 2000 habis dibagi 400, jadi kabisat.

    Cukup ganti nilai `tahun` pada kode contoh.

### Latihan 6: Ongkos kirim

Aturan: berat sampai 1 kg ongkirnya 10000, di atas 1 kg sampai 5 kg ongkirnya 25000, di atas itu 50000. Hitung ongkir untuk berat 3 kg.

??? success "Contoh jawaban"
    ```javascript
    const berat = 3;
    let ongkir;

    if (berat <= 1) {
      ongkir = 10000;
    } else if (berat <= 5) {
      ongkir = 25000;
    } else {
      ongkir = 50000;
    }

    console.log(`Ongkir: ${ongkir}`);
    ```
    Output: `Ongkir: 25000`

### Latihan 7: Stok barang dengan ternary

Variabel `stok` berisi 0. Tampilkan `Habis` jika stok falsy, atau `Tersedia: <jumlah>` jika ada, dalam satu ekspresi ternary.

??? success "Contoh jawaban"
    ```javascript
    const stok = 0;
    console.log(stok ? `Tersedia: ${stok}` : "Habis");
    ```
    Output: `Habis`

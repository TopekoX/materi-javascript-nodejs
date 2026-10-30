## Tujuan pembelajaran

Setelah mempelajari bab ini, kamu dapat:

- Menjelaskan apa itu variabel dengan bahasamu sendiri.
- Membuat variabel dengan `let` dan `const`, lalu memilih yang tepat.
- Mengisi, membaca, dan mengubah nilai variabel.
- Mengenali lima jenis nilai dasar: teks, angka, benar/salah, belum diisi, dan sengaja kosong.
- Menulis nama variabel yang valid dan mudah dibaca.
- Membaca pesan error sederhana dan memperbaikinya.

Kamu tidak perlu pengalaman memrogram. Kamu hanya membutuhkan Node.js yang sudah terpasang dan sebuah terminal.

## Apa itu variabel?

Bayangkan sebuah **kotak penyimpanan yang diberi label**. Pada label tertulis "buku", dan di dalam kotak ada sebuah buku. Kapan pun kamu butuh buku, kamu cukup mencari kotak berlabel "buku".

Variabel bekerja dengan cara yang sama:

- **Label** adalah nama variabel.
- **Isi kotak** adalah nilai yang disimpan.

Program komputer harus mengingat banyak hal: nama pengguna, harga barang, skor permainan. Variabel adalah tempat program menyimpan hal-hal itu agar bisa dipakai kembali nanti.

## Menjalankan JavaScript pertama kali

Sebelum membuat variabel, kita perlu cara untuk melihat hasilnya.

1. Buat sebuah folder latihan, lalu buat file bernama `halo.js`.
2. Isi file dengan kode berikut:

```javascript
console.log("Halo, dunia!");
```

3. Buka terminal di folder itu, lalu jalankan:

```bash
node halo.js
```

Output di terminal:

```text
Halo, dunia!
```

Penjelasan kode:

| Bagian | Artinya |
|---|---|
| `console.log(...)` | Perintah untuk menampilkan sesuatu ke terminal. |
| `"Halo, dunia!"` | Sebuah teks. Teks selalu diapit tanda kutip. |
| `;` | Tanda akhir sebuah perintah. Kita akan memakainya di setiap perintah. |
| `node halo.js` | Meminta Node.js menjalankan isi file `halo.js`. |

Mulai sekarang, semua contoh di bab ini dijalankan dengan cara yang sama: simpan ke file `.js`, lalu jalankan dengan `node`.

## Membuat variabel dengan `let`

Kata `let` dipakai untuk membuat variabel yang **nilainya boleh diganti**. Simpan kode ini di `let-dasar.js`:

```javascript
let nama = "Budi";
console.log(nama);
nama = "Sari";
console.log(nama);
```

Output:

```text
Budi
Sari
```

Penjelasan baris demi baris:

| Baris | Artinya |
|---|---|
| `let nama = "Budi";` | Buat kotak berlabel `nama`, lalu isi dengan teks `"Budi"`. |
| `console.log(nama);` | Tampilkan isi kotak `nama`. Perhatikan: tidak ada tanda kutip, karena kita meminta isi variabel, bukan teks "nama". |
| `nama = "Sari";` | Ganti isi kotak `nama` menjadi `"Sari"`. Tidak ada `let` di depan, karena kotaknya sudah ada. |
| `console.log(nama);` | Tampilkan lagi. Sekarang isinya sudah berubah. |

Beberapa istilah yang perlu diketahui:

- **Deklarasi**: membuat variabel (`let nama`).
- **Assignment** (penugasan): memasukkan nilai ke variabel, memakai tanda `=`. Di JavaScript, `=` berarti "isi dengan", bukan "sama dengan".
- **Inisialisasi**: mengisi nilai pertama kali, biasanya bersamaan dengan deklarasi.

### Variabel yang belum diisi

Variabel boleh dibuat tanpa nilai. Simpan sebagai `let-kosong.js`:

```javascript
let umur = 20;
console.log("Umur sekarang:", umur);
umur = 21;
console.log("Umur setelah ulang tahun:", umur);
let skor;
console.log(skor);
skor = 90;
console.log(skor);
```

Output:

```text
Umur sekarang: 20
Umur setelah ulang tahun: 21
undefined
90
```

`console.log` bisa menerima beberapa nilai yang dipisah koma, dan akan menampilkannya berdampingan.

Variabel `skor` dibuat tanpa nilai, jadi isinya adalah `undefined`, yaitu nilai khusus yang berarti "kotak sudah ada, tetapi belum diisi".

## Nilai tetap dengan `const`

Kadang sebuah nilai tidak boleh berubah, misalnya nilai pi. Untuk itu kita memakai `const` (singkatan dari *constant*, atau konstanta). Simpan sebagai `const-dasar.js`:

```javascript
const pi = 3.14;
console.log(pi);
pi = 3.14159;
console.log("Baris ini tidak pernah dijalankan");
```

Output (dipersingkat):

```text
3.14
...\const-dasar.js:3
pi = 3.14159;
   ^

TypeError: Assignment to constant variable.
```

Yang terjadi, langkah demi langkah:

1. Baris 1 membuat `pi` dan mengisinya dengan `3.14`.
2. Baris 2 menampilkan `3.14`.
3. Baris 3 mencoba mengganti isi `pi`. Karena `pi` dibuat dengan `const`, JavaScript menolak dan menghentikan program dengan **error**.
4. Baris 4 tidak pernah dijalankan.

Error ini bukan musibah. Ia melindungimu dari kesalahan tidak sengaja.

### `let` atau `const`?

| | `let` | `const` |
|---|---|---|
| Boleh diisi ulang | Ya | Tidak |
| Wajib diisi saat dibuat | Tidak | Ya |
| Dipakai untuk | Nilai yang berubah (skor, saldo) | Nilai yang tetap (pi, nama aplikasi) |

!!! tip "Kebiasaan yang baik"
    Mulailah dengan `const`. Ganti menjadi `let` hanya jika kamu memang perlu mengubah nilainya.

## Tipe data dasar

Isi sebuah variabel punya **tipe**. Ada lima tipe sederhana yang perlu kamu kenal:

| Tipe | Contoh | Dipakai untuk |
|---|---|---|
| `string` | `"Budi"` | Teks |
| `number` | `20`, `3.14` | Angka |
| `boolean` | `true`, `false` | Benar atau salah |
| `undefined` | (belum diisi) | Variabel tanpa nilai |
| `null` | `null` | Sengaja dikosongkan |

Operator `typeof` menampilkan tipe sebuah nilai. Simpan sebagai `tipe-data.js`:

```javascript
let nama = "Budi";
let umur = 20;
let mahasiswa = true;
let alamat;
let pasangan = null;
console.log(typeof nama);
console.log(typeof umur);
console.log(typeof mahasiswa);
console.log(typeof alamat);
console.log(typeof pasangan);
```

Output:

```text
string
number
boolean
undefined
object
```

Hasil terakhir tampak aneh: `null` ternyata dilaporkan sebagai `object`. Ini kekeliruan lama dalam JavaScript yang tidak pernah diperbaiki. Abaikan saja untuk saat ini: ingat bahwa `null` berarti "sengaja kosong".

!!! warning "Jangan tertukar"
    `"20"` (dengan tanda kutip) adalah **teks**, sedangkan `20` adalah **angka**. Keduanya terlihat mirip, tetapi diperlakukan berbeda.

## Aturan dan konvensi penamaan

**Aturan** (jika dilanggar, program error):

- Boleh berisi huruf, angka, `_`, dan `$`.
- Tidak boleh diawali angka.
- Tidak boleh mengandung spasi atau tanda minus.
- Tidak boleh memakai kata kunci JavaScript seperti `let` atau `const`.
- Huruf besar dan kecil dianggap **berbeda** (*case-sensitive*): `nama` dan `Nama` adalah dua variabel yang berbeda.

**Konvensi** (kebiasaan agar kode mudah dibaca):

- Gunakan **camelCase**: kata pertama huruf kecil, kata berikutnya diawali huruf besar, misalnya `namaLengkap` atau `hargaSatuan`.
- Beri nama yang menjelaskan isinya: `umur` lebih baik daripada `x`.

| Nama | Valid? | Catatan |
|---|---|---|
| `namaLengkap` | Ya | camelCase |
| `skor2` | Ya | angka di belakang boleh |
| `_total` | Ya | boleh diawali garis bawah |
| `2skor` | Tidak | diawali angka |
| `nama lengkap` | Tidak | ada spasi |
| `harga-satuan` | Tidak | tanda minus dibaca sebagai operasi pengurangan |
| `let` | Tidak | kata kunci |

## Menggunakan variabel

### Menggabungkan variabel dengan teks

Ada dua cara. Simpan sebagai `gabung-teks.js`:

```javascript
const nama = "Budi";
const umur = 20;
console.log("Nama saya " + nama + ", umur " + umur + " tahun.");
console.log(`Nama saya ${nama}, umur ${umur} tahun.`);
```

Output:

```text
Nama saya Budi, umur 20 tahun.
Nama saya Budi, umur 20 tahun.
```

- Cara pertama memakai tanda `+` untuk menyambung teks.
- Cara kedua memakai **template literal**: teks diapit tanda **backtick** (`` ` ``, biasanya di sebelah kiri angka 1), dan variabel disisipkan dengan `${...}`. Cara ini lebih mudah dibaca.

### Menghitung dengan variabel

Contoh praktis: menghitung belanja. Simpan sebagai `belanja.js`:

```javascript
const harga = 15000;
const jumlah = 3;
const total = harga * jumlah;
console.log(`Total: ${total}`);

let saldo = 100000;
saldo = saldo - total;
console.log(`Sisa saldo: ${saldo}`);
```

Output:

```text
Total: 45000
Sisa saldo: 55000
```

Proses eksekusinya:

1. `harga` bernilai 15000 dan `jumlah` bernilai 3, keduanya `const` karena tidak berubah.
2. `harga * jumlah` dihitung (`*` berarti kali) menjadi 45000, lalu disimpan di `total`.
3. `saldo` dibuat dengan `let` karena nilainya akan berubah.
4. `saldo - total` dihitung dari nilai `saldo` saat ini (100000 - 45000), lalu hasilnya dimasukkan kembali ke `saldo`. Sisi kanan dihitung lebih dulu, baru hasilnya disimpan.

## `var`: kenapa ada, kenapa dihindari

Sebelum `let` dan `const` ada, JavaScript hanya punya `var`. Simpan sebagai `var-dasar.js`:

```javascript
var kota = "Bandung";
console.log(kota);
kota = "Jakarta";
console.log(kota);
```

Output:

```text
Bandung
Jakarta
```

Untuk contoh sederhana ini, `var` berperilaku seperti `let`. Namun `var` punya beberapa perilaku membingungkan yang membuat `let` dan `const` dibuat sebagai pengganti. Kamu akan mempelajarinya di bab berikutnya.

Untuk sekarang, cukup ingat: **di kode baru, pakai `const` dan `let`**. Kamu hanya perlu mengenali `var` saat membaca kode lama.

## Kesalahan umum

### 1. Salah huruf besar atau kecil

```javascript
let nama = "Budi";
console.log(Nama);
```

```text
ReferenceError: Nama is not defined
```

`Nama` (huruf besar) tidak pernah dibuat, yang ada hanya `nama`. **Perbaikan:** samakan penulisan huruf.

### 2. Memakai variabel sebelum dibuat

```javascript
console.log(umur);
let umur = 20;
```

```text
ReferenceError: Cannot access 'umur' before initialization
```

JavaScript membaca kode dari atas ke bawah. **Perbaikan:** buat variabel lebih dulu, baru pakai.

### 3. Mengubah `const`

```text
TypeError: Assignment to constant variable.
```

Sudah kita lihat di bagian `const`. **Perbaikan:** ganti `const` menjadi `let` jika nilainya memang perlu berubah.

### 4. Menulis `=` seolah "sama dengan"

`umur = 20` berarti "isi `umur` dengan 20", bukan "umur sama dengan 20". Perbandingan memakai operator lain yang akan dipelajari di bab selanjutnya.

### Cara membaca pesan error

Perhatikan dua hal pada pesan error di terminal:

- **Jenis dan pesan** (misalnya `ReferenceError: Nama is not defined`), yang menjelaskan apa yang salah.
- **Nama file dan nomor baris** di bagian atas (misalnya `...\const-dasar.js:3`), yang menunjukkan lokasinya.

## Ringkasan

- **Variabel** adalah kotak berlabel untuk menyimpan nilai.
- `let` untuk nilai yang boleh berubah, `const` untuk nilai yang tetap. Mulailah dengan `const`.
- `=` berarti "isi dengan".
- Variabel yang belum diisi bernilai `undefined`.
- Tipe dasar: `string`, `number`, `boolean`, `undefined`, `null`. Cek dengan `typeof`.
- Nama variabel bersifat case-sensitive, tidak boleh diawali angka, dan sebaiknya memakai camelCase.
- Template literal (`` `...${variabel}...` ``) memudahkan menyisipkan variabel ke dalam teks.
- `var` ada di kode lama, tetapi hindari di kode baru.

## Latihan

Kerjakan sendiri dulu sebelum melihat jawaban.

### Latihan 1: Prediksi output

Tebak apa yang tampil, lalu jalankan untuk memeriksa:

```javascript
let a = 5;
let b = a;
a = 10;
console.log(a, b);
```

??? success "Jawaban"
    Output: `10 5`. Saat `b = a` dijalankan, `b` hanya menyalin nilai `a` saat itu (5). Mengubah `a` setelahnya tidak memengaruhi `b`.

### Latihan 2: Perbaiki error

Kode berikut punya dua kesalahan. Temukan dan perbaiki:

```javascript
const kota = "Medan";
kota = "Surabaya";
console.log(Kota);
```

??? success "Jawaban"
    - `kota` dibuat dengan `const` lalu diubah. Ganti menjadi `let kota = "Medan";`.
    - `Kota` seharusnya `kota` (huruf kecil).

### Latihan 3: Kartu perkenalan

Buat variabel `nama`, `umur`, dan `mahasiswa` (true/false) dengan nilai tentang dirimu, lalu tampilkan satu kalimat memakai template literal.

### Latihan 4: Pilih `let` atau `const`

Untuk tiap variabel, pilih `let` atau `const` dan jelaskan alasannya:

1. `namaAplikasi` yang selalu "Toko Kita".
2. `skorPermainan` yang bertambah selama bermain.
3. `jumlahHariDalamSeminggu`.
4. `saldoTabungan`.

??? success "Jawaban"
    1. `const`, namanya tidak berubah.
    2. `let`, nilainya bertambah.
    3. `const`, selalu 7.
    4. `let`, saldo naik dan turun.

### Latihan 5: Luas segitiga

Buat variabel `alas` dan `tinggi`, hitung luas segitiga (alas × tinggi ÷ 2) dengan `*` dan `/`, lalu tampilkan hasilnya.

??? success "Contoh jawaban"
    ```javascript
    const alas = 10;
    const tinggi = 6;
    const luas = alas * tinggi / 2;
    console.log(`Luas segitiga: ${luas}`);
    ```
    Output: `Luas segitiga: 30`

### Latihan 6: Tukar isi dua variabel

Diberikan `let a = "Apel";` dan `let b = "Jeruk";`. Tukar isinya sehingga `a` berisi "Jeruk" dan `b` berisi "Apel". Petunjuk: pakai satu variabel sementara, seperti menukar isi dua gelas dengan gelas ketiga.

??? success "Contoh jawaban"
    ```javascript
    let a = "Apel";
    let b = "Jeruk";
    let sementara = a;
    a = b;
    b = sementara;
    console.log(a, b);
    ```
    Output: `Jeruk Apel`

### Latihan 7: Struk belanja

Buat variabel untuk `hargaSatuan` (12500), `banyak` (4), dan `pajak` (0.1 untuk 10%). Hitung subtotal dan total bayar (subtotal + pajak), lalu tampilkan keduanya.

??? success "Contoh jawaban"
    ```javascript
    const hargaSatuan = 12500;
    const banyak = 4;
    const pajak = 0.1;
    const subtotal = hargaSatuan * banyak;
    const totalBayar = subtotal + subtotal * pajak;
    console.log(`Subtotal: ${subtotal}`);
    console.log(`Total bayar: ${totalBayar}`);
    ```
    Output:
    ```text
    Subtotal: 50000
    Total bayar: 55000
    ```

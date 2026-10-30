## Tujuan pembelajaran

Setelah mempelajari bab ini, kamu dapat:

- Menulis teks (*string*) dengan tiga jenis tanda kutip dan karakter khusus.
- Mengambil informasi dan potongan dari sebuah string, lalu mengubah bentuknya.
- Menjelaskan bagaimana JavaScript menyimpan angka, termasuk `Infinity` dan `NaN`.
- Menjelaskan mengapa `0.1 + 0.2` tidak sama dengan `0.3`, dan cara menanganinya.
- Mengubah teks menjadi angka dan sebaliknya secara sengaja.
- Memahami type coercion, yaitu perubahan tipe otomatis yang dilakukan JavaScript.

Bab ini memakai materi bab 1 dan 2: variabel, tipe data, operator, dan template literal. Jalankan setiap contoh dengan `node nama-file.js`.

## Pendahuluan: teks dan angka ada di mana-mana

Hampir setiap program mengolah dua jenis data: **teks** (nama, alamat, pesan) dan **angka** (harga, umur, jumlah). Keduanya tampak sederhana, tetapi di sinilah pemula paling sering terkejut: `"5" + 3` menghasilkan `53`, `0.1 + 0.2` tidak persis `0.3`, dan data yang diketik pengguna selalu berupa teks walaupun isinya angka.

Analoginya seperti kertas dan kalkulator. Kertas menyimpan tulisan, kalkulator menghitung bilangan. Keduanya berguna, tetapi hasilnya aneh jika kita memasukkan tulisan ke kalkulator tanpa sadar. Bab ini mengajarkan cara mengolah teks dan angka dengan benar, serta cara mengubah satu menjadi yang lain secara sengaja.

## Strings

### Tiga cara menulis string

String adalah teks, yaitu rangkaian karakter. Simpan sebagai `string-dasar.js`:

```javascript
const tunggal = 'Halo';
const ganda = "Dunia";
const backtick = `Node`;
console.log(tunggal, ganda, backtick);
```

Output:

```text
Halo Dunia Node
```

Ketiganya menghasilkan string biasa. Perbedaannya:

| Penulisan | Kegunaan |
|---|---|
| `'...'` dan `"..."` | Sama persis. Pilih salah satu dan konsisten. |
| `` `...` `` | Template literal: bisa menyisipkan `${...}` dan bisa terdiri dari beberapa baris. |

### Karakter khusus (escape)

Bagaimana jika teks mengandung tanda kutip yang sama dengan pembungkusnya? Gunakan **backslash** (`\`) di depannya. Simpan sebagai `escape.js`:

```javascript
console.log("Dia berkata \"hai\"");
console.log('Dia bilang \'ya\'');
console.log("Baris satu\nBaris dua");
console.log("Kolom A\tKolom B");
console.log("Folder C:\\Belajar");
console.log(`Baris A
Baris B`);
```

Output:

```text
Dia berkata "hai"
Dia bilang 'ya'
Baris satu
Baris dua
Kolom A	Kolom B
Folder C:\Belajar
Baris A
Baris B
```

| Tulisan | Artinya |
|---|---|
| `\"` dan `\'` | Tanda kutip sebagai teks biasa |
| `\n` | Pindah ke baris baru |
| `\t` | Tab |
| `\\` | Satu backslash |

Template literal tidak butuh `\n`: baris baru yang kamu ketik langsung ikut menjadi bagian teks.

### Panjang dan posisi karakter

Setiap karakter dalam string punya nomor posisi yang disebut **indeks**. Penghitungan indeks dimulai dari **0**, bukan 1.

```text
J  a  v  a  S  c  r  i  p  t
0  1  2  3  4  5  6  7  8  9
```

Simpan sebagai `string-posisi.js`:

```javascript
const kata = "JavaScript";
console.log(kata.length);
console.log(kata[0]);
console.log(kata[4]);
console.log(kata[kata.length - 1]);
console.log(kata[20]);
```

Output:

```text
10
J
S
t
undefined
```

Penjelasan:

- `kata.length` adalah **properti** yang menyimpan jumlah karakter, yaitu 10.
- `kata[0]` mengambil karakter pada indeks 0, yaitu `J`. Tanda `[ ]` dipakai untuk menyebut posisi.
- Karakter terakhir ada pada indeks `length - 1`, karena indeks mulai dari 0. Kamu sudah bisa menghitungnya dengan operator dari bab 2.
- `kata[20]` tidak ada, sehingga hasilnya `undefined` (kamu mengenal nilai ini dari bab 1).

!!! note "Properti"
    Titik (`.`) setelah nama variabel dipakai untuk mengakses sesuatu yang "menempel" pada nilai itu. `kata.length` artinya "panjang milik `kata`". Konsep ini dibahas lengkap di bab Objects.

### String tidak bisa diubah, hanya diganti

Sebuah string bersifat **immutable**: isinya tidak bisa diubah sebagian. Semua perintah di bawah menghasilkan string **baru** dan membiarkan string aslinya.

### Perintah bawaan untuk string

JavaScript menyediakan banyak **method**, yaitu perintah bawaan yang dipanggil dengan titik dan tanda kurung, misalnya `kata.toUpperCase()`. Nilai di dalam kurung (kalau ada) disebut **argumen**, yaitu bahan tambahan untuk perintah tersebut. Bab Functions membahas cara membuat perintah sendiri.

Simpan sebagai `string-method.js`:

```javascript
const kata = "JavaScript";
console.log(kata.toUpperCase());
console.log(kata.toLowerCase());
console.log(kata.slice(0, 4));
console.log(kata.slice(4));
console.log(kata.indexOf("Script"));
console.log(kata.indexOf("xyz"));
console.log(kata.includes("Script"));
console.log(kata.startsWith("Java"));
console.log(kata.endsWith("Java"));
console.log(kata.replace("Java", "Type"));
console.log("ha".repeat(3));
console.log(kata);
```

Output:

```text
JAVASCRIPT
javascript
Java
Script
4
-1
true
true
false
TypeScript
hahaha
JavaScript
```

| Method | Fungsi |
|---|---|
| `toUpperCase()` / `toLowerCase()` | Ubah ke huruf besar / kecil |
| `slice(awal, akhir)` | Ambil potongan dari indeks `awal` sampai **sebelum** `akhir`. Jika `akhir` dihilangkan, ambil sampai habis |
| `indexOf(teks)` | Indeks tempat `teks` pertama ditemukan, atau `-1` jika tidak ada |
| `includes(teks)` | `true` jika `teks` ada di dalam string |
| `startsWith(teks)` / `endsWith(teks)` | Cek awal / akhir string |
| `replace(lama, baru)` | Ganti kemunculan pertama `lama` dengan `baru` |
| `repeat(n)` | Ulangi string `n` kali |

Baris terakhir menampilkan `JavaScript` lagi: string aslinya tidak berubah.

`slice` juga menerima angka negatif yang menghitung dari belakang: `"ID-2025-007".slice(-3)` menghasilkan `"007"`.

### Contoh praktis: membersihkan input

Pengguna sering mengetik spasi berlebih atau huruf besar-kecil sembarangan. Simpan sebagai `string-bersih.js`:

```javascript
const input = "   budi santoso  ";
console.log(`[${input}]`);
console.log(`[${input.trim()}]`);

const email = "  Budi@Contoh.COM ";
const bersih = email.trim().toLowerCase();
console.log(bersih);
console.log(bersih.includes("@"));
```

Output:

```text
[   budi santoso  ]
[budi santoso]
budi@contoh.com
true
```

- `trim()` membuang spasi di awal dan akhir.
- Method bisa **dirantai**: `email.trim().toLowerCase()` menjalankan `trim()` dulu, lalu `toLowerCase()` pada hasilnya.
- Tanda `[ ]` pada `console.log` hanya agar spasi terlihat.

## Numbers

### Satu tipe untuk semua angka

Berbeda dari banyak bahasa lain, JavaScript memakai satu tipe `number` untuk bilangan bulat maupun pecahan. Simpan sebagai `angka-dasar.js`:

```javascript
console.log(typeof 42);
console.log(typeof 3.14);
console.log(typeof -7);
console.log(10 / 4);
console.log(10 / 0);
console.log(-10 / 0);
console.log(1_000_000);
```

Output:

```text
number
number
number
2.5
Infinity
-Infinity
1000000
```

- `10 / 4` menghasilkan `2.5`: tidak ada pembulatan otomatis.
- `10 / 0` tidak menyebabkan error. Hasilnya `Infinity` (tak hingga).
- Garis bawah `_` boleh dipakai agar angka besar mudah dibaca. Nilainya sama saja.

### Kenapa `0.1 + 0.2` bukan `0.3`?

Simpan sebagai `desimal.js`:

```javascript
console.log(0.1 + 0.2);
console.log(0.1 + 0.2 === 0.3);
console.log((0.1 + 0.2).toFixed(2));
console.log(3.14159.toFixed(2));
```

Output:

```text
0.30000000000000004
false
0.30
3.14
```

Ini bukan kesalahan JavaScript. Komputer menyimpan angka dalam sistem biner, dan pecahan seperti 0,1 tidak bisa ditulis tepat dalam biner, sama seperti 1/3 tidak bisa ditulis tepat dalam desimal (0,3333...). Akibatnya ada selisih sangat kecil. Hampir semua bahasa pemrograman mengalami hal yang sama.

Cara menangani:

- `toFixed(n)` membulatkan ke `n` angka di belakang koma. **Hasilnya berupa string**, jadi dipakai untuk menampilkan, bukan untuk menghitung lagi.
- Jangan membandingkan hasil hitungan desimal dengan `===`.
- Untuk uang, simpan dalam satuan terkecil (misalnya rupiah utuh atau sen) agar terhindar dari pecahan.

### Perintah matematika bawaan

Simpan sebagai `math.js`:

```javascript
console.log(Math.round(4.5), Math.round(4.4));
console.log(Math.floor(4.9), Math.ceil(4.1));
console.log(Math.max(3, 9, 5), Math.min(3, 9, 5));
console.log(Math.abs(-8));
console.log(Math.PI);
```

Output:

```text
5 4
4 5
9 3
8
3.141592653589793
```

| Perintah | Fungsi |
|---|---|
| `Math.round(x)` | Bulatkan ke bilangan bulat terdekat |
| `Math.floor(x)` | Bulatkan ke **bawah** |
| `Math.ceil(x)` | Bulatkan ke **atas** |
| `Math.max(...)` / `Math.min(...)` | Nilai terbesar / terkecil dari angka-angka yang diberikan |
| `Math.abs(x)` | Nilai mutlak (buang tanda minus) |
| `Math.PI` | Nilai pi |

### `NaN`: bukan angka

`NaN` (*Not a Number*) adalah nilai yang muncul ketika perhitungan tidak menghasilkan angka yang masuk akal, misalnya `"abc" * 2`. Simpan sebagai `nan.js`:

```javascript
console.log("abc" * 2);
console.log(typeof NaN);
console.log(NaN === NaN);
console.log(Number.isNaN(NaN));
console.log(Number.isNaN("abc"));
```

Output:

```text
NaN
number
false
true
false
```

Hal-hal yang perlu diingat:

- Tipe `NaN` tetap `number`.
- `NaN` tidak sama dengan apa pun, **termasuk dirinya sendiri**, sehingga `NaN === NaN` bernilai `false`.
- Untuk memeriksanya, gunakan `Number.isNaN(nilai)`. Perintah ini hanya `true` untuk `NaN` yang sebenarnya.

## Mengubah antar tipe

Ada dua cara tipe berubah: **sengaja** (kamu yang meminta) dan **otomatis** (JavaScript yang memutuskan). Mulai dari yang sengaja.

### Teks menjadi angka

Simpan sebagai `ke-angka.js`:

```javascript
console.log(Number("42"));
console.log(Number("3.5"));
console.log(Number("  7 "));
console.log(Number(""));
console.log(Number("abc"));
console.log(Number(true), Number(false));
console.log(Number(null));
console.log(Number(undefined));
console.log(parseInt("42px"));
console.log(parseFloat("3.5kg"));
console.log(parseInt("abc"));
```

Output:

```text
42
3.5
7
0
NaN
1 0
0
NaN
42
3.5
NaN
```

| Perintah | Perilaku |
|---|---|
| `Number(x)` | Ketat: seluruh isi harus berupa angka, kalau tidak hasilnya `NaN`. Spasi di tepi diabaikan. |
| `parseInt(x)` | Membaca bilangan bulat dari **awal** teks dan berhenti di karakter pertama yang bukan angka |
| `parseFloat(x)` | Sama, tetapi mengenali pecahan |

Beberapa hasil yang perlu diperhatikan: string kosong menjadi `0`, `true` menjadi `1`, `false` menjadi `0`, `null` menjadi `0`, sedangkan `undefined` menjadi `NaN`.

### Angka menjadi teks

Simpan sebagai `ke-teks.js`:

```javascript
console.log(String(123), typeof String(123));
console.log((255).toString(), typeof (255).toString());
console.log(String(true));
```

Output:

```text
123 string
255 string
true
```

`String(x)` dan `x.toString()` sama-sama mengubah nilai menjadi string.

## Type coercion: perubahan tipe otomatis

Ketika sebuah operator menerima dua nilai dengan tipe berbeda, JavaScript berusaha mengubah salah satunya agar operasi bisa dijalankan. Proses ini disebut **type coercion**. Simpan sebagai `coercion.js`:

```javascript
console.log("5" + 3);
console.log("5" - 3);
console.log("5" * "2");
console.log("abc" * 2);
console.log(true + 1);
console.log(1 + 2 + "3");
console.log("1" + 2 + 3);
```

Output:

```text
53
2
10
NaN
2
33
123
```

Aturan yang bekerja:

1. **`+` dengan salah satu operand string**: menyambung teks. Angka diubah menjadi teks. Maka `"5" + 3` adalah `"53"`.
2. **`-`, `*`, `/` selalu bekerja pada angka**: teks diubah menjadi angka. Maka `"5" - 3` adalah `2`, dan `"5" * "2"` adalah `10`. Jika teksnya bukan angka, hasilnya `NaN`.
3. `true` menjadi `1` dan `false` menjadi `0` dalam operasi angka, sehingga `true + 1` adalah `2`.
4. Operasi dikerjakan dari kiri ke kanan (urutan bab 2):
    - `1 + 2 + "3"`: `1 + 2` menjadi `3`, lalu `3 + "3"` menyambung menjadi `"33"`.
    - `"1" + 2 + 3`: `"1" + 2` menjadi `"12"`, lalu `"12" + 3` menjadi `"123"`.

### Kenapa `==` berbahaya

Sekarang kamu bisa memahami alasan saran di bab 2 untuk memakai `===`. Simpan sebagai `longgar.js`:

```javascript
console.log(5 == "5");
console.log(5 === "5");
console.log(0 == "");
console.log(null == undefined);
console.log(null === undefined);
```

Output:

```text
true
false
true
true
false
```

`==` memakai coercion sebelum membandingkan, sehingga hasilnya sering mengejutkan: angka `0` dianggap sama dengan string kosong, dan `null` dianggap sama dengan `undefined`. `===` tidak pernah mengubah tipe, jadi hasilnya mudah ditebak.

!!! tip "Aturan praktis"
    Gunakan `===` dan `!==`. Jika data berasal dari teks (misalnya input pengguna), ubah dulu ke angka dengan `Number(...)` secara eksplisit, lalu hitung.

## Contoh praktis: harga dari input teks

Data dari pengguna atau file hampir selalu berupa teks. Simpan sebagai `harga-teks.js`:

```javascript
const hargaTeks = "15000";
const jumlahTeks = "3";
console.log(hargaTeks + jumlahTeks);
console.log(hargaTeks * jumlahTeks);

const harga = Number(hargaTeks);
const jumlah = Number(jumlahTeks);
console.log(`Total: ${harga * jumlah}`);

const hargaAsli = 19999;
const diskon = 0.1;
console.log(hargaAsli * (1 - diskon));
console.log(Math.round(hargaAsli * (1 - diskon)));
```

Output:

```text
150003
45000
Total: 45000
17999.100000000002
17999
```

Proses eksekusinya:

1. `hargaTeks + jumlahTeks` menyambung dua string, sehingga hasilnya `"150003"`, bukan 45000.
2. `hargaTeks * jumlahTeks` kebetulan bekerja karena coercion mengubah keduanya menjadi angka. Mengandalkan ini berisiko, sebab kalau operatornya diganti `+`, hasilnya salah.
3. Cara yang aman: ubah dulu dengan `Number(...)`, lalu hitung.
4. Hasil diskon memunculkan ekor desimal `17999.100000000002`. `Math.round` merapikannya menjadi bilangan bulat.

## Kesalahan umum

### 1. Menyambung, bukan menjumlah

`"12" + "8"` menghasilkan `"128"`, bukan `20`. **Perbaikan:** `Number("12") + Number("8")`.

### 2. Mengira indeks mulai dari 1

Karakter pertama ada pada `kata[0]`, bukan `kata[1]`. Karakter terakhir ada pada `kata[kata.length - 1]`.

### 3. Mengharapkan string berubah

Simpan sebagai `lupa-simpan.js`:

```javascript
let nama = "budi";
nama.toUpperCase();
console.log(nama);
nama = nama.toUpperCase();
console.log(nama);
```

Output:

```text
budi
BUDI
```

`nama.toUpperCase()` menghasilkan string baru, tetapi tidak disimpan. **Perbaikan:** tampung hasilnya dengan `=`.

### 4. Membandingkan desimal dengan `===`

`0.1 + 0.2 === 0.3` bernilai `false`. **Perbaikan:** bulatkan dulu, atau hitung dalam satuan terkecil.

### 5. Memeriksa `NaN` dengan `===`

`hasil === NaN` selalu `false`. **Perbaikan:** `Number.isNaN(hasil)`.

### 6. Memakai hasil `toFixed` untuk menghitung

`(0.1 + 0.2).toFixed(2)` adalah string `"0.30"`. Jika ditambahkan lagi dengan `+`, kamu justru menyambung teks. Gunakan `toFixed` hanya untuk tampilan.

## Ringkasan

- String ditulis dengan `'...'`, `"..."`, atau `` `...` ``. Karakter khusus ditulis dengan `\`.
- Indeks string dimulai dari 0. `length` memberi panjang, `[ ]` mengambil satu karakter.
- String tidak bisa diubah. Method seperti `toUpperCase()`, `slice()`, `trim()`, dan `replace()` menghasilkan string baru.
- JavaScript hanya punya satu tipe angka, `number`. Pembagian dengan 0 menghasilkan `Infinity`.
- Pecahan desimal tidak selalu tepat (`0.1 + 0.2`). Gunakan `toFixed` atau `Math.round` untuk tampilan.
- `NaN` adalah hasil perhitungan yang tidak valid. Periksa dengan `Number.isNaN`.
- `Number()`, `parseInt()`, dan `parseFloat()` mengubah teks ke angka. `String()` mengubah ke teks.
- Type coercion terjadi otomatis. `+` menyambung jika ada string, sedangkan `-`, `*`, `/` mengubah ke angka.
- Pakai `===` dan `!==`, dan ubah tipe secara eksplisit.

## Latihan

Kerjakan sendiri dulu sebelum melihat jawaban.

### Latihan 1: Prediksi output

Tebak hasilnya, lalu jalankan untuk memeriksa:

```javascript
console.log("7" + 3, "7" - 3);
```

??? success "Jawaban"
    Output: `73 4`. Operator `+` dengan string menyambung menjadi `"73"`. Operator `-` mengubah `"7"` menjadi angka 7, lalu 7 - 3 = 4.

### Latihan 2: Perbaiki penjumlahan

Kode ini seharusnya menampilkan `20`, tetapi hasilnya `128`. Perbaiki:

```javascript
const a = "12";
const b = "8";
console.log(a + b);
```

??? success "Jawaban"
    ```javascript
    const a = "12";
    const b = "8";
    console.log(Number(a) + Number(b));
    ```
    Output: `20`

### Latihan 3: Data nama

Buat variabel `nama` berisi `"Siti Aminah"`. Tampilkan panjangnya, huruf pertamanya, dan versi huruf besarnya.

??? success "Contoh jawaban"
    ```javascript
    const nama = "Siti Aminah";
    console.log(nama.length);
    console.log(nama[0]);
    console.log(nama.toUpperCase());
    ```
    Output:
    ```text
    11
    S
    SITI AMINAH
    ```

### Latihan 4: Bersihkan email

Dari `"  Rina@Mail.COM "`, hasilkan `rina@mail.com` dalam satu baris.

??? success "Contoh jawaban"
    ```javascript
    console.log("  Rina@Mail.COM ".trim().toLowerCase());
    ```
    Output: `rina@mail.com`

### Latihan 5: Harga setelah diskon

Harga barang 19999 dengan diskon 10%. Tampilkan harga akhir yang dibulatkan ke bilangan bulat.

??? success "Contoh jawaban"
    ```javascript
    const harga = 19999;
    const diskon = 0.1;
    console.log(Math.round(harga * (1 - diskon)));
    ```
    Output: `17999`

### Latihan 6: Potong kode faktur

Dari `"INV-2025-0042"`, ambil tahunnya (`"2025"`), nomor urutnya (`"0042"`), lalu ubah nomor urut menjadi angka.

??? success "Contoh jawaban"
    ```javascript
    const kode = "INV-2025-0042";
    console.log(kode.slice(4, 8));
    console.log(kode.slice(-4));
    console.log(Number(kode.slice(-4)));
    ```
    Output:
    ```text
    2025
    0042
    42
    ```

### Latihan 7: Rata-rata nilai

Hitung rata-rata dari tiga nilai: 78, 85, dan 91. Tampilkan hasil lengkap dan hasil yang dibulatkan ke satu angka di belakang koma.

??? success "Contoh jawaban"
    ```javascript
    const rata = (78 + 85 + 91) / 3;
    console.log(rata);
    console.log(rata.toFixed(1));
    ```
    Output:
    ```text
    84.66666666666667
    84.7
    ```

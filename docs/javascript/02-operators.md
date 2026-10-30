## Tujuan pembelajaran

Setelah mempelajari bab ini, kamu dapat:

- Menjelaskan apa itu operator, operand, dan ekspresi.
- Memakai operator aritmetika: `+`, `-`, `*`, `/`, `%`, dan `**`.
- Memperbarui nilai variabel dengan `+=`, `-=`, `*=`, `/=`, `++`, dan `--`.
- Memahami urutan pengerjaan operasi dan memakai tanda kurung.
- Membandingkan nilai dengan `>`, `<`, `>=`, `<=`, `===`, dan `!==`.
- Menggabungkan kondisi dengan `&&`, `||`, dan `!`.

Bab ini memakai hal-hal dari bab sebelumnya: `let`, `const`, `console.log`, dan template literal. Jalankan setiap contoh dengan `node nama-file.js`.

## Apa itu operator dan ekspresi?

Dalam matematika, kamu sudah mengenal tanda `+` dan `×`. Tanda-tanda itu menyuruh kita melakukan sesuatu pada angka. Di JavaScript, tanda seperti itu disebut **operator**.

Perhatikan `3 + 4`:

- `3` dan `4` disebut **operand**, yaitu nilai yang diolah.
- `+` adalah **operator**, yaitu perintah untuk mengolahnya.
- Seluruh `3 + 4` disebut **ekspresi**, yaitu potongan kode yang menghasilkan sebuah nilai (di sini: `7`).

Analoginya seperti mesin kasir: kamu memasukkan angka (operand), menekan tombol (operator), dan keluar hasilnya (nilai ekspresi).

Ekspresi bisa disimpan di variabel, atau langsung ditampilkan:

```javascript
const hasil = 3 + 4;
console.log(hasil);
console.log(3 + 4);
```

Keduanya menampilkan `7`.

## Operator aritmetika

Operator aritmetika dipakai untuk menghitung. Simpan sebagai `aritmetika.js`:

```javascript
const a = 17;
const b = 5;
console.log(a + b);
console.log(a - b);
console.log(a * b);
console.log(a / b);
console.log(a % b);
console.log(a ** 2);
```

Output:

```text
22
12
85
3.4
2
289
```

| Operator | Nama | Contoh | Hasil |
|---|---|---|---|
| `+` | Tambah | `17 + 5` | `22` |
| `-` | Kurang | `17 - 5` | `12` |
| `*` | Kali | `17 * 5` | `85` |
| `/` | Bagi | `17 / 5` | `3.4` |
| `%` | Sisa bagi (modulo) | `17 % 5` | `2` |
| `**` | Pangkat | `17 ** 2` | `289` |

Dua operator yang mungkin baru bagimu:

- **`%` (modulo)** memberi **sisa pembagian**. 17 dibagi 5 hasilnya 3, dengan sisa 2. Jadi `17 % 5` adalah `2`. Bayangkan membagi 17 permen kepada 5 anak: tiap anak dapat 3, dan 2 permen tersisa.
- **`**` (pangkat)** menghitung perpangkatan. `17 ** 2` sama dengan 17 × 17.

!!! tip "Kegunaan `%`"
    Modulo sangat berguna untuk memeriksa genap atau ganjil. Angka genap jika sisa bagi dua adalah 0: `14 % 2` hasilnya `0`, sedangkan `7 % 2` hasilnya `1`.

### `+` juga dipakai untuk teks

Di bab sebelumnya kamu sudah memakai `+` untuk menyambung teks. Simpan sebagai `plus-teks.js`:

```javascript
console.log("5" + 3);
console.log(5 + 3);
console.log("Halo " + "dunia");
```

Output:

```text
53
8
Halo dunia
```

Jika salah satu operand adalah teks, `+` menyambung, bukan menjumlahkan. Jadi `"5" + 3` menghasilkan teks `"53"`, bukan angka `8`. Bab berikutnya membahas cara JavaScript mengubah tipe nilai secara lebih lengkap.

## Urutan pengerjaan operasi

Seperti di matematika, kali dan bagi dikerjakan lebih dulu daripada tambah dan kurang. Simpan sebagai `urutan.js`:

```javascript
console.log(2 + 3 * 4);
console.log((2 + 3) * 4);
console.log(10 - 4 - 3);
console.log(2 ** 3 * 2);
```

Output:

```text
14
20
3
16
```

Penjelasan:

| Ekspresi | Cara JavaScript menghitung | Hasil |
|---|---|---|
| `2 + 3 * 4` | `3 * 4 = 12` dulu, baru `2 + 12` | `14` |
| `(2 + 3) * 4` | Tanda kurung dikerjakan lebih dulu: `5 * 4` | `20` |
| `10 - 4 - 3` | Dari kiri ke kanan: `6 - 3` | `3` |
| `2 ** 3 * 2` | Pangkat dulu: `8 * 2` | `16` |

Urutan dari yang paling dulu: **tanda kurung**, lalu **pangkat**, lalu **kali, bagi, dan sisa bagi**, lalu **tambah dan kurang**. Operator dengan tingkat yang sama dikerjakan dari kiri ke kanan.

!!! tip "Pakai kurung agar jelas"
    Jika ragu, tambahkan tanda kurung. Kode seperti `(a + b) * c` lebih mudah dibaca dan tidak bergantung pada hafalan urutan.

## Memperbarui nilai variabel

Di bab 1 kamu menulis `saldo = saldo - total`. Cara itu sering dipakai, sehingga JavaScript menyediakan singkatannya. Simpan sebagai `perbarui.js`:

```javascript
let skor = 10;
skor = skor + 5;
console.log(skor);
skor += 5;
console.log(skor);
skor -= 3;
console.log(skor);
skor *= 2;
console.log(skor);
skor /= 4;
console.log(skor);

let hitung = 0;
hitung++;
hitung++;
console.log(hitung);
hitung--;
console.log(hitung);
```

Output:

```text
15
20
17
34
8.5
2
1
```

Penjelasan proses eksekusinya:

| Baris | Artinya | `skor` menjadi |
|---|---|---|
| `skor = skor + 5` | Hitung `10 + 5`, simpan kembali | `15` |
| `skor += 5` | Singkatan dari `skor = skor + 5` | `20` |
| `skor -= 3` | Singkatan dari `skor = skor - 3` | `17` |
| `skor *= 2` | Singkatan dari `skor = skor * 2` | `34` |
| `skor /= 4` | Singkatan dari `skor = skor / 4` | `8.5` |

Untuk menambah atau mengurangi **satu**, tersedia bentuk yang lebih singkat lagi:

- `hitung++` menambah 1 (sama dengan `hitung += 1`).
- `hitung--` mengurangi 1 (sama dengan `hitung -= 1`).

Semua operator ini mengubah nilai variabel, jadi variabelnya harus dibuat dengan `let`, bukan `const`.

## Operator perbandingan

Operator perbandingan membandingkan dua nilai dan menghasilkan `true` (benar) atau `false` (salah). Hasil seperti ini bertipe **boolean**, tipe yang sudah kamu kenal di bab 1. Simpan sebagai `perbandingan.js`:

```javascript
const umur = 17;
console.log(umur > 17);
console.log(umur >= 17);
console.log(umur < 20);
console.log(umur <= 16);
console.log(umur === 17);
console.log(umur !== 17);
```

Output:

```text
false
true
true
false
true
false
```

| Operator | Artinya | `umur` = 17 | Hasil |
|---|---|---|---|
| `>` | lebih besar dari | `umur > 17` | `false` |
| `>=` | lebih besar atau sama dengan | `umur >= 17` | `true` |
| `<` | lebih kecil dari | `umur < 20` | `true` |
| `<=` | lebih kecil atau sama dengan | `umur <= 16` | `false` |
| `===` | sama dengan | `umur === 17` | `true` |
| `!==` | tidak sama dengan | `umur !== 17` | `false` |

!!! warning "`=` bukan `===`"
    Satu tanda `=` berarti **mengisi** variabel. Tiga tanda `===` berarti **membandingkan**. Menukar keduanya adalah salah satu kesalahan paling umum bagi pemula.

### `===` dan `==`

JavaScript punya dua cara memeriksa kesamaan: `===` dan `==`. Simpan sebagai `sama-dengan.js`:

```javascript
console.log(5 === "5");
console.log(5 == "5");
```

Output:

```text
false
true
```

- `===` membandingkan nilai **dan tipe**. Angka `5` dan teks `"5"` berbeda tipe, jadi hasilnya `false`.
- `==` mencoba mengubah tipe lebih dulu, sehingga `5 == "5"` menghasilkan `true`. Perilaku ini sering membingungkan.

**Biasakan memakai `===` dan `!==`.** Alasan lengkap `==` bersikap seperti itu dibahas di bab 3.

## Operator logika

Kadang kita perlu menggabungkan beberapa pernyataan benar/salah. Ada tiga operator logika:

| Operator | Nama | Artinya |
|---|---|---|
| `&&` | AND (dan) | `true` jika **kedua** sisi `true` |
| `\|\|` | OR (atau) | `true` jika **salah satu** sisi `true` |
| `!` | NOT (bukan) | membalik `true` menjadi `false` dan sebaliknya |

Analogi: untuk masuk gedung kamu harus punya kartu **dan** tanda pengenal (`&&`). Untuk membayar, kamu boleh memakai tunai **atau** kartu (`||`).

Simpan sebagai `logika.js`:

```javascript
const sudahLogin = true;
const adalahAdmin = false;
console.log(sudahLogin && adalahAdmin);
console.log(sudahLogin || adalahAdmin);
console.log(!sudahLogin);
console.log(!adalahAdmin);

const nilai = 75;
const lulus = nilai >= 60 && nilai <= 100;
console.log(lulus);
```

Output:

```text
false
true
false
true
true
```

Penjelasan:

1. `sudahLogin && adalahAdmin`: `true` dan `false`. Karena satu sisi salah, hasilnya `false`.
2. `sudahLogin || adalahAdmin`: cukup satu sisi benar, jadi hasilnya `true`.
3. `!sudahLogin` membalik `true` menjadi `false`.
4. `!adalahAdmin` membalik `false` menjadi `true`.
5. `nilai >= 60 && nilai <= 100`: `75 >= 60` benar dan `75 <= 100` benar, jadi `lulus` bernilai `true`. Ini cara menulis "nilai berada di antara 60 dan 100".

Tabel kebenaran:

| `a` | `b` | `a && b` | `a \|\| b` |
|---|---|---|---|
| `true` | `true` | `true` | `true` |
| `true` | `false` | `false` | `true` |
| `false` | `true` | `false` | `true` |
| `false` | `false` | `false` | `false` |

## Contoh praktis: menghitung diskon

Simpan sebagai `diskon.js`:

```javascript
const harga = 120000;
const diskon = 0.15;
const jumlah = 2;

const subtotal = harga * jumlah;
const potongan = subtotal * diskon;
const total = subtotal - potongan;

console.log(`Subtotal: ${subtotal}`);
console.log(`Potongan: ${potongan}`);
console.log(`Total: ${total}`);

const totalMenit = 135;
const sisaMenit = totalMenit % 60;
const jam = (totalMenit - sisaMenit) / 60;
console.log(`${totalMenit} menit = ${jam} jam ${sisaMenit} menit`);
```

Output:

```text
Subtotal: 240000
Potongan: 36000
Total: 204000
135 menit = 2 jam 15 menit
```

Proses eksekusinya:

1. `subtotal` = 120000 × 2 = 240000.
2. `potongan` = 240000 × 0,15 = 36000 (diskon 15%).
3. `total` = 240000 - 36000 = 204000.
4. Untuk mengubah 135 menit menjadi jam: `135 % 60` memberi sisa 15 menit. Lalu `(135 - 15) / 60` memberi 2 jam. Inilah contoh nyata kegunaan modulo.

## Kesalahan umum

### 1. Memakai `=` untuk membandingkan

`umur = 17` mengisi variabel, sedangkan `umur === 17` membandingkan. Gunakan `===` ketika ingin bertanya "apakah sama?".

### 2. Mengira ekspresi mengubah variabel

Simpan sebagai `tidak-berubah.js`:

```javascript
let angka = 5;
angka + 3;
console.log(angka);
angka = angka + 3;
console.log(angka);
```

Output:

```text
5
8
```

Baris `angka + 3;` menghitung 8 tetapi hasilnya tidak disimpan ke mana pun, sehingga `angka` tetap 5. Agar nilainya berubah, hasil harus dimasukkan kembali dengan `=`, `+=`, atau `++`.

### 3. Menyambung teks dan angka tanpa sadar

`"5" + 3` menghasilkan `"53"`, bukan `8`. Jika hasil yang muncul aneh, periksa apakah salah satu nilainya berupa teks (dengan tanda kutip).

### 4. Mengubah variabel `const`

```javascript
const skor = 10;
skor += 5;
```

```text
TypeError: Assignment to constant variable.
```

Operator seperti `+=` dan `++` mengubah isi variabel, sehingga tidak bisa dipakai pada `const`. **Perbaikan:** buat variabel dengan `let`.

### 5. Urutan operasi yang tidak sesuai harapan

`10 + 5 * 2` hasilnya `20`, bukan `30`. Jika ingin menjumlah dulu, tulis `(10 + 5) * 2`.

## Ringkasan

- **Operator** adalah tanda yang melakukan sesuatu pada **operand**. **Ekspresi** adalah kode yang menghasilkan nilai.
- Aritmetika: `+`, `-`, `*`, `/`, `%` (sisa bagi), `**` (pangkat).
- Urutan: tanda kurung, pangkat, kali/bagi/sisa, lalu tambah/kurang. Pakai kurung agar jelas.
- Singkatan pembaruan: `+=`, `-=`, `*=`, `/=`, `++`, `--`. Hanya untuk variabel `let`.
- Perbandingan menghasilkan `true` atau `false`: `>`, `<`, `>=`, `<=`, `===`, `!==`. Biasakan memakai `===`.
- Logika: `&&` (dan), `||` (atau), `!` (bukan).
- `=` mengisi, `===` membandingkan.

## Latihan

Kerjakan sendiri dulu sebelum melihat jawaban.

### Latihan 1: Prediksi output

Tebak hasilnya, lalu jalankan untuk memeriksa:

```javascript
console.log(10 + 5 * 2 - 4 / 2);
```

??? success "Jawaban"
    Output: `18`. Kali dan bagi dikerjakan dulu: `5 * 2 = 10` dan `4 / 2 = 2`. Lalu `10 + 10 - 2 = 18`.

### Latihan 2: `===` atau `!==`

Tebak hasil kedua baris ini:

```javascript
console.log(3 === "3");
console.log(3 !== "3");
```

??? success "Jawaban"
    Output: `false` lalu `true`. Angka `3` dan teks `"3"` berbeda tipe, jadi tidak dianggap sama.

### Latihan 3: Luas dan keliling

Buat variabel `panjang` (8) dan `lebar` (5), lalu tampilkan luas dan keliling persegi panjang dengan template literal.

??? success "Contoh jawaban"
    ```javascript
    const panjang = 8;
    const lebar = 5;
    console.log(`Luas: ${panjang * lebar}`);
    console.log(`Keliling: ${2 * (panjang + lebar)}`);
    ```
    Output:
    ```text
    Luas: 40
    Keliling: 26
    ```

### Latihan 4: Konversi suhu

Ubah 25 derajat Celsius ke Fahrenheit dengan rumus `F = C × 9 / 5 + 32`, lalu tampilkan hasilnya.

??? success "Contoh jawaban"
    ```javascript
    const celsius = 25;
    const fahrenheit = celsius * 9 / 5 + 32;
    console.log(`${celsius} C = ${fahrenheit} F`);
    ```
    Output: `25 C = 77 F`

### Latihan 5: Genap atau ganjil

Tampilkan `true` jika 14 adalah bilangan genap, lalu periksa juga angka 7. Petunjuk: pakai `%` dan `===`.

??? success "Contoh jawaban"
    ```javascript
    console.log(14 % 2 === 0);
    console.log(7 % 2 === 0);
    ```
    Output: `true` lalu `false`.

### Latihan 6: Kembalian

Total tagihan adalah 87500 dan pembeli membayar 100000. Tampilkan apakah uangnya cukup (`true`/`false`) dan berapa kembaliannya.

??? success "Contoh jawaban"
    ```javascript
    const tagihan = 87500;
    const bayar = 100000;
    console.log(bayar >= tagihan);
    console.log(bayar - tagihan);
    ```
    Output: `true` lalu `12500`.

### Latihan 7: Syarat masuk

Seseorang boleh masuk jika umurnya minimal 17 **dan** punya KTP. Buat variabel `umur` (19) dan `punyaKtp` (`true`), lalu tampilkan apakah ia boleh masuk. Setelah itu tampilkan kebalikannya memakai `!`.

??? success "Contoh jawaban"
    ```javascript
    const umur = 19;
    const punyaKtp = true;
    const bolehMasuk = umur >= 17 && punyaKtp;
    console.log(bolehMasuk);
    console.log(!bolehMasuk);
    ```
    Output: `true` lalu `false`.

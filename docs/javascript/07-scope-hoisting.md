## Tujuan pembelajaran

Setelah mempelajari bab ini, kamu dapat:

- Menjelaskan apa itu *scope* dan mengapa variabel tidak terlihat dari mana saja.
- Membedakan scope global, scope fungsi, dan scope blok.
- Menjelaskan bagaimana JavaScript mencari variabel dari scope dalam ke luar.
- Menjelaskan *hoisting* dan *temporal dead zone*.
- Menjelaskan perbedaan `var` dengan `let` dan `const`, sehingga paham alasan `var` dihindari.
- Menghindari variabel global yang tidak disengaja.

Bab ini menepati janji dari bab 1 ("perilaku `var` yang membingungkan") dan bab 4 ("variabel di dalam blok"). Jalankan setiap contoh dengan `node nama-file.js`.

## Pendahuluan: siapa boleh melihat apa?

Bayangkan sebuah rumah. Barang di **ruang tamu** boleh dipakai siapa saja yang masuk rumah. Barang di **kamar pribadi** hanya dipakai oleh penghuni kamar itu. Dan orang di dalam kamar bisa melihat ke ruang tamu, tetapi tamu di ruang tamu tidak bisa melihat isi kamar.

**Scope** adalah aturan yang menentukan di bagian program mana sebuah variabel bisa dipakai. Di bab 4 dan 6 kamu sudah bertemu dua kasusnya: variabel di dalam blok `if` dan variabel di dalam fungsi tidak terlihat dari luar. Bab ini merangkum aturannya.

Mengapa dibuat begitu? Agar nama variabel tidak saling bentrok. Kamu boleh memakai nama `hasil` di sepuluh fungsi berbeda tanpa saling mengganggu.

## Tiga jenis scope

### Scope global

Variabel yang dibuat di luar fungsi dan blok berada di **scope global**: bisa dibaca dari mana saja dalam file. Simpan sebagai `scope-global.js`:

```javascript
const pesanGlobal = "Saya global";

function tampil() {
  const pesanLokal = "Saya lokal";
  console.log(pesanGlobal);
  console.log(pesanLokal);
}

tampil();
console.log(pesanGlobal);
```

Output:

```text
Saya global
Saya lokal
Saya global
```

`pesanGlobal` terlihat di dalam maupun di luar fungsi. `pesanLokal` hanya ada di dalam `tampil`.

### Scope fungsi

Variabel dan parameter di dalam fungsi hanya hidup selama fungsi itu berjalan, dan tidak terlihat dari luar. Karena itu dua fungsi boleh memakai nama yang sama. Simpan sebagai `scope-fungsi.js`:

```javascript
function pertama() {
  const x = 1;
  return x;
}

function kedua() {
  const x = 2;
  return x;
}

console.log(pertama());
console.log(kedua());
```

Output:

```text
1
2
```

Kedua `x` adalah dua variabel yang berbeda, masing-masing milik fungsinya.

### Scope blok

**Blok** adalah kode di dalam `{ }`, misalnya isi `if`, `for`, atau `while`. Variabel yang dibuat dengan `let` dan `const` di dalam blok hanya dikenal di dalam blok itu. Simpan sebagai `scope-blok.js`:

```javascript
const umur = 20;

if (umur >= 17) {
  const status = "Dewasa";
  let catatan = "boleh membuat KTP";
  console.log(status, catatan);
}

for (let i = 1; i <= 2; i++) {
  console.log(`Putaran ${i}`);
}
```

Output:

```text
Dewasa boleh membuat KTP
Putaran 1
Putaran 2
```

Variabel `status` dan `catatan` hanya ada di dalam blok `if`. Penghitung `i` hanya ada di dalam `for`. Mencoba memakainya di luar akan menghasilkan `ReferenceError`.

| Scope | Dibuat oleh | Variabel terlihat di |
|---|---|---|
| Global | Kode di luar fungsi dan blok | Seluruh file |
| Fungsi | `function` atau arrow function | Di dalam fungsi itu saja |
| Blok | `{ }` pada `if`, `for`, `while` | Di dalam blok itu saja (untuk `let` dan `const`) |

## Scope bersarang dan rantai scope

Scope bisa berada di dalam scope lain, seperti kamar di dalam rumah. Ketika JavaScript mencari sebuah variabel, ia mulai dari scope **terdalam**, lalu naik ke scope di luarnya, sampai ke global. Rangkaian ini disebut **scope chain**. Simpan sebagai `scope-bersarang.js`:

```javascript
const a = "luar";

function f() {
  const b = "tengah";

  if (true) {
    const c = "dalam";
    console.log(a, b, c);
  }

  console.log(a, b);
}

f();
```

Output:

```text
luar tengah dalam
luar tengah
```

Penjelasan:

1. Di blok `if`, variabel `c` ditemukan di scope sendiri, `b` di scope fungsi, dan `a` di scope global.
2. Di luar blok `if`, hanya `a` dan `b` yang terlihat. `c` sudah tidak ada.

Aturannya satu arah: **scope dalam bisa melihat ke luar, tetapi scope luar tidak bisa melihat ke dalam.**

### Variabel dengan nama sama (*shadowing*)

Jika scope dalam punya variabel dengan nama yang sama dengan scope luar, variabel dalam yang dipakai. Variabel luar seolah "tertutup bayangan". Simpan sebagai `shadowing.js`:

```javascript
const nama = "Global";

function tampil() {
  const nama = "Lokal";
  console.log(nama);
}

tampil();
console.log(nama);
```

Output:

```text
Lokal
Global
```

Di dalam `tampil`, `nama` merujuk ke yang lokal. Variabel global tidak berubah. Ini aman, tetapi bisa membingungkan, jadi usahakan memakai nama berbeda.

## Hoisting

Sekarang kita sampai pada hal yang sering mengejutkan pemula. Sebelum menjalankan kode, JavaScript membaca scope dan mencatat semua deklarasi di dalamnya. Efeknya seolah-olah deklarasi **diangkat ke atas** scope. Fenomena ini disebut **hoisting** (*hoist* artinya mengangkat).

Hoisting bekerja berbeda untuk setiap jenis deklarasi.

### Hoisting pada `var`

Simpan sebagai `hoisting-var.js`:

```javascript
console.log(nama);
var nama = "Budi";
console.log(nama);
```

Output:

```text
undefined
Budi
```

Baris pertama tidak error, tetapi menampilkan `undefined`. JavaScript memperlakukannya seolah kodenya begini:

```javascript
var nama;               // deklarasi diangkat, nilainya undefined
console.log(nama);      // undefined
nama = "Budi";          // pengisian nilai tetap di tempatnya
console.log(nama);      // Budi
```

Yang diangkat hanya **deklarasinya**, bukan pengisian nilainya. Hasilnya, variabel `var` bisa dipakai sebelum dibuat, dan nilainya `undefined`. Ini sumber banyak kesalahan yang sulit ditemukan.

### Hoisting pada `let` dan `const`: temporal dead zone

`let` dan `const` juga diangkat, tetapi tidak boleh dipakai sebelum baris deklarasinya dijalankan. Rentang antara awal scope dan baris deklarasi disebut **temporal dead zone** (TDZ), yaitu "zona terlarang". Simpan sebagai `hoisting-let.js`:

```javascript
console.log(umur);
let umur = 20;
```

Output (dipersingkat):

```text
ReferenceError: Cannot access 'umur' before initialization
```

Ini pesan error yang kamu temui di bab 1. Sekarang kamu tahu penyebabnya: `umur` berada di TDZ pada baris pertama. Perilaku ini justru **lebih baik** daripada `var`, karena kesalahan langsung ketahuan alih-alih diam-diam menghasilkan `undefined`.

### Hoisting pada fungsi

Fungsi yang dibuat dengan `function` (declaration) diangkat **beserta isinya**, sehingga boleh dipanggil sebelum ditulis. Simpan sebagai `hoisting-fungsi.js`:

```javascript
console.log(kuadrat(4));

function kuadrat(x) {
  return x * x;
}
```

Output:

```text
16
```

Namun fungsi yang disimpan dalam variabel (function expression atau arrow function, dari bab 6) mengikuti aturan variabelnya. Simpan sebagai `hoisting-arrow.js`:

```javascript
console.log(kali(2, 3));

const kali = (a, b) => a * b;
```

Output (dipersingkat):

```text
ReferenceError: Cannot access 'kali' before initialization
```

Ringkasannya:

| Deklarasi | Dipakai sebelum ditulis |
|---|---|
| `function nama() {}` | Bisa dipanggil, fungsinya lengkap |
| `var x = ...` | Bisa dibaca, nilainya `undefined` |
| `let x = ...` / `const x = ...` | `ReferenceError` (TDZ) |
| `const f = () => {}` | `ReferenceError` (TDZ) |

!!! tip "Kebiasaan yang aman"
    Selalu deklarasikan variabel dan fungsi **sebelum** dipakai. Dengan begitu kamu tidak perlu mengandalkan hoisting.

## Mengapa `var` dihindari

Kini kita bisa menjelaskan perbedaan `var` dengan `let` dan `const` yang dijanjikan di bab 1. Ada tiga perbedaan utama.

### 1. `var` tidak punya scope blok

`var` hanya mengenal scope fungsi, sehingga ia "bocor" keluar dari blok. Simpan sebagai `var-bocor.js`:

```javascript
if (true) {
  var x = "bocor";
  let y = "aman";
}

console.log(x);
```

Output:

```text
bocor
```

`x` yang dibuat dengan `var` di dalam `if` masih bisa dipakai di luar. Sementara `y` yang dibuat dengan `let` tidak bisa. Hal yang sama terjadi pada perulangan. Simpan sebagai `var-loop.js`:

```javascript
for (var j = 0; j < 3; j++) {
  // kosong
}

console.log(j);
```

Output:

```text
3
```

Penghitung `j` tetap ada setelah perulangan selesai. Kalau memakai `let j`, variabel itu hilang bersama blok `for`.

### 2. `var` boleh dideklarasikan ulang

Simpan sebagai `var-ulang.js`:

```javascript
var jumlah = 1;
var jumlah = 2;
console.log(jumlah);
```

Output:

```text
2
```

Deklarasi ganda tidak dilaporkan, sehingga kita bisa tidak sengaja menimpa variabel lain yang bernama sama. Dengan `let` atau `const`, hal ini dicegah. Simpan sebagai `let-ulang.js`:

```javascript
let jumlah = 1;
let jumlah = 2;
```

Output (dipersingkat):

```text
SyntaxError: Identifier 'jumlah' has already been declared
```

`SyntaxError` muncul sebelum satu baris pun dijalankan, yaitu saat JavaScript membaca kodenya.

### 3. Hoisting `var` menghasilkan `undefined`

Seperti yang dilihat di `hoisting-var.js`, `var` bisa dipakai sebelum dibuat tanpa error.

### Perbandingan lengkap

| | `var` | `let` | `const` |
|---|---|---|---|
| Scope | Fungsi | Blok | Blok |
| Bisa diisi ulang | Ya | Ya | Tidak |
| Bisa dideklarasikan ulang | Ya | Tidak | Tidak |
| Dipakai sebelum deklarasi | `undefined` | `ReferenceError` | `ReferenceError` |

**Kesimpulan:** di kode baru, gunakan `const` dan `let`. Kamu hanya perlu memahami `var` untuk membaca kode lama.

## Contoh praktis: variabel global yang tidak disengaja

Salah satu bug paling licin terjadi ketika sebuah fungsi mengubah variabel di luar dirinya. Simpan sebagai `efek-samping.js`:

```javascript
let i = 100;

function ulang() {
  for (i = 0; i < 3; i++) {
    // kosong
  }
}

ulang();
console.log(i);
```

Output:

```text
3
```

Perhatikan `for (i = 0; ...)`: tidak ada `let` di depan `i`. Akibatnya perulangan memakai variabel `i` yang ada di scope global, bukan variabel baru, dan mengubah nilainya dari 100 menjadi 3. Kode lain yang bergantung pada `i` akan rusak tanpa terlihat penyebabnya.

Perbaikannya: buat penghitung dengan `let` di dalam `for`.

```javascript
let i = 100;

function ulang() {
  for (let i = 0; i < 3; i++) {
    // kosong
  }
}

ulang();
console.log(i);
```

Sekarang `i` di dalam fungsi adalah variabel terpisah, dan `console.log(i)` tetap menampilkan `100`.

Bandingkan juga dua gaya berikut. Simpan sebagai `global-vs-parameter.js`:

```javascript
let total = 0;

function tambahKeTotal(x) {
  total += x;
}

function jumlah(a, b) {
  return a + b;
}

tambahKeTotal(5);
tambahKeTotal(10);
console.log(total);
console.log(jumlah(5, 10));
```

Output:

```text
15
15
```

Keduanya menghasilkan 15, tetapi `tambahKeTotal` bergantung pada dan mengubah variabel di luarnya, sehingga hasilnya bergantung pada kondisi program. `jumlah` hanya memakai parameter dan `return`, sehingga hasilnya selalu sama untuk masukan yang sama. Fungsi seperti `jumlah` lebih mudah dipahami dan diuji.

## Kesalahan umum

### 1. Memakai variabel blok di luar blok

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

**Perbaikan:** deklarasikan variabelnya di luar blok dengan `let`, lalu isi di dalam blok.

```javascript
let pesan;
if (true) {
  pesan = "Halo";
}
console.log(pesan);
```

### 2. Lupa `let` atau `const` (variabel global tidak sengaja)

Simpan sebagai `lupa-deklarasi.js`:

```javascript
function hitung() {
  hasil = 10;
}

hitung();
console.log(hasil);
```

Output:

```text
10
```

Pada `hasil = 10` tidak ada `let` atau `const`. Dalam kondisi default, JavaScript lalu membuat variabel global baru secara diam-diam. Program berjalan, tetapi `hasil` bocor ke luar fungsi. **Perbaikan:** selalu tulis `const` atau `let` saat pertama kali membuat variabel.

### 3. Memakai `var` di dalam blok lalu terkejut

Seperti pada `var-bocor.js`, `var` tetap hidup di luar blok. Ganti dengan `let` atau `const`.

### 4. Memanggil arrow function sebelum dibuat

`kali(2, 3)` yang dipanggil sebelum `const kali = ...` menghasilkan `ReferenceError`. Fungsi dengan `function` boleh dipanggil lebih dulu, tetapi arrow function tidak. Tulis arrow function sebelum dipakai.

### 5. Nama yang sama membuat bingung

Pada `shadowing.js`, `nama` di dalam fungsi berbeda dari `nama` di luar. Jika ingin mengubah variabel luar, jangan mendeklarasikan ulang dengan `const` atau `let` di dalam. Lebih baik pakai nama berbeda atau kembalikan nilai dengan `return`.

## Ringkasan

- **Scope** menentukan di mana sebuah variabel bisa dipakai. Ada scope global, fungsi, dan blok.
- `let` dan `const` punya scope blok. `var` hanya punya scope fungsi dan bocor dari blok.
- JavaScript mencari variabel dari scope terdalam ke luar (*scope chain*). Scope dalam bisa melihat ke luar, tetapi tidak sebaliknya.
- Variabel dalam yang bernama sama menutupi variabel luar (*shadowing*).
- **Hoisting**: deklarasi diangkat ke atas scope. `var` menjadi `undefined`, `let` dan `const` masuk TDZ (`ReferenceError`), fungsi declaration bisa dipanggil lebih dulu.
- `var` boleh dideklarasikan ulang, `let` dan `const` tidak (`SyntaxError`).
- Selalu tulis `const` atau `let` saat membuat variabel, agar tidak menciptakan variabel global tanpa sadar.
- Utamakan fungsi yang memakai parameter dan `return`, bukan mengubah variabel global.

## Latihan

Kerjakan sendiri dulu sebelum melihat jawaban.

### Latihan 1: Prediksi output

Apa yang tampil?

```javascript
let a = 1;

if (true) {
  let a = 2;
  console.log(a);
}

console.log(a);
```

??? success "Jawaban"
    Output: `2` lalu `1`. Di dalam blok `if`, `let a = 2` membuat variabel baru yang menutupi `a` di luar. Setelah blok selesai, `a` yang asli tetap 1.

### Latihan 2: Perbaiki akses variabel

Kode ini menghasilkan `ReferenceError`. Perbaiki agar menampilkan `Lulus`:

```javascript
const nilai = 80;

if (nilai >= 60) {
  let keterangan = "Lulus";
}

console.log(keterangan);
```

??? success "Jawaban"
    Deklarasikan `keterangan` di luar blok:
    ```javascript
    const nilai = 80;
    let keterangan;

    if (nilai >= 60) {
      keterangan = "Lulus";
    }

    console.log(keterangan);
    ```
    Output: `Lulus`

### Latihan 3: Hoisting

Tebak output dua potongan kode ini sebelum menjalankannya:

```javascript
console.log(a);
var a = 5;
```

```javascript
console.log(b);
let b = 5;
```

??? success "Jawaban"
    - Potongan pertama menampilkan `undefined`, karena deklarasi `var` diangkat tetapi nilainya belum diisi.
    - Potongan kedua menghasilkan `ReferenceError: Cannot access 'b' before initialization`, karena `b` berada di temporal dead zone.

### Latihan 4: Shadowing

Apa yang tampil?

```javascript
const x = "luar";

function f() {
  const x = "dalam";
  return x;
}

console.log(f(), x);
```

??? success "Jawaban"
    Output: `dalam luar`. Fungsi memakai `x` miliknya sendiri, dan `x` global tidak terpengaruh.

### Latihan 5: `var` di perulangan

Apa yang tampil? Lalu apa yang terjadi jika `var` diganti `let`?

```javascript
for (var i = 0; i < 3; i++) {
  // kosong
}

console.log(i);
```

??? success "Jawaban"
    Output: `3`, karena `var` bocor dari blok `for`. Dengan `let i`, variabel itu hilang setelah perulangan dan `console.log(i)` menghasilkan `ReferenceError: i is not defined`.

### Latihan 6: Perbaiki variabel global tidak sengaja

Fungsi ini membuat variabel global tanpa disengaja. Perbaiki:

```javascript
function hitung() {
  total = 5 * 2;
  return total;
}

console.log(hitung());
```

??? success "Jawaban"
    Tambahkan `const` agar `total` menjadi variabel lokal:
    ```javascript
    function hitung() {
      const total = 5 * 2;
      return total;
    }

    console.log(hitung());
    ```
    Output: `10`

### Latihan 7: Ubah menjadi fungsi murni

Fungsi `bayar` di bawah bergantung pada variabel global `diskon`. Ubah agar `diskon` menjadi parameter, lalu panggil dengan total 200000 dan diskon 10.

```javascript
let diskon = 10;

function bayar(total) {
  return total - total * diskon / 100;
}
```

??? success "Contoh jawaban"
    ```javascript
    function bayar(total, diskon) {
      return total - total * diskon / 100;
    }

    console.log(bayar(200000, 10));
    ```
    Output: `180000`. Sekarang hasil `bayar` hanya ditentukan oleh argumennya, tidak oleh variabel di luar.

## Tujuan pembelajaran

Setelah mempelajari bab ini, kamu dapat:

- Membedakan error sintaks, error saat program berjalan, dan error logika.
- Membaca pesan error dan *stack trace* untuk menemukan sumber masalah.
- Menangani error dengan `try`, `catch`, dan `finally`.
- Membuat error sendiri dengan `throw` dan memilih jenis error yang tepat.
- Memvalidasi masukan agar program tidak berhenti mendadak.
- Menelusuri bug dengan `console.log`, `console.table`, dan langkah debugging yang sistematis.

Bab ini memakai hampir semua materi sebelumnya: fungsi, array, objek, dan JSON dari bab 9. Jalankan setiap contoh dengan `node nama-file.js`.

## Pendahuluan: error itu normal

Sejak bab 1 kamu sudah bertemu `ReferenceError`, `TypeError`, dan teman-temannya. Pemrogram berpengalaman pun melihat error setiap hari. Error bukan tanda kamu gagal, melainkan **pesan dari JavaScript yang memberi tahu apa yang salah dan di mana**.

Ada dua keterampilan yang dilatih di bab ini:

1. **Debugging**: menemukan dan memperbaiki penyebab error dalam kode yang kamu tulis.
2. **Error handling**: menulis program yang menghadapi masalah dengan tenang, misalnya data rusak atau masukan salah, alih-alih berhenti mendadak.

Analogi: debugging seperti mekanik yang mencari kerusakan mobil, sedangkan error handling seperti sabuk pengaman dan ban serep yang disiapkan sebelum masalah terjadi.

## Tiga jenis kesalahan

### 1. Error sintaks (*syntax error*)

Kode ditulis dengan tata bahasa yang salah, sehingga JavaScript menolak menjalankannya **sama sekali**. Simpan sebagai `sintaks.js`:

```javascript
console.log("Halo"
```

Output (dipersingkat):

```text
SyntaxError: missing ) after argument list
```

Tanda kurung penutup `)` hilang. Karena kesalahan ini ketahuan saat JavaScript membaca file, tidak ada satu baris pun yang dijalankan.

### 2. Error saat program berjalan (*runtime error*)

Kodenya valid secara tata bahasa, tetapi sebuah perintah gagal dijalankan. Program berhenti di titik itu. Berikut jenis yang paling sering:

| Jenis | Penyebab umum | Contoh pesan |
|---|---|---|
| `ReferenceError` | Memakai variabel yang tidak ada | `undefinedVar is not defined` |
| `TypeError` | Memakai nilai dengan cara yang tidak cocok untuk tipenya | `o.fungsi is not a function`, `Cannot read properties of null (reading 'x')` |
| `RangeError` | Angka di luar jangkauan yang diizinkan | `Invalid count value: -1` |
| `SyntaxError` | Teks yang diolah (misalnya `JSON.parse`) bukan format yang valid | `Unexpected token 'b', "bukan json" is not valid JSON` |

### 3. Error logika (*logic error*)

Program berjalan tanpa error, tetapi **hasilnya salah**. Ini yang paling sulit ditemukan karena JavaScript tidak memberi peringatan. Simpan sebagai `logika.js`:

```javascript
const nilai = [80, 90];
const total = nilai[0] + nilai[1];
const rata = total / 2;
const rataSalah = nilai[0] + nilai[1] / 2;

console.log(rata);
console.log(rataSalah);
```

Output:

```text
85
125
```

`rataSalah` lupa tanda kurung, sehingga pembagian dikerjakan lebih dulu (aturan urutan operasi bab 2). Tidak ada pesan error, hanya angka yang keliru. Menemukan kesalahan semacam ini butuh teknik debugging, yang dibahas di bagian akhir.

## Membaca pesan error dan stack trace

Ketika error tidak ditangani, Node.js mencetak pesan lengkap lalu menghentikan program. Simpan sebagai `stack.js`:

```javascript
function c() {
  throw new Error("Ada yang salah");
}
function b() {
  c();
}
function a() {
  b();
}
a();
```

Output (dipersingkat, `...` mewakili lokasi folder):

```text
...\stack.js:2
  throw new Error("Ada yang salah");
  ^

Error: Ada yang salah
    at c (...\stack.js:2:9)
    at b (...\stack.js:5:3)
    at a (...\stack.js:8:3)
    at Object.<anonymous> (...\stack.js:10:1)
```

Cara membacanya:

| Bagian | Artinya |
|---|---|
| `...\stack.js:2` | File dan nomor baris tempat error muncul |
| Baris kode dengan `^` | Baris yang bermasalah |
| `Error: Ada yang salah` | **Jenis** error dan **pesannya** |
| Daftar `at ...` | **Stack trace**: urutan pemanggilan fungsi yang membawa program ke error, dari yang paling dalam ke luar |

Pada stack trace, `at c (stack.js:2:9)` berarti error terjadi di fungsi `c`, baris 2, kolom 9. Di bawahnya, `c` dipanggil oleh `b` (baris 5), yang dipanggil oleh `a` (baris 8), yang dipanggil dari kode utama (baris 10). Baris-baris `at` di dalam Node.js sendiri tidak perlu kamu perhatikan.

!!! tip "Urutan membaca"
    Mulai dari **pesan error** (apa yang salah), lalu **baris pertama stack trace** yang menunjuk ke file milikmu (di mana). Hampir selalu itu sudah cukup untuk menemukan masalahnya.

## `try`, `catch`, dan `finally`

### Menangkap error dengan `try...catch`

Kita bisa meminta JavaScript **mencoba** menjalankan kode berisiko, dan menyiapkan jalur cadangan jika terjadi error. Simpan sebagai `try-catch.js`:

```javascript
try {
  console.log("Mulai");
  const hasil = JSON.parse("bukan json");
  console.log("Tidak sampai sini");
} catch (error) {
  console.log("Terjadi error!");
  console.log(error.name);
  console.log(error.message);
}

console.log("Program lanjut");
```

Output:

```text
Mulai
Terjadi error!
SyntaxError
Unexpected token 'b', "bukan json" is not valid JSON
Program lanjut
```

Proses eksekusinya:

1. Blok `try` mulai dijalankan dan mencetak `Mulai`.
2. `JSON.parse("bukan json")` gagal dan melemparkan error.
3. JavaScript **langsung melompat** ke blok `catch`. Baris `Tidak sampai sini` dilewati.
4. Objek error diberikan ke variabel `error` (nama bebas). Kita membaca `error.name` (jenis) dan `error.message` (pesan).
5. Setelah `catch` selesai, program **berlanjut normal** dengan `Program lanjut`.

Tanpa `try...catch`, error di langkah 2 akan menghentikan seluruh program.

### Objek error

Error adalah objek (bab 9) dengan tiga properti yang berguna:

| Properti | Isi |
|---|---|
| `name` | Jenis error, misalnya `"TypeError"` |
| `message` | Penjelasan singkat |
| `stack` | Teks stack trace lengkap |

### `finally`: selalu dijalankan

Blok `finally` dijalankan **apa pun yang terjadi**, baik ada error maupun tidak. Cocok untuk pekerjaan beres-beres. Simpan sebagai `finally.js`:

```javascript
function bacaAngka(teks) {
  try {
    const n = Number(teks);
    if (Number.isNaN(n)) {
      throw new Error(`"${teks}" bukan angka`);
    }
    return n;
  } catch (e) {
    console.log("Gagal:", e.message);
    return 0;
  } finally {
    console.log("Selesai memeriksa", teks);
  }
}

console.log(bacaAngka("42"));
console.log(bacaAngka("abc"));
```

Output:

```text
Selesai memeriksa 42
42
Gagal: "abc" bukan angka
Selesai memeriksa abc
0
```

Perhatikan bahwa `finally` dijalankan **sebelum** nilai `return` sampai ke pemanggil. Itu sebabnya `Selesai memeriksa 42` tercetak lebih dulu daripada `42`, dan `finally` tetap berjalan walaupun `try` atau `catch` sudah mengeksekusi `return`.

## Melempar error sendiri dengan `throw`

Kita juga bisa **membuat** error ketika menemukan keadaan yang tidak boleh terjadi. Perintahnya `throw`. Simpan sebagai `throw.js`:

```javascript
function bagi(a, b) {
  if (b === 0) {
    throw new Error("Pembagi tidak boleh nol");
  }
  return a / b;
}

try {
  console.log(bagi(10, 2));
  console.log(bagi(5, 0));
} catch (e) {
  console.log(`Error: ${e.message}`);
}
```

Output:

```text
5
Error: Pembagi tidak boleh nol
```

- `new Error("pesan")` membuat objek error baru dengan pesan tertentu. Kata `new` dipakai untuk membuat objek dari "cetakan" bawaan; bab Classes menjelaskannya lebih dalam.
- `throw` melempar error itu. Fungsi berhenti seketika, dan error naik ke pemanggil sampai ada `catch` yang menangkapnya.

!!! warning "Lempar objek `Error`, bukan teks"
    Tulis `throw new Error("pesan")`, bukan `throw "pesan"`. Hanya objek `Error` yang membawa stack trace. Teks biasa tidak punya `name`, `message`, maupun `stack`.

### Memilih jenis error

Selain `Error` umum, ada jenis yang lebih spesifik, termasuk `TypeError` dan `RangeError`. Memilih jenis yang tepat membuat penyebab masalah lebih jelas. Simpan sebagai `validasi.js`:

```javascript
function buatAkun(nama, umur) {
  if (typeof nama !== "string" || nama.trim() === "") {
    throw new TypeError("Nama harus berupa teks yang tidak kosong");
  }
  if (typeof umur !== "number") {
    throw new TypeError("Umur harus berupa angka");
  }
  if (umur < 0 || umur > 150) {
    throw new RangeError("Umur di luar jangkauan");
  }
  return { nama, umur };
}

const percobaan = [
  ["Budi", 20],
  ["", 20],
  ["Sari", "dua"],
  ["Andi", 200],
];

percobaan.forEach(([nama, umur]) => {
  try {
    console.log("OK:", buatAkun(nama, umur));
  } catch (e) {
    console.log(`${e.name}: ${e.message}`);
  }
});
```

Output:

```text
OK: { nama: 'Budi', umur: 20 }
TypeError: Nama harus berupa teks yang tidak kosong
TypeError: Umur harus berupa angka
RangeError: Umur di luar jangkauan
```

Fungsi `buatAkun` memeriksa masukan di awal (**validasi**) dan menolak yang tidak masuk akal. Perhatikan `try...catch` berada **di dalam** `forEach`, sehingga satu masukan yang salah tidak menghentikan pemeriksaan masukan berikutnya.

### Menangani sebagian, melempar ulang sisanya

Kadang kita hanya ingin menangani jenis error tertentu. Error lain sebaiknya dilempar lagi (**rethrow**) agar tidak tersembunyi. Simpan sebagai `rethrow.js`:

```javascript
function parseAman(teks) {
  try {
    return JSON.parse(teks);
  } catch (e) {
    if (e.name === "SyntaxError") {
      return null;
    }
    throw e;
  }
}

console.log(parseAman('{"a":1}'));
console.log(parseAman("rusak"));
```

Output:

```text
{ a: 1 }
null
```

Jika error-nya `SyntaxError` (data tidak valid), fungsi mengembalikan `null`. Jenis error lain di-`throw` ulang agar kesalahan yang tidak terduga tetap terlihat.

## Contoh praktis: membaca pengaturan dari JSON

Data dari luar program (file, pengguna, jaringan) bisa rusak kapan saja, sehingga pembacaannya perlu dilindungi. Simpan sebagai `pengaturan.js`:

```javascript
const bawaan = { tema: "terang", ukuran: 14 };

function bacaPengaturan(teks) {
  try {
    return { ...bawaan, ...JSON.parse(teks) };
  } catch {
    console.log("Pengaturan rusak, memakai bawaan.");
    return bawaan;
  }
}

console.log(bacaPengaturan('{"tema":"gelap"}'));
console.log(bacaPengaturan('{"tema":"gelap"'));
```

Output:

```text
{ tema: 'gelap', ukuran: 14 }
Pengaturan rusak, memakai bawaan.
{ tema: 'terang', ukuran: 14 }
```

- Pemanggilan pertama berisi JSON valid. Spread menggabungkannya dengan pengaturan bawaan (bab 9).
- Pemanggilan kedua kehilangan `}` di akhir, sehingga `JSON.parse` gagal. Program tidak berhenti, melainkan memakai pengaturan bawaan.
- `catch { ... }` tanpa `(error)` diperbolehkan jika kita tidak membutuhkan objek errornya.

## Debugging: menelusuri bug

### Menampilkan nilai dengan `console.log`

Cara paling sederhana untuk melihat apa yang terjadi di dalam program adalah mencetak nilai variabel. Simpan sebagai `debug-log.js`:

```javascript
const total = 409;
const rata = 81.8;
const nilai = [
  { nama: "Budi", nilai: 80 },
  { nama: "Sari", nilai: 92 },
];

console.log("total:", total);
console.log({ total, rata });
console.table(nilai);
```

Output:

```text
total: 409
{ total: 409, rata: 81.8 }
┌─────────┬────────┬───────┐
│ (index) │ nama   │ nilai │
├─────────┼────────┼───────┤
│ 0       │ 'Budi' │ 80    │
│ 1       │ 'Sari' │ 92    │
└─────────┴────────┴───────┘
```

Tiga teknik:

- `console.log("total:", total)` memberi **label** supaya tidak bingung angka mana yang sedang dilihat.
- `console.log({ total, rata })` memakai properti singkat (bab 9) untuk mencetak nama sekaligus nilai beberapa variabel.
- `console.table(...)` menampilkan array objek sebagai tabel yang mudah dibaca.

Ada juga `console.error(...)` dan `console.warn(...)` untuk pesan masalah. Keduanya keluar lewat jalur khusus yang disebut **stderr**, terpisah dari output normal.

### Contoh: mencari bug

Fungsi ini seharusnya menghitung rata-rata, tetapi hasilnya salah. Simpan sebagai `bug.js`:

```javascript
function rataRata(angka) {
  let total = 0;
  for (let i = 1; i <= angka.length; i++) {
    total += angka[i];
  }
  return total / angka.length;
}

console.log(rataRata([80, 90, 70]));
```

Output:

```text
NaN
```

Hasilnya `NaN` (dari bab 3), tanda ada angka yang tidak valid ikut dihitung. Mari telusuri dengan menambahkan `console.log` di dalam perulangan. Simpan sebagai `bug-debug.js`:

```javascript
function rataRata(angka) {
  let total = 0;
  for (let i = 1; i <= angka.length; i++) {
    console.log({ i, elemen: angka[i], total });
    total += angka[i];
  }
  return total / angka.length;
}

console.log(rataRata([80, 90, 70]));
```

Output:

```text
{ i: 1, elemen: 90, total: 0 }
{ i: 2, elemen: 70, total: 90 }
{ i: 3, elemen: undefined, total: 160 }
NaN
```

Jejak ini langsung membuka masalahnya:

1. Perulangan dimulai dari `i = 1`, sehingga elemen pertama (80) terlewat.
2. Pada `i = 3` elemennya `undefined`, karena indeks terakhir sebenarnya 2. Menjumlahkan `undefined` menghasilkan `NaN`.

Ini kesalahan klasik **selisih satu** (bab 5). Perbaikan: mulai dari `i = 0` dan gunakan `i < angka.length`.

```javascript
function rataRata(angka) {
  let total = 0;
  for (let i = 0; i < angka.length; i++) {
    total += angka[i];
  }
  return total / angka.length;
}

console.log(rataRata([80, 90, 70]));
```

Output:

```text
80
```

### Langkah debugging yang sistematis

1. **Baca pesan error** dengan teliti, jangan hanya melihat warna merahnya.
2. **Temukan barisnya** dari stack trace, lalu baca kode di sekitarnya.
3. **Persempit masalah.** Kecilkan kode sampai contoh paling pendek yang masih salah.
4. **Periksa nilai nyata**, jangan menebak. Cetak variabel yang dicurigai.
5. **Ubah satu hal sekali**, lalu jalankan lagi. Mengubah banyak hal sekaligus membuat sebab-akibat tidak jelas.
6. **Jelaskan kode dengan kata-kata**, bahkan kepada benda mati. Seringkali kesalahan terlihat saat kamu menjelaskannya baris demi baris.

### Debugger

Selain `console.log`, ada alat bernama **debugger** yang memungkinkan program dijeda di baris tertentu (**breakpoint**) dan dijalankan baris demi baris sambil melihat isi semua variabel. Pada editor seperti VS Code, kamu menaruh breakpoint dengan mengklik di sisi kiri nomor baris, lalu menjalankan program dalam mode debug. Di dalam kode, perintah `debugger;` juga berfungsi sebagai breakpoint ketika program dijalankan lewat alat debug. Untuk sekarang `console.log` sudah cukup; coba debugger saat programmu mulai besar.

## Kesalahan umum

### 1. `catch` kosong

Simpan sebagai `catch-kosong.js`:

```javascript
try {
  JSON.parse("rusak");
} catch (e) {
  // tidak melakukan apa-apa
}

console.log("Selesai tanpa kabar apa pun");
```

Output:

```text
Selesai tanpa kabar apa pun
```

Error ditelan diam-diam, sehingga bug tersembunyi dan sulit ditemukan nanti. **Perbaikan:** minimal catat pesannya (`console.log(e.message)`), atau tangani dengan jalur cadangan yang jelas.

### 2. Membungkus semuanya dalam satu `try`

Menaruh seluruh program di satu `try...catch` besar membuat sulit mengetahui bagian mana yang gagal. Bungkus hanya kode yang memang berisiko, dengan blok `try` sependek mungkin.

### 3. Memakai `try...catch` untuk alur normal

Jika sesuatu bisa diperiksa dengan `if`, pakai `if`. `try...catch` untuk keadaan yang **tidak terduga**, bukan untuk percabangan sehari-hari.

### 4. Melempar teks biasa

Simpan sebagai `throw-teks.js`:

```javascript
try {
  throw "ada masalah";
} catch (e) {
  console.log(typeof e);
  console.log(e.stack);
}
```

Output:

```text
string
undefined
```

Teks tidak punya stack trace. **Perbaikan:** `throw new Error("ada masalah")`.

### 5. Berharap `try...catch` menangkap error sintaks

Error sintaks muncul saat file dibaca, sebelum `try` sempat dijalankan, sehingga tidak bisa ditangkap. Perbaiki kodenya.

### 6. Mengubah banyak hal sekaligus saat debugging

Jika tiga hal diubah bersamaan dan masalah hilang, kamu tidak tahu mana yang memperbaikinya. Ubah satu per satu.

### 7. Lupa menghapus `console.log` debugging

Setelah bug ketemu, hapus pencetakan sementara agar output program tetap bersih.

## Ringkasan

- Tiga jenis kesalahan: **sintaks** (kode tidak bisa dibaca), **runtime** (gagal saat berjalan), dan **logika** (hasil salah tanpa error).
- Baca pesan error lalu stack trace. Cari baris pertama yang menunjuk ke file milikmu.
- `try { ... } catch (e) { ... }` menangkap error dan membuat program berlanjut. `finally` selalu dijalankan.
- Objek error punya `name`, `message`, dan `stack`.
- `throw new Error("pesan")` melempar error sendiri. Pilih jenis yang tepat, misalnya `TypeError` atau `RangeError`.
- Validasi masukan di awal fungsi dan tolak yang tidak masuk akal.
- Tangkap error yang kamu tahu cara menanganinya, lempar ulang (`throw e`) sisanya.
- Debugging: cetak nilai nyata (`console.log`, `console.table`), persempit masalah, dan ubah satu hal sekali.
- Hindari `catch` kosong dan `throw` teks biasa.

## Latihan

Kerjakan sendiri dulu sebelum melihat jawaban.

### Latihan 1: Prediksi output

Apa yang tampil, dan dalam urutan apa?

```javascript
try {
  console.log("A");
  null.x;
  console.log("B");
} catch (e) {
  console.log("C");
} finally {
  console.log("D");
}
```

??? success "Jawaban"
    Output: `A`, `C`, `D` (masing-masing di baris sendiri). Mengakses `null.x` menyebabkan `TypeError`, sehingga `B` dilewati dan program melompat ke `catch`. `finally` selalu berjalan di akhir.

### Latihan 2: Lindungi `JSON.parse`

Kode ini akan menghentikan program. Bungkus dengan `try...catch` agar mencetak `Data rusak` dan tetap menampilkan `Selesai`:

```javascript
const data = JSON.parse('{"a":');
console.log(data);
console.log("Selesai");
```

??? success "Contoh jawaban"
    ```javascript
    try {
      const data = JSON.parse('{"a":');
      console.log(data);
    } catch (e) {
      console.log("Data rusak");
    }
    console.log("Selesai");
    ```
    Output: `Data rusak` lalu `Selesai`.

### Latihan 3: Akar kuadrat yang aman

Buat fungsi `akar(n)` yang melempar `RangeError` dengan pesan `Angka tidak boleh negatif` jika `n` negatif, dan selain itu mengembalikan `Math.sqrt(n)`. Uji dengan 16 dan -4, dan tangkap errornya.

??? success "Contoh jawaban"
    ```javascript
    function akar(n) {
      if (n < 0) {
        throw new RangeError("Angka tidak boleh negatif");
      }
      return Math.sqrt(n);
    }

    [16, -4].forEach((n) => {
      try {
        console.log(akar(n));
      } catch (e) {
        console.log(`${e.name}: ${e.message}`);
      }
    });
    ```
    Output:
    ```text
    4
    RangeError: Angka tidak boleh negatif
    ```

### Latihan 4: Validasi password

Buat fungsi `cekPassword(p)` yang melempar `Error` dengan pesan `Password minimal 8 karakter` jika panjangnya kurang dari 8, dan `Password tidak boleh mengandung spasi` jika ada spasi (`includes(" ")`). Jika lolos, kembalikan `"OK"`. Uji dengan `"abc"`, `"rahasia123"`, dan `"ada spasi nih1"`.

??? success "Contoh jawaban"
    ```javascript
    function cekPassword(p) {
      if (p.length < 8) {
        throw new Error("Password minimal 8 karakter");
      }
      if (p.includes(" ")) {
        throw new Error("Password tidak boleh mengandung spasi");
      }
      return "OK";
    }

    ["abc", "rahasia123", "ada spasi nih1"].forEach((p) => {
      try {
        console.log(cekPassword(p));
      } catch (e) {
        console.log(e.message);
      }
    });
    ```
    Output:
    ```text
    Password minimal 8 karakter
    OK
    Password tidak boleh mengandung spasi
    ```

### Latihan 5: Cari bug

Fungsi ini seharusnya mencari angka terbesar, tetapi menghasilkan `undefined`. Temukan penyebabnya dengan `console.log`, lalu perbaiki:

```javascript
function terbesar(angka) {
  let max;
  for (let i = 0; i < angka.length; i++) {
    if (angka[i] > max) {
      max = angka[i];
    }
  }
  return max;
}

console.log(terbesar([3, 9, 4]));
```

??? success "Jawaban"
    `max` tidak pernah diberi nilai awal, sehingga bernilai `undefined`, dan `3 > undefined` selalu `false`. Perbaikan: beri nilai awal elemen pertama.
    ```javascript
    function terbesar(angka) {
      let max = angka[0];
      for (let i = 1; i < angka.length; i++) {
        if (angka[i] > max) {
          max = angka[i];
        }
      }
      return max;
    }

    console.log(terbesar([3, 9, 4]));
    ```
    Output: `9`

### Latihan 6: Parse dengan nilai bawaan

Buat fungsi `parseDenganBawaan(teks, bawaan)` yang mengembalikan hasil `JSON.parse(teks)`, atau `bawaan` jika teksnya tidak valid. Uji dengan `'{"x":1}'` dan `"rusak"`, dengan `bawaan` berupa `{ x: 0 }`.

??? success "Contoh jawaban"
    ```javascript
    function parseDenganBawaan(teks, bawaan) {
      try {
        return JSON.parse(teks);
      } catch {
        return bawaan;
      }
    }

    console.log(parseDenganBawaan('{"x":1}', { x: 0 }));
    console.log(parseDenganBawaan("rusak", { x: 0 }));
    ```
    Output:
    ```text
    { x: 1 }
    { x: 0 }
    ```

### Latihan 7: Urutan `finally`

Tebak urutan output kedua pemanggilan ini:

```javascript
function proses(angka) {
  console.log("Buka");
  try {
    if (angka < 0) {
      throw new Error("negatif");
    }
    return angka * 2;
  } catch (e) {
    return "gagal";
  } finally {
    console.log("Tutup");
  }
}

console.log(proses(5));
console.log(proses(-1));
```

??? success "Jawaban"
    Output:
    ```text
    Buka
    Tutup
    10
    Buka
    Tutup
    gagal
    ```
    `finally` selalu berjalan sebelum nilai `return` dikirim ke pemanggil, baik lewat `return` di `try` (untuk 5) maupun di `catch` (untuk -1).

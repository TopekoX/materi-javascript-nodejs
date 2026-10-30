## Tujuan pembelajaran

Setelah mempelajari bab ini, kamu dapat:

- Menjelaskan apa itu fungsi dan mengapa fungsi berguna.
- Membuat dan memanggil fungsi dengan `function`.
- Memakai parameter, argumen, dan nilai default.
- Mengembalikan hasil dengan `return`, dan membedakannya dari `console.log`.
- Menulis fungsi dengan *function expression* dan *arrow function*.
- Menyusun fungsi kecil yang saling memanggil.

Bab ini memakai materi bab 1 sampai 5, terutama `if` dan perulangan. Jalankan setiap contoh dengan `node nama-file.js`.

## Pendahuluan: resep yang bisa dipakai berulang

Bayangkan sebuah **resep**. Kamu menulis langkah-langkahnya sekali, lalu bisa memasaknya kapan pun tanpa menulis ulang. Kalau resepnya menerima bahan yang berbeda (misalnya jenis mie), hasil masakannya ikut berbeda.

**Fungsi** (*function*) bekerja dengan cara yang sama: sekumpulan perintah yang diberi nama, ditulis sekali, dan bisa dijalankan berkali-kali dengan bahan yang berbeda. Kamu sebenarnya sudah memakai fungsi bawaan sejak bab 1: `console.log(...)`, `Number(...)`, dan `Math.round(...)` adalah fungsi. Di bab ini kamu belajar membuatnya sendiri.

Manfaat fungsi:

- **Tidak mengulang kode.** Tulis sekali, pakai di banyak tempat.
- **Mudah dibaca.** Nama seperti `hitungDiskon` lebih jelas daripada deretan perhitungan.
- **Mudah diperbaiki.** Kesalahan cukup diperbaiki di satu tempat.

## Membuat dan memanggil fungsi

### Sintaks

```text
function namaFungsi() {
  // perintah yang dijalankan saat fungsi dipanggil
}
```

Simpan sebagai `fungsi-dasar.js`:

```javascript
function sapa() {
  console.log("Halo!");
}

sapa();
sapa();
```

Output:

```text
Halo!
Halo!
```

| Bagian | Artinya |
|---|---|
| `function` | Kata kunci untuk membuat fungsi |
| `sapa` | Nama fungsi. Aturan penamaannya sama dengan variabel, biasanya camelCase dan berupa kata kerja |
| `()` | Tempat parameter (dijelaskan di bawah). Kosong berarti fungsi tidak butuh bahan |
| `{ ... }` | **Badan fungsi**: perintah yang akan dijalankan |
| `sapa();` | **Pemanggilan** fungsi: menjalankan badan fungsi |

Penting: **menulis** fungsi tidak menjalankannya. Fungsi baru berjalan ketika dipanggil dengan tanda kurung, `sapa()`. Pada contoh, `sapa()` dipanggil dua kali sehingga `Halo!` tercetak dua kali.

## Parameter dan argumen

Fungsi menjadi berguna ketika bisa menerima "bahan". Bahan itu ditulis di dalam tanda kurung. Simpan sebagai `parameter.js`:

```javascript
function sapaNama(nama) {
  console.log(`Halo, ${nama}!`);
}

sapaNama("Budi");
sapaNama("Sari");
```

Output:

```text
Halo, Budi!
Halo, Sari!
```

Dua istilah yang sering tertukar:

- **Parameter** adalah nama bahan yang ditulis saat **membuat** fungsi (`nama`). Ia berperan seperti variabel di dalam fungsi.
- **Argumen** adalah nilai sebenarnya yang diberikan saat **memanggil** fungsi (`"Budi"`).

Proses eksekusinya: ketika `sapaNama("Budi")` dipanggil, parameter `nama` diisi `"Budi"`, lalu badan fungsi dijalankan. Pada pemanggilan berikutnya `nama` berisi `"Sari"`.

Parameter boleh lebih dari satu, dipisah koma, dan argumen diberikan dengan urutan yang sama.

### Nilai default

Parameter bisa diberi nilai awal yang dipakai jika argumennya tidak diberikan. Simpan sebagai `default.js`:

```javascript
function sapaDefault(nama = "teman") {
  console.log(`Halo, ${nama}!`);
}

sapaDefault("Budi");
sapaDefault();
```

Output:

```text
Halo, Budi!
Halo, teman!
```

Pada pemanggilan kedua tidak ada argumen, sehingga `nama` memakai nilai default `"teman"`.

## Mengembalikan nilai dengan `return`

Fungsi sering dipakai untuk **menghitung sesuatu** dan menyerahkan hasilnya. Itu tugas `return`. Simpan sebagai `return.js`:

```javascript
function tambah(a, b) {
  return a + b;
}

const hasil = tambah(3, 4);
console.log(hasil);
console.log(tambah(10, 20));
console.log(tambah(1, 2) * 10);
```

Output:

```text
7
30
30
```

Penjelasan:

1. `tambah(3, 4)` dipanggil, `a` berisi 3 dan `b` berisi 4.
2. `return a + b` menghitung 7 dan **menyerahkannya** ke tempat fungsi dipanggil.
3. Pada `const hasil = tambah(3, 4)`, nilai 7 disimpan ke `hasil`. Artinya pemanggilan fungsi bisa dipakai seperti nilai biasa.
4. Pada `tambah(1, 2) * 10`, pemanggilan menghasilkan 3, lalu dikalikan 10.

### `return` menghentikan fungsi

Begitu `return` dijalankan, fungsi langsung selesai. Perintah setelahnya tidak dijalankan. Simpan sebagai `return-berhenti.js`:

```javascript
function cekUmur(umur) {
  if (umur >= 17) {
    return "Dewasa";
  }
  return "Anak";
}

console.log(cekUmur(20));
console.log(cekUmur(10));
```

Output:

```text
Dewasa
Anak
```

Untuk `cekUmur(20)`, kondisinya `true`, `return "Dewasa"` dijalankan, dan fungsi selesai sehingga `return "Anak"` tidak tercapai. Pola ini membuat kode lebih ringkas daripada memakai `else`.

### `return` bukan `console.log`

Dua hal ini sering dikira sama, padahal berbeda:

| | `console.log(x)` | `return x` |
|---|---|---|
| Fungsinya | **Menampilkan** x di terminal untuk dibaca manusia | **Menyerahkan** x kepada kode yang memanggil |
| Hasilnya bisa dipakai lagi? | Tidak | Ya |

Simpan sebagai `return-vs-log.js`:

```javascript
function tambahTampil(a, b) {
  console.log(a + b);
}

function tambahKembali(a, b) {
  return a + b;
}

console.log(tambahKembali(2, 3) * 2);
console.log(tambahTampil(2, 3) * 2);
```

Output:

```text
10
5
NaN
```

- `tambahKembali(2, 3) * 2` menghasilkan 5 × 2 = `10`.
- `tambahTampil(2, 3)` hanya mencetak `5` (baris kedua), lalu fungsi selesai **tanpa mengembalikan apa pun**. Hasilnya `undefined`, dan `undefined * 2` menghasilkan `NaN`.

Fungsi yang tidak punya `return` otomatis mengembalikan `undefined`.

## Cara lain menulis fungsi

Selain `function` biasa (disebut *function declaration*), ada dua bentuk lain. Ketiganya membuat fungsi yang bekerja sama.

### Function expression

Fungsi adalah **nilai**, sama seperti angka atau teks, sehingga bisa disimpan di variabel. Simpan sebagai `expression.js`:

```javascript
const kali = function (a, b) {
  return a * b;
};

console.log(kali(6, 7));
```

Output:

```text
42
```

Fungsinya tidak punya nama sendiri. Namanya adalah nama variabel yang menyimpannya (`kali`).

### Arrow function

Bentuk yang lebih ringkas, memakai tanda panah `=>`. Simpan sebagai `arrow.js`:

```javascript
const kuadrat = (x) => {
  return x * x;
};

const kurang = (a, b) => a - b;

const sapaSingkat = () => "Halo!";

console.log(kuadrat(5));
console.log(kurang(10, 4));
console.log(sapaSingkat());
```

Output:

```text
25
6
Halo!
```

| Bentuk | Penjelasan |
|---|---|
| `(x) => { return x * x; }` | Badan dengan `{ }` dan `return` yang ditulis sendiri |
| `(a, b) => a - b` | Jika badannya **satu ekspresi**, `{ }` dan `return` boleh dihilangkan. Hasil ekspresi otomatis dikembalikan |
| `() => "Halo!"` | Tanpa parameter, kurung kosong tetap ditulis |

### Mana yang dipakai?

| Bentuk | Contoh | Kapan |
|---|---|---|
| Declaration | `function tambah(a, b) { ... }` | Fungsi biasa dengan nama jelas |
| Expression | `const kali = function (a, b) { ... }` | Jarang untuk pemula |
| Arrow | `const kali = (a, b) => a * b` | Fungsi pendek dan satu baris |

Di bab ini kita memakai declaration dan arrow function. Perbedaan lebih dalam di antara bentuk-bentuk ini dibahas di bab Scope & Hoisting.

## Fungsi yang memanggil fungsi

Fungsi kecil bisa dirangkai menjadi fungsi yang lebih besar. Simpan sebagai `fungsi-gabung.js`:

```javascript
function hitungDiskonPersen(totalBelanja) {
  if (totalBelanja >= 500000) {
    return 20;
  } else if (totalBelanja >= 250000) {
    return 10;
  }
  return 0;
}

function hitungBayar(totalBelanja, member) {
  let diskon = hitungDiskonPersen(totalBelanja);
  if (member) {
    diskon += 5;
  }
  return totalBelanja - totalBelanja * diskon / 100;
}

console.log(hitungBayar(350000, true));
console.log(hitungBayar(100000, false));
console.log(hitungBayar(600000, true));
```

Output:

```text
297500
100000
450000
```

Ini adalah contoh diskon dari bab 4 yang dipecah menjadi dua fungsi, masing-masing dengan satu tugas. `hitungBayar` memanggil `hitungDiskonPersen`, lalu menambahkan bonus member. Bandingkan hasil pertama dengan bab 4: 297500 sama persis.

Variabel `diskon` hanya ada di dalam `hitungBayar`. Hal ini dijelaskan di bagian kesalahan umum di bawah.

## Contoh praktis

Simpan sebagai `praktis.js`:

```javascript
const celsiusKeFahrenheit = (c) => c * 9 / 5 + 32;

const adalahGenap = (n) => n % 2 === 0;

function faktorial(n) {
  let hasil = 1;
  for (let i = 2; i <= n; i++) {
    hasil *= i;
  }
  return hasil;
}

function hitungVokal(kata) {
  let jumlah = 0;
  for (let i = 0; i < kata.length; i++) {
    const huruf = kata[i];
    if (huruf === "a" || huruf === "i" || huruf === "u" || huruf === "e" || huruf === "o") {
      jumlah++;
    }
  }
  return jumlah;
}

console.log(celsiusKeFahrenheit(100));
console.log(adalahGenap(8), adalahGenap(7));
console.log(faktorial(5));
console.log(hitungVokal("belajar"));
```

Output:

```text
212
true false
120
3
```

Perhatikan bahwa `faktorial` dan `hitungVokal` hanyalah kode dari bab 5 yang dibungkus fungsi. Sekarang keduanya bisa dipanggil kapan saja dengan angka atau kata yang berbeda tanpa menulis ulang perulangannya.

## Kesalahan umum

### 1. Lupa `return`

Simpan sebagai `lupa-return.js`:

```javascript
function luas(p, l) {
  p * l;
}

console.log(luas(4, 5));
```

Output:

```text
undefined
```

`p * l` dihitung tetapi tidak diserahkan ke mana pun. **Perbaikan:** `return p * l;`.

### 2. Memakai `console.log` padahal butuh `return`

Sudah terlihat pada contoh `return-vs-log.js`: hasil dari fungsi yang hanya mencetak tidak bisa dipakai untuk menghitung lagi.

### 3. Lupa tanda kurung saat memanggil

Simpan sebagai `lupa-kurung.js`:

```javascript
function sapa() {
  return "Halo!";
}

console.log(sapa);
console.log(sapa());
```

Output:

```text
[Function: sapa]
Halo!
```

`sapa` tanpa kurung adalah fungsinya sendiri (sebagai nilai), bukan hasil menjalankannya. **Perbaikan:** tulis `sapa()`.

### 4. Jumlah argumen tidak sesuai

Simpan sebagai `argumen.js`:

```javascript
function tambah(a, b) {
  return a + b;
}

console.log(tambah(5));
console.log(tambah(1, 2, 3));
```

Output:

```text
NaN
3
```

- `tambah(5)`: `b` tidak diberi argumen sehingga bernilai `undefined`, dan `5 + undefined` menghasilkan `NaN`.
- `tambah(1, 2, 3)`: argumen ketiga diabaikan.

### 5. Memakai variabel milik fungsi dari luar

Simpan sebagai `variabel-lokal.js`:

```javascript
function hitung() {
  const rahasia = 42;
  return rahasia;
}

console.log(hitung());
console.log(rahasia);
```

Output (dipersingkat):

```text
42
ReferenceError: rahasia is not defined
```

Variabel yang dibuat di dalam fungsi disebut **variabel lokal** dan hanya dikenal di dalam fungsi itu. Parameter juga begitu. **Perbaikan:** kembalikan nilainya dengan `return` lalu simpan di luar. Aturan lengkapnya dibahas di bab Scope & Hoisting.

### 6. Menulis kode setelah `return`

Perintah setelah `return` tidak pernah dijalankan. Jika ada pekerjaan yang harus dilakukan, tulis sebelum `return`.

## Ringkasan

- **Fungsi** adalah blok kode bernama yang bisa dipanggil berulang kali. Menulisnya tidak menjalankannya; memanggil dengan `nama()` yang menjalankannya.
- **Parameter** adalah nama bahan saat membuat fungsi. **Argumen** adalah nilai yang diberikan saat memanggil.
- Parameter bisa punya nilai default: `function f(x = 1)`.
- `return` menyerahkan hasil dan menghentikan fungsi. Tanpa `return`, hasilnya `undefined`.
- `console.log` hanya menampilkan, `return` mengembalikan nilai untuk dipakai lagi.
- Tiga bentuk: `function` declaration, function expression, dan arrow function `(a, b) => a + b`.
- Satu fungsi sebaiknya mengerjakan satu tugas, dan fungsi boleh memanggil fungsi lain.
- Variabel di dalam fungsi bersifat lokal.

## Latihan

Kerjakan sendiri dulu sebelum melihat jawaban.

### Latihan 1: Prediksi output

Apa yang tampil?

```javascript
function f(x) {
  return x * 2;
}

console.log(f(f(3)));
```

??? success "Jawaban"
    Output: `12`. Bagian dalam `f(3)` dihitung lebih dulu menjadi 6, lalu `f(6)` menjadi 12.

### Latihan 2: Perbaiki fungsi

Kode ini mencetak `undefined`, padahal seharusnya `20`. Perbaiki:

```javascript
function luas(p, l) {
  p * l;
}

console.log(luas(4, 5));
```

??? success "Jawaban"
    ```javascript
    function luas(p, l) {
      return p * l;
    }

    console.log(luas(4, 5));
    ```
    Output: `20`

### Latihan 3: Genap atau bukan

Buat fungsi `adalahGenap(n)` yang mengembalikan `true` jika `n` genap. Uji dengan 10 dan 7.

??? success "Contoh jawaban"
    ```javascript
    function adalahGenap(n) {
      return n % 2 === 0;
    }

    console.log(adalahGenap(10));
    console.log(adalahGenap(7));
    ```
    Output: `true` lalu `false`.

### Latihan 4: Konversi suhu

Buat fungsi `celsiusKeFahrenheit(c)`. Uji dengan 100 dan 0.

??? success "Contoh jawaban"
    ```javascript
    const celsiusKeFahrenheit = (c) => c * 9 / 5 + 32;

    console.log(celsiusKeFahrenheit(100));
    console.log(celsiusKeFahrenheit(0));
    ```
    Output: `212` lalu `32`.

### Latihan 5: Faktorial

Buat fungsi `faktorial(n)` dengan perulangan. Uji dengan 5 dan 0. Tebak hasil untuk 0 sebelum menjalankannya.

??? success "Contoh jawaban"
    ```javascript
    function faktorial(n) {
      let hasil = 1;
      for (let i = 2; i <= n; i++) {
        hasil *= i;
      }
      return hasil;
    }

    console.log(faktorial(5));
    console.log(faktorial(0));
    ```
    Output: `120` lalu `1`. Untuk `n = 0`, perulangan tidak berjalan sama sekali sehingga `hasil` tetap 1, yang memang nilai 0! secara matematika.

### Latihan 6: Hitung vokal

Buat fungsi `hitungVokal(kata)` yang mengembalikan jumlah huruf vokal. Uji dengan `"pemrograman"`.

??? success "Contoh jawaban"
    ```javascript
    function hitungVokal(kata) {
      let jumlah = 0;
      for (let i = 0; i < kata.length; i++) {
        const huruf = kata[i];
        if (huruf === "a" || huruf === "i" || huruf === "u" || huruf === "e" || huruf === "o") {
          jumlah++;
        }
      }
      return jumlah;
    }

    console.log(hitungVokal("pemrograman"));
    ```
    Output: `4` (huruf e, o, a, a).

### Latihan 7: Rata-rata dan predikat

Buat arrow function `rataRata(a, b)`, dan fungsi `predikat(nilai)` yang mengembalikan `"A"` untuk nilai 90 ke atas, `"B"` untuk 80 ke atas, `"C"` untuk 70 ke atas, selain itu `"D"`. Tampilkan predikat dari rata-rata nilai 78 dan 92.

??? success "Contoh jawaban"
    ```javascript
    const rataRata = (a, b) => (a + b) / 2;

    function predikat(nilai) {
      if (nilai >= 90) {
        return "A";
      } else if (nilai >= 80) {
        return "B";
      } else if (nilai >= 70) {
        return "C";
      }
      return "D";
    }

    console.log(predikat(rataRata(78, 92)));
    ```
    Output: `B`. Rata-ratanya 85, dan `predikat(85)` mengembalikan `"B"`. Perhatikan bahwa hasil satu fungsi langsung dipakai sebagai argumen fungsi lain.

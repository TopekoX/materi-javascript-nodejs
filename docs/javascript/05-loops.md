## Tujuan pembelajaran

Setelah mempelajari bab ini, kamu dapat:

- Menjelaskan apa itu perulangan dan kapan perlu memakainya.
- Menulis perulangan `for`, `while`, dan `do...while`.
- Menjelaskan tiga bagian dari `for` dan urutan pengerjaannya.
- Menghentikan perulangan dengan `break` dan melewati satu putaran dengan `continue`.
- Membuat perulangan di dalam perulangan.
- Menghindari perulangan tanpa akhir dan kesalahan selisih satu (*off-by-one*).

Bab ini memakai materi bab 1 sampai 4, terutama operator `++`, `+=`, dan `if`. Jalankan setiap contoh dengan `node nama-file.js`.

## Pendahuluan: mengapa perlu perulangan?

Bayangkan kamu diminta mencetak angka 1 sampai 5. Dengan cara yang sudah kita kenal:

```javascript
console.log(1);
console.log(2);
console.log(3);
console.log(4);
console.log(5);
```

Untuk 5 angka masih mudah. Bagaimana kalau 1000 angka? Menulis 1000 baris jelas tidak masuk akal. Dalam hidup sehari-hari kita berkata: *"Ulangi sampai selesai"*, misalnya mengaduk adonan sampai rata atau menghitung langkah sampai sepuluh.

**Perulangan** (*loop*) adalah struktur yang menjalankan blok kode berulang kali selama suatu syarat masih benar. Satu kali pengulangan disebut satu **iterasi**.

## Perulangan `for`

Gunakan `for` ketika kamu tahu **berapa kali** pengulangan akan dilakukan.

### Sintaks

```text
for (inisialisasi; kondisi; perubahan) {
  // dijalankan di setiap iterasi
}
```

Simpan sebagai `for-dasar.js`:

```javascript
for (let i = 1; i <= 5; i++) {
  console.log(`Putaran ke-${i}`);
}
```

Output:

```text
Putaran ke-1
Putaran ke-2
Putaran ke-3
Putaran ke-4
Putaran ke-5
```

Tanda kurung `for` punya tiga bagian yang dipisah `;`:

| Bagian | Pada contoh | Artinya |
|---|---|---|
| Inisialisasi | `let i = 1` | Dijalankan **sekali** di awal. Membuat variabel penghitung |
| Kondisi | `i <= 5` | Diperiksa **sebelum** setiap iterasi. Jika `false`, perulangan berhenti |
| Perubahan | `i++` | Dijalankan **setelah** setiap iterasi |

Variabel penghitung biasanya diberi nama `i` (singkatan *index* atau *iterator*). Karena nilainya akan berubah, ia dibuat dengan `let`.

### Proses eksekusi langkah demi langkah

1. `let i = 1` dijalankan sekali.
2. Periksa `i <= 5` (1 <= 5, `true`), jalankan isi blok, cetak `Putaran ke-1`.
3. Jalankan `i++`, sehingga `i` menjadi 2.
4. Periksa `i <= 5` lagi (`true`), jalankan blok, dan seterusnya.
5. Setelah `i` menjadi 6, kondisi `6 <= 5` bernilai `false`, perulangan berhenti, dan program lanjut ke baris setelah blok.

### Menghitung mundur dan melompat

Bagian perubahan boleh apa saja, tidak harus `i++`. Simpan sebagai `for-variasi.js`:

```javascript
for (let i = 3; i >= 1; i--) {
  console.log(i);
}
console.log("Mulai!");

for (let i = 0; i <= 20; i += 5) {
  console.log(i);
}
```

Output:

```text
3
2
1
Mulai!
0
5
10
15
20
```

- `i--` mengurangi 1 setiap putaran, dan kondisinya diganti menjadi `i >= 1`.
- `i += 5` melompat lima angka sekali putaran.

### Mengakumulasi hasil

Pola yang sangat sering dipakai: siapkan variabel di luar perulangan, lalu tambahkan ke dalamnya di setiap iterasi. Simpan sebagai `for-jumlah.js`:

```javascript
let total = 0;

for (let i = 1; i <= 5; i++) {
  total += i;
}

console.log(`Total: ${total}`);
```

Output:

```text
Total: 15
```

Jejak nilai di setiap iterasi:

| Iterasi | `i` | `total` sebelum | `total` sesudah |
|---|---|---|---|
| 1 | 1 | 0 | 1 |
| 2 | 2 | 1 | 3 |
| 3 | 3 | 3 | 6 |
| 4 | 4 | 6 | 10 |
| 5 | 5 | 10 | 15 |

Variabel `total` dibuat **di luar** perulangan agar masih ada setelah perulangan selesai. Variabel `i` hanya dikenal di dalam `for`.

## Perulangan `while`

Gunakan `while` ketika kamu **tidak tahu** berapa kali pengulangan akan terjadi, tetapi tahu syarat untuk berhenti.

```text
while (kondisi) {
  // dijalankan selama kondisi true
}
```

Contoh: berapa bulan sampai tabungan 100.000 dengan bunga 10% per bulan mencapai 150.000? Simpan sebagai `while-tabungan.js`:

```javascript
let saldo = 100000;
let bulan = 0;

while (saldo < 150000) {
  saldo += saldo * 10 / 100;
  bulan++;
  console.log(`Bulan ${bulan}: ${saldo}`);
}

console.log(`Mencapai target setelah ${bulan} bulan.`);
```

Output:

```text
Bulan 1: 110000
Bulan 2: 121000
Bulan 3: 133100
Bulan 4: 146410
Bulan 5: 161051
Mencapai target setelah 5 bulan.
```

Proses eksekusinya:

1. Sebelum setiap putaran, `saldo < 150000` diperiksa.
2. Di dalam blok, saldo ditambah bunga 10% dan `bulan` bertambah satu.
3. Setelah bulan ke-5, saldo 161051 sudah tidak kurang dari 150000, kondisi menjadi `false`, dan perulangan berhenti.

Kita tidak bisa menebak jumlah putaran sebelumnya, jadi `while` lebih cocok daripada `for`.

!!! warning "Pastikan kondisinya bisa menjadi false"
    Di dalam blok `while` harus ada sesuatu yang akhirnya membuat kondisi menjadi `false` (di sini `saldo` terus bertambah). Jika tidak, perulangan berjalan selamanya.

## Perulangan `do...while`

`do...while` mirip `while`, tetapi kondisinya diperiksa **setelah** blok dijalankan. Akibatnya blok **minimal berjalan satu kali**. Simpan sebagai `do-while.js`:

```javascript
let angka = 10;

do {
  console.log(`do...while: ${angka}`);
  angka++;
} while (angka < 5);

let angka2 = 10;

while (angka2 < 5) {
  console.log(`while: ${angka2}`);
  angka2++;
}

console.log("Selesai.");
```

Output:

```text
do...while: 10
Selesai.
```

Kondisi `10 < 5` sudah `false` sejak awal. `do...while` tetap menjalankan blok satu kali, sedangkan `while` tidak menjalankan blok sama sekali. Perhatikan titik koma setelah `while (...)` pada `do...while`.

`do...while` jarang dipakai. Gunakan hanya jika sesuatu memang harus dilakukan setidaknya sekali.

## Memilih jenis perulangan

| Situasi | Pilihan |
|---|---|
| Jumlah putaran diketahui (1 sampai 10, mundur dari 5) | `for` |
| Berulang sampai syarat tertentu terpenuhi | `while` |
| Harus berjalan minimal sekali | `do...while` |

## `break` dan `continue`

Dua kata kunci ini mengubah jalannya perulangan dari dalam blok.

- **`break`** menghentikan seluruh perulangan.
- **`continue`** melewati sisa blok pada putaran saat ini, lalu lanjut ke putaran berikutnya.

Simpan sebagai `break-continue.js`:

```javascript
for (let i = 1; i <= 100; i++) {
  if (i % 7 === 0 && i % 5 === 0) {
    console.log(`Angka pertama yang habis dibagi 7 dan 5: ${i}`);
    break;
  }
}

for (let i = 1; i <= 9; i++) {
  if (i % 2 === 0) {
    continue;
  }
  console.log(i);
}
```

Output:

```text
Angka pertama yang habis dibagi 7 dan 5: 35
1
3
5
7
9
```

- Perulangan pertama sebenarnya diatur sampai 100, tetapi `break` menghentikannya begitu menemukan 35.
- Perulangan kedua melewati angka genap dengan `continue`, sehingga hanya angka ganjil yang dicetak.

## Perulangan di dalam perulangan

Perulangan boleh ditaruh di dalam perulangan lain. Perulangan **dalam** menyelesaikan semua putarannya untuk setiap satu putaran perulangan **luar**. Analoginya jarum jam: jarum menit berputar penuh (60 kali) untuk setiap satu langkah jarum jam.

Simpan sebagai `bersarang.js`:

```javascript
for (let i = 1; i <= 3; i++) {
  let baris = "";
  for (let j = 1; j <= 3; j++) {
    baris = baris + i * j + " ";
  }
  console.log(baris.trim());
}

for (let i = 1; i <= 4; i++) {
  console.log("*".repeat(i));
}
```

Output:

```text
1 2 3
2 4 6
3 6 9
*
**
***
****
```

Penjelasan:

1. Untuk `i = 1`, perulangan dalam berjalan dengan `j` dari 1 sampai 3 dan membentuk teks `"1 2 3 "`. `trim()` membuang spasi di ujungnya (dari bab 3).
2. Lalu `i` menjadi 2, perulangan dalam dimulai lagi dari awal, dan seterusnya.
3. Perulangan kedua mencetak pola segitiga: `repeat(i)` mengulang tanda `*` sebanyak `i` kali.

## Perulangan pada string

String punya `length` dan indeks (bab 3), sehingga kita bisa memeriksanya karakter demi karakter. Simpan sebagai `loop-string.js`:

```javascript
const kata = "belajar";
let jumlahVokal = 0;
let terbalik = "";

for (let i = 0; i < kata.length; i++) {
  const huruf = kata[i];
  if (huruf === "a" || huruf === "i" || huruf === "u" || huruf === "e" || huruf === "o") {
    jumlahVokal++;
  }
}

for (let i = kata.length - 1; i >= 0; i--) {
  terbalik += kata[i];
}

console.log(`Jumlah vokal: ${jumlahVokal}`);
console.log(`Kata terbalik: ${terbalik}`);
```

Output:

```text
Jumlah vokal: 3
Kata terbalik: rajaleb
```

Penjelasan:

- Indeks dimulai dari 0, sehingga kondisinya `i < kata.length` (bukan `<=`). Indeks terakhir adalah `kata.length - 1`.
- Perulangan pertama memeriksa setiap huruf. Hurufnya `b, e, l, a, j, a, r`, dan vokalnya `e`, `a`, `a`, sehingga hasilnya 3.
- Perulangan kedua berjalan **mundur** dari indeks terakhir ke 0 dan menyambung huruf demi huruf.

## Contoh praktis: faktorial

Faktorial dari 5 (ditulis 5!) adalah 5 × 4 × 3 × 2 × 1. Simpan sebagai `faktorial.js`:

```javascript
const n = 5;
let hasil = 1;

for (let i = 2; i <= n; i++) {
  hasil *= i;
}

console.log(`${n}! = ${hasil}`);
```

Output:

```text
5! = 120
```

Perhatikan dua keputusan penting:

- `hasil` dimulai dari **1**, bukan 0. Jika dimulai dari 0, semua perkalian akan menghasilkan 0.
- Perulangan dimulai dari `i = 2`, karena perkalian dengan 1 tidak mengubah hasil.

Jejaknya: `hasil` menjadi 2, lalu 6, lalu 24, lalu 120.

## Kesalahan umum

### 1. Perulangan tanpa akhir (*infinite loop*)

Simpan sebagai `tanpa-akhir.js`, tetapi **jangan dijalankan sampai selesai**:

```javascript
let n = 5;

while (n > 0) {
  console.log(n);
}
```

Nilai `n` tidak pernah berubah, jadi `n > 0` selalu `true` dan terminal terus mencetak `5` tanpa henti. Jika ini terjadi, tekan `Ctrl + C` di terminal untuk menghentikan program. **Perbaikan:** tambahkan `n--;` di dalam blok.

### 2. Selisih satu (*off-by-one*)

Simpan sebagai `selisih-satu.js`:

```javascript
const kata = "abc";

for (let i = 0; i <= kata.length; i++) {
  console.log(kata[i]);
}
```

Output:

```text
a
b
c
undefined
```

Panjang `"abc"` adalah 3, tetapi indeksnya hanya 0, 1, 2. Dengan `<=`, perulangan ikut memproses indeks 3 yang tidak ada. **Perbaikan:** gunakan `i < kata.length`.

### 3. Penghitung dibuat dengan `const`

Simpan sebagai `penghitung-const.js`:

```javascript
for (const i = 0; i < 3; i++) {
  console.log(i);
}
```

Output (dipersingkat):

```text
0
TypeError: Assignment to constant variable.
```

`i++` mengubah `i`, sedangkan `const` tidak boleh diubah. **Perbaikan:** gunakan `let`.

### 4. Mengubah penghitung di dalam blok

Mengubah `i` secara manual di dalam blok `for` (misalnya `i += 2`) membuat jumlah putaran sulit ditebak. Biarkan bagian perubahan di tanda kurung `for` yang mengaturnya.

### 5. Menaruh variabel akumulator di dalam perulangan

Jika `let total = 0;` ditulis **di dalam** blok, nilainya di-reset menjadi 0 pada setiap putaran, dan variabelnya tidak bisa dipakai setelah perulangan selesai. Buat variabel akumulator **sebelum** perulangan.

## Ringkasan

- **Perulangan** menjalankan blok berulang kali. Satu pengulangan disebut **iterasi**.
- `for (awal; kondisi; perubahan)` cocok ketika jumlah putaran diketahui.
- `while (kondisi)` cocok ketika berulang sampai syarat terpenuhi. Pastikan kondisinya akhirnya menjadi `false`.
- `do...while` menjalankan blok minimal satu kali.
- `break` menghentikan perulangan, `continue` melewati satu putaran.
- Perulangan bisa bersarang. Perulangan dalam selesai penuh untuk setiap putaran perulangan luar.
- Untuk string, gunakan `i < kata.length`, karena indeks terakhir adalah `length - 1`.
- Buat penghitung dengan `let`, dan buat akumulator sebelum perulangan.
- Hentikan program yang macet dengan `Ctrl + C`.

## Latihan

Kerjakan sendiri dulu sebelum melihat jawaban.

### Latihan 1: Prediksi output

Apa yang tampil?

```javascript
for (let i = 1; i <= 10; i += 3) {
  console.log(i);
}
```

??? success "Jawaban"
    Output: `1`, `4`, `7`, `10` (masing-masing di baris sendiri). Nilai `i` bertambah 3 setiap putaran, dan 10 masih memenuhi `i <= 10`.

### Latihan 2: Perbaiki perulangan tanpa akhir

Hitung mundur ini tidak pernah berhenti. Perbaiki:

```javascript
let n = 5;

while (n > 0) {
  console.log(n);
}
```

??? success "Jawaban"
    Tambahkan `n--;` di dalam blok:
    ```javascript
    let n = 5;

    while (n > 0) {
      console.log(n);
      n--;
    }
    ```
    Output: `5`, `4`, `3`, `2`, `1` (masing-masing di baris sendiri).

### Latihan 3: Jumlah bilangan genap

Hitung jumlah semua bilangan genap dari 1 sampai 10, lalu tampilkan hasilnya.

??? success "Contoh jawaban"
    ```javascript
    let total = 0;

    for (let i = 1; i <= 10; i++) {
      if (i % 2 === 0) {
        total += i;
      }
    }

    console.log(`Total: ${total}`);
    ```
    Output: `Total: 30` (2 + 4 + 6 + 8 + 10).

### Latihan 4: Tabel perkalian

Tampilkan tabel perkalian 7 dari 7 × 1 sampai 7 × 5 dengan format `7 x 1 = 7`.

??? success "Contoh jawaban"
    ```javascript
    for (let i = 1; i <= 5; i++) {
      console.log(`7 x ${i} = ${7 * i}`);
    }
    ```
    Output:
    ```text
    7 x 1 = 7
    7 x 2 = 14
    7 x 3 = 21
    7 x 4 = 28
    7 x 5 = 35
    ```

### Latihan 5: Faktorial 6

Ubah contoh faktorial untuk menghitung 6!. Tebak hasilnya dulu.

??? success "Jawaban"
    Ganti `const n = 5;` menjadi `const n = 6;`. Hasilnya `6! = 720` (6 × 5 × 4 × 3 × 2 × 1).

### Latihan 6: Hitung huruf

Hitung berapa kali huruf `a` muncul di `"banana"`, lalu tampilkan juga kata itu terbalik.

??? success "Contoh jawaban"
    ```javascript
    const kata = "banana";
    let jumlahA = 0;
    let terbalik = "";

    for (let i = 0; i < kata.length; i++) {
      if (kata[i] === "a") {
        jumlahA++;
      }
    }

    for (let i = kata.length - 1; i >= 0; i--) {
      terbalik += kata[i];
    }

    console.log(jumlahA);
    console.log(terbalik);
    ```
    Output:
    ```text
    3
    ananab
    ```

### Latihan 7: FizzBuzz

Untuk angka 1 sampai 15: cetak `Fizz` jika habis dibagi 3, `Buzz` jika habis dibagi 5, `FizzBuzz` jika habis dibagi keduanya, dan jika tidak, cetak angkanya. Petunjuk: periksa syarat "keduanya" lebih dulu.

??? success "Contoh jawaban"
    ```javascript
    for (let i = 1; i <= 15; i++) {
      if (i % 15 === 0) {
        console.log("FizzBuzz");
      } else if (i % 3 === 0) {
        console.log("Fizz");
      } else if (i % 5 === 0) {
        console.log("Buzz");
      } else {
        console.log(i);
      }
    }
    ```
    Output (angka 1 sampai 15): `1`, `2`, `Fizz`, `4`, `Buzz`, `Fizz`, `7`, `8`, `Fizz`, `Buzz`, `11`, `Fizz`, `13`, `14`, `FizzBuzz`. Syarat `i % 15` harus lebih dulu, karena angka yang habis dibagi 15 juga habis dibagi 3 dan 5.

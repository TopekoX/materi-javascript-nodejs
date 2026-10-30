## Tujuan pembelajaran

Setelah mempelajari bab ini, kamu dapat:

- Menjelaskan apa itu array dan kapan memakainya.
- Membuat array, mengakses, dan mengubah elemennya.
- Menambah dan menghapus elemen dengan `push`, `pop`, `shift`, dan `unshift`.
- Mencari, memotong, menggabung, dan mengurutkan array.
- Memproses array dengan `for`, `for...of`, `forEach`, `map`, `filter`, `find`, dan `reduce`.
- Membedakan perintah yang mengubah array asli dari yang menghasilkan array baru.
- Memahami mengapa `const` pada array tidak membuat isinya terkunci.

Bab ini memakai materi bab 1 sampai 7, terutama perulangan dan fungsi (termasuk arrow function). Jalankan setiap contoh dengan `node nama-file.js`.

## Pendahuluan: satu variabel untuk banyak nilai

Sampai sekarang, satu variabel menyimpan satu nilai. Bayangkan kamu perlu menyimpan nilai ujian lima siswa:

```javascript
const nilai1 = 78;
const nilai2 = 85;
const nilai3 = 92;
const nilai4 = 66;
const nilai5 = 88;
```

Cara ini melelahkan, dan mustahil untuk 500 siswa. Kita butuh **satu wadah untuk banyak nilai**, seperti **rak berlaci bernomor**: satu rak, banyak laci, setiap laci punya nomor.

**Array** adalah daftar nilai yang berurutan dan disimpan dalam satu variabel. Setiap nilai di dalamnya disebut **elemen**, dan setiap elemen punya nomor posisi yang disebut **indeks**, dimulai dari 0 (sama seperti string di bab 3).

## Membuat dan mengakses array

Array ditulis dengan tanda kurung siku `[ ]`, elemennya dipisah koma. Simpan sebagai `array-dasar.js`:

```javascript
const buah = ["apel", "jeruk", "mangga"];

console.log(buah);
console.log(buah.length);
console.log(buah[0]);
console.log(buah[buah.length - 1]);
console.log(buah.at(-1));
console.log(buah[10]);

buah[1] = "pisang";
console.log(buah);
console.log(typeof buah, Array.isArray(buah));
```

Output:

```text
[ 'apel', 'jeruk', 'mangga' ]
3
apel
mangga
mangga
undefined
[ 'apel', 'pisang', 'mangga' ]
object true
```

Penjelasan:

| Kode | Artinya |
|---|---|
| `["apel", "jeruk", "mangga"]` | Array berisi tiga string |
| `buah.length` | Jumlah elemen (3). Properti yang sama dengan `length` pada string |
| `buah[0]` | Elemen pada indeks 0 (pertama) |
| `buah[buah.length - 1]` | Elemen terakhir. Indeks terakhir selalu `length - 1` |
| `buah.at(-1)` | Cara singkat mengambil elemen terakhir. Angka negatif menghitung dari belakang |
| `buah[10]` | Indeks yang tidak ada menghasilkan `undefined`, bukan error |
| `buah[1] = "pisang"` | Mengganti isi elemen pada indeks 1 |
| `Array.isArray(buah)` | Memeriksa apakah sebuah nilai adalah array. `typeof` hanya menjawab `object` |

Pada tampilan `console.log`, Node.js menampilkan array dengan tanda `[ ... ]` dan string berkutip tunggal. Array boleh kosong (`[]`) dan boleh berisi tipe campuran (`[1, "dua", true]`), tetapi biasanya isinya satu jenis saja.

## Menambah dan menghapus elemen

Ada empat perintah untuk ujung depan dan belakang array. Simpan sebagai `tambah-hapus.js`:

```javascript
const antrian = ["Budi", "Sari"];

antrian.push("Andi");
console.log(antrian);

const keluar = antrian.shift();
console.log(keluar, antrian);

antrian.unshift("Rina");
console.log(antrian);

const terakhir = antrian.pop();
console.log(terakhir, antrian);

console.log(antrian.push("Dewi"));
```

Output:

```text
[ 'Budi', 'Sari', 'Andi' ]
Budi [ 'Sari', 'Andi' ]
[ 'Rina', 'Sari', 'Andi' ]
Andi [ 'Rina', 'Sari' ]
3
```

| Perintah | Fungsi | Mengembalikan |
|---|---|---|
| `push(x)` | Tambah `x` di **belakang** | Panjang array yang baru |
| `pop()` | Hapus elemen **terakhir** | Elemen yang dihapus |
| `unshift(x)` | Tambah `x` di **depan** | Panjang array yang baru |
| `shift()` | Hapus elemen **pertama** | Elemen yang dihapus |

Analogi antrian: `push` dan `shift` seperti orang yang bergabung di belakang lalu dilayani dari depan. Perhatikan bahwa semua perintah ini **mengubah array aslinya**.

## `const` dan array

Array yang dibuat dengan `const` tetap bisa diubah isinya. Simpan sebagai `const-array.js`:

```javascript
const angka = [1, 2];
angka.push(3);
console.log(angka);

const salinan = angka;
salinan.push(4);
console.log(angka);

const copy = angka.slice();
copy.push(5);
console.log(angka, copy);

console.log([1] === [1]);
```

Output:

```text
[ 1, 2, 3 ]
[ 1, 2, 3, 4 ]
[ 1, 2, 3, 4 ] [ 1, 2, 3, 4, 5 ]
false
```

Ada tiga hal penting di sini:

1. **`const` hanya mengunci nama variabelnya**, bukan isinya. `angka.push(3)` boleh karena kita tidak mengganti variabel `angka`, hanya mengubah isi di dalamnya. Yang tidak boleh adalah mengisi ulang variabelnya:

    ```javascript
    const angka = [1];
    angka = [2];
    ```

    ```text
    TypeError: Assignment to constant variable.
    ```

2. **Array disimpan sebagai referensi.** `const salinan = angka` tidak membuat array baru, melainkan membuat nama kedua untuk array yang sama (seperti dua kunci untuk satu rumah). Karena itu `salinan.push(4)` ikut mengubah `angka`.
3. **Untuk membuat salinan sungguhan**, pakai `slice()` tanpa argumen. Ia menghasilkan array baru, jadi `copy.push(5)` tidak memengaruhi `angka`.

Karena array adalah referensi, `[1] === [1]` bernilai `false`: keduanya dua array berbeda yang kebetulan berisi sama.

## Mencari elemen

Simpan sebagai `cari.js`:

```javascript
const kota = ["Jakarta", "Bandung", "Medan"];

console.log(kota.includes("Medan"));
console.log(kota.includes("medan"));
console.log(kota.indexOf("Bandung"));
console.log(kota.indexOf("Surabaya"));
```

Output:

```text
true
false
1
-1
```

- `includes(x)` menjawab `true` atau `false`: apakah `x` ada di array. Pencocokannya persis, termasuk huruf besar-kecil.
- `indexOf(x)` memberi indeks tempat `x` pertama kali ditemukan, atau `-1` jika tidak ada. Sama seperti pada string di bab 3.

## Memotong, menggabung, dan mengubah bagian array

Simpan sebagai `potong.js`:

```javascript
const huruf = ["a", "b", "c", "d", "e"];

console.log(huruf.slice(1, 3));
console.log(huruf.slice(-2));
console.log(huruf.join("-"));
console.log(huruf.concat(["f"]));
console.log(huruf);

const hapus = ["a", "b", "c", "d"];
hapus.splice(1, 2);
console.log(hapus);
hapus.splice(1, 0, "x");
console.log(hapus);
```

Output:

```text
[ 'b', 'c' ]
[ 'd', 'e' ]
a-b-c-d-e
[ 'a', 'b', 'c', 'd', 'e', 'f' ]
[ 'a', 'b', 'c', 'd', 'e' ]
[ 'a', 'd' ]
[ 'a', 'x', 'd' ]
```

| Perintah | Fungsi | Array asli |
|---|---|---|
| `slice(awal, akhir)` | Ambil potongan dari indeks `awal` sampai **sebelum** `akhir` | Tidak berubah |
| `join(pemisah)` | Gabungkan semua elemen menjadi satu string | Tidak berubah |
| `concat(array2)` | Gabungkan dua array menjadi array baru | Tidak berubah |
| `splice(awal, jumlahHapus, ...baru)` | Hapus `jumlahHapus` elemen mulai `awal`, lalu (opsional) sisipkan elemen baru di situ | **Berubah** |

Pada `splice(1, 2)`, dua elemen mulai dari indeks 1 (`"b"` dan `"c"`) dihapus. Pada `splice(1, 0, "x")`, tidak ada yang dihapus dan `"x"` disisipkan di indeks 1.

## Memproses semua elemen

### Dengan `for` dan `for...of`

Karena array punya `length` dan indeks, perulangan `for` dari bab 5 langsung bisa dipakai. Ada juga `for...of` yang lebih ringkas: ia langsung memberikan **elemennya**, tanpa indeks. Simpan sebagai `perulangan.js`:

```javascript
const nilai = [80, 65, 90];

for (let i = 0; i < nilai.length; i++) {
  console.log(`Indeks ${i}: ${nilai[i]}`);
}

for (const n of nilai) {
  console.log(n);
}
```

Output:

```text
Indeks 0: 80
Indeks 1: 65
Indeks 2: 90
80
65
90
```

Pada `for (const n of nilai)`, di setiap putaran `n` berisi satu elemen. Di sini `const` boleh dipakai, karena `n` dibuat baru di setiap putaran, bukan diubah. Gunakan `for...of` jika kamu tidak butuh indeksnya.

### Dengan perintah yang menerima fungsi

Array punya beberapa perintah bawaan yang menerima **sebuah fungsi sebagai argumen** dan menjalankannya untuk setiap elemen. Ingat dari bab 6 bahwa fungsi adalah nilai, sehingga boleh diberikan seperti nilai lain. Fungsi yang diberikan seperti ini disebut **callback**, yaitu fungsi yang "dipanggil balik" oleh perintah tersebut. Biasanya ditulis sebagai arrow function.

Perintah paling sederhana adalah `forEach`, yang menjalankan fungsi untuk setiap elemen. Simpan sebagai `foreach.js`:

```javascript
const nilai = [80, 65, 90];

nilai.forEach((n, i) => {
  console.log(`${i + 1}. ${n}`);
});
```

Output:

```text
1. 80
2. 65
3. 90
```

Callback menerima elemen (`n`) dan, jika ditulis, indeksnya (`i`). `forEach` tidak mengembalikan apa-apa; ia hanya menjalankan fungsi.

## `map`, `filter`, `find`, dan `reduce`

Perintah-perintah ini sangat sering dipakai. Semuanya **tidak mengubah array asli**. Simpan sebagai `map-filter.js`:

```javascript
const angka = [1, 2, 3, 4, 5, 6];

const dobel = angka.map((x) => x * 2);
const genap = angka.filter((x) => x % 2 === 0);
const pertama = angka.find((x) => x > 3);
const adaBesar = angka.some((x) => x > 5);
const semuaPositif = angka.every((x) => x > 0);
const total = angka.reduce((jumlah, x) => jumlah + x, 0);

console.log(dobel);
console.log(genap);
console.log(pertama);
console.log(adaBesar);
console.log(semuaPositif);
console.log(total);
console.log(angka);
```

Output:

```text
[ 2, 4, 6, 8, 10, 12 ]
[ 2, 4, 6 ]
4
true
true
21
[ 1, 2, 3, 4, 5, 6 ]
```

| Perintah | Fungsi | Hasil |
|---|---|---|
| `map(f)` | Ubah **setiap** elemen dengan `f` | Array baru dengan panjang yang sama |
| `filter(f)` | Simpan elemen yang `f`-nya bernilai `true` | Array baru yang lebih pendek atau sama |
| `find(f)` | Cari elemen **pertama** yang `f`-nya `true` | Elemennya, atau `undefined` |
| `some(f)` | Apakah **ada** elemen yang `f`-nya `true`? | `true` atau `false` |
| `every(f)` | Apakah **semua** elemen `f`-nya `true`? | `true` atau `false` |
| `reduce(f, awal)` | Gabungkan semua elemen menjadi **satu nilai** | Satu nilai |

Baris terakhir menunjukkan bahwa `angka` tetap utuh.

### Cara kerja `reduce`

`reduce` paling sulit dipahami, jadi mari lihat jejaknya untuk `angka.reduce((jumlah, x) => jumlah + x, 0)`:

| Putaran | `jumlah` (penampung) | `x` (elemen) | Hasil `jumlah + x` |
|---|---|---|---|
| 1 | 0 (nilai awal) | 1 | 1 |
| 2 | 1 | 2 | 3 |
| 3 | 3 | 3 | 6 |
| 4 | 6 | 4 | 10 |
| 5 | 10 | 5 | 15 |
| 6 | 15 | 6 | 21 |

Hasil setiap putaran menjadi `jumlah` untuk putaran berikutnya. Setelah elemen terakhir, nilai akhirnya (21) dikembalikan. Argumen kedua, `0`, adalah nilai awal penampung.

## Mengurutkan dan membalik

Simpan sebagai `urut.js`:

```javascript
const a = [10, 9, 1, 100];

console.log(a.slice().sort());
console.log(a.slice().sort((x, y) => x - y));
console.log(a.slice().sort((x, y) => y - x));
console.log(a.slice().reverse());

const nama = ["Sari", "Andi", "budi"];
console.log(nama.sort());
```

Output:

```text
[ 1, 10, 100, 9 ]
[ 1, 9, 10, 100 ]
[ 100, 10, 9, 1 ]
[ 100, 1, 9, 10 ]
[ 'Andi', 'Sari', 'budi' ]
```

Hal-hal yang perlu diperhatikan:

- **`sort()` tanpa argumen mengurutkan sebagai teks**, bukan angka. Karena itu `10` dianggap lebih kecil dari `9` (seperti `"10"` datang sebelum `"9"` dalam kamus). Hasilnya `[ 1, 10, 100, 9 ]`.
- **Untuk angka, berikan fungsi pembanding.** `(x, y) => x - y` mengurutkan naik, dan `(x, y) => y - x` mengurutkan turun. Fungsi itu mengembalikan angka negatif jika `x` harus di depan `y`, positif jika di belakang, dan 0 jika sama.
- Huruf besar diurutkan lebih dulu daripada huruf kecil, sehingga `"budi"` berada setelah `"Sari"`.
- **`sort()` dan `reverse()` mengubah array aslinya.** Itulah sebabnya contoh di atas memakai `a.slice()` terlebih dulu untuk menyalin. Baris terakhir memanggil `nama.sort()` langsung, sehingga `nama` ikut berubah.

## Contoh praktis

### Statistik nilai

Simpan sebagai `praktis-nilai.js`:

```javascript
const nilai = [78, 85, 92, 66, 88];

const total = nilai.reduce((a, b) => a + b, 0);
const rata = total / nilai.length;
const tertinggi = nilai.reduce((a, b) => (a > b ? a : b));
const terendah = nilai.reduce((a, b) => (a < b ? a : b));
const lulus = nilai.filter((n) => n >= 70);

console.log(`Rata-rata: ${rata}`);
console.log(`Tertinggi: ${tertinggi}`);
console.log(`Terendah: ${terendah}`);
console.log(`Yang lulus: ${lulus.join(", ")}`);
console.log(`Jumlah lulus: ${lulus.length} dari ${nilai.length}`);
```

Output:

```text
Rata-rata: 81.8
Tertinggi: 92
Terendah: 66
Yang lulus: 78, 85, 92, 88
Jumlah lulus: 4 dari 5
```

Proses eksekusinya:

1. `reduce` dengan penjumlahan menghasilkan 409, dan dibagi 5 menjadi 81,8.
2. `reduce` tanpa nilai awal memakai elemen pertama sebagai penampung. Ternary `a > b ? a : b` (dari bab 4) selalu menyimpan yang lebih besar, hasilnya 92. Sebaliknya untuk terendah.
3. `filter` menyaring nilai yang minimal 70, dan `join(", ")` merangkainya menjadi teks.

### Keranjang belanja

Simpan sebagai `praktis-keranjang.js`:

```javascript
const harga = [15000, 8000, 22000, 5000];

const total = harga.reduce((a, b) => a + b, 0);
const mahal = harga.filter((h) => h > 10000);
const setelahDiskon = harga.map((h) => (h * 90) / 100);
const totalDiskon = setelahDiskon.reduce((a, b) => a + b, 0);

console.log(`Total: ${total}`);
console.log(`Barang di atas 10000: ${mahal}`);
console.log(`Setelah diskon 10%: ${setelahDiskon}`);
console.log(`Total setelah diskon: ${totalDiskon}`);
```

Output:

```text
Total: 50000
Barang di atas 10000: 15000,22000
Setelah diskon 10%: 13500,7200,19800,4500
Total setelah diskon: 45000
```

Ketiga perintah dirangkai: `filter` memilih, `map` mengubah, `reduce` menjumlah. Perhatikan bahwa array yang disisipkan ke template literal ditampilkan sebagai teks dengan elemen dipisah koma.

## Kesalahan umum

### 1. `map` dengan `{ }` tapi tanpa `return`

Simpan sebagai `map-tanpa-return.js`:

```javascript
const dobel = [1, 2, 3].map((x) => {
  x * 2;
});

console.log(dobel);
```

Output:

```text
[ undefined, undefined, undefined ]
```

Arrow function dengan `{ }` harus menulis `return` sendiri (aturan bab 6). **Perbaikan:** `(x) => x * 2` atau `(x) => { return x * 2; }`.

### 2. `reduce` pada array kosong tanpa nilai awal

Simpan sebagai `reduce-kosong.js`:

```javascript
console.log([].reduce((a, b) => a + b, 0));
console.log([].reduce((a, b) => a + b));
```

Output (dipersingkat):

```text
0
TypeError: Reduce of empty array with no initial value
```

**Perbaikan:** selalu berikan nilai awal jika array bisa kosong.

### 3. Mengurutkan angka tanpa pembanding

`[10, 9, 1].sort()` menghasilkan `[1, 10, 9]`. **Perbaikan:** `sort((x, y) => x - y)`.

### 4. Lupa bahwa `sort`, `reverse`, `push`, `splice` mengubah array asli

Simpan sebagai `mengubah-asli.js`:

```javascript
const x = [3, 1, 2];
x.sort();
console.log(x);
```

Output:

```text
[ 1, 2, 3 ]
```

Jika array asli masih dibutuhkan, salin dulu dengan `slice()`.

### 5. Mengira `const b = a` membuat salinan

Seperti pada `const-array.js`, itu hanya nama kedua untuk array yang sama. **Perbaikan:** `const b = a.slice();`.

### 6. Selisih satu pada indeks

Elemen terakhir ada di `array[array.length - 1]`, bukan `array[array.length]` (yang bernilai `undefined`). Pada perulangan, gunakan `i < array.length`, bukan `<=`.

### 7. Membandingkan array dengan `===`

`[1, 2] === [1, 2]` bernilai `false`, karena yang dibandingkan adalah referensinya. Untuk membandingkan isi, bandingkan `join()`-nya atau periksa elemen satu per satu.

## Ringkasan

- **Array** menyimpan banyak nilai berurutan dalam satu variabel: `[a, b, c]`. Indeks dimulai dari 0.
- `length` memberi jumlah elemen. Elemen terakhir ada di `length - 1`, atau `at(-1)`.
- `push`/`pop` bekerja di belakang, `unshift`/`shift` di depan. Semuanya mengubah array asli.
- `includes` dan `indexOf` mencari. `slice`, `join`, dan `concat` tidak mengubah array asli, sedangkan `splice` mengubahnya.
- `for`, `for...of`, dan `forEach` memproses semua elemen.
- `map` mengubah, `filter` menyaring, `find` mencari satu, `some`/`every` memeriksa kondisi, dan `reduce` merangkum menjadi satu nilai. Semuanya tidak mengubah array asli.
- `sort()` bawaan mengurutkan sebagai teks. Untuk angka, pakai pembanding `(x, y) => x - y`.
- `const` pada array hanya mengunci nama variabel, bukan isinya. Array adalah referensi, jadi salin dengan `slice()`.

## Latihan

Kerjakan sendiri dulu sebelum melihat jawaban.

### Latihan 1: Prediksi output

Apa yang tampil?

```javascript
const a = [1, 2, 3];
a[1] = 10;
a.push(4);
console.log(a, a.length);
```

??? success "Jawaban"
    Output: `[ 1, 10, 3, 4 ] 4`. Elemen indeks 1 diganti menjadi 10, lalu 4 ditambahkan di belakang sehingga panjangnya 4.

### Latihan 2: Perbaiki `map`

Kode ini menghasilkan `[ undefined, undefined, undefined ]`. Perbaiki agar menghasilkan `[ 2, 4, 6 ]`:

```javascript
const hasil = [1, 2, 3].map((x) => {
  x * 2;
});

console.log(hasil);
```

??? success "Jawaban"
    ```javascript
    const hasil = [1, 2, 3].map((x) => x * 2);

    console.log(hasil);
    ```
    Output: `[ 2, 4, 6 ]`

### Latihan 3: Rata-rata nilai

Hitung rata-rata dari `[70, 85, 90, 65]` memakai `for...of` (jumlahkan dulu, lalu bagi dengan `length`).

??? success "Contoh jawaban"
    ```javascript
    const nilai = [70, 85, 90, 65];
    let total = 0;

    for (const n of nilai) {
      total += n;
    }

    console.log(total / nilai.length);
    ```
    Output: `77.5`

### Latihan 4: Genap lalu kuadrat

Dari angka 1 sampai 10, ambil yang genap saja, lalu kuadratkan hasilnya. Tampilkan array genapnya dan array kuadratnya.

??? success "Contoh jawaban"
    ```javascript
    const angka = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10];
    const genap = angka.filter((x) => x % 2 === 0);
    const kuadrat = genap.map((x) => x * x);

    console.log(genap);
    console.log(kuadrat);
    ```
    Output:
    ```text
    [ 2, 4, 6, 8, 10 ]
    [ 4, 16, 36, 64, 100 ]
    ```

### Latihan 5: Cari nama

Dari `["Budi", "Sari", "Andi", "Rina"]`, cari nama pertama yang diawali huruf `A` (`startsWith`), periksa apakah `"Rina"` ada, lalu tampilkan array yang sudah diurutkan abjad tanpa mengubah array asli.

??? success "Contoh jawaban"
    ```javascript
    const nama = ["Budi", "Sari", "Andi", "Rina"];

    console.log(nama.find((n) => n.startsWith("A")));
    console.log(nama.includes("Rina"));
    console.log(nama.slice().sort());
    console.log(nama);
    ```
    Output:
    ```text
    Andi
    true
    [ 'Andi', 'Budi', 'Rina', 'Sari' ]
    [ 'Budi', 'Sari', 'Andi', 'Rina' ]
    ```

### Latihan 6: Urutkan angka

Urutkan `[5, 25, 100, 1]` dari terbesar ke terkecil. Tebak juga apa hasilnya jika memakai `sort()` tanpa pembanding.

??? success "Jawaban"
    ```javascript
    const angka = [5, 25, 100, 1];
    console.log(angka.slice().sort((a, b) => b - a));
    console.log(angka.slice().sort());
    ```
    Output:
    ```text
    [ 100, 25, 5, 1 ]
    [ 1, 100, 25, 5 ]
    ```
    Tanpa pembanding, angka diurutkan sebagai teks, sehingga `"100"` datang sebelum `"25"`.

### Latihan 7: Keranjang belanja

Harga barang: `[15000, 8000, 22000, 5000]`. Hitung total harga, ambil barang yang harganya di atas 10000, lalu hitung total setelah diskon 10% untuk semua barang.

??? success "Contoh jawaban"
    ```javascript
    const harga = [15000, 8000, 22000, 5000];

    const total = harga.reduce((a, b) => a + b, 0);
    const mahal = harga.filter((h) => h > 10000);
    const totalDiskon = harga
      .map((h) => (h * 90) / 100)
      .reduce((a, b) => a + b, 0);

    console.log(total);
    console.log(mahal);
    console.log(totalDiskon);
    ```
    Output:
    ```text
    50000
    [ 15000, 22000 ]
    45000
    ```
    Perintah `map` dan `reduce` bisa dirangkai, karena `map` menghasilkan array baru yang langsung diproses `reduce`.

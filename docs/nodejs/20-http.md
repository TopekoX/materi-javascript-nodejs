## Tujuan pembelajaran

Setelah mempelajari bab ini, kamu dapat:

- Menjelaskan cara kerja HTTP: permintaan (*request*), respons (*response*), metode, jalur, header, body, dan kode status.
- Membuat server HTTP dengan `node:http`.
- Membaca metode, alamat, query, dan body dari permintaan.
- Mengirim respons teks dan JSON dengan kode status dan header yang benar.
- Membuat *routing* secara manual.
- Memanggil server dengan `fetch` untuk mengujinya.
- Menangani error dan masukan yang tidak valid dengan kode status yang tepat.
- Menjalankan dan menghentikan server dengan rapi.

Bab ini memakai class dan error (bab 10 dan 13), async (bab 12), `URL` (bab 17), serta `process.env` dan `SIGINT` (bab 18). Semua contoh dijalankan dari folder `node-belajar`.

## Pendahuluan: restoran di internet

Setiap kali kamu membuka sebuah halaman web atau memakai aplikasi di ponsel, perangkatmu **meminta** sesuatu kepada komputer lain dan menerima **jawaban**. Aturan percakapan itu bernama **HTTP** (*HyperText Transfer Protocol*).

Analogi restoran:

| Di restoran | Di HTTP |
|---|---|
| Pelanggan | **Klien** (browser, aplikasi, atau program lain) |
| Pelayan dan dapur | **Server** (program yang kita tulis) |
| Memesan: "satu nasi goreng" | **Permintaan** (*request*) |
| Pesanan datang, atau "maaf, habis" | **Respons** (*response*) |
| Alamat meja atau nama menu | **URL** (alamat sumber daya) |

Server tidak tahu kapan pelanggan datang, jadi ia **menunggu** dan bereaksi setiap kali ada permintaan. Itu pola event (bab 19): server memancarkan event `request` untuk setiap permintaan yang masuk.

## Anatomi HTTP

Sebuah permintaan membawa:

| Bagian | Fungsi | Contoh |
|---|---|---|
| **Metode** | Apa yang ingin dilakukan | `GET` (ambil), `POST` (buat), `PUT` (ganti), `PATCH` (ubah sebagian), `DELETE` (hapus) |
| **URL** | Sumber daya yang dituju | `/api/produk/2?harga_maks=3000` |
| **Header** | Keterangan tambahan | `Content-Type: application/json` |
| **Body** | Isi data (opsional, umumnya untuk POST/PUT/PATCH) | `{"nama": "Pensil"}` |

Sebuah respons membawa **kode status**, header, dan body. Kode status adalah angka tiga digit yang menjelaskan hasilnya:

| Kelompok | Arti | Kode yang sering dipakai |
|---|---|---|
| **2xx** | Berhasil | `200 OK`, `201 Created` (data baru dibuat), `204 No Content` (berhasil, tanpa isi) |
| **3xx** | Pengalihan | `301`, `302` |
| **4xx** | Kesalahan dari **klien** | `400 Bad Request` (masukan salah), `404 Not Found`, `405 Method Not Allowed`, `413 Payload Too Large` |
| **5xx** | Kesalahan dari **server** | `500 Internal Server Error` |

Membedakan 4xx dan 5xx penting: "kamu salah mengirim" (4xx) berbeda dari "saya yang rusak" (5xx).

## Server pertama

Simpan sebagai `halo-server.js`:

```javascript title="halo-server.js"
import { createServer } from "node:http";

const server = createServer((req, res) => {
  res.writeHead(200, { "Content-Type": "text/plain; charset=utf-8" });
  res.end("Halo dari Node.js!");
});

server.listen(3000, () => {
  console.log("Server berjalan di http://localhost:3000");
});
```

```bash
# berjalan terus, hentikan dengan Ctrl + C
node halo-server.js
```

Output:

```text
Server berjalan di http://localhost:3000
```

Terminal tidak kembali ke prompt, karena server sedang menunggu permintaan (ingat bab 15: program berhenti hanya ketika tidak ada pekerjaan tertunda). Buka `http://localhost:3000` di browser, dan kamu akan melihat `Halo dari Node.js!`. Hentikan dengan `Ctrl + C`.

Penjelasan kode:

| Kode | Artinya |
|---|---|
| `createServer(fungsi)` | Membuat server. Fungsi itu dipanggil **setiap kali ada permintaan** dengan dua argumen: `req` dan `res` |
| `req` | Objek permintaan (*IncomingMessage*): berisi metode, URL, header, dan body |
| `res` | Objek respons (*ServerResponse*): dipakai untuk mengirim jawaban |
| `res.writeHead(200, {...})` | Menentukan kode status dan header respons |
| `res.end("...")` | Mengirim body dan **menyelesaikan** respons |
| `server.listen(3000, ...)` | Mulai menerima koneksi di **port** 3000. Port adalah "nomor pintu" di komputer. Fungsi kedua dipanggil saat server siap |
| `localhost` | Nama untuk "komputer ini sendiri" |

!!! warning "`res.end()` wajib dipanggil"
    Respons baru dikirim lengkap setelah `res.end()`. Tanpa itu, klien menunggu tanpa akhir (lihat kesalahan umum).

## Menguji server dengan `fetch`

Menjalankan server lalu membuka browser setiap kali tidak praktis untuk belajar. Kita bisa menjalankan server **dan** klien dalam satu skrip: server mendengarkan di port acak (port `0` berarti "pilihkan port yang kosong"), lalu `fetch` (bab 12) memanggilnya, lalu server ditutup. Buat fungsi bantu `bantu-http.js` yang akan kita pakai di sisa bab ini:

```javascript title="bantu-http.js"
export async function dengarkan(server) {
  await new Promise((resolve) => server.listen(0, "127.0.0.1", resolve));
  const { port } = server.address();

  return {
    url: `http://127.0.0.1:${port}`,
    tutup: () =>
      new Promise((resolve) => {
        server.close(resolve);
        server.closeAllConnections();
      }),
  };
}
```

Fungsi `dengarkan` menjalankan server di port acak dan mengembalikan `url`-nya, serta fungsi `tutup` untuk menghentikannya. Sekarang kita bisa melihat permintaan dan respons sekaligus. Simpan sebagai `permintaan.js`:

```javascript title="permintaan.js"
import { createServer } from "node:http";
import { dengarkan } from "./bantu-http.js";

const app = await dengarkan(
  createServer((req, res) => {
    console.log("server menerima:", req.method, req.url, req.headers["content-type"]);
    res.writeHead(200, { "Content-Type": "text/plain; charset=utf-8" });
    res.end("Halo dari server");
  }),
);

const respons = await fetch(app.url + "/halo?x=1", {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({ a: 1 }),
});

console.log(respons.status, respons.ok, respons.headers.get("content-type"));
console.log(await respons.text());

await app.tutup();
console.log("server ditutup");
```

```bash
node permintaan.js
```

Output:

```text
server menerima: POST /halo?x=1 application/json
200 true text/plain; charset=utf-8
Halo dari server
server ditutup
```

Dua sisi percakapan terlihat sekaligus:

- **Di server**: `req.method` adalah `POST`, `req.url` adalah `/halo?x=1` (jalur **beserta** query), dan `req.headers` berisi header (nama header selalu huruf kecil).
- **Di klien**: `fetch(url, opsi)` mengembalikan Promise berisi objek respons. `respons.status` adalah kode status (200), `respons.ok` bernilai `true` untuk 2xx, `respons.headers.get(...)` membaca header, dan `respons.text()` atau `respons.json()` membaca body (keduanya Promise).

### Kode status dan header

Simpan sebagai `status.js`:

```javascript title="status.js"
import { createServer } from "node:http";
import { dengarkan } from "./bantu-http.js";

const app = await dengarkan(
  createServer((req, res) => {
    res.writeHead(201, { "Content-Type": "application/json", "X-Contoh": "ya" });
    res.end(JSON.stringify({ pesan: "dibuat" }));
  }),
);

const respons = await fetch(app.url);
console.log(respons.status, respons.statusText, respons.ok, respons.headers.get("x-contoh"));
console.log(await respons.json());

await app.tutup();
```

```bash
node status.js
```

Output:

```text
201 Created true ya
{ pesan: 'dibuat' }
```

Dua hal penting:

- Body respons harus berupa **teks atau Buffer**. Mengirim objek langsung (`res.end({ a: 1 })`) menghasilkan error. Ubah dulu menjadi JSON dengan `JSON.stringify`, dan beri tahu klien lewat header `Content-Type: application/json`.
- Header sendiri boleh apa saja (`X-Contoh`). Nama yang diawali `X-` secara tradisi dipakai untuk header buatan sendiri.

## Membangun API toko mini

Sekarang kita buat server yang sungguhan: sebuah API sederhana untuk daftar produk, dengan beberapa jalur. Server ditulis dalam dua berkas agar mudah diuji: `toko-app.js` berisi logika (dan **tidak** langsung mendengarkan port), sedangkan `toko-server.js` yang menjalankannya.

```javascript title="toko-app.js"
import { createServer } from "node:http";

const produk = [
  { id: 1, nama: "Pensil", harga: 2000 },
  { id: 2, nama: "Buku", harga: 8000 },
  { id: 3, nama: "Pulpen", harga: 3000 },
];

class GalatHttp extends Error {
  constructor(status, pesan) {
    super(pesan);
    this.status = status;
  }
}

function kirim(res, status, data, headers = {}) {
  const json = typeof data !== "string";
  res.writeHead(status, {
    "Content-Type": json ? "application/json; charset=utf-8" : "text/plain; charset=utf-8",
    ...headers,
  });
  res.end(json ? JSON.stringify(data) : data);
}

async function bacaJson(req) {
  let teks = "";
  for await (const potongan of req) {
    teks += potongan;
  }
  if (teks === "") {
    throw new GalatHttp(400, "Body kosong");
  }
  try {
    return JSON.parse(teks);
  } catch {
    throw new GalatHttp(400, "JSON tidak valid");
  }
}

export function buatServer() {
  return createServer(async (req, res) => {
    try {
      const url = new URL(req.url, `http://${req.headers.host}`);
      const bagian = url.pathname.split("/").filter(Boolean);

      if (url.pathname === "/") {
        return kirim(res, 200, "Selamat datang di Toko API");
      }

      if (bagian[0] === "api" && bagian[1] === "produk" && bagian.length === 2) {
        if (req.method === "GET") {
          const hargaMaks = url.searchParams.get("harga_maks");
          if (hargaMaks === null) {
            return kirim(res, 200, produk);
          }
          if (Number.isNaN(Number(hargaMaks))) {
            throw new GalatHttp(400, "harga_maks harus angka");
          }
          return kirim(res, 200, produk.filter((p) => p.harga <= Number(hargaMaks)));
        }

        if (req.method === "POST") {
          const { nama, harga } = await bacaJson(req);
          if (typeof nama !== "string" || nama.trim() === "") {
            throw new GalatHttp(400, "nama wajib diisi");
          }
          if (typeof harga !== "number" || harga <= 0) {
            throw new GalatHttp(400, "harga harus angka lebih dari 0");
          }
          const baru = { id: produk.at(-1).id + 1, nama, harga };
          produk.push(baru);
          return kirim(res, 201, baru, { Location: `/api/produk/${baru.id}` });
        }

        return kirim(res, 405, { galat: "Metode tidak diizinkan" }, { Allow: "GET, POST" });
      }

      if (bagian[0] === "api" && bagian[1] === "produk" && bagian.length === 3) {
        const id = Number(bagian[2]);
        if (!Number.isInteger(id)) {
          throw new GalatHttp(400, "id harus bilangan bulat");
        }
        const ditemukan = produk.find((p) => p.id === id);
        if (!ditemukan) {
          throw new GalatHttp(404, `Produk ${id} tidak ditemukan`);
        }
        if (req.method === "GET") {
          return kirim(res, 200, ditemukan);
        }
        return kirim(res, 405, { galat: "Metode tidak diizinkan" }, { Allow: "GET" });
      }

      throw new GalatHttp(404, "Jalur tidak ditemukan");
    } catch (error) {
      if (error instanceof GalatHttp) {
        return kirim(res, error.status, { galat: error.message });
      }
      console.error(error);
      return kirim(res, 500, { galat: "Kesalahan server" });
    }
  });
}
```

Kode ini panjang, jadi mari bedah bagian demi bagian:

| Bagian | Penjelasan |
|---|---|
| `produk` | Data disimpan di array di memori. Setiap kali server dimatikan, perubahan hilang (penyimpanan permanen dibahas di bab 23) |
| `class GalatHttp extends Error` | Error khusus yang membawa **kode status** (bab 13). Kapan pun kita ingin menjawab dengan 4xx, cukup `throw new GalatHttp(400, "...")` |
| `kirim(res, status, data, headers)` | Fungsi bantu: jika `data` berupa teks, kirim sebagai teks. Jika objek atau array, ubah menjadi JSON dan atur header yang sesuai |
| `bacaJson(req)` | Membaca body permintaan. `req` adalah stream (bab 19), jadi dibaca dengan `for await`. Body kosong atau JSON rusak menjadi `GalatHttp(400)` |
| `new URL(req.url, "http://" + host)` | `req.url` hanya berisi jalur dan query, jadi kita bentuk URL penuh agar bisa memakai `pathname` dan `searchParams` (bab 17) |
| `url.pathname.split("/").filter(Boolean)` | Memecah `/api/produk/2` menjadi `["api", "produk", "2"]`. `filter(Boolean)` membuang elemen kosong |
| Rangkaian `if` | **Routing**: memilih tindakan berdasarkan jalur dan metode |
| `try ... catch` besar | Menjamin setiap permintaan mendapat jawaban. `GalatHttp` dijawab sesuai statusnya, error lain dijawab `500` |

Sekarang uji semua jalurnya dengan satu skrip klien. Simpan sebagai `toko-uji.js`:

```javascript title="toko-uji.js"
import { buatServer } from "./toko-app.js";
import { dengarkan } from "./bantu-http.js";

const app = await dengarkan(buatServer());

async function panggil(metode, jalur, body) {
  const respons = await fetch(app.url + jalur, {
    method: metode,
    headers: body === undefined ? {} : { "Content-Type": "application/json" },
    body: body === undefined ? undefined : JSON.stringify(body),
  });
  const teks = await respons.text();
  const isi = respons.headers.get("content-type")?.includes("json") ? JSON.parse(teks) : teks;
  console.log(metode, jalur, "->", respons.status, isi);
  return respons;
}

await panggil("GET", "/");
await panggil("GET", "/api/produk");
await panggil("GET", "/api/produk?harga_maks=3000");
await panggil("GET", "/api/produk?harga_maks=murah");
await panggil("GET", "/api/produk/2");
await panggil("GET", "/api/produk/99");
await panggil("GET", "/api/produk/abc");

const dibuat = await panggil("POST", "/api/produk", { nama: "Penghapus", harga: 1500 });
console.log("Location:", dibuat.headers.get("location"));

await panggil("POST", "/api/produk", { nama: "", harga: -1 });
await panggil("POST", "/api/produk", { nama: "Spidol", harga: "mahal" });

const salah = await panggil("DELETE", "/api/produk");
console.log("Allow:", salah.headers.get("allow"));

await panggil("GET", "/tidak-ada");
await panggil("GET", "/api/produk/4");

const rusak = await fetch(app.url + "/api/produk", {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: "{rusak",
});
console.log("POST rusak ->", rusak.status, await rusak.json());

const kosong = await fetch(app.url + "/api/produk", { method: "POST" });
console.log("POST kosong ->", kosong.status, await kosong.json());

await app.tutup();
```

```bash
node toko-uji.js
```

Output:

```text
GET / -> 200 Selamat datang di Toko API
GET /api/produk -> 200 [
  { id: 1, nama: 'Pensil', harga: 2000 },
  { id: 2, nama: 'Buku', harga: 8000 },
  { id: 3, nama: 'Pulpen', harga: 3000 }
]
GET /api/produk?harga_maks=3000 -> 200 [
  { id: 1, nama: 'Pensil', harga: 2000 },
  { id: 3, nama: 'Pulpen', harga: 3000 }
]
GET /api/produk?harga_maks=murah -> 400 { galat: 'harga_maks harus angka' }
GET /api/produk/2 -> 200 { id: 2, nama: 'Buku', harga: 8000 }
GET /api/produk/99 -> 404 { galat: 'Produk 99 tidak ditemukan' }
GET /api/produk/abc -> 400 { galat: 'id harus bilangan bulat' }
POST /api/produk -> 201 { id: 4, nama: 'Penghapus', harga: 1500 }
Location: /api/produk/4
POST /api/produk -> 400 { galat: 'nama wajib diisi' }
POST /api/produk -> 400 { galat: 'harga harus angka lebih dari 0' }
DELETE /api/produk -> 405 { galat: 'Metode tidak diizinkan' }
Allow: GET, POST
GET /tidak-ada -> 404 { galat: 'Jalur tidak ditemukan' }
GET /api/produk/4 -> 200 { id: 4, nama: 'Penghapus', harga: 1500 }
POST rusak -> 400 { galat: 'JSON tidak valid' }
POST kosong -> 400 { galat: 'Body kosong' }
```

Perhatikan pilihan kode statusnya:

| Situasi | Status | Alasan |
|---|---|---|
| Daftar atau satu produk ditemukan | `200` | Berhasil |
| Produk baru dibuat | `201` + header `Location` | `201` khusus untuk "sesuatu baru telah dibuat", dan `Location` menunjuk alamat sumber daya baru |
| `harga_maks=murah`, `id` bukan angka, nama kosong, JSON rusak | `400` | Masukan dari klien salah |
| Produk 99 tidak ada, jalur tidak dikenal | `404` | Sumber daya tidak ditemukan |
| `DELETE` ke jalur yang hanya menerima `GET`/`POST` | `405` + header `Allow` | Metode tidak didukung, dan `Allow` memberi tahu metode yang boleh |

Setelah `POST` berhasil, produk `4` benar-benar bisa diambil (`GET /api/produk/4`), karena datanya tersimpan di array selama server hidup.

## Menjalankan server sungguhan

Berkas yang mendengarkan port dipisah, supaya `toko-app.js` tetap bisa diuji tanpa port tetap. Simpan sebagai `toko-server.js`:

```javascript title="toko-server.js"
import { buatServer } from "./toko-app.js";

const PORT = Number(process.env.PORT) || 3000;
const server = buatServer();

server.listen(PORT, () => {
  console.log(`Server berjalan di http://localhost:${PORT}`);
});

process.on("SIGINT", () => {
  console.log("\nMenutup server...");
  server.close(() => console.log("Server ditutup"));
});
```

Port dibaca dari variabel lingkungan `PORT` dengan nilai bawaan 3000 (bab 18), dan `SIGINT` (Ctrl + C) menutup server dengan rapi: `server.close()` berhenti menerima koneksi baru dan menunggu yang sedang berjalan selesai. Jalankan:

```bash
# berjalan terus, hentikan dengan Ctrl + C
node toko-server.js
```

Output:

```text
Server berjalan di http://localhost:3000
```

Biarkan server berjalan, lalu buka terminal **kedua** dan buat klien kecil `klien.js`:

```javascript title="klien.js"
const respons = await fetch("http://localhost:3000/api/produk/2");
console.log(respons.status, await respons.json());
```

```bash
node klien.js
```

Output:

```text
200 { id: 2, nama: 'Buku', harga: 8000 }
```

Kamu juga bisa membuka `http://localhost:3000/api/produk` di browser untuk melihat JSON mentah, atau memakai program `curl`:

```bash
curl http://localhost:3000/api/produk/2
```

Output:

```text
{"id":2,"nama":"Buku","harga":8000}
```

(Di Windows PowerShell, `curl` adalah alias untuk perintah lain; gunakan `curl.exe`.) Kembali ke terminal server dan tekan `Ctrl + C`:

```text
Menutup server...
Server ditutup
```

## Kesalahan umum

### 1. Lupa `res.end()`

Simpan sebagai `lupa-end.js`:

```javascript title="lupa-end.js"
import { createServer } from "node:http";
import { dengarkan } from "./bantu-http.js";

const app = await dengarkan(
  createServer((req, res) => {
    res.statusCode = 200;
    res.write("halo");
  }),
);

try {
  const respons = await fetch(app.url, { signal: AbortSignal.timeout(300) });
  console.log("fetch selesai, status", respons.status);
  await respons.text();
} catch (e) {
  console.log(e.name);
}

await app.tutup();
```

```bash
node lupa-end.js
```

Output:

```text
fetch selesai, status 200
TimeoutError
```

Status dan header sudah terkirim (`res.write` mengirimnya), sehingga `fetch` selesai dengan status 200. Tetapi tanpa `res.end()` isi respons tidak pernah lengkap, jadi `respons.text()` menunggu tanpa akhir sampai dibatalkan (di sini oleh `AbortSignal.timeout(300)`, batas waktu 300 ms). Jika handler bahkan tidak menulis apa pun, `fetch` itu sendiri yang menggantung. **Perbaikan:** setiap jalur harus berakhir dengan `res.end()`. Jika ada percabangan, pastikan setiap cabang memanggilnya.

### 2. Mengubah respons setelah dikirim

Simpan sebagai `terkirim.js`:

```javascript title="terkirim.js"
import { createServer } from "node:http";
import { dengarkan } from "./bantu-http.js";

const app = await dengarkan(
  createServer((req, res) => {
    res.end("pertama");
    try {
      res.writeHead(500);
    } catch (e) {
      console.log(e.code, e.message);
    }
  }),
);

console.log(await (await fetch(app.url)).text());
await app.tutup();
```

```bash
node terkirim.js
```

Output:

```text
ERR_HTTP_HEADERS_SENT Cannot write headers after they are sent to the client
pertama
```

Setelah `res.end()`, status dan header sudah terkirim dan tidak bisa diubah. Penyebab paling umum: lupa `return` setelah mengirim respons di sebuah `if`, sehingga kode terus berjalan ke bagian yang mengirim respons lagi. Pada `toko-app.js` itu sebabnya setiap `kirim(...)` ditulis dengan `return`.

### 3. Mengirim objek langsung ke `res.end`

Simpan sebagai `kirim-objek.js`:

```javascript title="kirim-objek.js"
import { createServer } from "node:http";
import { dengarkan } from "./bantu-http.js";

const app = await dengarkan(
  createServer((req, res) => {
    try {
      res.end({ a: 1 });
    } catch (e) {
      console.log(e.name, e.code);
      res.end("ok");
    }
  }),
);

await (await fetch(app.url)).text();
await app.tutup();
```

```bash
node kirim-objek.js
```

Output:

```text
TypeError ERR_INVALID_ARG_TYPE
```

Body hanya boleh teks atau `Buffer`. **Perbaikan:** `res.end(JSON.stringify(objek))` dengan header `Content-Type: application/json`.

### 4. Handler `async` tanpa `try...catch`

Simpan sebagai `async-crash.js`:

```javascript title="async-crash.js"
import { createServer } from "node:http";
import { dengarkan } from "./bantu-http.js";

const app = await dengarkan(
  createServer(async (req, res) => {
    const data = JSON.parse("{rusak");
    res.end("tidak sampai");
  }),
);

await fetch(app.url, { signal: AbortSignal.timeout(500) }).catch(() => {});
await app.tutup();
```

```bash
node async-crash.js
```

Output (dipersingkat):

```text
SyntaxError: Expected property name or '}' in JSON at position 1 (line 1 column 2)
```

Handler `async` yang melempar error menghasilkan Promise yang ditolak tanpa penanganan (bab 12 dan 18), yang menghentikan **seluruh server**, bukan hanya satu permintaan. Karena itu `toko-app.js` membungkus semuanya dengan `try...catch` dan menjawab `500` untuk error yang tak terduga. Server yang mati karena satu permintaan buruk adalah celah serius.

### 5. Port sudah dipakai

Menjalankan dua server di port yang sama menghasilkan error:

```text
Error: listen EADDRINUSE: address already in use :::3000
```

Biasanya penyebabnya adalah server lama masih berjalan di terminal lain. Hentikan dulu, atau gunakan `PORT` berbeda lewat variabel lingkungan.

### 6. Mengira `req.url` hanya jalur

`req.url` adalah jalur **beserta query**: untuk `/api/produk?harga_maks=3000` nilainya `/api/produk?harga_maks=3000`, sehingga `req.url === "/api/produk"` gagal. Gunakan `new URL(req.url, base).pathname` dan `searchParams`.

### 7. Menjawab error dengan status 200

Mengirim `200` dengan isi `{"galat": "..."}` membuat klien mengira permintaannya berhasil (`respons.ok` bernilai `true`). Gunakan status yang sesuai (400, 404, 500).

### 8. Percaya begitu saja pada masukan klien

Body, query, dan header berasal dari luar dan bisa berisi apa saja: teks kosong, tipe salah, atau data raksasa. Selalu validasi (seperti pada `POST /api/produk`) dan balas `400` bila tidak valid.

### 9. Lupa bahwa data di memori hilang

Array `produk` hanya hidup selama server berjalan. Memulai ulang server mengembalikan data awal. Untuk data permanen, simpan ke berkas atau database (bab 23).

## Ringkasan

- **HTTP**: klien mengirim *request* (metode, URL, header, body), server menjawab *response* (kode status, header, body).
- Kode status: `2xx` berhasil (`200`, `201`, `204`), `4xx` kesalahan klien (`400`, `404`, `405`), `5xx` kesalahan server (`500`).
- `createServer((req, res) => { ... })` dari `node:http` membuat server. `server.listen(port)` mulai menerima koneksi.
- Pada `req`: `req.method`, `req.url` (jalur + query), `req.headers`, dan body yang dibaca seperti stream (`for await`). Pecah alamat dengan `new URL(req.url, base)`.
- Pada `res`: `res.writeHead(status, headers)` lalu **wajib** `res.end(body)`. Body harus teks atau Buffer, jadi JSON dikirim dengan `JSON.stringify` dan header `Content-Type: application/json`.
- **Routing** memilih tindakan berdasarkan jalur dan metode. Beri `405` plus header `Allow` untuk metode yang tidak didukung.
- Bungkus handler dengan `try...catch` supaya satu permintaan buruk tidak mematikan server. Gunakan error khusus yang membawa status.
- `fetch` adalah klien HTTP bawaan. Menjalankan server di port `0` plus `fetch` dalam satu skrip memudahkan pengujian.
- Pisahkan logika server (`buatServer`) dari bagian yang mendengarkan port, dan tutup dengan rapi saat `SIGINT`.

## Latihan

Kerjakan sendiri dulu sebelum melihat jawaban. Latihan memakai `bantu-http.js` dan dijalankan dari folder `node-belajar`.

### Latihan 1: Prediksi

Apa yang tercetak?

```javascript
import { createServer } from "node:http";
import { dengarkan } from "./bantu-http.js";

const app = await dengarkan(
  createServer((req, res) => {
    if (req.url === "/ping") {
      res.end("pong");
      return;
    }
    res.statusCode = 404;
    res.end("tidak ada");
  }),
);

const a = await fetch(app.url + "/ping");
console.log(a.status, await a.text());
const b = await fetch(app.url + "/x");
console.log(b.status, await b.text());

await app.tutup();
```

??? success "Jawaban"
    Output: `200 pong` lalu `404 tidak ada`. Kode status bawaan `res` adalah 200, sehingga `/ping` menjawab 200. Jalur lain diberi status 404 secara manual.

### Latihan 2: Perbaiki respons yang menggantung

Klien menunggu tanpa akhir (hasilnya `TimeoutError`). Perbaiki agar mencetak `halo`:

```javascript
import { createServer } from "node:http";
import { dengarkan } from "./bantu-http.js";

const app = await dengarkan(
  createServer((req, res) => {
    res.statusCode = 200;
    res.write("halo");
  }),
);

try {
  console.log(await (await fetch(app.url, { signal: AbortSignal.timeout(300) })).text());
} catch (e) {
  console.log(e.name);
}

await app.tutup();
```

??? success "Jawaban"
    Ganti `res.write("halo")` dengan `res.end("halo")` (atau tambahkan `res.end()` setelah `write`).
    ```javascript
    import { createServer } from "node:http";
    import { dengarkan } from "./bantu-http.js";

    const app = await dengarkan(
      createServer((req, res) => {
        res.statusCode = 200;
        res.end("halo");
      }),
    );

    console.log(await (await fetch(app.url, { signal: AbortSignal.timeout(300) })).text());

    await app.tutup();
    ```

    Output: `halo`

### Latihan 3: Jalur dengan query

Buat server yang menjawab `GET /api/halo?nama=Budi` dengan JSON `{ pesan: "Halo, Budi!" }`. Jika `nama` tidak ada, pakai `teman`. Jalur lain dijawab `404`.

??? success "Contoh jawaban"
    ```javascript
    import { createServer } from "node:http";
    import { dengarkan } from "./bantu-http.js";

    const app = await dengarkan(
      createServer((req, res) => {
        const url = new URL(req.url, `http://${req.headers.host}`);
        if (url.pathname === "/api/halo") {
          const nama = url.searchParams.get("nama") || "teman";
          res.writeHead(200, { "Content-Type": "application/json" });
          res.end(JSON.stringify({ pesan: `Halo, ${nama}!` }));
          return;
        }
        res.writeHead(404);
        res.end();
      }),
    );

    console.log(await (await fetch(app.url + "/api/halo?nama=Budi")).json());
    console.log(await (await fetch(app.url + "/api/halo")).json());

    await app.tutup();
    ```

    Output:
    ```text
    { pesan: 'Halo, Budi!' }
    { pesan: 'Halo, teman!' }
    ```

### Latihan 4: POST dengan validasi

Buat `POST /api/hitung` yang menerima body JSON `{ "a": 2, "b": 3 }` dan menjawab `{ hasil: 5 }`. Jika `a` atau `b` bukan angka, jawab `400` dengan `{ galat: "a dan b harus angka" }`. Uji dengan `{ a: 2, b: 3 }` dan `{ a: "2", b: 3 }`.

??? success "Contoh jawaban"
    ```javascript
    import { createServer } from "node:http";
    import { dengarkan } from "./bantu-http.js";

    const app = await dengarkan(
      createServer(async (req, res) => {
        let teks = "";
        for await (const potongan of req) {
          teks += potongan;
        }
        const { a, b } = JSON.parse(teks);
        const adaGalat = typeof a !== "number" || typeof b !== "number";
        res.writeHead(adaGalat ? 400 : 200, { "Content-Type": "application/json" });
        res.end(JSON.stringify(adaGalat ? { galat: "a dan b harus angka" } : { hasil: a + b }));
      }),
    );

    async function hitung(body) {
      const r = await fetch(app.url + "/api/hitung", {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify(body),
      });
      console.log(r.status, await r.json());
    }

    await hitung({ a: 2, b: 3 });
    await hitung({ a: "2", b: 3 });

    await app.tutup();
    ```

    Output:
    ```text
    200 { hasil: 5 }
    400 { galat: 'a dan b harus angka' }
    ```

### Latihan 5: Metode yang tidak didukung

Buat jalur `/api/info` yang hanya menerima `GET` (jawaban `{ aplikasi: "Toko API", versi: 1 }`). Metode lain dijawab `405` dengan header `Allow: GET`. Cetak status dan, untuk yang `405`, nilai header `Allow`.

??? success "Contoh jawaban"
    ```javascript
    import { createServer } from "node:http";
    import { dengarkan } from "./bantu-http.js";

    const app = await dengarkan(
      createServer((req, res) => {
        if (req.method !== "GET") {
          res.writeHead(405, { Allow: "GET" });
          res.end();
          return;
        }
        res.writeHead(200, { "Content-Type": "application/json" });
        res.end(JSON.stringify({ aplikasi: "Toko API", versi: 1 }));
      }),
    );

    const a = await fetch(app.url + "/api/info");
    console.log(a.status, await a.json());

    const b = await fetch(app.url + "/api/info", { method: "POST" });
    console.log(b.status, b.headers.get("allow"));

    await app.tutup();
    ```

    Output:
    ```text
    200 { aplikasi: 'Toko API', versi: 1 }
    405 GET
    ```

### Latihan 6: Status 204

Buat `DELETE /api/item/1` yang menjawab `204 No Content` tanpa body. Cetak status dan isi body (sebagai JSON string agar kelihatan jika kosong).

??? success "Contoh jawaban"
    ```javascript
    import { createServer } from "node:http";
    import { dengarkan } from "./bantu-http.js";

    const app = await dengarkan(
      createServer((req, res) => {
        if (req.method === "DELETE" && req.url === "/api/item/1") {
          res.writeHead(204);
          res.end();
          return;
        }
        res.writeHead(404);
        res.end();
      }),
    );

    const r = await fetch(app.url + "/api/item/1", { method: "DELETE" });
    console.log(r.status, JSON.stringify(await r.text()));

    await app.tutup();
    ```

    Output: `204 ""`. Status `204` berarti berhasil tanpa isi, umum dipakai setelah penghapusan.

### Latihan 7: Batas ukuran body

Buat server `POST` yang menolak body lebih dari 20 byte dengan `413` dan `{ galat: "Body terlalu besar" }`, dan menjawab `200` dengan `{ diterima: <jumlah byte> }` jika tidak. Uji dengan body `"halo"` dan body sepanjang 50 karakter.

??? success "Contoh jawaban"
    ```javascript
    import { createServer } from "node:http";
    import { dengarkan } from "./bantu-http.js";

    const BATAS = 20;

    const app = await dengarkan(
      createServer(async (req, res) => {
        let jumlah = 0;
        for await (const potongan of req) {
          jumlah += potongan.length;
          if (jumlah > BATAS) {
            res.writeHead(413, { "Content-Type": "application/json" });
            res.end(JSON.stringify({ galat: "Body terlalu besar" }));
            return;
          }
        }
        res.writeHead(200, { "Content-Type": "application/json" });
        res.end(JSON.stringify({ diterima: jumlah }));
      }),
    );

    async function kirimBody(isi) {
      const r = await fetch(app.url, { method: "POST", body: isi });
      console.log(r.status, await r.json());
    }

    await kirimBody("halo");
    await kirimBody("x".repeat(50));

    await app.tutup();
    ```

    Output:
    ```text
    200 { diterima: 4 }
    413 { galat: 'Body terlalu besar' }
    ```

    Server memeriksa ukuran sambil membaca, sehingga body raksasa ditolak tanpa dimuat seluruhnya ke memori.

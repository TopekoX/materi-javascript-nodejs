## Tujuan pembelajaran

Setelah mempelajari bab ini, kamu dapat:

- Menjelaskan apa itu Express dan mengapa orang memakainya dibanding `node:http` saja.
- Memasang Express dengan versi yang dikunci dan membuat aplikasi pertama.
- Membuat *route* untuk berbagai metode HTTP, dan membaca parameter jalur, query, serta body JSON.
- Mengirim respons dengan `res.send`, `res.json`, `res.status`, dan `res.sendStatus`.
- Menjelaskan *middleware* dan menulis middleware sendiri, termasuk memahami pentingnya urutan.
- Menangani error dengan *error-handling middleware*.
- Memecah aplikasi dengan `express.Router`.
- Melayani berkas statis dengan `express.static`.

Bab ini memakai bab 16 (npm), bab 20 (HTTP), serta async dan class (bab 12 dan 13). Karena memakai paket dari luar, proyeknya dibuat di folder baru bernama `belajar-express`.

## Pendahuluan: dari tukang batu ke kerangka bangunan

Di bab 20 kamu membuat server toko dengan `node:http`. Hasilnya berfungsi, tetapi lihat berapa banyak pekerjaan manual: memecah URL, mencocokkan jalur dengan rangkaian `if`, membaca body dari stream, menulis header, mengubah objek menjadi JSON, dan membungkus semuanya dengan `try...catch`. Setiap server baru mengulang pekerjaan yang sama.

**Express** adalah kerangka web (*web framework*) minimalis yang membungkus pekerjaan-pekerjaan berulang itu. Ia tidak mengganti Node.js; di dalamnya Express tetap memakai `node:http`. Yang ia tambahkan:

| Pekerjaan | `node:http` | Express |
|---|---|---|
| Mencocokkan jalur dan metode | Rangkaian `if` manual | `app.get("/api/produk/:id", ...)` |
| Mengambil bagian jalur | Memecah `pathname` sendiri | `req.params.id` |
| Mengambil query | `new URL(...).searchParams` | `req.query` |
| Membaca body JSON | `for await` dan `JSON.parse` | `express.json()` lalu `req.body` |
| Mengirim JSON | `writeHead` + `JSON.stringify` + `end` | `res.json(data)` |
| Pekerjaan bersama (log, autentikasi) | Ditulis di satu fungsi besar | **Middleware** yang bisa disusun |
| Menangani error | `try...catch` di mana-mana | Satu *error handler* terpusat |

Express adalah kerangka web Node.js yang paling banyak dipakai, sehingga kamu akan sering menjumpainya. Ada alternatif seperti Fastify, Koa, dan Hono, dengan gagasan yang mirip.

## Menyiapkan proyek

Buat folder proyek baru, lalu pasang Express dengan versi tertentu. Versi dikunci agar contoh di bab ini berperilaku sama seperti milikmu (bab 16). Bab ini ditulis dengan **Express 5.3.0**.

```bash
mkdir belajar-express
cd belajar-express
npm init -y
npm pkg set type=module
npm pkg set private=true --json
npm install express@5.3.0
```

Output (contoh):

```text
added 65 packages, and audited 66 packages in 2s

30 packages are looking for funding
  run `npm fund` for details

found 0 vulnerabilities
```

Express membawa 28 paket langsung dan puluhan paket turunan, itu sebabnya 65 paket terpasang. Pastikan `package.json` mencatatnya:

```bash
npm pkg get dependencies
```

Output:

```text
{
  "express": "^5.3.0"
}
```

Karena folder ini bertipe `module`, kita memakai `import express from "express"`. Salin juga fungsi bantu `bantu-http.js` dari bab 20 ke folder ini; kita memakainya untuk menguji server tanpa port tetap:

```javascript title="belajar-express/bantu-http.js"
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

## Aplikasi Express pertama

Simpan sebagai `halo.js`:

```javascript title="belajar-express/halo.js"
import express from "express";

const app = express();
const PORT = Number(process.env.PORT) || 3000;

app.get("/", (req, res) => {
  res.send("Halo dari Express!");
});

app.listen(PORT, () => {
  console.log(`Server berjalan di http://localhost:${PORT}`);
});
```

```bash
# berjalan terus, hentikan dengan Ctrl + C
node halo.js
```

Output:

```text
Server berjalan di http://localhost:3000
```

Buka `http://localhost:3000` di browser dan kamu akan melihat `Halo dari Express!`. Hentikan dengan `Ctrl + C`.

| Kode | Artinya |
|---|---|
| `express()` | Membuat aplikasi Express (`app`) |
| `app.get("/", fungsi)` | Mendaftarkan *route*: jika ada permintaan `GET` ke jalur `/`, jalankan fungsi itu |
| `(req, res) => {...}` | *Handler*. `req` dan `res` adalah objek permintaan dan respons, yang diperkaya Express |
| `res.send("...")` | Mengirim respons dan menyelesaikannya (setara `writeHead` + `end`) |
| `app.listen(PORT, ...)` | Mulai mendengarkan port. Di dalamnya, Express memanggil `http.createServer(app)` |

Fakta menarik: `app` sendiri adalah **sebuah fungsi** `(req, res)`, tepat seperti handler yang diberikan ke `createServer` di bab 20. Itu sebabnya `createServer(app)` bisa dipakai, dan kita memanfaatkannya untuk pengujian di bawah.

## Routing

*Routing* adalah memilih handler berdasarkan **metode** dan **jalur**. Express menyediakan satu method untuk setiap metode HTTP: `app.get`, `app.post`, `app.put`, `app.patch`, `app.delete`, dan `app.all` (semua metode). Simpan sebagai `routing.js`:

```javascript title="belajar-express/routing.js"
import express from "express";
import { createServer } from "node:http";
import { dengarkan } from "./bantu-http.js";

const app = express();

app.get("/", (req, res) => {
  res.send("Halo Express");
});

app.get("/api/halo/:nama", (req, res) => {
  res.json({ pesan: `Halo, ${req.params.nama}!`, query: req.query });
});

app.post("/api/echo", (req, res) => {
  res.json({ tipeBody: typeof req.body });
});

app.get("/status", (req, res) => {
  res.sendStatus(204);
});

app.get("/dibuat", (req, res) => {
  res.status(201).set("X-Contoh", "ya").json({ ok: true });
});

const server = await dengarkan(createServer(app));

async function panggil(metode, jalur, opsi = {}) {
  const respons = await fetch(server.url + jalur, { method: metode, ...opsi });
  const teks = await respons.text();
  console.log(metode, jalur, "->", respons.status, respons.headers.get("content-type"), teks.slice(0, 60));
}

await panggil("GET", "/");
await panggil("GET", "/api/halo/Budi?x=1&y=a&y=b");
await panggil("POST", "/api/echo", {
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({ a: 1 }),
});
await panggil("GET", "/status");
await panggil("GET", "/dibuat");

await server.tutup();
```

```bash
node routing.js
```

Output:

```text
GET / -> 200 text/html; charset=utf-8 Halo Express
GET /api/halo/Budi?x=1&y=a&y=b -> 200 application/json; charset=utf-8 {"pesan":"Halo, Budi!","query":{"x":"1","y":["a","b"]}}
POST /api/echo -> 200 application/json; charset=utf-8 {"tipeBody":"undefined"}
GET /status -> 204 null 
GET /dibuat -> 201 application/json; charset=utf-8 {"ok":true}
```

### Membaca permintaan

| Properti | Isi | Contoh |
|---|---|---|
| `req.params` | Bagian jalur yang diberi nama dengan `:` | Jalur `/api/halo/:nama` dan permintaan `/api/halo/Budi` menghasilkan `req.params.nama === "Budi"` |
| `req.query` | Query string sebagai objek | `?x=1&y=a&y=b` menjadi `{ x: "1", y: ["a", "b"] }` |
| `req.body` | Body yang sudah diurai (butuh middleware, lihat berikut) | `{ a: 1 }` |
| `req.get("nama-header")` | Nilai sebuah header | `req.get("content-type")` |
| `req.method`, `req.path`, `req.originalUrl` | Metode, jalur tanpa query, dan alamat lengkap | `GET`, `/api/halo/Budi`, `/api/halo/Budi?x=1` |

Hal yang harus diingat: semua nilai di `req.params` dan `req.query` adalah **teks** (atau array teks jika diulang). Angka harus diubah dengan `Number(...)`. Pada contoh di atas, `req.body` bernilai `undefined` karena belum ada middleware yang membaca body.

### Mengirim respons

| Method | Fungsi |
|---|---|
| `res.send(data)` | Mengirim teks, HTML, atau Buffer. Mengatur `Content-Type` secara otomatis |
| `res.json(objek)` | Mengubah menjadi JSON, mengatur `Content-Type: application/json`, lalu mengirim |
| `res.status(kode)` | Mengatur kode status dan mengembalikan `res`, sehingga bisa dirangkai: `res.status(201).json(...)` |
| `res.sendStatus(kode)` | Mengirim status beserta teks standarnya (`204` dikirim tanpa isi) |
| `res.set(nama, nilai)` | Mengatur header |
| `res.location(url)`, `res.redirect(url)` | Mengatur header `Location`, atau mengalihkan klien |

Bandingkan dengan bab 20: `res.status(201).set("X-Contoh", "ya").json({ ok: true })` menggantikan `writeHead`, `JSON.stringify`, dan `end`.

!!! tip "Urutan route penting"
    Express mencocokkan route dari atas ke bawah dan memakai yang **pertama** cocok. Letakkan jalur yang spesifik (`/produk/baru`) sebelum jalur berparameter (`/produk/:id`), kalau tidak `"baru"` akan dianggap sebagai `:id`.

## Middleware

**Middleware** adalah fungsi yang berdiri di antara permintaan yang masuk dan handler akhir. Bayangkan satpam di pintu masuk gedung: setiap pengunjung melewatinya dulu, ia boleh mencatat, memeriksa, mengubah, atau menolak, lalu meneruskan pengunjung ke dalam.

Bentuknya selalu `(req, res, next)`:

- Jika middleware **meneruskan** permintaan, ia memanggil `next()`.
- Jika middleware **menjawab sendiri** (misalnya menolak dengan 401), ia memanggil `res.json(...)` dan **tidak** memanggil `next()`.

Middleware dipasang dengan `app.use(...)` (berlaku untuk semua route di bawahnya), atau langsung di route tertentu sebagai argumen tengah. Simpan sebagai `middleware.js`:

```javascript title="belajar-express/middleware.js"
import express from "express";
import { createServer } from "node:http";
import { dengarkan } from "./bantu-http.js";

const app = express();
const catatan = [];

app.use((req, res, next) => {
  res.on("finish", () => catatan.push(`${req.method} ${req.originalUrl} -> ${res.statusCode}`));
  next();
});

app.use(express.json());

function periksaKunci(req, res, next) {
  if (req.get("x-kunci") !== "rahasia") {
    res.status(401).json({ galat: "kunci salah" });
    return;
  }
  next();
}

app.get("/publik", (req, res) => {
  res.json({ pesan: "boleh dilihat siapa saja" });
});

app.get("/rahasia", periksaKunci, (req, res) => {
  res.json({ pesan: "hanya untuk pemegang kunci" });
});

app.post("/api/echo", (req, res) => {
  res.json({ diterima: req.body });
});

const server = await dengarkan(createServer(app));

async function panggil(metode, jalur, opsi = {}) {
  const respons = await fetch(server.url + jalur, { method: metode, ...opsi });
  console.log(metode, jalur, "->", respons.status, await respons.text());
}

await panggil("GET", "/publik");
await panggil("GET", "/rahasia");
await panggil("GET", "/rahasia", { headers: { "x-kunci": "rahasia" } });
await panggil("POST", "/api/echo", {
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({ nama: "Budi" }),
});

console.log(catatan);
await server.tutup();
```

```bash
node middleware.js
```

Output:

```text
GET /publik -> 200 {"pesan":"boleh dilihat siapa saja"}
GET /rahasia -> 401 {"galat":"kunci salah"}
GET /rahasia -> 200 {"pesan":"hanya untuk pemegang kunci"}
POST /api/echo -> 200 {"diterima":{"nama":"Budi"}}
[
  'GET /publik -> 200',
  'GET /rahasia -> 401',
  'GET /rahasia -> 200',
  'POST /api/echo -> 200'
]
```

Tiga jenis middleware muncul di sini:

1. **Middleware pencatat** (`app.use(fungsi)`): berlaku untuk **semua** permintaan. Ia mendaftarkan pendengar `finish` pada `res` (event dari bab 19) untuk mencatat status setelah respons selesai, lalu memanggil `next()`.
2. **`express.json()`**: middleware bawaan yang membaca body berformat JSON dan menaruh hasilnya di `req.body`. Tanpa ini, `req.body` adalah `undefined` (lihat `routing.js`). Express juga menyediakan `express.urlencoded()` untuk data formulir HTML dan `express.static()` untuk berkas.
3. **Middleware per-route** (`periksaKunci`): hanya dipasang di route `/rahasia`. Jika header `x-kunci` salah, ia menjawab 401 dan **berhenti** (tanpa `next()`), sehingga handler `/rahasia` tidak pernah berjalan.

Karena middleware berjalan berurutan, **urutan pemasangan** menentukan hasil. Kesalahan paling umum adalah memasang `express.json()` setelah route yang membutuhkannya. Simpan sebagai `urutan-salah.js`:

```javascript title="belajar-express/urutan-salah.js"
import express from "express";
import { createServer } from "node:http";
import { dengarkan } from "./bantu-http.js";

const app = express();

app.post("/api/echo", (req, res) => {
  res.json({ tipeBody: typeof req.body });
});

app.use(express.json());

const server = await dengarkan(createServer(app));
const respons = await fetch(server.url + "/api/echo", {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({ a: 1 }),
});
console.log(await respons.json());
await server.tutup();
```

```bash
node urutan-salah.js
```

Output:

```text
{ tipeBody: 'undefined' }
```

Body dikirim dengan benar, tetapi route `/api/echo` sudah dijalankan **sebelum** `express.json()` sempat membacanya. **Perbaikan:** pasang `app.use(express.json())` di atas semua route yang memakai `req.body`.

## Penanganan error

Di Express, error ditangani oleh **error-handling middleware**: middleware khusus dengan **empat** parameter `(error, req, res, next)`. Aturannya:

- Error yang dilempar (`throw`) di handler, baik biasa maupun `async`, otomatis diteruskan ke error handler. (Ini salah satu perbaikan Express 5: di Express 4, error dari handler `async` harus diteruskan manual.)
- Error handler dipasang **setelah** semua route.
- Sebelum error handler, pasang middleware "catch-all" untuk jalur yang tidak dikenal (404).

Tanpa error handler buatan sendiri, Express menjawab dengan halaman HTML bawaan (`Cannot GET /tidak-ada` untuk 404, dan stack trace untuk 500). Itu kurang cocok untuk API yang seharusnya menjawab JSON. Kita akan menulisnya di proyek toko berikut.

## Contoh praktis: API toko dengan Express

Kita bangun ulang API toko dari bab 20, sekarang dengan Express, `Router`, middleware, dan error handler. Buat folder `toko` di dalam `belajar-express`.

Struktur akhirnya:

```text
belajar-express/
  bantu-http.js
  toko/
    galat.js          error khusus yang membawa status
    produk.js         Router untuk /api/produk
    app.js            merakit aplikasi (middleware, router, error handler)
    server.js         menjalankan di port tertentu
    uji.js            skrip uji
    publik/index.html berkas statis
```

Error khusus dari bab 20 ditaruh di berkasnya sendiri:

```javascript title="belajar-express/toko/galat.js"
export class GalatHttp extends Error {
  constructor(status, pesan) {
    super(pesan);
    this.status = status;
  }
}
```

### Router: mengelompokkan route

`express.Router()` membuat "aplikasi mini" yang hanya berisi route. Router lalu dipasang di jalur tertentu, sehingga route di dalamnya tidak perlu menulis awalan `/api/produk` berulang kali.

```javascript title="belajar-express/toko/produk.js"
import { Router } from "express";
import { GalatHttp } from "./galat.js";

const produk = [
  { id: 1, nama: "Pensil", harga: 2000 },
  { id: 2, nama: "Buku", harga: 8000 },
  { id: 3, nama: "Pulpen", harga: 3000 },
];

const metodeTidakDiizinkan = (izin) => (req, res) => {
  res.set("Allow", izin).status(405).json({ galat: "Metode tidak diizinkan" });
};

export const rute = Router();

rute
  .route("/")
  .get((req, res) => {
    const { harga_maks: hargaMaks } = req.query;
    if (hargaMaks === undefined) {
      return res.json(produk);
    }
    if (Number.isNaN(Number(hargaMaks))) {
      throw new GalatHttp(400, "harga_maks harus angka");
    }
    res.json(produk.filter((p) => p.harga <= Number(hargaMaks)));
  })
  .post((req, res) => {
    if (req.body === undefined) {
      throw new GalatHttp(400, "Body kosong");
    }
    const { nama, harga } = req.body;
    if (typeof nama !== "string" || nama.trim() === "") {
      throw new GalatHttp(400, "nama wajib diisi");
    }
    if (typeof harga !== "number" || harga <= 0) {
      throw new GalatHttp(400, "harga harus angka lebih dari 0");
    }
    const baru = { id: produk.at(-1).id + 1, nama, harga };
    produk.push(baru);
    res.status(201).location(`/api/produk/${baru.id}`).json(baru);
  })
  .all(metodeTidakDiizinkan("GET, POST"));

rute
  .route("/:id")
  .get((req, res) => {
    const id = Number(req.params.id);
    if (!Number.isInteger(id)) {
      throw new GalatHttp(400, "id harus bilangan bulat");
    }
    const ditemukan = produk.find((p) => p.id === id);
    if (!ditemukan) {
      throw new GalatHttp(404, `Produk ${id} tidak ditemukan`);
    }
    res.json(ditemukan);
  })
  .all(metodeTidakDiizinkan("GET"));
```

Penjelasan:

- `rute.route("/")` membuat rantai untuk satu jalur: `.get(...)`, `.post(...)`, dan `.all(...)`. `.all` menangkap **metode lain** yang belum dikenali, yang kita pakai untuk menjawab `405` plus header `Allow`.
- Handler cukup `throw new GalatHttp(...)` untuk menolak. Express meneruskannya ke error handler. Tidak ada lagi `try...catch` di setiap handler.
- `const { harga_maks: hargaMaks } = req.query` memakai destructuring dengan nama baru (bab 9).
- `res.status(201).location(...).json(baru)` merangkai tiga hal dalam satu baris.

### Merakit aplikasi

```javascript title="belajar-express/toko/app.js"
import express from "express";
import { join } from "node:path";
import { rute as produk } from "./produk.js";
import { GalatHttp } from "./galat.js";

export function buatApp() {
  const app = express();
  app.disable("x-powered-by");

  app.use(express.json());
  app.use("/halaman", express.static(join(import.meta.dirname, "publik")));

  app.get("/", (req, res) => {
    res.send("Selamat datang di Toko API");
  });

  app.use("/api/produk", produk);

  app.use((req, res) => {
    throw new GalatHttp(404, "Jalur tidak ditemukan");
  });

  app.use((error, req, res, next) => {
    if (error.type === "entity.parse.failed") {
      return res.status(400).json({ galat: "JSON tidak valid" });
    }
    if (error instanceof GalatHttp) {
      return res.status(error.status).json({ galat: error.message });
    }
    console.error(error);
    res.status(500).json({ galat: "Kesalahan server" });
  });

  return app;
}
```

Urutan pemasangan di file ini sangat disengaja:

| Urutan | Pemasangan | Alasan |
|---|---|---|
| 1 | `app.disable("x-powered-by")` | Mematikan header `X-Powered-By: Express`, yang tidak perlu dibocorkan ke klien |
| 2 | `express.json()` | Harus sebelum route yang membaca `req.body` |
| 3 | `express.static(...)` di `/halaman` | Melayani berkas dari folder `publik`. `import.meta.dirname` (bab 17) membuat alamatnya tidak bergantung folder kerja |
| 4 | Route biasa dan `app.use("/api/produk", produk)` | Memasang Router di bawah awalan jalur |
| 5 | Middleware 404 | Dijalankan hanya jika **tidak ada** route di atasnya yang menjawab |
| 6 | Error handler (4 parameter) | Paling akhir. Mengubah setiap error menjadi JSON |

Pada error handler: `error.type === "entity.parse.failed"` adalah tanda yang diberikan `express.json()` ketika body bukan JSON yang valid. `GalatHttp` dijawab sesuai statusnya, dan error tak terduga dijawab `500` tanpa membocorkan detail ke klien (detail tetap dicatat dengan `console.error`).

Berkas statis yang akan dilayani:

```html title="belajar-express/toko/publik/index.html"
<h1>Halo dari Toko</h1>
```

Dan pintu masuk yang menjalankan server:

```javascript title="belajar-express/toko/server.js"
import { buatApp } from "./app.js";

const PORT = Number(process.env.PORT) || 3000;

const server = buatApp().listen(PORT, (error) => {
  if (error) {
    console.error("Gagal memulai server:", error.message);
    process.exitCode = 1;
    return;
  }
  console.log(`Server berjalan di http://localhost:${PORT}`);
});

process.on("SIGINT", () => {
  console.log("\nMenutup server...");
  server.close(() => console.log("Server ditutup"));
});
```

Pada Express 5, callback `listen` menerima `error` jika server gagal dimulai (misalnya port sudah dipakai), sehingga kita bisa melaporkannya dengan rapi dan memberi exit code 1 (bab 18).

### Menguji semuanya

Skrip uji ini sama dengan yang kita pakai di bab 20, hanya servernya yang berbeda:

```javascript title="belajar-express/toko/uji.js"
import { createServer } from "node:http";
import { buatApp } from "./app.js";
import { dengarkan } from "../bantu-http.js";

const server = await dengarkan(createServer(buatApp()));

async function panggil(metode, jalur, body) {
  const respons = await fetch(server.url + jalur, {
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

const rusak = await fetch(server.url + "/api/produk", {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: "{rusak",
});
console.log("POST rusak ->", rusak.status, await rusak.json());

const kosong = await fetch(server.url + "/api/produk", { method: "POST" });
console.log("POST kosong ->", kosong.status, await kosong.json());

const halaman = await fetch(server.url + "/halaman/");
console.log("GET /halaman/ ->", halaman.status, halaman.headers.get("content-type"), JSON.stringify((await halaman.text()).trim()));
console.log("X-Powered-By:", halaman.headers.get("x-powered-by"));

await server.tutup();
```

```bash
node toko/uji.js
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
GET /halaman/ -> 200 text/html; charset=utf-8 "<h1>Halo dari Toko</h1>"
X-Powered-By: null
```

Seluruh 16 baris pertama **identik dengan keluaran bab 20**: perilaku API tidak berubah, hanya cara menuliskannya yang lebih terstruktur. Tiga baris terakhir menunjukkan hal baru: berkas statis dilayani di `/halaman/`, dan header `X-Powered-By` tidak lagi dikirim.

Untuk menjalankan server sungguhan, gunakan:

```bash
# berjalan terus, hentikan dengan Ctrl + C
node toko/server.js
```

Output:

```text
Server berjalan di http://localhost:3000
```

Buka `http://localhost:3000/api/produk` di browser. Setelah `Ctrl + C`:

```text
Menutup server...
Server ditutup
```

## Kesalahan umum

### 1. Memasang `express.json()` setelah route

Sudah terlihat pada `urutan-salah.js`: `req.body` tetap `undefined`. Urutan pemasangan middleware menentukan hasil.

### 2. Lupa memanggil `next()`

Middleware yang tidak menjawab dan tidak memanggil `next()` membuat permintaan menggantung selamanya. Simpan sebagai `lupa-next.js`:

```javascript title="belajar-express/lupa-next.js"
import express from "express";
import { createServer } from "node:http";
import { dengarkan } from "./bantu-http.js";

const app = express();

app.use((req, res, next) => {
  console.log("middleware berjalan");
});

app.get("/", (req, res) => {
  res.send("tidak pernah sampai");
});

const server = await dengarkan(createServer(app));

try {
  await fetch(server.url, { signal: AbortSignal.timeout(300) });
} catch (e) {
  console.log(e.name);
}
await server.tutup();
```

```bash
node lupa-next.js
```

Output:

```text
middleware berjalan
TimeoutError
```

**Perbaikan:** setiap middleware harus berakhir dengan menjawab (`res.send`, `res.json`, ...) **atau** memanggil `next()`. Jangan keduanya sekaligus.

### 3. Menjawab dua kali

Memanggil `res.json(...)` lalu melanjutkan eksekusi sehingga ada `res.json(...)` kedua menghasilkan `ERR_HTTP_HEADERS_SENT` (bab 20). Gunakan `return res.json(...)` agar fungsi berhenti.

### 4. Error handler dengan parameter yang kurang

Express mengenali error handler hanya jika fungsinya punya **tepat empat** parameter `(error, req, res, next)`. Dengan tiga parameter, fungsi itu diperlakukan sebagai middleware biasa dan tidak pernah menerima error.

### 5. Error handler atau middleware 404 di tempat yang salah

Middleware 404 dan error handler harus dipasang **setelah** semua route. Jika dipasang di atas, ia akan menjawab 404 sebelum route sempat dicocokkan.

### 6. Menganggap `req.params` dan `req.query` sudah berupa angka

Semuanya teks. `req.params.id + 1` untuk `id` bernilai `"5"` menghasilkan `"51"`. Ubah dengan `Number(...)` dan validasi hasilnya (`Number.isInteger`, `Number.isNaN`).

### 7. Mempercayai `req.body` tanpa validasi

Klien bebas mengirim apa saja: tipe salah, field hilang, atau tanpa body sama sekali (`req.body` jadi `undefined` pada Express 5). Selalu periksa dulu seperti pada `POST /api/produk`.

### 8. Menulis sintaks Express 4 di Express 5

Bab ini memakai Express 5. Beberapa hal berubah dari versi 4: jalur wildcard `*` harus diberi nama (`/file/*splat`), parameter opsional ditulis `{/:bagian}`, dan `req.body` bernilai `undefined` (bukan `{}`) jika tidak ada body. Sebagian tutorial lama di internet masih memakai sintaks Express 4. Periksa versi Express yang kamu pakai (`npm ls express`).

### 9. Membocorkan detail error ke klien

Mengirim `error.message` atau stack trace dari error tak terduga memberi petunjuk berharga kepada penyerang. Seperti pada `app.js`: catat detailnya di server (`console.error`), dan jawab pesan umum `Kesalahan server`.

### 10. Mengira Express sudah aman

Express sengaja minimalis. Hal-hal seperti membatasi laju permintaan, header keamanan, dan CORS (aturan akses lintas-situs untuk browser) belum diurus. Untuk aplikasi sungguhan, tambahkan middleware yang sesuai (misalnya paket `helmet` dan `cors`) dan pelajari dasar keamanan web.

## Ringkasan

- **Express** adalah kerangka web minimalis di atas `node:http`: routing, middleware, dan helper respons. Pasang dengan `npm install express@versi`; bab ini memakai Express 5.
- `app.get/post/put/patch/delete/all(jalur, handler)` mendaftarkan route. Route dicocokkan dari atas ke bawah.
- `req.params` (bagian jalur), `req.query` (query), dan `req.body` (butuh `express.json()`) selalu berupa teks atau data mentah dari klien: validasi dan ubah tipenya.
- `res.json`, `res.send`, `res.status(...)`, `res.sendStatus`, dan `res.set` mengirim respons. `res.status(201).json(...)` bisa dirangkai.
- **Middleware** adalah fungsi `(req, res, next)`. Panggil `next()` untuk meneruskan, atau jawab sendiri untuk menghentikan. **Urutan pemasangan** menentukan perilaku.
- Middleware bawaan: `express.json()`, `express.urlencoded()`, `express.static()`.
- **Error handler** memiliki empat parameter `(error, req, res, next)`, dipasang paling akhir. Error yang dilempar di handler (termasuk `async`) otomatis diteruskan ke sana pada Express 5.
- `express.Router()` memecah aplikasi menjadi modul route yang dipasang di awalan jalur dengan `app.use("/awalan", router)`.
- Pisahkan perakitan aplikasi (`buatApp`) dari bagian yang mendengarkan port agar mudah diuji.

## Latihan

Kerjakan sendiri dulu sebelum melihat jawaban. Latihan dijalankan dari folder `belajar-express` dan memakai `bantu-http.js`.

### Latihan 1: Urutan route

Apa yang tercetak untuk permintaan `/a/spesial`, dan bagaimana cara agar route khususnya yang dipakai?

```javascript
import express from "express";
import { createServer } from "node:http";
import { dengarkan } from "./bantu-http.js";

const app = express();
app.get("/a/:x", (req, res) => res.send(`param:${req.params.x}`));
app.get("/a/spesial", (req, res) => res.send("khusus"));

const server = await dengarkan(createServer(app));
console.log(await (await fetch(server.url + "/a/spesial")).text());
await server.tutup();
```

??? success "Jawaban"
    Output: `param:spesial`. Route `/a/:x` terdaftar lebih dulu dan cocok untuk `spesial`, sehingga route khusus tidak pernah tercapai. Perbaikannya, daftarkan `/a/spesial` **sebelum** `/a/:x`.

### Latihan 2: Perbaiki middleware yang menggantung

Permintaan ini selalu menunggu sampai timeout. Perbaiki agar mencetak `halo`:

```javascript
import express from "express";
import { createServer } from "node:http";
import { dengarkan } from "./bantu-http.js";

const app = express();
app.use((req, res, next) => {
  req.waktu = "sekarang";
});
app.get("/", (req, res) => res.send("halo"));

const server = await dengarkan(createServer(app));
try {
  console.log(await (await fetch(server.url, { signal: AbortSignal.timeout(300) })).text());
} catch (e) {
  console.log(e.name);
}
await server.tutup();
```

??? success "Jawaban"
    Middleware harus memanggil `next()`:
    ```javascript
    import express from "express";
    import { createServer } from "node:http";
    import { dengarkan } from "./bantu-http.js";

    const app = express();
    app.use((req, res, next) => {
      req.waktu = "sekarang";
      next();
    });
    app.get("/", (req, res) => res.send("halo"));

    const server = await dengarkan(createServer(app));
    console.log(await (await fetch(server.url, { signal: AbortSignal.timeout(300) })).text());
    await server.tutup();
    ```

    Output: `halo`

### Latihan 3: Penjumlahan lewat query

Buat route `GET /api/jumlah?a=2&b=3` yang menjawab `{ hasil: 5 }`. Jika `a` atau `b` bukan angka, jawab `400` dengan `{ galat: "a dan b harus angka" }`. Uji dengan `a=2&b=3` dan `a=dua&b=3`.

??? success "Contoh jawaban"
    ```javascript
    import express from "express";
    import { createServer } from "node:http";
    import { dengarkan } from "./bantu-http.js";

    const app = express();

    app.get("/api/jumlah", (req, res) => {
      const a = Number(req.query.a);
      const b = Number(req.query.b);
      if (Number.isNaN(a) || Number.isNaN(b)) {
        return res.status(400).json({ galat: "a dan b harus angka" });
      }
      res.json({ hasil: a + b });
    });

    const server = await dengarkan(createServer(app));

    for (const kueri of ["a=2&b=3", "a=dua&b=3"]) {
      const r = await fetch(`${server.url}/api/jumlah?${kueri}`);
      console.log(r.status, await r.json());
    }

    await server.tutup();
    ```

    Output:
    ```text
    200 { hasil: 5 }
    400 { galat: 'a dan b harus angka' }
    ```

### Latihan 4: Middleware kunci rahasia

Buat middleware `periksaKunci` yang menjawab `401` dengan `{ galat: "tanpa izin" }` jika header `x-kunci` bukan `abc`. Pasang hanya pada `GET /rahasia`. Uji tanpa header dan dengan header yang benar.

??? success "Contoh jawaban"
    ```javascript
    import express from "express";
    import { createServer } from "node:http";
    import { dengarkan } from "./bantu-http.js";

    const app = express();

    function periksaKunci(req, res, next) {
      if (req.get("x-kunci") !== "abc") {
        return res.status(401).json({ galat: "tanpa izin" });
      }
      next();
    }

    app.get("/rahasia", periksaKunci, (req, res) => res.json({ isi: "data penting" }));

    const server = await dengarkan(createServer(app));

    const a = await fetch(server.url + "/rahasia");
    console.log(a.status, await a.json());
    const b = await fetch(server.url + "/rahasia", { headers: { "x-kunci": "abc" } });
    console.log(b.status, await b.json());

    await server.tutup();
    ```

    Output:
    ```text
    401 { galat: 'tanpa izin' }
    200 { isi: 'data penting' }
    ```

### Latihan 5: Router untuk catatan

Buat `Router` untuk `/api/catatan` dengan `GET /` (daftar, awalnya kosong) dan `POST /` (menambah catatan dari body `{ "teks": "..." }`, menjawab `201` dengan catatan berisi `id` dan `teks`). Pasang `express.json()`. Tambahkan dua catatan, lalu ambil daftarnya.

??? success "Contoh jawaban"
    ```javascript
    import express, { Router } from "express";
    import { createServer } from "node:http";
    import { dengarkan } from "./bantu-http.js";

    const catatan = [];
    const rute = Router();

    rute.get("/", (req, res) => res.json(catatan));
    rute.post("/", (req, res) => {
      const baru = { id: catatan.length + 1, teks: req.body.teks };
      catatan.push(baru);
      res.status(201).json(baru);
    });

    const app = express();
    app.use(express.json());
    app.use("/api/catatan", rute);

    const server = await dengarkan(createServer(app));

    for (const teks of ["beli buku", "belajar Express"]) {
      const r = await fetch(server.url + "/api/catatan", {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({ teks }),
      });
      console.log(r.status, await r.json());
    }
    console.log(await (await fetch(server.url + "/api/catatan")).json());

    await server.tutup();
    ```

    Output:
    ```text
    201 { id: 1, teks: 'beli buku' }
    201 { id: 2, teks: 'belajar Express' }
    [ { id: 1, teks: 'beli buku' }, { id: 2, teks: 'belajar Express' } ]
    ```

### Latihan 6: Error handler yang aman

Buat route `GET /meledak` yang melempar `new Error("rahasia internal")`. Pasang error handler yang menjawab `500` dengan `{ galat: "Kesalahan server" }` tanpa membocorkan pesan aslinya, dan mencatat pesan aslinya dengan `console.error`. Cetak status dan body klien.

??? success "Contoh jawaban"
    ```javascript
    import express from "express";
    import { createServer } from "node:http";
    import { dengarkan } from "./bantu-http.js";

    const app = express();

    app.get("/meledak", () => {
      throw new Error("rahasia internal");
    });

    app.use((error, req, res, next) => {
      console.error("dicatat di server:", error.message);
      res.status(500).json({ galat: "Kesalahan server" });
    });

    const server = await dengarkan(createServer(app));
    const r = await fetch(server.url + "/meledak");
    console.log(r.status, await r.json());
    await server.tutup();
    ```

    Output (baris `dicatat di server` dikirim ke stderr, jadi bisa muncul lebih dulu atau belakangan):
    ```text
    dicatat di server: rahasia internal
    500 { galat: 'Kesalahan server' }
    ```

### Latihan 7: Berkas statis

Buat folder `publik` berisi `halo.txt` dengan isi `Halo statis`, layani di jalur `/berkas` dengan `express.static`, lalu ambil `/berkas/halo.txt` dan cetak isinya.

??? success "Contoh jawaban"
    ```javascript
    import express from "express";
    import { createServer } from "node:http";
    import { mkdir, writeFile } from "node:fs/promises";
    import { join } from "node:path";
    import { dengarkan } from "./bantu-http.js";

    const folder = join(import.meta.dirname, "publik");
    await mkdir(folder, { recursive: true });
    await writeFile(join(folder, "halo.txt"), "Halo statis");

    const app = express();
    app.use("/berkas", express.static(folder));

    const server = await dengarkan(createServer(app));
    const r = await fetch(server.url + "/berkas/halo.txt");
    console.log(r.status, await r.text());
    await server.tutup();
    ```

    Output: `200 Halo statis`. `express.static` melayani semua berkas di folder itu dan menolak upaya keluar dari folder dengan `..`.

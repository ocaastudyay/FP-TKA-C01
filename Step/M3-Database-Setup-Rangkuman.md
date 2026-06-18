# 🗄️ Rangkuman Setup MongoDB — M3 Database Engineer
### Final Project Cloud Computing | `vm-be2`

---

## 📋 Info Penting

| Info | Detail |
|------|--------|
| **VM Target** | `vm-be2` — `azureuser@70.153.145.176` |
| **Internal IP `vm-be2`** | `10.0.0.7` |
| **Database Name** | `orderdb` (sesuai `app.py` line 43 dan struktur dump asli `dump/orderdb/`) |
| **MongoDB Admin User** | `admin` / `FpTka2026!` (kredensial server MongoDB, **bukan** kredensial login aplikasi) |
| **Port** | `27017` |

---

## Langkah 1 — SSH ke VM

```bash
ssh azureuser@70.153.145.176
```

---

## Langkah 2 — Install MongoDB 7.x + Tools

```bash
sudo apt-get update

curl -fsSL https://www.mongodb.org/static/pgp/server-7.0.asc | \
  sudo gpg -o /usr/share/keyrings/mongodb-server-7.0.gpg --dearmor

echo "deb [ arch=amd64,arm64 signed-by=/usr/share/keyrings/mongodb-server-7.0.gpg ] \
  https://repo.mongodb.org/apt/ubuntu jammy/mongodb-org/7.0 multiverse" | \
  sudo tee /etc/apt/sources.list.d/mongodb-org-7.0.list

sudo apt-get update && sudo apt-get install -y mongodb-org mongodb-database-tools

sudo systemctl start mongod
sudo systemctl enable mongod
sudo systemctl status mongod   # harus "Active: active (running)"
```

---

## Langkah 3 — Buat User Admin MongoDB

```bash
mongosh
```

```javascript
use admin

db.createUser({
  user: "admin",
  pwd: "FpTka2026!",
  roles: [{ role: "root", db: "admin" }]
})

db.getUsers()
exit
```

---

## Langkah 4 — Aktifkan Auth & Buka Bind IP

```bash
sudo nano /etc/mongod.conf
```

```yaml
net:
  port: 27017
  bindIp: 0.0.0.0

security:
  authorization: enabled
```

> ⚠️ `security` harus jadi section tersendiri di level paling atas, bukan di bawah `processManagement`.

```bash
sudo systemctl restart mongod
sudo systemctl status mongod
```

---

## Langkah 5 — Restore DB Dump

```bash
git clone https://github.com/fuaddary/fp-tka-26.git
cd fp-tka-26/Resources/DB

mongorestore \
  --uri='mongodb://admin:FpTka2026!@localhost:27017/?authSource=admin' \
  --drop \
  dump/
```

Verifikasi:

```bash
mongosh 'mongodb://admin:FpTka2026!@localhost:27017/orderdb?authSource=admin' \
  --eval '
    print("users:", db.users.countDocuments());
    print("products:", db.products.countDocuments({is_active: true}));
    print("orders:", db.orders.countDocuments());
  '
```

✅ Hasil: `users: 505`, `products: 96`, `orders: 10000`.

---

## Langkah 6 — Buat Index Performa

Dibuat berdasarkan analisis query nyata di `app.py` (bukan cuma tebakan), termasuk satu index tambahan (`orders.status`) setelah ditemukan bahwa `/admin/stats` — query paling berat menurut `locustfile.py` — melakukan `$match: {status: "completed"}` tanpa index pendukung.

```bash
mongosh 'mongodb://admin:FpTka2026!@localhost:27017/orderdb?authSource=admin'
```

```javascript
// ── ORDERS ──────────────────────────────────────
db.orders.createIndex({ created_at: -1 })                    // GET /orders default sort
db.orders.createIndex({ order_id: 1 }, { unique: true })      // GET/PUT /orders/<id>
db.orders.createIndex({ user_id: 1, created_at: -1 })         // GET /orders (user biasa)
db.orders.createIndex({ status: 1 })                          // GET /admin/stats ($match status)

// ── PRODUCTS ─────────────────────────────────────
db.products.createIndex({ is_active: 1, category: 1 })        // GET /products?category=
db.products.createIndex({ is_active: 1, created_at: -1 })     // sort default produk

// ── USERS ────────────────────────────────────────
db.users.createIndex({ email: 1 }, { unique: true })          // POST /auth/login & /register
```

Hasil verifikasi akhir (`getIndexes()`):

| Collection | Index |
|---|---|
| **orders** | `_id_`, `created_at_-1`, `order_id_1`, `user_id_1_created_at_-1`, `status_1` |
| **products** | `_id_`, `is_active_1_category_1`, `is_active_1_created_at_-1` |
| **users** | `_id_`, `email_1` |

---

## Langkah 7 — Verifikasi Akhir

```bash
sudo systemctl status mongod
ss -tlnp | grep 27017

mongosh 'mongodb://admin:FpTka2026!@localhost:27017/orderdb?authSource=admin' \
  --eval "print(db.users.findOne({email: 'admin1@tka.its.ac.id'}).role)"
# → admin ✅
```

> **Catatan penting soal email testing:** Akun admin (`admin1`–`admin5@tka.its.ac.id`, password `Admin@12345`) memang akun resmi dari `generate_dump.py` milik dosen. Sementara email user biasa **tidak** mengikuti pola `userN@example.com` — itu di-generate acak pakai Faker (`fake.unique.email()`). Pola `userN@example.com` di `locustfile.py` itu memang **sengaja dibuat 404** oleh dosen (ada komentar `# akan 404 tapi itu ok`), dipakai untuk simulasi load test, bukan akun asli.

---

## 🔗 Link Koneksi untuk Anggota Tim Lain

| Untuk | URI Koneksi |
|-------|-------------|
| **M4** (dari `vm-be1`) | `mongodb://admin:FpTka2026!@10.0.0.7:27017/orderdb?authSource=admin` |
| **M5 / M2 BE** (di `vm-be2` sendiri) | `mongodb://admin:FpTka2026!@localhost:27017/orderdb?authSource=admin` |
| **Kalau diakses dari luar lewat IP publik** | `mongodb://admin:FpTka2026!@70.153.145.176:27017/orderdb?authSource=admin` |

> ⚠️ DB name adalah `orderdb`, **bukan** `orders_db` (nama lama sebelum di-fix — masih ada sebagai database kosong sisa, boleh di-drop: `mongosh '...orders_db?authSource=admin' --eval 'db.dropDatabase()'`).

---

## ⚙️ Catatan untuk M2 / Backend Engineer (`app.py`)

`app.py` **tidak** memakai `python-dotenv` — env var dibaca langsung dari OS lewat `os.environ.get(...)`, jadi file `.env` saja **tidak otomatis terbaca**. Karena auth sudah aktif, default `MONGO_URI` (`mongodb://localhost:27017/`) akan gagal connect. Wajib set environment variable manual sebelum run:

```bash
export MONGO_URI='mongodb://admin:FpTka2026!@localhost:27017/?authSource=admin'
export JWT_SECRET='ganti-string-rahasia-sendiri'
python app.py
```

Kalau BE jalan di VM lain (bukan `vm-be2`), ganti `localhost` dengan `10.0.0.7`.

---

## 🐛 Troubleshooting yang Pernah Ditemui

| Error | Solusi |
|---|---|
| `event not found` saat pakai `!` di bash | Selalu pakai single quotes `'...'` untuk URI MongoDB |
| `Unrecognized option: processManagement.authorization` | Pastikan `security` jadi section tersendiri di `mongod.conf` |
| `ECONNREFUSED 127.0.0.1:27017` | Cek `sudo systemctl status mongod`, restart kalau perlu |
| `mongorestore: command not found` | `sudo apt-get install -y mongodb-database-tools` |
| `mongorestore` Authentication failed | Pastikan auth sudah enabled **sebelum** restore, dan URI pakai credentials |
| `findOne(...).email` → `TypeError: Cannot read properties of null` | Email test salah asumsi (lihat catatan Langkah 7) — bukan masalah restore |

---

*Rangkuman ini mencakup seluruh proses setup database dari instalasi MongoDB sampai verifikasi akhir untuk peran M3 — Database Engineer, Final Project Cloud Computing.*

# M3 — Database Engineer Setup Guide
### MongoDB di `vm-be2` | Final Project Cloud Computing

---

## 📋 Overview

| Info | Detail |
|------|--------|
| **VM Target** | `vm-be2` — `azureuser@70.153.145.176` |
| **Database Name** | `orders_db` |
| **MongoDB User** | `admin` / `FpTka2026!` |
| **Port** | `27017` |

---

## Langkah 1 — SSH ke VM

```bash
ssh azureuser@70.153.145.176
```

---

## Langkah 2 — Install MongoDB 7.x

```bash
# 1. Update sistem
sudo apt-get update

# 2. Import GPG Key MongoDB
curl -fsSL https://www.mongodb.org/static/pgp/server-7.0.asc | \
  sudo gpg -o /usr/share/keyrings/mongodb-server-7.0.gpg --dearmor

# 3. Tambah repository MongoDB
echo "deb [ arch=amd64,arm64 signed-by=/usr/share/keyrings/mongodb-server-7.0.gpg ] \
  https://repo.mongodb.org/apt/ubuntu jammy/mongodb-org/7.0 multiverse" | \
  sudo tee /etc/apt/sources.list.d/mongodb-org-7.0.list

# 4. Install MongoDB
sudo apt-get update && sudo apt-get install -y mongodb-org

# 5. Start dan enable agar otomatis jalan saat VM reboot
sudo systemctl start mongod
sudo systemctl enable mongod

# 6. Verifikasi — harusnya muncul "Active: active (running)"
sudo systemctl status mongod
```

> ✅ **Sukses jika:** output status menunjukkan `Active: active (running)`

---

## Langkah 3 — Buat User Admin & Indexes

Masuk ke mongosh (tanpa auth dulu karena belum diaktifkan):

```bash
mongosh
```

Lalu copy-paste semua perintah berikut sekaligus:

```javascript
// Pindah ke database admin untuk buat user
use admin

db.createUser({
  user: "admin",
  pwd: "FpTka2026!",
  roles: [{ role: "root", db: "admin" }]
})

// Pindah ke database kita
use orders_db

// Buat collection orders
db.createCollection("orders")

// Buat index untuk sorting by waktu (GET /orders terbaru)
db.orders.createIndex({ created_at: -1 })

// Buat index untuk lookup by order_id (harus unique)
db.orders.createIndex({ order_id: 1 }, { unique: true })

// Verifikasi — harus muncul 3 index
db.orders.getIndexes()
```

> ✅ **Sukses jika:** `getIndexes()` menampilkan **3 indexes**: `_id_`, `created_at_-1`, `order_id_1`

```bash
# Keluar dari mongosh
exit
```

---

## Langkah 4 — Konfigurasi `mongod.conf`

```bash
sudo nano /etc/mongod.conf
```

Edit file sehingga bagian `net` dan `security` seperti ini:

```yaml
net:
  port: 27017
  bindIp: 0.0.0.0       # Buka akses dari vm-be1

security:
  authorization: enabled  # Wajib pakai user/password
```

> ⚠️ **Perhatian:** `security` harus jadi **section tersendiri** di level paling atas, BUKAN di bawah `processManagement`.

Simpan file (`Ctrl+O` → Enter → `Ctrl+X`), lalu restart MongoDB:

```bash
sudo systemctl restart mongod
sudo systemctl status mongod
```

---

## Langkah 5 — Verifikasi Koneksi dengan Auth

Test login menggunakan credential yang sudah dibuat:

```bash
# Gunakan single quotes agar karakter ! tidak error di bash
mongosh 'mongodb://admin:FpTka2026!@localhost:27017/orders_db?authSource=admin'
```

> ✅ **Sukses jika:** masuk ke shell `orders_db>` tanpa error

Kalau sudah masuk, verifikasi indexes sekali lagi:

```javascript
db.orders.getIndexes()
```

Lalu keluar:

```javascript
exit
```

---

## Langkah 6 — Verifikasi Akhir

```bash
# MongoDB masih jalan?
sudo systemctl status mongod

# Port 27017 terbuka?
ss -tlnp | grep 27017

# Data ada?
mongosh 'mongodb://admin:FpTka2026!@localhost:27017/orders_db?authSource=admin' \
  --eval "db.orders.countDocuments()"

# Indexes ada?
mongosh 'mongodb://admin:FpTka2026!@localhost:27017/orders_db?authSource=admin' \
  --eval "db.orders.getIndexes().forEach(i => print(i.name))"
```

---

## 🔗 Koordinasi dengan Anggota Lain

Setelah semua selesai, **share info berikut ke M4 dan M5:**

| Untuk | URI Koneksi |
|-------|-------------|
| **M4** (dari vm-be1) | `mongodb://admin:FpTka2026!@10.0.0.7:27017/orders_db?authSource=admin` |
| **M5** (di vm-be2) | `mongodb://admin:FpTka2026!@localhost:27017/orders_db?authSource=admin` |

---

## 🐛 Troubleshooting

### Error: `event not found` saat pakai `!`

**Penyebab:** Bash menginterpretasi `!` sebagai history expansion dalam double quotes.

**Solusi:** Selalu gunakan **single quotes** `'...'` untuk URI MongoDB:

```bash
# ❌ Salah
mongosh "mongodb://admin:FpTka2026!@localhost:27017/..."

# ✅ Benar
mongosh 'mongodb://admin:FpTka2026!@localhost:27017/...'
```

---

### Error: `Unrecognized option: processManagement.authorization`

**Penyebab:** `authorization` ditaruh di section yang salah di `mongod.conf`.

**Solusi:** Pastikan `security` adalah section **tersendiri**, bukan di bawah `processManagement`:

```yaml
# ❌ Salah
processManagement:
  authorization: enabled

# ✅ Benar
security:
  authorization: enabled
```

---

### Error: `ECONNREFUSED 127.0.0.1:27017`

**Penyebab:** MongoDB tidak jalan (mungkin crash setelah edit config).

**Solusi:**

```bash
# Cek status
sudo systemctl status mongod

# Cek log error
sudo journalctl -u mongod --no-pager | tail -20

# Start ulang
sudo systemctl start mongod
```

---

*Guide ini dibuat berdasarkan implementation plan revisi M3 — Final Project Cloud Computing*

# M6 — Setup Nginx Load Balancer + Serve Frontend

**Role:** DevOps / Load Balancer Engineer  
**VM Target:** vm-lb (`48.193.46.109`)  
**Deliverable:** Nginx LB routing ke BE1 & BE2 + Frontend live di `http://48.193.46.109`

---

## Prasyarat

Pastikan M4 dan M5 sudah selesai (backend running di port 5000) sebelum mengerjakan M6.

| VM | Public IP | Private IP | Port |
|---|---|---|---|
| vm-lb | `48.193.46.109` | `10.0.0.4` | `80` |
| vm-be1 | `48.193.41.35` | `10.0.0.5` | `5000` |
| vm-be2 | `70.153.145.176` | `10.0.0.7` | `5000` |

---

## Langkah 1 — SSH ke vm-lb

Jalankan dari PowerShell laptop:

```bash
ssh azureuser@48.193.46.109
```

---

## Langkah 2 — Clone Repository

```bash
cd ~
git clone https://github.com/fuaddary/fp-tka-26.git
ls
# Output: fp-tka-26
```

---

## Langkah 3 — Install Nginx

```bash
sudo apt-get update
sudo apt-get install -y nginx
```

---

## Langkah 4 — Buat Konfigurasi Nginx

```bash
sudo nano /etc/nginx/sites-available/fp-tka
```

Paste konfigurasi berikut:

```nginx
upstream backend {
    server 10.0.0.5:5000;   # vm-be1 private IP
    server 10.0.0.7:5000;   # vm-be2 private IP
    keepalive 32;            # reuse koneksi → RPS lebih tinggi
}

server {
    listen 80;

    # Serve Frontend
    location / {
        root /var/www/fp-tka/;
        index index.html;
        try_files $uri $uri/ /index.html;
    }

    # Route semua API endpoint ke backend
    location ~ ^/(order|orders|auth|products|admin|health) {
        proxy_pass http://backend;
        proxy_http_version 1.1;
        proxy_set_header Connection "";
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_connect_timeout 10s;
        proxy_read_timeout 30s;
    }
}
```

Simpan: `Ctrl+O` → Enter → `Ctrl+X`

> **Catatan:** `location ~ ^/(...)` menggunakan regex agar semua path seperti `/orders/<id>` ikut ter-route ke backend dengan benar.

---

## Langkah 5 — Copy Frontend ke Web Root

```bash
sudo mkdir -p /var/www/fp-tka/
sudo cp ~/fp-tka-26/Resources/FE/* /var/www/fp-tka/

# Verifikasi file ada
ls /var/www/fp-tka/
# Output: index.html  styles.css
```

---

## Langkah 6 — Update index.html

File `index.html` bawaan repo masih hardcode ke `localhost:5001` dan tidak memiliki fitur login. Karena backend menggunakan JWT auth, `index.html` perlu diganti dengan versi yang memiliki:

- Form Login / Register
- Search produk dari API
- Semua request otomatis membawa JWT token

### Cara update via nano:

```bash
sudo nano /var/www/fp-tka/index.html
```

1. Hapus semua isi lama
2. Paste isi `index.html` baru (lihat file terlampir di repo)
3. Simpan: `Ctrl+O` → Enter → `Ctrl+X`

> **Catatan:** `styles.css` tidak perlu diubah, tetap pakai file bawaan repo.

---

## Langkah 7 — Aktifkan Konfigurasi Nginx

```bash
# Aktifkan config baru
sudo ln -s /etc/nginx/sites-available/fp-tka /etc/nginx/sites-enabled/

# Hapus config default
sudo rm /etc/nginx/sites-enabled/default

# Test syntax — JANGAN skip!
sudo nginx -t
# Output yang diharapkan:
# nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
# nginx: configuration file /etc/nginx/nginx.conf test is successful

# Restart & enable auto-start
sudo systemctl restart nginx
sudo systemctl enable nginx
```

---

## Langkah 8 — Verifikasi

### Test dari vm-lb:

```bash
# Test Frontend muncul HTML
curl http://localhost/
# Output: <!DOCTYPE html>...

# Test health check ke backend
curl http://localhost/health
# Output: {"status":"ok","timestamp":"..."}
```

### Test dari browser laptop:

Buka `http://48.193.46.109` → harusnya muncul halaman **Login / Register**.

Login dengan:
- **Email:** `admin1@tka.its.ac.id`
- **Password:** `Admin@12345`

Setelah login, coba fitur:
- **Buat Pesanan** — ketik nama produk → pilih dari hasil search → isi jumlah → Buat Pesanan
- **Tampilkan Riwayat** — harusnya muncul data orders dari database

---

## Hasil Akhir

| Komponen | Status |
|---|---|
| Nginx installed & running | ✅ |
| Load balancer round-robin ke BE1 & BE2 | ✅ |
| Keepalive 32 aktif (boost RPS) | ✅ |
| Frontend live di `http://48.193.46.109` | ✅ |
| Login / Register berfungsi | ✅ |
| Search produk & buat pesanan berfungsi | ✅ |
| Riwayat pesanan tampil | ✅ |
| Health check response | ✅ |

---

## Troubleshooting

### 403 Forbidden saat buka `http://48.193.46.109`

**Penyebab:** Folder `/var/www/fp-tka/` belum dibuat atau file FE belum dicopy.

**Solusi:**
```bash
sudo mkdir -p /var/www/fp-tka/
sudo cp ~/fp-tka-26/Resources/FE/* /var/www/fp-tka/
ls /var/www/fp-tka/
# Harus ada: index.html  styles.css
```

---

### 502 Bad Gateway

**Penyebab:** Backend di vm-be1 atau vm-be2 belum running.

**Solusi:** Koordinasi dengan M4 dan M5, pastikan Gunicorn sudah jalan di kedua VM:
```bash
# Cek dari vm-lb apakah port backend bisa dijangkau
curl http://10.0.0.5:5000/health
curl http://10.0.0.7:5000/health
```

---

### `nginx -t` error: syntax error

**Penyebab:** Ada typo di file konfigurasi, biasanya kurang titik koma atau kurung kurawal.

**Solusi:** Buka ulang config dan cek baris yang disebutkan di error:
```bash
sudo nano /etc/nginx/sites-available/fp-tka
```

---

### `{"error":"Token tidak ditemukan"}` saat curl `/orders`

**Penyebab:** Endpoint `/orders` memerlukan JWT token — ini normal karena backend pakai auth.

**Solusi:** Akses lewat browser menggunakan FE yang sudah ada form login, bukan via curl langsung.

---

### FE muncul tapi login gagal

**Penyebab:** MONGO_URI di backend salah (pakai `orders_db` padahal harusnya `orderdb`).

**Solusi:** Koordinasi dengan M4/M5, pastikan `.env` mereka pakai:
```
MONGO_URI=mongodb://admin:FpTka2026!@10.0.0.7:27017/orderdb?authSource=admin
```

---

### Search produk tidak muncul hasil

**Penyebab:** Mungkin ketikan kurang dari 2 huruf, atau koneksi ke backend lambat.

**Solusi:** Ketik minimal 2 karakter dan tunggu sebentar (ada delay 300ms). Kalau tetap tidak muncul, cek di browser DevTools (F12 → Network) apakah request ke `/products?search=...` berhasil.

---

## Screenshot untuk Dokumentasi (M8)

1. Output `sudo nginx -t` (syntax ok)
2. Output `sudo systemctl status nginx` (active running)  
3. Output `curl http://localhost/health`
4. Tampilan halaman Login di browser `http://48.193.46.109`
5. Tampilan halaman utama setelah login (form order, riwayat)

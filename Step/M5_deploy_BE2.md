# Deployment Backend (vm-be2)

## Step 1 — SSH ke vm-be2

Masuk ke VM menggunakan SSH:

```bash
ssh azureuser@70.153.145.176
```

Masukkan password saat diminta:

```text
TkaFP2026!Azure
```

---

## Step 2 — Update System & Install Dependencies

Update package dan install dependency yang dibutuhkan.

```bash
sudo apt-get update
sudo apt-get install -y python3 python3-pip git

pip3 install flask pymongo gunicorn gevent python-dotenv bcrypt PyJWT
```

---

## Step 3 — Clone Repository

Clone repository kemudian masuk ke folder backend.

```bash
git clone https://github.com/fuaddary/fp-tka-26.git
cd fp-tka-26/Resources/BE
```

---

## Step 4 — Test Manual (Verifikasi Koneksi MongoDB)

Jalankan aplikasi secara manual menggunakan environment variable `MONGO_URI`.

```bash
export MONGO_URI='mongodb://admin:FpTka2026!@localhost:27017/orderdb?authSource=admin'
PORT=5000 python3 app.py
```

### Output yang diharapkan

```text
* Running on http://127.0.0.1:5000
* Running on http://10.0.0.7:5000
```

Buka terminal atau tab baru, kemudian login kembali ke VM.

```bash
ssh azureuser@70.153.145.176
```

Lakukan pengujian endpoint.

```bash
curl http://localhost:5000/products
```

Jika endpoint mengembalikan data JSON produk, maka koneksi ke MongoDB berhasil.

✅ **Koneksi MongoDB sukses**

Setelah selesai melakukan pengujian, kembali ke terminal pertama lalu hentikan aplikasi.

```text
Ctrl + C
```

---

## Step 5 — Buat Systemd Service

Buat file service baru.

```bash
sudo nano /etc/systemd/system/flask-be2.service
```

Isi file dengan konfigurasi berikut.

```ini
[Unit]
Description=Flask Backend BE2
After=network.target mongod.service

[Service]
User=azureuser
WorkingDirectory=/home/azureuser/fp-tka-26/Resources/BE
Environment="MONGO_URI=mongodb://admin:FpTka2026!@localhost:27017/orderdb?authSource=admin"
Environment="JWT_SECRET=fp-tka-2026-rahasia-kelompok-26"
ExecStart=/home/azureuser/.local/bin/gunicorn -w 3 --threads 4 -k gthread --timeout 30 --bind 0.0.0.0:5000 app:app
Restart=always

[Install]
WantedBy=multi-user.target
```

Simpan file:

```text
Ctrl + X
Y
Enter
```

---

## Step 6 — Aktifkan & Jalankan Service

Reload konfigurasi systemd, jalankan service, dan aktifkan agar otomatis berjalan saat boot.

```bash
sudo systemctl daemon-reload
sudo systemctl start flask-be2
sudo systemctl enable flask-be2
```

---

## Step 7 — Verifikasi Service Running

Periksa status service.

```bash
sudo systemctl status flask-be2
```

### Output yang diharapkan

```text
Active: active (running)
...
Listening at: http://0.0.0.0:5000
Using worker: gthread
Booting worker with pid: xxxxx (x3)
```

Tekan:

```text
q
```

untuk keluar dari tampilan status.

---

## Step 8 — Verifikasi Endpoint

Pastikan endpoint dapat diakses.

```bash
curl http://localhost:5000/health

curl http://localhost:5000/products
```

Jika kedua endpoint memberikan respons yang sesuai, maka deployment backend pada **vm-be2** telah berhasil.
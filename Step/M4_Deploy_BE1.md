# M4 — Deploy Backend #1 (vm-be1)

Dokumentasi langkah deploy Flask backend di **vm-be1** untuk FP TKA 2026, Kelompok 26.

| Item | Nilai |
|---|---|
| VM | vm-be1 |
| Public IP | `48.193.41.35` |
| Private IP | `10.0.0.5` |
| OS | Ubuntu 22.04 (jammy) |
| Port aplikasi | 5000 |
| Worker | Gunicorn `gthread`, 5 worker × 4 thread |
| MongoDB | di vm-be2 (`10.0.0.7:27017`) |

---

## 1. SSH masuk ke vm-be1

Masuk ke VM. Sebut terminal ini **Tab A** — semua langkah 2 sampai 5 dikerjakan di sini.

```bash
ssh azureuser@48.193.41.35
```

## 2. Install dependencies sistem

```bash
sudo apt-get update
sudo apt-get install -y python3 python3-pip git
```

## 3. Clone repo dan masuk folder backend

```bash
git clone https://github.com/fuaddary/fp-tka-26.git
cd fp-tka-26/Resources/BE
```

## 4. Install library Python

VM ini Ubuntu 22.04 (pip lama), jadi **tanpa** flag `--break-system-packages` (flag itu hanya ada di pip Ubuntu 24+).

```bash
pip3 install -r requirements.txt gevent
```

Gunicorn terpasang di `/home/azureuser/.local/bin/`. Akan muncul warning "not on PATH" — abaikan, nanti dipakai dengan path lengkap.

## 5. Test manual (memastikan koneksi ke MongoDB)

Tujuannya: **Tab A** menjalankan aplikasi, **Tab B** mengetes dari samping selagi aplikasi hidup. Ini memastikan koneksi ke MongoDB benar sebelum repot membuat service.

**Di Tab A**, set `MONGO_URI` lalu jalankan aplikasi. Wajib pakai **kutip tunggal** `'...'` karena password mengandung `!` yang akan error kalau pakai kutip ganda (`!` ditafsirkan bash sebagai history expansion):

```bash
export MONGO_URI='mongodb://admin:FpTka2026!@10.0.0.7:27017/orderdb?authSource=admin'
echo "$MONGO_URI"
python3 app.py
```

Setelah perintah terakhir, Tab A akan menampilkan `Running on http://...` lalu **diam** (prompt tidak kembali). Itu normal — aplikasi sedang jalan dan menahan tab ini. **Jangan tekan apa-apa di Tab A.** Biarkan terbuka.

**Buka terminal baru (Tab B)**, SSH ke VM yang sama:

```bash
ssh azureuser@48.193.41.35
```

Di **Tab B**, jalankan tes. Port-nya **5001** karena `python3 app.py` manual berjalan di 5001:

```bash
curl http://localhost:5001/products
```

Kalau keluar JSON produk (mis. `"total":92...`), koneksi ke MongoDB di vm-be2 sukses.

Lalu matikan aplikasi manual: **kembali ke Tab A**, tekan **Ctrl+C**. Prompt kembali normal. Sisa langkah dikerjakan di Tab A.

## 6. Buat systemd service

Di Tab A:

```bash
sudo nano /etc/systemd/system/flask-be1.service
```

Isi berikut (`JWT_SECRET` **wajib** disepakati sama dengan M5):

```ini
[Unit]
Description=Flask Backend BE1
After=network.target

[Service]
User=azureuser
WorkingDirectory=/home/azureuser/fp-tka-26/Resources/BE
Environment="MONGO_URI=mongodb://admin:FpTka2026!@10.0.0.7:27017/orderdb?authSource=admin"
Environment="JWT_SECRET=fp-tka-2026-rahasia-kelompok-26"
ExecStart=/home/azureuser/.local/bin/gunicorn -w 5 --threads 4 -k gthread --timeout 30 --bind 0.0.0.0:5000 app:app
Restart=always

[Install]
WantedBy=multi-user.target
```

Simpan: `Ctrl+O`, `Enter`, lalu `Ctrl+X`.

## 7. Jalankan dan aktifkan auto-start

```bash
sudo systemctl daemon-reload
sudo systemctl start flask-be1
sudo systemctl enable flask-be1
sudo systemctl status flask-be1
```

Harus `active (running)`. Tekan `q` untuk keluar dari tampilan status.

## 8. Verifikasi final

Service jalan di background, tidak perlu tab lain. Langsung di Tab A:

```bash
curl http://localhost:5000/products
```

Keluar JSON produk → **BE1 beres**, jalan permanen di port 5000.

---

## Catatan: kenapa gthread, bukan gevent?

Implementation plan awalnya menyarankan worker **gevent**. Saat deploy, gevent gagal boot dengan error `ModuleNotFoundError: No module named 'zope.event'` — sebuah konflik *namespace package* `zope` di Ubuntu 22.04 (paket `zope.event` dan `zope.interface` terinstall di lokasi berbeda sehingga Python gagal menemukan submodulnya). Karena itu worker diganti ke **gthread**, yang merupakan bawaan Gunicorn dan tidak butuh `zope` sama sekali.

### Perbedaan gevent vs gthread (singkat)

| Aspek | gevent | gthread |
|---|---|---|
| Model konkurensi | Green threads (coroutine) | Thread OS sungguhan |
| Kapasitas konkuren | Sangat tinggi (ratusan+ per worker) | Dibatasi `worker × threads` |
| Memori per koneksi | Sangat hemat | Lebih berat dari green thread |
| Dependency | Butuh `gevent` + `zope` | Bawaan Gunicorn, tanpa dependency tambahan |
| Cocok untuk | Konkurensi ekstrem | Beban I/O-bound umum |

Untuk aplikasi ini yang **I/O-bound** (tiap request menghabiskan waktu menunggu MongoDB membalas, bukan menghitung di CPU), keduanya menghasilkan RPS yang berdekatan. Bottleneck sebenarnya kemungkinan ada di MongoDB (satu instance dipakai dua backend) atau di Nginx LB, bukan di model worker BE1. Jadi penggantian ini tidak mengorbankan performa secara berarti.

### Cara kerja konfigurasi gthread yang dipakai

Konfigurasi: `-w 5 --threads 4` → **5 worker × 4 thread = 20 koneksi aktif** sekaligus per backend.

- Gunicorn menjalankan **5 proses worker** terpisah (memanfaatkan banyak core CPU dan saling terisolasi).
- Tiap worker punya **4 thread** yang berbagi menangani request.
- Saat sebuah thread mengirim query ke MongoDB lalu **menunggu balasan**, Python melepas GIL sehingga thread lain di worker yang sama bisa langsung melayani request lain. Inilah yang membuat gthread efektif untuk beban I/O: waktu tunggu DB "diisi" pekerjaan request lain, bukan menganggur.
- Karena request order cepat selesai (query DB lalu balas), thread berputar cepat, sehingga 20 thread per backend mampu melayani throughput tinggi.

> Jika saat load testing BE1 ternyata jenuh lebih dulu (CPU belum penuh tapi failure sudah muncul), `--threads` bisa dinaikkan (mis. ke 8) tanpa perlu kembali ke gevent.

---

## Catatan penutup & koordinasi

- Service **auto-restart** kalau crash (`Restart=always`) dan **auto-start** kalau VM reboot (`enable`). Aman ditinggal logout.
- Ambil **screenshot** output langkah 7 (`status active (running)`) dan langkah 8 (`curl` JSON produk) untuk laporan M8.
- **Kabari M6:** BE1 siap di `10.0.0.5:5000`.
- **Sepakati `JWT_SECRET` dengan M5** — nilai harus identik di kedua backend, kalau tidak auth akan pecah secara acak di belakang load balancer.
- Dua penyimpangan dari plan untuk dicatat di laporan: (1) VM ternyata Ubuntu 22.04 bukan 24, jadi `--break-system-packages` tidak dipakai; (2) gevent diganti gthread karena konflik namespace `zope`.

### Troubleshooting

Kalau service gagal start, lihat penyebabnya:

```bash
sudo journalctl -u flask-be1 -n 40 --no-pager
```

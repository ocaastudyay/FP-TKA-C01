# FP-TKA-C01

# Final Project Teknologi Komputasi Awan 2026

## Order Processing Service menggunakan Microsoft Azure

### Kelompok 1

| No | Nama | NRP |
| :---: | :--- | :---: |
| 1 | Diva Aulia Rosa | 5027241003 |
| 2 | Zaenal Mustofa | 5027241018 |
| 3 | Raya Ahmad Syarif | 5027241041 |
| 4 | Daniswara Fausta Novanto | 5027241050 |
| 5 | Hafiz Ramadhan | 5027241096 |
| 6 | Naruna Vicranthyo Putra Gangga | 5027241105 |
| 7 | Adinda Cahya Pramesti | 5027241117 |

---

# Daftar Isi

* [1. Introduction](#1-introduction)
* [2. Arsitektur Cloud](#2-arsitektur-cloud)
* [3. Implementasi](#3-implementasi)
* [4. Hasil Pengujian Endpoint](#4-hasil-pengujian-endpoint)
* [5. Hasil Load Testing](#5-hasil-load-testing)
* [6. Kesimpulan dan Saran](#6-kesimpulan-dan-saran)

---

# 1. Introduction

## 1.1 Latar Belakang

Perkembangan layanan cloud computing memungkinkan aplikasi modern dibangun dengan arsitektur yang lebih fleksibel, scalable, dan mudah dikelola. Dalam lingkungan produksi, aplikasi tidak hanya dituntut untuk berjalan dengan baik, tetapi juga mampu menangani banyak permintaan pengguna secara bersamaan tanpa mengalami penurunan performa yang signifikan.

Pada Final Project Teknologi Komputasi Awan 2026, kelompok diminta untuk membangun dan melakukan deployment sebuah layanan **Order Processing Service** menggunakan Microsoft Azure. Sistem yang dibangun harus mampu melayani operasi CRUD sederhana melalui REST API, menggunakan MongoDB sebagai basis data, serta menerapkan load balancing untuk meningkatkan kemampuan sistem dalam menangani beban kerja.

Selain implementasi aplikasi, proyek ini juga menekankan pada aspek deployment cloud, konfigurasi infrastruktur, monitoring performa, serta pengujian beban menggunakan Locust.

---

## 1.2 Permasalahan

Permasalahan utama yang ingin diselesaikan dalam proyek ini adalah:

* Bagaimana membangun layanan Order Processing Service yang dapat diakses melalui internet.
* Bagaimana mendistribusikan request ke beberapa backend server menggunakan load balancer.
* Bagaimana mengintegrasikan backend dengan database MongoDB pada lingkungan cloud.
* Bagaimana mengukur performa sistem ketika menerima request dalam jumlah besar secara bersamaan.
* Bagaimana mengoptimalkan penggunaan resource cloud dengan tetap memenuhi batas anggaran yang diberikan.

---

## 1.3 Tujuan

Tujuan dari proyek ini adalah:

1. Membangun layanan Order Processing Service berbasis Flask.
2. Melakukan deployment aplikasi pada Microsoft Azure.
3. Mengimplementasikan Nginx sebagai Load Balancer.
4. Menggunakan MongoDB sebagai database utama.
5. Melakukan pengujian endpoint menggunakan Postman.
6. Melakukan load testing menggunakan Locust.
7. Menganalisis performa sistem berdasarkan hasil pengujian.

---

# 2. Arsitektur Cloud

## 2.1 Gambaran Umum Arsitektur

Sistem dibangun menggunakan tiga Virtual Machine yang berada dalam satu Azure Virtual Network (VNet) dengan subnet private `10.0.0.0/24`.

Arsitektur terdiri dari:

* 1 VM sebagai Nginx Load Balancer dan Frontend Server.
* 2 VM sebagai Backend Flask Server.
* MongoDB 7.0 sebagai database utama.
* Frontend berbasis static files yang disajikan melalui Nginx.

Seluruh request dari pengguna akan diterima oleh Nginx Load Balancer dan didistribusikan ke backend menggunakan algoritma Round Robin.

---

## 2.2 Diagram Arsitektur

### Diagram Infrastruktur

<img width="1489" height="891" alt="image" src="https://github.com/user-attachments/assets/a6492267-e7a3-4cd3-812e-069901ae8df8" />

Gambar di atas menunjukkan arsitektur cloud yang digunakan pada proyek ini. Nginx bertindak sebagai reverse proxy sekaligus load balancer yang mendistribusikan request ke dua backend Flask menggunakan metode Round Robin. Backend kemudian berkomunikasi dengan MongoDB melalui jaringan private Azure.

---

## 2.3 Spesifikasi Virtual Machine

| VM     | Azure Size        | vCPU | RAM  | Fungsi                                  |
| ------ | ----------------- | ---- | ---- | --------------------------------------- |
| vm-lb  | Standard_B2ats_v2 | 2    | 1 GB | Nginx Load Balancer dan Frontend Server |
| vm-be1 | Standard_B2als_v2 | 2    | 4 GB | Backend Flask #1                        |
| vm-be2 | Standard_B2als_v2 | 2    | 4 GB | Backend Flask #2 dan MongoDB            |

### Tabel Spesifikasi VM & Biaya

| VM | Size (Azure) | vCPU | RAM | Role | Harga/bulan |
|---|---|:---:|:---:|---|:---:|
| vm-lb | Standard_B2ats_v2 | 2 | 1 GB | Nginx Load Balancer — Round-robin + Keepalive 32 + Serve Frontend | $7.81 |
| vm-be1 | Standard_B2als_v2 | 2 | 4 GB | Backend #1 — Flask + Gunicorn gevent (5 workers) | $31.24 |
| vm-be2 | Standard_B2als_v2 | 2 | 4 GB | Backend #2 — Flask + Gunicorn gevent (3 workers) + MongoDB 7.0 | $31.24 |
| **Total** | | | | | **$70.29/bulan** |

> **Budget maksimal:** $75/bulan (sisa $4.71 digunakan sebagai buffer biaya network egress)

## 2.4 Estimasi Biaya Infrastruktur

| VM        | Fungsi                      | Harga/Bulan |
| --------- | --------------------------- | ----------- |
| vm-lb     | Load Balancer + Frontend    | $7.81       |
| vm-be1    | Backend Server #1           | $31.24      |
| vm-be2    | Backend Server #2 + MongoDB | $31.24      |
| **Total** |                             | **$70.29**  |

Total biaya infrastruktur yang digunakan adalah **$70.29 per bulan**, masih berada di bawah batas maksimal anggaran sebesar **$75 per bulan**.

---

## 2.5 Konfigurasi IP

| VM | Public IP | Private IP | Port |
|---|---|---|:---:|
| vm-lb | 48.193.46.109 | 10.0.0.4 | 80 |
| vm-be1 | 48.193.41.35 | 10.0.0.5 | 5000 |
| vm-be2 | 70.153.145.176 | 10.0.0.7 | 5000, 27017 |

---

## 2.6 Konfigurasi Network Security Group (NSG)

Untuk mengontrol akses terhadap setiap layanan, digunakan Azure Network Security Group dengan konfigurasi sebagai berikut.

| Priority | Rule Name          | Port  | Protocol | Action |
| -------- | ------------------ | ----- | -------- | ------ |
| 100      | Allow-SSH          | 22    | TCP      | Allow  |
| 110      | Allow-HTTP         | 80    | TCP      | Allow  |
| 120      | Allow-Backend-5000 | 5000  | TCP      | Allow  |
| 130      | Allow-MongoDB      | 27017 | TCP      | Allow  |

### Dokumentasi Konfigurasi NSG

<img width="1600" height="422" alt="M1_Inbound_Security_Rules" src="https://github.com/user-attachments/assets/4e0228a3-c20f-4479-a068-efca1f5a31ca" />

Konfigurasi tersebut memungkinkan administrator melakukan akses SSH, pengguna mengakses aplikasi melalui HTTP, backend berkomunikasi melalui port 5000, dan database MongoDB menerima koneksi melalui port 27017.

---

## 2.7 Alasan Pemilihan Konfigurasi

### 1. vm-lb — Standard_B2ats_v2 (1 GB RAM, $7.81/bulan)

Nginx bertugas hanya sebagai reverse proxy dan load balancer — meneruskan request masuk ke salah satu backend, bukan memproses logika aplikasi. Karena itu, Nginx sangat efisien dalam konsumsi resource: bahkan di bawah ribuan koneksi konkuren, penggunaan RAM-nya tetap di kisaran ratusan MB. Memilih VM dengan RAM lebih besar untuk layer ini tidak memberikan peningkatan RPS yang berarti, sehingga Standard_B2ats_v2 dengan 1 GB RAM adalah pilihan paling efisien dari sisi biaya.

Fitur tambahan yang diaktifkan:
- **Keepalive 32 koneksi** ke upstream — menghilangkan overhead TCP handshake berulang, berkontribusi sekitar 10–15% peningkatan throughput tanpa biaya tambahan.
- **Serve static file** — Frontend (index.html + styles.css) langsung dikirim dari Nginx tanpa menyentuh backend, menghemat resource BE untuk memproses request API.

### 2. vm-be1 — Standard_B2als_v2 (4 GB RAM, $31.24/bulan)

Backend Flask yang dijalankan dengan Gunicorn membutuhkan RAM yang cukup untuk menampung banyak worker process secara bersamaan. Dengan 4 GB RAM yang seluruhnya bebas digunakan untuk proses aplikasi (tidak berbagi dengan MongoDB), vm-be1 dapat menjalankan **5 gevent worker** secara optimal.

Alasan memilih **gevent worker** dibanding sync worker default:
- Sync worker memblokir seluruh process selama menunggu respons database, sehingga hanya bisa melayani satu request per worker dalam satu waktu.
- Gevent worker menggunakan model kooperatif (green thread), sehingga satu worker bisa menangani puluhan request sekaligus selama ada jeda I/O.
- Estimasi peningkatan RPS dari perubahan ini saja sekitar **30–40%**.

### 3. vm-be2 — Standard_B2als_v2 (4 GB RAM, $31.24/bulan)

Sama seperti vm-be1 namun hanya menjalankan **3 gevent worker** karena sebagian RAM-nya harus disisihkan untuk MongoDB yang berjalan di VM yang sama. Pembagian ini mencegah kondisi Out-of-Memory (OOM) saat load test tinggi yang dapat menyebabkan crash pada salah satu proses.

Estimasi pembagian RAM vm-be2:
| Proses | Estimasi RAM |
|---|---|
| Flask + Gunicorn (3 workers) | ~600 MB |
| MongoDB 7.0 (buffer + index) | ~1.5 GB |
| Sistem operasi (Ubuntu) | ~300 MB |
| Buffer/headroom | ~1.6 GB |
| **Total** | **~4 GB** |

### 4. MongoDB di vm-be2 — bukan VM terpisah

Dengan budget $75/bulan, menambah VM keempat khusus untuk database akan memaksa downgrade spesifikasi backend atau melampaui anggaran. Penempatan MongoDB di vm-be2 merupakan trade-off yang terukur karena:

1. Koneksi dari vm-be1 ke MongoDB tetap melalui **Private VNet (10.0.0.0/24)** dengan latensi sangat rendah di bawah 1 ms — jauh lebih baik dibanding koneksi lintas internet.
2. Query yang dijalankan (`insert`, `find by order_id`, `find all sort by created_at`) relatif ringan jika index sudah dibuat dengan benar.
3. **Index pada `created_at DESC` dan `order_id` (unique)** memastikan query tidak melakukan full collection scan meski data sudah mencapai ratusan ribu dokumen.

> **Catatan:** Jika budget bertambah, prioritas pertama adalah memindahkan MongoDB ke VM dedicated agar vm-be2 dapat menjalankan 5 worker penuh, kemudian menambahkan backend ketiga untuk meningkatkan total kapasitas.

## 2.8 Estimasi Performa

| Kondisi | Estimasi RPS |
|---|:---:|
| Tanpa optimasi (sync worker, tanpa index) | ~50–70 |
| + MongoDB index (`created_at`, `order_id`) | ~100–130 |
| + Gevent worker | ~130–170 |
| + Nginx keepalive 32 | ~150–190 |

Target minimum untuk nilai load testing: **≥ 150 RPS**
---

# 3. Implementasi

## 3.1 Implementasi Database MongoDB (M3)

MongoDB 7.0 digunakan sebagai database utama pada sistem Order Processing Service. Database ditempatkan pada VM `vm-be2` dengan alamat private `10.0.0.7` sehingga dapat diakses oleh backend melalui jaringan internal Azure Virtual Network.

### Spesifikasi Database

| Parameter       | Nilai       |
| --------------- | ----------- |
| Database Engine | MongoDB 7.0 |
| Database Name   | `orderdb`   |
| Host            | vm-be2      |
| Private IP      | 10.0.0.7    |
| Port            | 27017       |
| Authentication  | Enabled     |

---

### Instalasi dan Konfigurasi MongoDB

Tahapan implementasi MongoDB meliputi:

1. Instalasi MongoDB 7.0 dan MongoDB Database Tools.
2. Aktivasi service MongoDB menggunakan systemd.
3. Pembuatan user administrator database.
4. Aktivasi authentication untuk meningkatkan keamanan akses database.
5. Konfigurasi `bindIp` agar backend dapat mengakses database melalui jaringan internal Azure.
6. Restore dataset awal menggunakan `mongorestore`.

Konfigurasi utama MongoDB:

```yaml
net:
  port: 27017
  bindIp: 0.0.0.0

security:
  authorization: enabled
```

Konfigurasi tersebut memungkinkan MongoDB menerima koneksi dari backend server sambil tetap menerapkan mekanisme autentikasi.

---

### Restore Database

Dataset awal diperoleh dari repository Final Project dan direstore ke database `orderdb`.

Hasil restore:

| Collection | Jumlah Data |
| ---------- | ----------- |
| users      | 505         |
| products   | 96          |
| orders     | 10000       |

Data tersebut digunakan sebagai sumber data utama selama proses pengujian endpoint maupun load testing.

---

### Optimasi Database Menggunakan Index

Untuk meningkatkan performa query, dibuat beberapa index berdasarkan pola akses endpoint aplikasi.

#### Collection Orders

| Index                | Fungsi                         |
| -------------------- | ------------------------------ |
| created_at (-1)      | Sorting order terbaru          |
| order_id (unique)    | Pencarian order berdasarkan ID |
| user_id + created_at | Riwayat order pengguna         |
| status               | Statistik dan agregasi admin   |

#### Collection Products

| Index                  | Fungsi                 |
| ---------------------- | ---------------------- |
| is_active + category   | Filter kategori produk |
| is_active + created_at | Sorting produk aktif   |

#### Collection Users

| Index          | Fungsi                        |
| -------------- | ----------------------------- |
| email (unique) | Login dan registrasi pengguna |

Pembuatan index dilakukan berdasarkan analisis query yang digunakan oleh aplikasi Flask dan skenario load testing sehingga dapat mengurangi waktu pencarian data saat sistem menerima beban tinggi.

---

### Integrasi Backend dengan MongoDB

MongoDB ditempatkan pada VM `vm-be2` dengan alamat private `10.0.0.7`.

| Backend                   | Host Database |
| ------------------------- | ------------- |
| Backend Server 1 (vm-be1) | 10.0.0.7      |
| Backend Server 2 (vm-be2) | localhost     |

Penggunaan private IP memungkinkan komunikasi database dilakukan melalui Azure Virtual Network tanpa melewati jaringan publik sehingga lebih aman dan memiliki latensi yang lebih rendah.

---

### Kendala yang Ditemui

| Permasalahan                             | Solusi                                                      |
| ---------------------------------------- | ----------------------------------------------------------- |
| Authentication gagal saat restore        | Menggunakan URI dengan kredensial administrator             |
| MongoDB tidak dapat diakses dari backend | Mengubah bindIp menjadi 0.0.0.0                             |
| mongorestore tidak tersedia              | Menginstal mongodb-database-tools                           |
| Error konfigurasi authorization          | Memindahkan section security ke level root pada mongod.conf |

---

### Dokumentasi

<img width="833" height="157" alt="WhatsApp Image 2026-06-19 at 08 39 10" src="https://github.com/user-attachments/assets/4d844e5a-529e-45d6-b3e7-97f4e1672607" />

---

## 3.2 Implementasi Backend Server 1 (M4)

Backend Server 1 ditempatkan pada VM `vm-be1` dengan alamat private `10.0.0.5` dan public IP `48.193.41.35`. Server ini bertugas menangani request aplikasi yang diteruskan oleh Nginx Load Balancer serta berkomunikasi dengan MongoDB yang berada pada VM `vm-be2`.

### Spesifikasi Backend

| Parameter        | Nilai        |
| ---------------- | ------------ |
| VM               | vm-be1       |
| Public IP        | 48.193.41.35 |
| Private IP       | 10.0.0.5     |
| Operating System | Ubuntu 22.04 |
| Framework        | Flask        |
| WSGI Server      | Gunicorn     |
| Port             | 5000         |
| Database Host    | 10.0.0.7     |
| Database         | MongoDB 7.0  |

---

### Instalasi Backend

Tahap deployment backend dilakukan dengan:

1. Instalasi Python 3, Pip, dan Git.
2. Clone repository Final Project.
3. Instalasi seluruh dependency menggunakan `requirements.txt`.
4. Pengujian aplikasi secara manual.
5. Konfigurasi service menggunakan systemd.
6. Deployment menggunakan Gunicorn.

Pendekatan ini memastikan aplikasi dapat diverifikasi terlebih dahulu sebelum dijalankan secara permanen sebagai service.

---

### Pengujian Koneksi Database

Sebelum backend dijalankan menggunakan Gunicorn, dilakukan pengujian manual menggunakan Flask development server.

Backend dikonfigurasi untuk terhubung ke MongoDB menggunakan jaringan internal Azure sehingga komunikasi antar layanan tidak melewati jaringan publik.

Pengujian dilakukan dengan mengakses endpoint produk dan memastikan data berhasil diambil dari database MongoDB pada `vm-be2`.

---

### Deployment Menggunakan Gunicorn

Backend dijalankan menggunakan Gunicorn sebagai production WSGI server.

Konfigurasi yang digunakan:

| Parameter         | Nilai        |
| ----------------- | ------------ |
| Worker Type       | gthread      |
| Worker            | 5            |
| Thread per Worker | 4            |
| Total Thread      | 20           |
| Timeout           | 30 detik     |
| Binding Address   | 0.0.0.0:5000 |

Dengan konfigurasi tersebut Backend Server 1 mampu menangani beberapa request secara bersamaan tanpa harus membuat proses baru untuk setiap koneksi yang masuk.

---

### Konfigurasi Systemd

Aplikasi dikonfigurasi sebagai service bernama:

```text
flask-be1.service
```

Konfigurasi ini memberikan beberapa keuntungan:

* Service berjalan otomatis saat VM melakukan booting.
* Service akan restart otomatis jika mengalami crash.
* Backend tetap aktif meskipun administrator logout dari terminal.

Pendekatan ini meningkatkan reliabilitas aplikasi selama proses pengujian maupun deployment.

---

### Pemilihan Worker gthread

Rencana awal deployment menggunakan worker tipe **gevent**. Namun selama implementasi ditemukan konflik dependency pada Ubuntu 22.04 yang menyebabkan Gunicorn gagal menjalankan worker gevent.

Sebagai solusi digunakan worker tipe **gthread** yang merupakan implementasi bawaan Gunicorn.

| Aspek                   | gevent       | gthread         |
| ----------------------- | ------------ | --------------- |
| Model Konkurensi        | Green Thread | Native Thread   |
| Dependency Tambahan     | Ya           | Tidak           |
| Kompleksitas Deployment | Lebih tinggi | Lebih sederhana |
| Cocok untuk I/O Bound   | Ya           | Ya              |

Karena aplikasi lebih banyak melakukan operasi I/O ke MongoDB dibanding komputasi CPU-intensive, penggunaan gthread tetap mampu memberikan performa yang baik.

---

### Analisis Konfigurasi Backend

Konfigurasi 5 worker dan 4 thread dipilih untuk memanfaatkan sumber daya VM secara optimal.

Ketika sebuah thread sedang menunggu respons dari MongoDB, thread lain tetap dapat melayani request yang masuk. Karakteristik ini membuat model gthread cukup efektif untuk aplikasi berbasis database seperti Order Processing Service.

Selain itu, backend dikonfigurasi menggunakan fitur auto-restart pada systemd sehingga layanan tetap tersedia apabila terjadi crash atau restart pada VM.

---

### Dokumentasi

<img width="1600" height="623" alt="WhatsApp Image 2026-06-17 at 22 23 45 (1)" src="https://github.com/user-attachments/assets/78a8f0d3-ef41-4d47-b585-d96147056c65" />
<img width="1472" height="639" alt="WhatsApp Image 2026-06-17 at 22 23 45" src="https://github.com/user-attachments/assets/4d31e128-349c-4c33-9016-ff32a50814de" />

---

## 3.3 Implementasi Backend Server 2 (M5)

Backend Server 2 ditempatkan pada VM `vm-be2` dengan public IP `70.153.145.176` dan private IP `10.0.0.7`. Selain menjalankan aplikasi Flask, VM ini juga berfungsi sebagai host MongoDB sehingga backend dapat mengakses database secara langsung melalui localhost.

### Spesifikasi Backend

| Parameter        | Nilai          |
| ---------------- | -------------- |
| VM               | vm-be2         |
| Public IP        | 70.153.145.176 |
| Private IP       | 10.0.0.7       |
| Operating System | Ubuntu 22.04   |
| Framework        | Flask          |
| WSGI Server      | Gunicorn       |
| Port             | 5000           |
| Database Host    | localhost      |
| Database         | MongoDB 7.0    |

---

### Instalasi Backend

Tahapan deployment backend meliputi:

1. Melakukan update package sistem Ubuntu.
2. Menginstal Python 3, Pip, dan Git.
3. Menginstal dependency aplikasi seperti Flask, PyMongo, Gunicorn, Gevent, Python-dotenv, Bcrypt, dan PyJWT.
4. Melakukan clone repository Final Project dari GitHub.
5. Melakukan pengujian koneksi database secara manual.
6. Menjalankan backend sebagai service menggunakan systemd dan Gunicorn.

Pendekatan ini memastikan seluruh komponen backend telah terpasang dan dapat berkomunikasi dengan database sebelum dijalankan sebagai service permanen.

---

### Verifikasi Koneksi Database

Sebelum deployment dilakukan, backend diuji secara manual menggunakan Flask Development Server.

URI koneksi yang digunakan:

```text
mongodb://<user>:<password>@localhost:27017/orderdb?authSource=admin
```

Karena MongoDB berada pada VM yang sama, komunikasi dilakukan melalui localhost sehingga menghasilkan latensi yang lebih rendah dibanding komunikasi melalui jaringan antar server.

Keberhasilan pengujian diverifikasi melalui endpoint `/products` yang berhasil mengembalikan data dalam format JSON.

---

### Deployment Menggunakan Gunicorn

Backend dijalankan menggunakan Gunicorn sebagai production WSGI server.

Konfigurasi yang digunakan:

| Parameter         | Nilai        |
| ----------------- | ------------ |
| Worker Type       | gthread      |
| Worker            | 3            |
| Thread per Worker | 4            |
| Total Thread      | 12           |
| Timeout           | 30 detik     |
| Binding Address   | 0.0.0.0:5000 |

Jumlah worker yang digunakan lebih sedikit dibanding Backend Server 1 karena VM ini juga menjalankan MongoDB sehingga resource sistem harus dibagi secara seimbang.

---

### Konfigurasi Systemd

Backend dikonfigurasi sebagai service bernama:

```text
flask-be2.service
```

Service ini dijalankan setelah MongoDB aktif dengan konfigurasi:

```ini
After=network.target mongod.service
```

Konfigurasi tersebut memastikan backend tidak berjalan sebelum database siap menerima koneksi.

Keuntungan penggunaan systemd antara lain:

* Service berjalan otomatis saat VM booting.
* Service melakukan restart otomatis apabila terjadi kegagalan.
* Backend tetap aktif meskipun sesi SSH ditutup.

---

### Analisis Konfigurasi Backend

Konfigurasi 3 worker dan 4 thread dipilih untuk menjaga keseimbangan resource antara Flask dan MongoDB yang berjalan pada VM yang sama.

Dengan total 12 thread aktif, backend tetap mampu melayani banyak request secara bersamaan tanpa mengganggu performa database. Pendekatan ini memberikan stabilitas yang lebih baik selama pengujian endpoint maupun load testing.

---

### Dokumentasi

<img width="1217" height="296" alt="WhatsApp Image 2026-06-17 at 22 36 49" src="https://github.com/user-attachments/assets/847d8a17-f642-4188-9a78-0ea404cf6400" />
<img width="1361" height="637" alt="WhatsApp Image 2026-06-17 at 22 36 54" src="https://github.com/user-attachments/assets/f8098d90-b1a7-4880-b77f-92b3dd3465b4" />

---

## 3.4 Implementasi Nginx Load Balancer (M6)

Nginx digunakan sebagai Load Balancer untuk mendistribusikan request dari client ke dua backend server yang berjalan secara paralel, yaitu Backend Server 1 (`vm-be1`) dan Backend Server 2 (`vm-be2`).

Arsitektur ini bertujuan untuk meningkatkan ketersediaan layanan (availability), membagi beban trafik (load distribution), serta mengurangi kemungkinan bottleneck pada satu server backend.

### Spesifikasi Infrastruktur

| VM     | Public IP      | Private IP | Port |
| ------ | -------------- | ---------- | ---- |
| vm-lb  | 48.193.46.109  | 10.0.0.4   | 80   |
| vm-be1 | 48.193.41.35   | 10.0.0.5   | 5000 |
| vm-be2 | 70.153.145.176 | 10.0.0.7   | 5000 |

---

### Instalasi Nginx

Nginx diinstal pada VM `vm-lb` menggunakan package manager Ubuntu.

Tahapan implementasi meliputi:

1. Clone repository Final Project.
2. Instalasi Nginx.
3. Konfigurasi upstream backend.
4. Konfigurasi reverse proxy.
5. Aktivasi virtual host.
6. Pengujian konfigurasi.
7. Aktivasi service Nginx.

---

### Konfigurasi Load Balancer

Nginx dikonfigurasi menggunakan mekanisme upstream yang mengarah ke dua backend server.

```nginx
upstream backend {
    server 10.0.0.5:5000;
    server 10.0.0.7:5000;
    keepalive 32;
}
```

Konfigurasi tersebut memungkinkan Nginx mendistribusikan request secara otomatis ke kedua backend server menggunakan metode round-robin bawaan Nginx.

---

### Reverse Proxy API

Semua endpoint API diarahkan ke backend cluster menggunakan konfigurasi:

```nginx
location ~ ^/(order|orders|auth|products|admin|health) {
    proxy_pass http://backend;
}
```

Pendekatan ini memungkinkan frontend dan backend diakses melalui domain/IP yang sama sehingga tidak memerlukan konfigurasi CORS tambahan.

Endpoint yang dilayani antara lain:

* `/auth`
* `/products`
* `/orders`
* `/order`
* `/admin`
* `/health`

---

### Optimasi Menggunakan Keepalive

Konfigurasi:

```nginx
keepalive 32;
```

digunakan untuk mempertahankan koneksi antara Nginx dan backend server sehingga koneksi tidak perlu dibuat ulang untuk setiap request.

Keuntungan konfigurasi ini:

* Mengurangi latency.
* Menurunkan overhead TCP connection.
* Meningkatkan Request Per Second (RPS).
* Mengurangi penggunaan CPU backend.

---

### Verifikasi Konfigurasi

Setelah konfigurasi selesai dibuat, dilakukan validasi menggunakan:

```bash
sudo nginx -t
```

Kemudian service dijalankan menggunakan:

```bash
sudo systemctl restart nginx
sudo systemctl enable nginx
```

Keberhasilan implementasi ditunjukkan dengan status service Nginx yang berada pada kondisi **active (running)**.

---

### Analisis Implementasi Load Balancer

Dengan adanya Nginx Load Balancer, request pengguna tidak lagi bergantung pada satu backend server.

Apabila salah satu backend mengalami beban tinggi, request berikutnya dapat diteruskan ke backend lainnya sehingga utilisasi resource menjadi lebih merata.

Selain itu, penggunaan private IP antar VM meningkatkan keamanan komunikasi karena trafik backend tidak melewati jaringan publik.

---

### Dokumentasi

#### Validasi Konfigurasi Nginx

*(Tambahkan screenshot hasil `sudo nginx -t` di sini)*

#### Status Service Nginx

*(Tambahkan screenshot hasil `sudo systemctl status nginx` di sini)*

#### Health Check Backend

*(Tambahkan screenshot hasil `curl http://localhost/health` di sini)*

---

## 3.5 Implementasi Frontend Deployment (M6)

Frontend aplikasi ditempatkan pada VM Load Balancer (`vm-lb`) dan disajikan langsung oleh Nginx melalui direktori web root.

Dengan pendekatan ini, seluruh layanan dapat diakses melalui satu alamat IP yang sama, yaitu:

```text
http://48.193.46.109
```

---

### Deployment Frontend

File frontend yang berasal dari repository dipindahkan ke direktori:

```text
/var/www/fp-tka/
```

Struktur file yang digunakan:

```text
/var/www/fp-tka/
├── index.html
└── styles.css
```

Nginx dikonfigurasi untuk menyajikan file tersebut sebagai halaman utama aplikasi.

---

### Integrasi dengan Backend API

Frontend telah dimodifikasi agar dapat berkomunikasi langsung dengan backend melalui Load Balancer.

Fitur yang tersedia meliputi:

* Login pengguna.
* Registrasi pengguna.
* Pencarian produk.
* Pembuatan pesanan.
* Menampilkan riwayat pesanan.
* Integrasi JWT Authentication.

Seluruh request API dikirim ke endpoint yang sama sehingga pengguna tidak perlu mengetahui lokasi backend yang sebenarnya.

---

### Pengujian Fungsionalitas

Pengujian dilakukan menggunakan akun administrator yang tersedia pada database:

| Parameter | Nilai                                               |
| --------- | --------------------------------------------------- |
| Email     | [admin1@tka.its.ac.id](mailto:admin1@tka.its.ac.id) |
| Password  | Admin@12345                                         |

Fitur yang diuji:

1. Login.
2. Registrasi akun.
3. Pencarian produk.
4. Pembuatan pesanan.
5. Menampilkan riwayat pesanan.

Seluruh fitur berhasil berjalan melalui Nginx Load Balancer dan terhubung dengan backend cluster.

---

### Analisis Implementasi Frontend

Dengan menempatkan frontend pada Nginx, pengguna hanya perlu mengakses satu alamat IP untuk menggunakan seluruh layanan aplikasi.

Pendekatan ini menyederhanakan deployment, mengurangi kompleksitas konfigurasi client, serta memberikan pengalaman pengguna yang lebih baik karena frontend dan backend berada dalam satu entry point yang sama.

---

### Dokumentasi

#### Halaman Login

*(Tambahkan screenshot halaman Login di browser di sini)*

#### Halaman Utama Setelah Login

*(Tambahkan screenshot dashboard atau halaman utama aplikasi di sini)*

#### Fitur Pembuatan Pesanan

*(Tambahkan screenshot form pembuatan pesanan di sini)*

#### Riwayat Pesanan

*(Tambahkan screenshot tampilan riwayat pesanan di sini)*

# 4. Hasil Pengujian Endpoint

Pengujian endpoint dilakukan menggunakan Postman untuk memastikan seluruh API dapat berfungsi sesuai dengan spesifikasi sistem. Pengujian mencakup autentikasi pengguna, pengelolaan produk, pengelolaan pesanan, endpoint administrator, serta health check service.

---

## 4.1 Pengujian Authentication Endpoint

### Login User

Endpoint login digunakan untuk melakukan autentikasi pengguna dan menghasilkan JWT token yang digunakan pada endpoint yang memerlukan otorisasi.

**Endpoint**

```http
POST /auth/login
```

### Dokumentasi

<img width="1037" height="802" alt="WhatsApp Image 2026-06-20 at 20 20 31" src="https://github.com/user-attachments/assets/140df773-c915-42a2-9986-9c1770348d35" />

---

## 4.2 Pengujian Product Endpoint

### Menampilkan Daftar Produk

Endpoint ini digunakan untuk menampilkan seluruh produk aktif yang tersedia pada database.

**Endpoint**

```http
GET /products
```

### Dokumentasi

<img width="1041" height="792" alt="WhatsApp Image 2026-06-20 at 20 20 31 (1)" src="https://github.com/user-attachments/assets/6367612b-3408-42f0-95c8-f4f23d56b1d8" />

---

## 4.3 Pengujian Order Endpoint

### Membuat Pesanan

Endpoint ini digunakan untuk membuat pesanan baru oleh pengguna yang telah terautentikasi.

**Endpoint**

```http
POST /orders
```

### Dokumentasi

<img width="1046" height="792" alt="image" src="https://github.com/user-attachments/assets/5843cf2a-6921-47a4-a241-38919d3ab85e" />

---

### Menampilkan Riwayat Pesanan

Endpoint ini digunakan untuk menampilkan daftar pesanan yang dimiliki pengguna.

**Endpoint**

```http
GET /orders
```

### Dokumentasi

<img width="1055" height="800" alt="WhatsApp Image 2026-06-20 at 20 20 31 (2)" src="https://github.com/user-attachments/assets/9e634bea-626d-4309-9ef3-f5f5f67161b6" />

---

## 4.4 Pengujian Admin Endpoint

### Statistik Sistem

Endpoint administrator digunakan untuk menampilkan statistik transaksi dan informasi sistem.

**Endpoint**

```http
GET /admin/stats
```

### Dokumentasi

<img width="1037" height="790" alt="WhatsApp Image 2026-06-20 at 20 20 32" src="https://github.com/user-attachments/assets/3ccdb44d-7f4b-4133-bfa5-efef0f541e5a" />

---

## 4.5 Pengujian Health Check Endpoint

Endpoint health check digunakan untuk memastikan backend service berjalan dengan baik dan dapat menerima request.

**Endpoint**

```http
GET /health
```

### Dokumentasi

<img width="1042" height="298" alt="WhatsApp Image 2026-06-20 at 20 20 32 (1)" src="https://github.com/user-attachments/assets/2223f360-5289-4e90-b455-56731ddc52d9" />

---

## 4.6 Pengujian Admin Endpoint

Endpoint administrator digunakan untuk menampilkan statistik sistem berdasarkan data transaksi yang tersimpan pada database MongoDB.

### Statistik Sistem

Endpoint ini menampilkan informasi agregasi seperti jumlah pesanan, status pesanan, dan data statistik lain yang digunakan oleh administrator untuk memantau kondisi sistem.

**Endpoint**

```http
GET /admin/stats
```

### Hasil Pengujian

Pengujian dilakukan menggunakan Postman dengan akun administrator yang telah berhasil melakukan autentikasi. Endpoint berhasil mengembalikan data statistik dengan status **200 OK**, menunjukkan bahwa proses agregasi data pada MongoDB berjalan dengan baik. Data yang ditampilkan mencakup jumlah transaksi dan total pendapatan dari berbagai kota yang tersimpan pada database.
### Dokumentasi

<img width="1037" height="790" alt="WhatsApp Image 2026-06-20 at 20 20 32 (2)" src="https://github.com/user-attachments/assets/2a086497-699b-4392-a44e-40d3aaf38913" />

---

# 5. Hasil Load Testing

Load testing dilakukan menggunakan **Locust** untuk mengevaluasi performa sistem ketika menerima beban pengguna secara bersamaan. Pengujian dilakukan pada lima skenario berbeda untuk mengukur response time, throughput, serta tingkat kegagalan request.

---

## 5.1 Skenario 1 (100 User)

### Hasil Pengujian

| Metrik | Nilai |
|---------|---------|
| Average Response Time | 23 ms (P50) |
| Requests per Second (RPS) | 37.43 |
| Failure Rate | 0% |

### Dokumentasi

<img width="1497" height="902" alt="WhatsApp Image 2026-06-20 at 20 41 47" src="https://github.com/user-attachments/assets/ccf960b6-7d3c-46bd-899b-b2317829b942" />

### Analisis

Pada pengujian dengan 100 pengguna simultan, sistem mampu mempertahankan throughput sebesar **37.43 request per second** tanpa kegagalan request. Nilai median response time berada pada kisaran **23 ms** dan menunjukkan bahwa seluruh komponen sistem, mulai dari Nginx Load Balancer, Flask Backend, hingga MongoDB, mampu menangani beban ringan dengan sangat baik.

---

## 5.2 Skenario 2 (300 User)

### Hasil Pengujian

| Metrik | Nilai |
|---------|---------|
| Average Response Time | 24 ms (P50) |
| Requests per Second (RPS) | 115.37 |
| Failure Rate | 0% |

### Dokumentasi

<img width="1393" height="903" alt="WhatsApp Image 2026-06-20 at 20 42 13" src="https://github.com/user-attachments/assets/72530134-89e9-4926-82ad-524d091963d9" />

### Analisis

Ketika jumlah pengguna meningkat menjadi **300 user**, throughput meningkat hingga **115.37 request per second** tanpa adanya request yang gagal. Nilai response time tetap rendah sehingga menunjukkan bahwa mekanisme load balancing dan konfigurasi backend masih mampu melakukan scaling secara efektif.

---

## 5.3 Skenario 3 (500 User)

### Hasil Pengujian

| Metrik | Nilai |
| :--- | :--- |
| **Average Response Time** | 26 ms (P50) / 190 ms (P95) |
| **Requests per Second (RPS)** | 182.96 |
| **Failure Rate** | 0% |

### Dokumentasi

<img width="1492" height="907" alt="WhatsApp Image 2026-06-20 at 20 42 46" src="https://github.com/user-attachments/assets/5934ab93-5f2b-44b5-af11-99283f48980a" />

### Analisis

Pada pengujian dengan beban maksimal mencapai **500 user**, sistem menunjukkan performa yang sangat stabil dan optimal. Throughput berhasil meningkat secara signifikan hingga mencapai **182.96 request per second (RPS)** tanpa adanya kegagalan sistem (*Failure Rate* tetap **0%**). 

Dari sisi pemrosesan, *Response Time* pada *50th percentile* (P50) berada di angka **26 ms**, sementara pada kondisi puncak atau *95th percentile* (P95) tercatat sebesar **190 ms** (mengalami perbaikan waktu respons yang signifikan setelah stabil, dari yang sebelumnya sempat menyentuh angka di atas 1500 ms pada awal proses *ramp-up*). Hal ini membuktikan bahwa infrastruktur backend dan mekanisme *load balancing* mampu menangani lonjakan beban tinggi (*high concurrency*) secara efisien serta mempertahankan konsistensi performa dengan sangat baik.

## 5.4 Skenario 4 (550 User)

### Konfigurasi Pengujian

| Parameter | Nilai |
|------------|---------|
| Jumlah User | 550 |
| Durasi Pengujian | 4 menit 15 detik |

### Hasil Pengujian

| Metrik | Nilai |
|---------|---------|
| Average Response Time | 151.76 ms |
| Requests per Second (RPS) | 201.4 |
| Failure Rate | 0% |

### Dokumentasi

<img width="1195" height="881" alt="WhatsApp Image 2026-06-19 at 07 59 43" src="https://github.com/user-attachments/assets/bdae3a9f-e675-4b0c-8c0e-b55bb4c246f8" />

### Analisis

Pada pengujian dengan **550 pengguna simultan**, sistem menghasilkan throughput sebesar **201.4 request per second** dengan tingkat kegagalan **0%**. Total request yang berhasil diproses mencapai **51.257 request** selama pengujian berlangsung.

Endpoint yang paling sering diakses adalah:

| Endpoint | Total Request |
|-----------|---------------|
| `/products?[filters]` | 28.021 |
| `/products/<id>` | 13.958 |
| `/admin/stats` | 3.477 |
| `/orders` | 2.596 |

Hasil ini menunjukkan bahwa sistem masih mampu menangani beban tinggi secara stabil dengan performa yang baik.

---

## 5.5 Skenario 5 (600 User)

### Hasil Pengujian

| Metrik | Nilai |
|---------|---------|
| Average Response Time | 95 ms (P50) |
| Requests per Second (RPS) | 193.1 |
| Failure Rate | 1% |

### Dokumentasi

<img width="1343" height="911" alt="WhatsApp Image 2026-06-20 at 20 45 26" src="https://github.com/user-attachments/assets/1e45ab87-e4b2-4e2a-9eca-b053e85e7cb4" />

### Analisis

Pada beban **600 pengguna simultan**, sistem masih mampu mempertahankan throughput mendekati **200 request per second**. Namun mulai muncul kegagalan request sebesar **1%**, yang menunjukkan bahwa sistem mulai mendekati batas kapasitas optimalnya.

Walaupun demikian, performa sistem masih tergolong baik karena sebagian besar request tetap dapat diproses dengan sukses.

---

## 5.6 Analisis Keseluruhan Load Testing

### Ringkasan Hasil

| Skenario | User | Avg Response Time | RPS | Failure Rate |
|----------|------|-------------------|------|--------------|
| 1 | 100 | 23 ms | 37.43 | 0% |
| 2 | 300 | 24 ms | 115.37 | 0% |
| 3 | 500 | Menunggu Data | Menunggu Data | Menunggu Data |
| 4 | 550 | 151.76 ms | 201.4 | 0% |
| 5 | 600 | 95 ms | 193.1 | 1% |

### Analisis

Berdasarkan seluruh pengujian yang dilakukan, sistem menunjukkan kemampuan scaling yang baik seiring bertambahnya jumlah pengguna simultan.

Pada skenario **100 hingga 300 pengguna**, response time tetap rendah dan tidak ditemukan kegagalan request. Saat jumlah pengguna meningkat menjadi **550 pengguna**, throughput mencapai lebih dari **200 request per second** dengan tingkat keberhasilan **100%**.

Pada skenario tertinggi yaitu **600 pengguna**, sistem masih mampu mempertahankan throughput yang tinggi meskipun mulai muncul kegagalan sebesar **1%**. Kondisi ini menunjukkan bahwa sistem mulai mendekati batas kapasitasnya namun masih mampu beroperasi secara stabil.

Secara keseluruhan, kombinasi **Nginx Load Balancer**, **Flask Backend (Gunicorn)**, dan **MongoDB** berhasil menangani ratusan pengguna secara bersamaan dengan tingkat reliabilitas yang tinggi. Hasil pengujian menunjukkan bahwa arsitektur cloud yang diimplementasikan telah memenuhi kebutuhan performa dan skalabilitas untuk skenario Order Processing Service.

# 6. Kesimpulan dan Saran

> Kesimpulan akhir akan disusun berdasarkan hasil implementasi dan load testing yang diperoleh selama proyek berlangsung.


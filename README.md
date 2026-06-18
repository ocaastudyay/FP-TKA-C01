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

*(Tambahkan dokumentasi/screenshot hasil implementasi MongoDB dari M3 di sini)*

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

*(Tambahkan dokumentasi/screenshot hasil implementasi Backend Server 1 dari M4 di sini)*


# 4. Hasil Pengujian Endpoint

> Dokumentasi Postman dan frontend akan ditambahkan setelah hasil dari M7 dikumpulkan.

---

# 5. Hasil Load Testing

> Hasil pengujian Locust untuk lima skenario akan ditambahkan setelah seluruh pengujian selesai dilakukan.

---

# 6. Kesimpulan dan Saran

> Kesimpulan akhir akan disusun berdasarkan hasil implementasi dan load testing yang diperoleh selama proyek berlangsung.








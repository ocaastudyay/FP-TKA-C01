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

> Dokumentasi implementasi MongoDB, Backend, dan Nginx akan ditambahkan setelah hasil dari M3, M4, M5, dan M6 dikumpulkan.

---

# 4. Hasil Pengujian Endpoint

> Dokumentasi Postman dan frontend akan ditambahkan setelah hasil dari M7 dikumpulkan.

---

# 5. Hasil Load Testing

> Hasil pengujian Locust untuk lima skenario akan ditambahkan setelah seluruh pengujian selesai dilakukan.

---

# 6. Kesimpulan dan Saran

> Kesimpulan akhir akan disusun berdasarkan hasil implementasi dan load testing yang diperoleh selama proyek berlangsung.








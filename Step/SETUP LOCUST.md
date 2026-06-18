# Menjalankan Load Testing dengan Locust

## 1. Install Locust

```powershell
pip install locust
```

Cek instalasi:

```powershell
pip show locust
```

Jika muncul informasi versi Locust, berarti berhasil terpasang.

---

## 2. Clone Repository

```powershell
git clone https://github.com/fuaddary/fp-tka-26.git
cd fp-tka-26/Resources/Test
```

---

## 3. Jalankan Locust

Karena `locust` tidak terbaca di PATH Windows, jalankan melalui Python:

```powershell
python -m locust -f locustfile.py --host=http://48.193.46.109
```

Jika `python` tidak dikenali:

```powershell
py -m locust -f locustfile.py --host=http://48.193.46.109
```

---

## 4. Buka Dashboard Locust

Buka browser:

```text
http://localhost:8089
```

Isi:

* Host: `http://48.193.46.109`
* Number of Users: sesuai skenario
* Spawn Rate: sesuai skenario

Klik **Start Swarming**.

---

## 5. Skenario Pengujian

| Skenario | Users | Spawn Rate | Durasi   |
| -------- | ----- | ---------- | -------- |
| 1        | 100   | 10         | 60 detik |
| 2        | 300   | 50         | 60 detik |
| 3        | 500   | 100        | 60 detik |
| 4        | 800   | 200        | 60 detik |
| 5        | 1000  | 500        | 60 detik |

> Jika muncul failure, turunkan jumlah user sampai kembali 0% failure.

---

## 6. Dokumentasi Hasil

Untuk setiap skenario ambil screenshot:

* Tab **Statistics**
* Tab **Charts**
* Tab **Failures**
* Resource VM (`htop`)

Catat:

* Total Requests
* Requests per Second (RPS)
* Average Response Time
* Failure Rate

---

## 7. Verifikasi Sistem

Pastikan website tetap dapat diakses:

```text
http://48.193.46.109
```

dan endpoint tetap merespons selama testing berlangsung.

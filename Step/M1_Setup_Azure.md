# Setup Azure — Member 1 (Ringkas)

Region: **Indonesia Central**. Resource group: `fp-tka-2026`. Arsitektur: **3 VM** (vm-be2 dual-role: Backend #2 + MongoDB).

---

## 1. Akun Azure Free Trial
- https://azure.microsoft.com/en-us/free → **Start free**
- Pakai email yang belum pernah daftar Azure (kalau akun Students sudah habis credit)
- Isi negara Indonesia, verifikasi HP (OTP), verifikasi kartu Visa/Mastercard (tidak dicharge)
- Cek di portal: Subscriptions → harus ada **"Free Trial"** dengan $200 credit aktif

## 2. Budget Alert (lakukan sebelum buat resource)
- Search **"Cost Management"** → **Budgets** → **+ Add**
- Name: `FP-TKA-Budget`, Reset: Monthly, Amount: `75`
- Tambah alert di 80% dan 100% → email kamu → **Create**

## 3. Resource Group
- Search **"Resource groups"** → **+ Create**
- Name: `fp-tka-2026`, Region: **Indonesia Central**
- Kalau region tidak muncul → pakai **Southeast Asia (Singapore)** sebagai alternatif

## 4. Virtual Network
- Search **"Virtual networks"** → **+ Create**
- Name: `vnet-fp-tka`, Region: Indonesia Central, RG: `fp-tka-2026`
- Address space & subnet: biarkan default (`10.0.0.0/16` / `10.0.0.0/24`)

## 5. Network Security Group
- Search **"Network security groups"** → **+ Create** → Name: `nsg-fp-tka`
- Tambahkan 4 inbound rules (Inbound security rules → + Add):

| Name | Port | Priority |
|------|------|----------|
| Allow-SSH | 22 | 100 |
| Allow-HTTP | 80 | 110 |
| Allow-Backend-5000 | 5000 | 120 |
| Allow-MongoDB | 27017 | 130 |

Semua: Source Any, Protocol TCP, Action Allow.

## 6. Buat 3 VM
Semua VM pakai: Ubuntu Server 22.04 LTS - x64 Gen2, RG `fp-tka-2026`, Region Indonesia Central, Auth: Password (`azureuser` / `TkaFP2026!Azure`), Availability: **No infrastructure redundancy required**, NIC NSG: **Advanced** → `nsg-fp-tka`, Disk: Standard SSD (default size OK).

| VM | Size | Fungsi |
|----|------|--------|
| `vm-lb` | Standard_B2ats_v2 (2 vCPU, 1GB) | Nginx LB + Frontend |
| `vm-be1` | Standard_B2als_v2 (2 vCPU, 4GB) | Flask Backend #1 (utama) |
| `vm-be2` | Standard_B2als_v2 (2 vCPU, 4GB) | Flask Backend #2 + MongoDB |

Kalau size tidak muncul → **See all sizes** → ketik nama size di search box.

Setelah tiap VM jadi → **catat Public IP** (vm-lb) dan **Private IP** (vm-be1, vm-be2) dari tab Networking.

Total estimasi: **~$70.29/bln** (budget $75).

## 7. SSH ke VM
```bash
ssh azureuser@<PUBLIC_IP_VM>
# Password: TkaFP2026!Azure
```
Atau via portal: buka VM → **Connect** → **Native SSH**.

## 8. Verifikasi Jaringan (dari vm-be1)
```bash
ping -c 3 <PRIVATE_IP_vm-be2>
ping -c 3 <PRIVATE_IP_vm-lb>
```
Semua reply → VNet & NSG OK ✅

## 9. IP Address Kelompok
```
vm-lb  Public: 48.193.46.109   Private: 10.0.0.4
vm-be1 Public: 48.193.41.35    Private: 10.0.0.5   ← Member 4 deploy (workers=5)
vm-be2 Public: 70.153.145.176  Private: 10.0.0.7   ← Member 3 setup MongoDB, Member 5 deploy (workers=3)

SSH: ssh azureuser@<PUBLIC_IP>  |  Password: TkaFP2026!Azure
MONGO_URI: mongodb://10.0.0.7:27017/?maxPoolSize=100&minPoolSize=10
```

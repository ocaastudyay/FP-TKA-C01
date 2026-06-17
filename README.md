# FP-TKA-C01
---
### Anggota Kelompok

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

### Tabel Spesifikasi VM
 
| VM | Size | vCPU | RAM | Role | Harga/bln |
|---|---|---|---|---|---|
| vm-lb | Standard_B2ats_v2 | 2 | 1 GB | Nginx Load Balancer | $7.81 |
| vm-be1 | Standard_B2als_v2 | 2 | 4 GB | Backend #1 (5 Gunicorn workers) | $31.24 |
| vm-be2 | Standard_B2als_v2 | 2 | 4 GB | Backend #2 (3 workers) + MongoDB | $31.24 |
| **Total** | | | | | **$70.29** |


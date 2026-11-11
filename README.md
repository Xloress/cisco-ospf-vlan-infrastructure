# Enterprise Network Infrastructure: Multi-Area OSPF & VLAN Routing

Dokumentasi implementasi dan perancangan infrastruktur jaringan skala enterprise berbasis **Dynamic Routing Open Shortest Path First (OSPFv2)** dan **Virtual Local Area Network (VLAN)** menggunakan simulator **Cisco IOL** pada platform **PNETLab**.

Proyek praktikum ini dikerjakan sebagai bagian dari mata kuliah *Infrastruktur & Layanan Jaringan Internet*, Program Studi D4 Teknik Telekomunikasi, Jurusan Teknik Elektro, **Politeknik Negeri Bandung (POLBAN)**.

---

## 📌 Ringkasan Proyek

* **Mata Kuliah:** Infrastruktur & Layanan Jaringan Internet
* **Dosen Pengampu:** Taufik Irfan, S.ST., M.T. & M. Soebagja, S.T., M.T.
* **Tim Pelaksana (Kelompok 2 / Kelas 3A-TNK):**
  1. Andrean Noviansyah (241344004) - *R-ISP-1 & R-CORE-1*
  2. **Azlan Abdurahman (241344006) - *R-CORE-3 & SW-HRD***
  3. Jihan Nabila Nurhidayat (241344013) - *R-CORE-2 & SW-IT*
  4. Keysa Kalam Setiadi (241344014) - *R-ISP-2 & SW-FINANCE*
* **Periode Praktikum:** 6 Oktober 2026 – 13 Oktober 2026
* **Platform & Tools:** PNETLab, Cisco IOL (IOS on Linux), VMware Workstation Pro

---

## 🛠️ Arsitektur & Peran Teknis

Dalam proyek ini, fokus tanggung jawab mencakup konfigurasi inti router dan switch lantai/divisi:

1. **R-CORE-3 (Core Router):**
   * Alokasi IP Loopback: `192.168.0.3/32`
   * Konfigurasi OSPF Area 0 (Backbone) dan interkoneksi ke Area 1.
   * Optimasi parameter OSPF: *router-id*, *hello-interval*, *dead-interval*, dan penyesuaian *cost metric*.
   * Manajemen tabel routing dinamis dan pengujian redundansi failover jalur link data.
2. **SW-HRD (Distribution / Access Switch):**
   * Konfigurasi VLAN segmentasi traffic pengguna HRD.
   * Trunking 802.1Q dan interkoneksi uplink ke router gateway.
   * Konfigurasi gateway redundansi dan verifikasi konektivitas end-to-end.

---

## 📂 Struktur Repositori

```text
├── docs/
│   └── Laporan_Praktikum_OSPF_VLAN_Kelompok2.pdf   # Laporan resmi lengkap praktikum
├── configs/
│   ├── R-CORE-3_Azlan.cfg                         # Running-configuration Router R-CORE-3
│   └── SW-HRD_Azlan.cfg                          # Running-configuration Switch SW-HRD
└── README.md
```

---

## 🔍 Hasil & Verifikasi
* Verifikasi tetangga OSPF (`show ip ospf neighbor`) berstatus `FULL/BDR` dan `FULL/DR`.
* Tabel routing dinamis terkonvergensi penuh (`show ip route ospf`).
* Pengujian ping dan traceroute antar segmen VLAN menunjukkan latensi stabil dengan mekanisme pengalihan rute otomatis (failover) saat salah satu link inti diputus.

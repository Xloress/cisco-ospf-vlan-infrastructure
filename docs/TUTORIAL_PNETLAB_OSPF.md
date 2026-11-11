# Panduan Lengkap Konfigurasi OSPFv2 Multi-Area & VLAN di PNETLab (Step-by-Step Pemula)

Panduan praktis ini dirancang untuk memandu mahasiswa atau insinyur jaringan pemula dalam mereplikasi dan mengonfigurasi arsitektur jaringan enterprise bertingkat (*Core, Distribution, Access*) menggunakan image **Cisco IOL (IOS on Linux)** pada platform **PNETLab** atau **EVE-NG**.

---

## 🛠️ Persiapan Lingkungan Laboratorium

1. **Platform Simulator:** PNETLab v5+ atau EVE-NG Community Edition yang terpasang pada VMware Workstation Pro.
2. **Image Perangkat yang Digunakan:**
   * **Cisco IOL L3 (Router):** `i86bi-linux-l3-adventerprisek9-ms.155-2.T.bin`
   * **Cisco IOL L2 (Switch):** `i86bi-linux-l2-adventerprisek9-m.SSA.high_iron_20180510.bin`
   * **Client Endpoints:** VPCS (*Virtual PC Simulator*)

---

## 🗺️ Skema Pengalamatan IP & Alokasi VLAN

| Segmen / Subnet | Fungsi Jaringan | Area OSPF | Default Gateway |
|:---|:---|:---:|:---|
| `192.168.0.0/24` | Jaringan Antar-Router Inti (Inter-Core Mesh) | **Area 0** | N/A (P2P / Broadcast) |
| `192.168.11.0/24` | VLAN 11 (Divisi HRD - Staff Lantai 1) | **Area 1** | `192.168.11.254` |
| `192.168.12.0/24` | VLAN 12 (Divisi HRD - Manajerial) | **Area 1** | `192.168.12.254` |
| `192.168.13.0/24` | VLAN 13 (Divisi HRD - Ruang Rapat) | **Area 1** | `192.168.13.254` |
| `192.168.14.0/24` | VLAN 14 (Divisi IT Infrastructure) | **Area 2** | `192.168.14.254` |
| `192.168.16.0/24` | VLAN 16 (Divisi Finance & Akuntansi) | **Area 2** | `192.168.16.254` |

---

## 📝 Langkah Konfigurasi Step-by-Step

### Langkah 1: Konfigurasi Router Core ABR (`R-CORE-3`)

Masuk ke mode konfigurasi global pada router `R-CORE-3`:

```cisco
enable
configure terminal
hostname R-CORE-3
enable secret admin
username admin privilege 15 secret admin
ip domain name telmat.net

! 1. Konfigurasi Loopback untuk OSPF Router-ID
interface Loopback0
 ip address 192.168.0.3 255.255.255.255
 no shutdown

! 2. Konfigurasi Antarmuka Uplink Backbone (Area 0)
interface Ethernet0/0
 description Ke R-CORE-1 (Backbone Mesh)
 ip address 192.168.113.2 255.255.255.252
 ip ospf priority 100
 no shutdown

! 3. Konfigurasi Antarmuka Downlink Menuju Switch HRD (Area 1)
interface Ethernet0/1
 description Ke SW-HRD (Distribution Link)
 ip address 192.168.33.1 255.255.255.252
 no shutdown

! 4. Mengaktifkan Routing Protokol OSPF Multi-Area
router ospf 123
 router-id 192.168.0.3
 log-adjacency-changes
 network 192.168.0.3 0.0.0.0 area 0
 network 192.168.113.0 0.0.0.3 area 0
 network 192.168.33.0 0.0.0.3 area 1
 exit
```

---

### Langkah 2: Konfigurasi Switch Layer 3 Departemen (`SW-HRD`)

Konfigurasi Switch distribusi HRD untuk mengaktifkan perutean Inter-VLAN (*Switch Virtual Interface / SVI*) dan propagasi rute ke OSPF:

```cisco
enable
configure terminal
hostname SW-HRD
ip routing
enable secret admin
username admin privilege 15 secret admin

! 1. Definisi Basis Data VLAN
vlan 11
 name HRD_Staff
vlan 12
 name HRD_Manager
vlan 13
 name HRD_MeetingRoom
exit

! 2. Konfigurasi Uplink Routed Port Menuju Core Router
interface Ethernet0/0
 description Uplink ke R-CORE-3
 no switchport
 ip address 192.168.33.2 255.255.255.252
 no shutdown

! 3. Konfigurasi Gateway SVI Inter-VLAN
interface Vlan11
 ip address 192.168.11.254 255.255.255.0
 no shutdown

interface Vlan12
 ip address 192.168.12.254 255.255.255.0
 no shutdown

interface Vlan13
 ip address 192.168.13.254 255.255.255.0
 no shutdown

! 4. Mengarahkan Port Akses ke PC Klien
interface Ethernet1/0
 switchport mode access
 switchport access vlan 13
 spanning-tree portfast edge
 no shutdown

! 5. Advertisikan Subnet VLAN ke OSPF Area 1
router ospf 123
 router-id 192.168.0.13
 network 192.168.33.0 0.0.0.3 area 1
 network 192.168.11.0 0.0.0.255 area 1
 network 192.168.12.0 0.0.0.255 area 1
 network 192.168.13.0 0.0.0.255 area 1
 exit
```

---

## 🔍 Checklist Verifikasi & Perintah Troubleshooting

Jalankan perintah ini di CLI Cisco untuk memastikan seluruh link berjalan sempurna:

1. **Periksa Tetangga OSPF (Wajib Berstatus FULL):**
   ```cisco
   R-CORE-3# show ip ospf neighbor
   ```
   *Jika muncul status `FULL/DR` atau `FULL/BDR`, berarti sinkronisasi Link-State telah sukses.*

2. **Periksa Pangkalan Data Topologi (LSDB):**
   ```cisco
   R-CORE-3# show ip ospf database
   ```

3. **Periksa Tabel Routing:**
   ```cisco
   SW-HRD# show ip route ospf
   ```
   *Rute dari area lain akan ditandai dengan kode `O IA` (OSPF Inter-Area).*

4. **Uji Konektivitas Klien & Redundansi Failover:**
   Dari terminal client `PC1`, jalankan ping kontinu menuju server IT:
   ```bash
   PC1> ping 192.168.16.101 -t
   ```
   *Coba putuskan salah satu kabel core di PNETLab (klik kanan link -> disconnect). Amati bahwa ping hanya putus 1–2 kali (*request timed out*) lalu jalur otomatis beralih lewat router cadangan!*

# cisco-packet-tracer-lab-infrastruktur-jaringan-kantor-
Pembuatan infrastruktur jaringan sebuah kantor yang memaksimalkan keamanan 3 lapis yang terdiri dari Network Segmentation (VLAN &amp; Inter-VLAN Routing), Traffic Control (Extended ACL), Physical Edge Security (Port Security), dan Centralized Management (AAA RADIUS).

````markdown
# PROJECT CISCO PACKET TRACER  
## Infrastruktur Jaringan Kantor (Level Enterprise)

## 📌 Deskripsi Project
Project ini merupakan simulasi infrastruktur jaringan kantor level enterprise menggunakan Cisco Packet Tracer.  
Topologi dirancang dengan konsep:

- Redundancy
- High Availability
- Routing Layer 3
- VLAN Segmentation
- HSRP Failover
- OSPF Dynamic Routing
- ACL Security
- Port Security
- AAA RADIUS Authentication

Tujuan utama project ini adalah membangun jaringan kantor yang:
- Stabil
- Aman
- Memiliki jalur cadangan
- Mampu melakukan failover otomatis
- Mendukung manajemen jaringan enterprise

---

# 1. Penyusunan Topologi

## 🔹 Edge / WAN Layer
### Perangkat:
- 2x Cisco Router 2911

### Fungsi:
Bertindak sebagai gerbang jaringan kantor cabang menuju:
- Kantor pusat
- ISP / jaringan eksternal

Penggunaan 2 router bertujuan untuk simulasi:
- Jalur utama
- Jalur backup (redundancy)

---

## 🔹 Core / Distribution Layer
### Perangkat:
- 2x Cisco Multilayer Switch Layer 3 seri 3650

### Fungsi:
Sebagai pusat utama jaringan yang bertugas untuk:
- Inter-VLAN Routing
- Menjalankan OSPF
- Menjalankan HSRP
- Routing berkecepatan tinggi antar departemen

Konfigurasi HSRP memungkinkan:
- Switch cadangan otomatis aktif jika switch utama gagal

---

## 🔹 Access Layer
### Perangkat:
- 3x Cisco Switch Layer 2 seri 2960

### Fungsi:
Sebagai penghubung langsung perangkat pengguna:
- Zona Teller
- Zona Back Office
- Zona Server Lokal

Fitur keamanan:
- Port Security

---

## 🔹 End Device & Server
### Perangkat:
- 6x PC Desktop
- 1x Server-PT

### Fungsi:
PC digunakan untuk simulasi client jaringan.

Server digunakan sebagai:
- AAA Server
- RADIUS Server
- Authentication Center

---

## 🔹 Media Transmisi Kabel

### Kabel Straight-Through:
Digunakan untuk:
- PC → Switch L2
- Switch L2 → Server
- Switch L2 → Switch L3
- Switch L3 → Router

### Kabel Cross-Over:
Digunakan untuk:
- Switch L3 ↔ Switch L3

Tujuan:
- Jalur redundansi
- Backup koneksi antar core switch

---

# 2. Konfigurasi VLAN

## 📌 Membuat Database VLAN

Konfigurasi dilakukan pada:
- 2 Multilayer Switch L3
- 3 Switch Access L2

```bash
enable
configure terminal

vlan 10
name TELLER
exit

vlan 20
name BACK_OFFICE
exit

vlan 30
name SERVER_LOCAL
exit
```

## 📌 Penjelasan
Seluruh switch wajib memiliki database VLAN yang sama agar:
- Trafik VLAN dapat diteruskan
- Trunk berjalan normal
- Inter-VLAN dapat bekerja

---

# 3. Verifikasi VLAN

```bash
show vlan brief
```

## 📌 Hasil yang Diharapkan
Status VLAN:
- VLAN 10 → TELLER
- VLAN 20 → BACK_OFFICE
- VLAN 30 → SERVER_LOCAL

Semua harus berstatus:
```bash
active
```

---

## 📌 Catatan STP (Spanning Tree Protocol)

Jika terdapat kabel berwarna orange:
- STP sedang bekerja
- Mencegah looping
- Menyiapkan jalur backup otomatis

Ketika jalur utama gagal:
- Jalur orange akan berubah menjadi hijau

---

# 4. Konfigurasi Trunking

## 📌 Trunk pada Switch Access L2

```bash
interface range fa0/3-4
switchport mode trunk
description UPLINK_TO_CORE
```

---

## 📌 Trunk pada Multilayer Switch L3

```bash
interface range gig1/0/1-6
switchport
switchport mode trunk
description LINK_TO_ACCESS_AND_CORE
```

---

# 5. Konfigurasi EtherChannel

## 📌 Switch1_L3

```bash
interface range gig1/0/4-5
channel-group 1 mode desirable
```

## 📌 Switch2_L3

```bash
interface range gig1/0/2-3
channel-group 1 mode desirable
switchport mode trunk
```

---

## 📌 Verifikasi Neighbor

```bash
show cdp neighbors
```

## 📌 Verifikasi Trunk

```bash
show interfaces trunk
```

---

# 6. Pencegahan VLAN Hopping

Jika terdapat port dengan mode:
```bash
auto
```

Maka ubah menjadi:

```bash
switchport mode trunk
switchport nonegotiate
```

## 📌 Tujuan
Mencegah serangan:
- VLAN Hopping
- Fake DTP Negotiation

---

# 7. Inter-VLAN Routing (SVI)

## 📌 Mengaktifkan Routing Layer 3

```bash
enable
configure terminal
ip routing
```

---

## 📌 Membuat SVI Gateway

```bash
interface vlan 10
ip address 192.168.10.1 255.255.255.0
no shutdown

interface vlan 20
ip address 192.168.20.1 255.255.255.0
no shutdown

interface vlan 30
ip address 192.168.30.1 255.255.255.0
no shutdown
```

---

# 8. Konfigurasi IP PC

## VLAN 10 - Teller
- IP: 192.168.10.10
- Gateway: 192.168.10.1

## VLAN 20 - Back Office
- IP: 192.168.20.10
- Gateway: 192.168.20.1

## VLAN 30 - Server
- IP: 192.168.30.10
- Gateway: 192.168.30.1

---

# 9. Static Routing

## 📌 Switch1_L3

```bash
interface gig1/0/3
no switchport
ip address 10.10.10.2 255.255.255.252
no shutdown

ip route 0.0.0.0 0.0.0.0 10.10.10.1
```

## 📌 Switch2_L3

```bash
interface gig1/0/5
no switchport
ip address 10.10.20.2 255.255.255.252
no shutdown

ip route 0.0.0.0 0.0.0.0 10.10.20.1
```

---

# 10. HSRP (Hot Standby Router Protocol)

## 📌 Tujuan
Membuat gateway virtual agar:
- Failover otomatis
- Gateway tetap sama
- Tidak perlu mengubah setting PC

## 📌 Virtual Gateway
| VLAN | Virtual IP |
|---|---|
| VLAN 10 | 192.168.10.1 |
| VLAN 20 | 192.168.20.1 |
| VLAN 30 | 192.168.30.1 |

---

## 📌 Konfigurasi HSRP Switch1_L3

```bash
interface vlan 10
ip address 192.168.10.2 255.255.255.0
standby 10 ip 192.168.10.1
standby 10 priority 110
standby 10 preempt
```

---

## 📌 Verifikasi HSRP

```bash
show standby brief
```

---

# 11. Konfigurasi OSPF

## 📌 Aktivasi OSPF

```bash
router ospf 1
network 10.10.10.0 0.0.0.3 area 0
network 192.168.10.0 0.0.0.255 area 0
network 192.168.20.0 0.0.0.255 area 0
network 192.168.30.0 0.0.0.255 area 0
```

---

## 📌 Verifikasi OSPF

```bash
show ip route
```

Status route OSPF akan muncul dengan kode:

```bash
O
```

---

# 12. Extended ACL

## 📌 Tujuan
Membatasi akses antar VLAN.

## 📌 Aturan ACL
- VLAN Teller boleh akses Web Server
- VLAN Teller tidak boleh ping Server
- VLAN Teller tidak boleh akses Back Office

---

## 📌 Konfigurasi ACL

```bash
access-list 100 permit tcp 192.168.10.0 0.0.0.255 192.168.30.0 0.0.0.255 eq 80
access-list 100 deny icmp 192.168.10.0 0.0.0.255 192.168.30.0 0.0.0.255
access-list 100 deny ip 192.168.10.0 0.0.0.255 192.168.20.0 0.0.0.255
access-list 100 permit ip any any
```

---

# 13. Port Security

## 📌 Tujuan
Mencegah perangkat asing terhubung ke switch access.

---

## 📌 Konfigurasi

```bash
interface range fa0/1-2
switchport mode access
switchport port-security
switchport port-security maximum 1
switchport port-security mac-address sticky
switchport port-security violation shutdown
```

---

## 📌 Verifikasi

```bash
show port-security interface fa0/1
```

---

# 14. AAA & RADIUS Authentication

## 📌 Tujuan
Membuat autentikasi login administrator secara terpusat.

---

## 📌 Konfigurasi AAA di Switch

```bash
aaa new-model
radius-server host 192.168.30.10 key rahasiaBI
aaa authentication login default group radius local

username admin_local privilege 15 password superadmin

line vty 0 4
login authentication default
```

---

# 15. Maintenance & Monitoring

## 📌 Monitoring OSPF

```bash
show ip ospf neighbor
```

## 📌 Monitoring HSRP

```bash
show standby brief
```

## 📌 Monitoring EtherChannel

```bash
show etherchannel summary
```

---

# 📊 Teknologi yang Digunakan

- VLAN
- Inter-VLAN Routing
- EtherChannel
- STP
- HSRP
- OSPF
- ACL
- Port Security
- AAA Authentication
- RADIUS Server
- Cisco Catalyst 3650
- Cisco Catalyst 2960
- Cisco Router 2911

---

# 🎯 Hasil Akhir Project

Project berhasil membangun:
- Infrastruktur jaringan enterprise
- Jalur redundansi
- Failover otomatis
- Dynamic Routing
- Segmentasi jaringan
- Sistem keamanan jaringan enterprise
- AAA Authentication terpusat

---

# 👨‍💻 Author

Nama: M. HAZIMI RAMIZAN, S.Tr.Kom. 
Project: Cisco Packet Tracer Enterprise Network Infrastructure
````


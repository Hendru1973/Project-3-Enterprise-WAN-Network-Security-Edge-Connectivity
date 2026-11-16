# Project 3: Enterprise WAN, Network Security & Edge Connectivity

Simulasi dan implementasi arsitektur jaringan *multi-site enterprise* menggunakan Cisco Packet Tracer yang menghubungkan Kantor Pusat (**HQ**) dan Kantor Cabang (**Branch**) melalui infrastruktur ISP publik dengan mengamankan pertukaran data internal menggunakan terowongan **Site-to-Site IPsec VPN (AES-256)** serta menyediakan akses internet melalui **NAT/PAT Overload**.

![Network Topology](<img width="976" height="697" alt="Topologi" src="https://github.com/user-attachments/assets/a408eea1-d9a7-444f-8919-2a1ac42384db" />
.png)

---

## Arsitektur & Perencanaan IP

### Tabel Pengalamatan Perangkat
| Perangkat | Tipe | Interface | Alamat IP / Subnet | Keterangan |
| :--- | :--- | :--- | :--- | :--- |
| **R1-HQ** | Cisco ISR4331 | `Gig0/0/0.10` | `192.168.10.1 /24` | Gateway VLAN 10 (Staff-HQ) |
| **R1-HQ** | Cisco ISR4331 | `Gig0/0/0.30` | `192.168.30.1 /24` | Gateway VLAN 30 (Server-HQ) |
| **R1-HQ** | Cisco ISR4331 | `Gig0/0/1` | `200.10.10.1 /30` | WAN Public Interface (Outside) |
| **R-ISP** | Cisco 2911 | `Gig0/1` | `200.10.10.2 /30` | Koneksi ke R1-HQ |
| **R-ISP** | Cisco 2911 | `Gig0/0` | `200.20.20.1 /30` | Koneksi ke R2-Branch |
| **R-ISP** | Cisco 2911 | `Gig0/2` | `8.8.8.1 /24` | Gateway Simulasi Internet |
| **R2-Branch** | Cisco ISR4331 | `Gig0/0/1` | `200.20.20.2 /30` | WAN Public Interface (Outside) |
| **R2-Branch** | Cisco ISR4331 | `Gig0/0/0` | `192.168.20.1 /24` | Gateway Branch LAN |
| **PC-HQ** | End Device | `FastEthernet0` | `192.168.10.10 /24` | Klien VLAN 10 (`GW: 192.168.10.1`) |
| **Server-HQ** | Server | `FastEthernet0` | `192.168.30.2 /24` | Server Internal HQ (`GW: 192.168.30.1`) |
| **PC-BRANCH** | End Device | `FastEthernet0` | `192.168.20.10 /24` | Klien Kantor Cabang (`GW: 192.168.20.1`) |
| **Server-Internet** | Server | `FastEthernet0` | `8.8.8.8 /24` | Simulasi Server Publik (`GW: 8.8.8.1`) |

---

## Fitur Utama yang Diimplementasikan

1. **Inter-VLAN Routing (*Router-on-a-Stick*):**
   - Segmentasi jaringan lokal HQ menjadi **VLAN 10 (Staff-HQ)** dan **VLAN 30 (Server-HQ)** pada `Switch-HQ` dengan enkapsulasi `802.1Q` menuju sub-interface `R1-HQ`.
2. **Site-to-Site IPsec VPN (Enkripsi Antar-Kantor):**
   - **IKE Phase 1 (ISAKMP):** Enkripsi `AES-256`, hashing `SHA`, Diffie-Hellman `Group 2`, autentikasi `Pre-Shared Key`.
   - **IKE Phase 2 (IPsec Transform Set):** `esp-aes 256` dan `esp-sha-hmac` dalam mode `Tunnel` untuk mengamankan trafik antar-subnet privat (`192.168.10.0/24` & `192.168.30.0/24` <-> `192.168.20.0/24`).
3. **Edge Connectivity & NAT/PAT Overload:**
   - Menerjemahkan alamat IP privat klien internal menjadi alamat IP publik interface WAN (`ip nat inside source list NAT_ACL interface GigabitEthernet0/0/1 overload`) dengan pengecualian (`deny`) pada trafik menuju subnet VPN agar tidak bentrok dengan proses enkripsi.

---

## Verifikasi & Pengujian

- **End-to-End Connectivity (`ping`):** Seluruh host antar-VLAN, antar-site (via VPN), dan menuju publik (`8.8.8.8`) berhasil terhubung (*Reply 100%*).
- <img width="596" height="422" alt="Screenshot 2026-11-17 003546" src="https://github.com/user-attachments/assets/233f0732-d381-4909-9241-81575caeaadc" />
-<img width="581" height="341" alt="Screenshot 2026-11-17 003728" src="https://github.com/user-attachments/assets/721ae78f-3e86-43a0-b996-c3954aff2cdc" />
-<img width="532" height="378" alt="Screenshot 2026-11-17 003819" src="https://github.com/user-attachments/assets/4f1825df-585d-42c3-8777-90725a8cf1f3" />


- **NAT Translation (`show ip nat translations`):** Terbukti alamat lokal `192.168.10.10` ditranslasikan secara dinamis ke IP publik `200.10.10.1` menggunakan port PAT unik.
- <img width="643" height="175" alt="Screenshot 2026-11-17 004619" src="https://github.com/user-attachments/assets/b42c1f06-4ad3-4dc1-86df-ee66eb4a6aa8" />

- **IPsec Security Association (`show crypto ipsec sa`):** Terowongan berstatus `ACTIVE` dengan penghitung paket terenkripsi (`#pkts encaps` & `#pkts decaps`) meningkat dua arah menggunakan `esp-aes 256`.
- <img width="563" height="125" alt="Screenshot 2026-11-17 005122" src="https://github.com/user-attachments/assets/48f2123e-9a6a-4a9c-a043-8c908ed3dc73" />
-<img width="596" height="501" alt="Screenshot 2026-11-17 005913" src="https://github.com/user-attachments/assets/3a7ae7fa-8c5d-424b-99d2-3e423597c3f3" />


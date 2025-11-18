# **PANDUAN PROYEK MAHASISWA: Open5GS Deployment dengan K3s**

**Program Studi:** Sarjana Ilmu Komputer
**Topik:** Implementasi dan Testing 5G Core Network
**Tahun Akademik:** 2025
**Repository Acuan:** [https://github.com/rayhanegar/Open5GS-Testbed](https://github.com/rayhanegar/Open5GS-Testbed)

---

## 📘 **Daftar Isi**

1. Pendahuluan
2. Tujuan Pembelajaran
3. Prasyarat
4. Arsitektur Sistem
5. Instalasi dan Setup
6. Verifikasi Deployment
7. Tugas 1: Konektivitas Dasar
8. Tugas 2: Analisis Packet dengan Wireshark
9. Tugas 3: Sequence Diagram
10. Troubleshooting
11. Referensi

---

## 📌 **Pendahuluan**

Panduan ini memandu mahasiswa dalam melakukan deployment **Open5GS** (5G Core Network open-source) menggunakan **Kubernetes (K3s)** dengan **Calico CNI**.

Mahasiswa akan mempelajari:

* Konsep arsitektur **5G SA (Standalone)**
* Deployment **microservices** menggunakan Kubernetes
* Networking dalam lingkungan containerized
* **Protocol analysis** untuk 5G
* Troubleshooting infrastruktur jaringan

---

## 📡 **Apa itu Open5GS?**

Open5GS adalah implementasi open-source dari **5G Core Network (5GC)** sesuai standar 3GPP.

Komponen utama Network Function (NF):

| NF       | Fungsi                    |
| -------- | ------------------------- |
| **AMF**  | Registrasi & mobilitas UE |
| **SMF**  | Manajemen sesi PDU        |
| **UPF**  | Forwarding traffic UE     |
| **NRF**  | Service discovery         |
| **AUSF** | Authentication            |
| **UDM**  | Data subscriber           |
| **UDR**  | Repository subscriber     |
| **PCF**  | QoS & policy control      |
| **NSSF** | Network slice selection   |
| **SCP**  | Routing antar NF          |

---

## 🟦 **Apa itu K3s?**

K3s adalah distribusi Kubernetes yang **ringan**, cocok untuk edge computing & testbed.

Mahasiswa akan belajar:

* Orchestration container
* Deployment aplikasi terdistribusi
* Workflow DevOps
* Networking Kubernetes

---

## 🎯 **Tujuan Pembelajaran**

### **Pengetahuan**

* Memahami arsitektur 5G SA
* Menjelaskan fungsi NF
* Memahami network slicing
* Memahami protokol 5G (NGAP, NAS, GTP-U)

### **Keterampilan**

* Deployment Open5GS di K3s
* Konfigurasi jaringan 5G
* Packet analysis via Wireshark
* Membuat sequence diagram
* Troubleshooting jaringan

### **Kompetensi**

* Infrastructure as Code
* Cloud-native deployment
* Network debugging
* Technical documentation

---

## 🖥️ **Prasyarat**

### **Hardware**

**Minimum:**

* 2 CPU
* 4 GB RAM
* 20 GB storage

**Recommended:**

* 4+ CPU
* 8 GB RAM
* 50+ GB storage

### **Software**

* Ubuntu 22.04 / 24.04
* K3s
* Docker / Containerd
* kubectl
* Wireshark
* git, curl

### **Pengetahuan Dasar**

* Linux CLI
* Networking dasar
* Docker
* YAML
* REST APIs

### **Akses Sistem**

```bash
sudo whoami   # harus 'root'
git clone https://github.com/rayhanegar/Open5GS-Testbed.git
cd Open5GS-Testbed
```

---

## 🏗️ **Arsitektur Sistem**

```
┌────────────────────────────────────────────────────────────┐
│                      UERANSIM (External)                    │
│  gNB <──NGAP──> K3s AMF                                     │
│  gNB <──GTP-U──> K3s UPF                                    │
│  UE  <──N1/N3──> Open5GS Core                               │
│                                                            │
│          ┌────────── Control Plane NFs ──────────┐         │
│          │ AMF, SMF, NRF, AUSF, UDM, UDR, SCP,   │         │
│          │ PCF, NSSF                              │         │
│          └────────────────────────────────────────┘         │
│                                                            │
│          ┌──────────────────────┐                           │
│          │       UPF            │                           │
│          └──────────────────────┘                           │
└────────────────────────────────────────────────────────────┘
```

### **Static IP NF Deployment**

| Komponen | Pod   | IP          | Port       | Fungsi            |
| -------- | ----- | ----------- | ---------- | ----------------- |
| NRF      | nrf-0 | 10.10.0.10  | 7777       | Service discovery |
| SCP      | scp-0 | 10.10.0.200 | 7777       | Routing           |
| AMF      | amf-0 | 10.10.0.5   | 7777/38412 | UE registration   |
| SMF      | smf-0 | 10.10.0.4   | 7777       | Session mgmt      |
| UPF      | upf-0 | 10.10.0.7   | 2152       | User plane        |
| UDM      | ...   |             |            |                   |
| UDR      | ...   |             |            |                   |
| AUSF     | ...   |             |            |                   |
| PCF      | ...   |             |            |                   |
| NSSF     | ...   |             |            |                   |

---

## 🌐 **Network Slice Configuration**

| Slice | SST | DNN          | Subnet       | Gateway   |
| ----- | --- | ------------ | ------------ | --------- |
| eMBB  | 1   | embb.testbed | 10.45.0.0/24 | 10.45.0.1 |
| URLLC | 2   | urllc.v2x    | 10.45.1.0/24 | 10.45.1.1 |
| mMTC  | 3   | mmtc.testbed | 10.45.2.0/24 | 10.45.2.1 |

---

## ⚙️ **Instalasi dan Setup**

### **1. Update Sistem**

```bash
sudo apt update
sudo apt upgrade -y
sudo apt install -y curl git iptables iptables-persistent \
net-tools iputils-ping traceroute tcpdump wireshark wireshark-common

sudo mkdir -p /mnt/data/open5gs-logs
sudo chmod 777 /mnt/data/open5gs-logs
```

---

### **2. Setup K3s dengan Calico**

```bash
cd ~/Open5GS-Testbed/open5gs/open5gs-k3s-calico
chmod +x setup-k3s-environment-calico.sh
sudo ./setup-k3s-environment-calico.sh
```

### **Verifikasi**

```bash
sudo systemctl status k3s
kubectl get nodes
```

---

### **3. Build & Import Container Images**

```bash
chmod +x build-import-containers.sh
sudo ./build-import-containers.sh

sudo k3s crictl images
```

---

### **4. Deploy Open5GS**

```bash
chmod +x deploy-k3s-calico.sh
sudo ./deploy-k3s-calico.sh
kubectl get pods -n open5gs -w
```

---

## ✅ **Verifikasi Deployment**

### **Cek Semua NF**

```bash
kubectl get pods -n open5gs -o wide
kubectl logs -n open5gs amf-0
```

### **Verifikasi Static IP**

```bash
sudo ./verify-static-ips.sh
```

### **Verifikasi MongoDB**

```bash
sudo ./verify-mongodb.sh
```

---

# 🧪 **Tugas 1 — Konektivitas Dasar**

### **1. Setup UERANSIM**

Edit AMF IP di:

```
configs/open5gs-gnb-k3s.yaml
```

---

### **2. Jalankan gNB**

```bash
./build/nr-gnb -c configs/open5gs-gnb-k3s.yaml
```

### **3. Jalankan UE**

```bash
sudo ./build/nr-ue -c configs/open5gs-ue-embb.yaml
```

---

### **4. Test Connectivity**

```bash
ip addr show uesimtun0
ping -I uesimtun0 10.45.0.1
ping -I uesimtun0 8.8.8.8
curl --interface uesimtun0 -I https://www.google.com
```

---

### **5. Template Laporan Tugas**

```
## Tugas 1: Konektivitas Dasar
**Tanggal**:  
**Nama**:  
**Status K3s**:  

### gNB Registration
- Status: 
- Time: 
- AMF Connection:

### UE Registration
- Status:
- IMSI:
- TUN Interface:
- IP UE:

### Connectivity Tests
| Test | Result | RTT |
|------|--------|------|
| UPF Gateway | ... | ... |
| Internet | ... | ... |
| DNS | ... | ... |

### Issues
-  

### Resolution
-
```

---


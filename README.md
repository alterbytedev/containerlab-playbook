# 🚀 CONTAINERLAB EVPN SETUP GUIDE

### 1. INSTALL CONTAINERLAB
```bash
curl -sL [https://containerlab.dev/setup](https://containerlab.dev/setup) | sudo -E bash -s "all"
2. IMPORT ARISTA cEOS IMAGE
Bash
# Pastikan file .tar.xz ada di folder saat ini
docker import cEOS64-lab-4.32.0F.tar.xz ceos:4.32.0F
3. BUAT FILE TOPOLOGI (evpn-lab.yml)
Bash
cat <<EOF > evpn-lab.yml
name: evpn-lab

mgmt:
  network: clab-mgmt
  ipv4-subnet: 192.168.0.0/24

topology:
  nodes:
    # ===== Spine =====
    spine1:
      kind: ceos
      image: ceos:4.36.0F
      mgmt-ipv4: 192.168.0.11

    spine2:
      kind: ceos
      image: ceos:4.36.0F
      mgmt-ipv4: 192.168.0.12

    # ===== Leaf =====
    leaf1:
      kind: ceos
      image: ceos:4.36.0F
      mgmt-ipv4: 192.168.0.21

    leaf2:
      kind: ceos
      image: ceos:4.36.0F
      mgmt-ipv4: 192.168.0.22

    # ===== Clients =====
    client1:
      kind: linux
      image: alpine:latest
      mgmt-ipv4: 192.168.0.31

    client2:
      kind: linux
      image: alpine:latest
      mgmt-ipv4: 192.168.0.32

  links:
    # Spine ↔ Leaf
    - endpoints: ["spine1:eth1", "leaf1:eth1"]
    - endpoints: ["spine1:eth2", "leaf2:eth1"]
    - endpoints: ["spine2:eth1", "leaf1:eth2"]
    - endpoints: ["spine2:eth2", "leaf2:eth2"]

    # Leaf ↔ Client (data plane)
    - endpoints: ["leaf1:eth3", "client1:eth1"]
    - endpoints: ["leaf2:eth3", "client2:eth1"]
EOF
4. OPERASIONAL LAB
Bash
# DEPLOY: Menjalankan lab
sudo containerlab deploy -t evpn-lab.yml

# GRAPH: Melihat topologi di browser (Port 5001)
sudo containerlab graph -t evpn-lab.yml

# DESTROY: Mematikan lab
sudo containerlab destroy -t evpn-lab.yml

# CLEANUP: Mematikan lab & hapus semua log/folder sisa
sudo containerlab destroy -t evpn-lab.yml --cleanup
5. AKSES NODE
Bash
# Masuk ke CLI Arista via Docker
docker exec -it clab-evpn-lab-spine1 Cli

# SSH (User: admin | Pass: admin atau kosong)
ssh admin@192.168.0.11
# **traefik-wireguard-openvpn-stack**

**Self-hosted VPN infrastructure** using Docker, Traefik, WireGuard, and OpenVPN.

This repository documents a **production-style VPN stack** deployed on a single virtual machine using **Docker Compose**.

It combines:
- **Traefik** as a reverse proxy with automatic HTTPS (**Let’s Encrypt**)
- **WireGuard (wg-easy)** for modern VPN access
- **OpenVPN Access Server** for legacy compatibility

---

## **Architecture Overview**

The stack consists of the following components:

### **Traefik (v3.6.1)**
- Acts as a reverse proxy
- Terminates HTTPS using Let’s Encrypt (**HTTP-01 challenge**)
- Routes traffic based on domain names
- Proxies internal-only services securely

### **WireGuard (wg-easy)**
- Provides a modern VPN tunnel (`UDP 51820`)
- Web UI is **NOT exposed directly**
- UI is accessible **only via Traefik over HTTPS**

### **OpenVPN Access Server**
- Provides a traditional OpenVPN VPN
- Includes admin and client web portals
- Runs in parallel with WireGuard

### **Docker**
- All services run in containers
- Single bridge network: `vpn-net`

---

## **Virtual Machine Requirements**

This stack is designed to run on a **single Linux VM**.

### **Tested environment**
- OS: **Ubuntu 22.04 LTS**
- Architecture: **x86_64**
- Docker Engine + Docker Compose plugin

### **Minimum recommended specs**
- **1 vCPU** (2 recommended)
- **1–2 GB RAM**
- **20 GB disk**
- **Public IPv4 address (required)**
- **IPv6 recommended** (used for SSH hardening)

### **Network requirements**
- VM must be directly reachable from the internet
- Required ports must not be blocked by provider firewall or NAT

---

## **Network & Ports**

### **Publicly exposed ports**
- `80/tcp` → Traefik (Let’s Encrypt HTTP-01 challenge)
- `443/tcp` → Traefik HTTPS entrypoint
- `51820/udp` → WireGuard tunnel
- `1194/udp` → OpenVPN tunnel
- `943/tcp` → OpenVPN admin & client UI (**temporary**)

### **Internal-only ports**
- `51821/tcp` → WireGuard UI (proxied via Traefik)

---

## **DNS Setup (Required)**

To access the services securely over HTTPS, you **must own a domain name**  
(e.g. Namecheap, Cloudflare, GoDaddy).

This stack uses **hostname-based routing**, meaning Traefik decides where to send traffic
based on the domain name in the request.

### **Step 1: Choose a domain**

Example domain used below:

```text
**example.com**
```

### **Step 2: Create DNS A records

Create the following A records, all pointing to your VM’s public IPv4 address:

traefik.example.com → <VM_PUBLIC_IP>
wg-ui.example.com   → <VM_PUBLIC_IP>
vpn-ui.example.com  → <VM_PUBLIC_IP>

### **Step 3: Verify DNS resolution

Before starting Docker, verify DNS works:
```text
nslookup wg-ui.example.com
```
The hostname must resolve to your VM IP, otherwise:

TLS certificates will fail

Traefik routing will not work

### **Security Measures

UFW firewall enabled (default deny incoming)

Only required ports are allowed

SSH (22/tcp) restricted to a single IPv6 address

WireGuard UI is not exposed directly

Traefik handles TLS termination and routing

Docker socket mounted read-only

**Passwords and secrets are NOT stored in this repository **

### **Deployment

Step 1: Clone the repository (on the VM)

```text
git clone https://github.com/tseno-stamenov/traefik-wireguard-openvpn-stack
cd traefik-wireguard-openvpn-stack

'''
Step 2: Create required directories
'''text
mkdir -p letsencrypt wg-data openvpn-data
'''
Step 3: Start the stack
'''text
docker compose up -d
```

Step 4: Verify access
```text
https://traefik.example.com
https://wg-ui.example.com
https://vpn-ui.example.com:943
```
### **Repository Structure

```text
.
├── docker-compose.yml        # Main stack definition
├── README.md                 # Documentation
├── letsencrypt/              # TLS certificates (mounted into Traefik)
│   └── acme.json
├── wg-data/                  # WireGuard persistent data
├── openvpn-data/             # OpenVPN persistent data
└── scripts/
    └── setup-ufw.sh          # Firewall hardening script
```

### **Why this project exists**

This project was built to:

Learn reverse proxies and TLS automation

Compare WireGuard vs OpenVPN in practice

Practice Docker-based infrastructure design

Build a secure, self-hosted VPN solution

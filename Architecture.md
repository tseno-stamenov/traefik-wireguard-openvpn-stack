**## Architecture Overview **

The stack consists of the following components:

- **Traefik v3.6.1**
  - Acts as a reverse proxy
  - Terminates HTTPS using Let's Encrypt (HTTP-01)
  - Routes traffic based on domain names

- **WireGuard (wg-easy)**
  - Provides a modern VPN tunnel (UDP 51820)
  - Has a web UI that is NOT exposed directly
  - UI is accessible only through Traefik over HTTPS

- **OpenVPN Access Server**
  - Provides a traditional OpenVPN VPN
  - Includes admin and client web portals
  - Runs in parallel with WireGuard

- **Docker**
  - All services run in containers
  - Single bridge network (`vpn-net`)

**## Network & Ports**

Publicly exposed ports:

- 80/tcp   → Traefik (Let's Encrypt HTTP-01 challenge)
- 443/tcp  → Traefik HTTPS entrypoint
- 51820/udp → WireGuard tunnel
- 1194/udp → OpenVPN tunnel
- 943/tcp  → OpenVPN admin & client UI (temporary)

Internal-only ports:

- 51821/tcp → WireGuard UI (proxied via Traefik)

**## DNS Setup**

The following DNS records are required:

- traefik.example.com → VM public IP
- wg-ui.example.com   → VM public IP
- vpn-ui.example.com  → VM public IP

**## Security Measures**

- UFW firewall enabled (default deny incoming)
- Only required ports are allowed
- SSH (22/tcp) restricted to a single IPv6 address
- WireGuard UI is not exposed directly
- Traefik handles TLS and routing
- Docker socket mounted read-only

⚠️ Passwords and secrets are NOT stored in this repository.

**## Deployment**

1. Clone the repository on the VM:
   git clone https://github.com/tseno-stamenov/traefik-wireguard-openvpn-stack

2. Enter the directory:
   cd traefik-wireguard-openvpn-stack

3. Create required directories:
   mkdir -p letsencrypt wg-data openvpn-data

4. Start the stack:
   docker compose up -d

5. Verify:
   - https://traefik.example.com
   - https://wg-ui.example.com
   - https://<VM-IP>:943


**## Repository Structure**
.
├── docker-compose.yml
├── README.md
├── letsencrypt/
│   └── acme.json
├── wg-data/
│   └── (WireGuard configs)
├── openvpn-data/
│   └── (OpenVPN data)
└── scripts/
    └── setup-ufw.sh

**## Why this project exists**

This project was built to:
- Learn reverse proxies and TLS automation
- Compare WireGuard vs OpenVPN
- Practice Docker-based infrastructure
- Build a secure self-hosted VPN solution



# Docker Service Discovery with Cloudflare Tunnel

## Project Overview

This setup demonstrates:

* Docker Service Discovery
* Cloudflare Tunnel Integration
* Internal Docker DNS Communication
* Container-to-Container Communication
* Removing Static Private IP Dependency

Instead of using:

```bash
http://SERVER IP:3000
```

We use Docker internal DNS:

```bash
http://shubh-app:3000
```

---

# Architecture

```text
User
 ↓
eye.main.com
 ↓
Cloudflare
 ↓
Cloudflare Tunnel
 ↓
cloudflared container
 ↓
Docker Internal DNS
 ↓
shubh-app container
 ↓
Next.js Application
```

---

# Benefits of Service Discovery

| Static IP Method        | Service Discovery Method |
| ----------------------- | ------------------------ |
| IP can change           | Stable service name      |
| Hard to scale           | Easy scaling             |
| Manual management       | Automatic Docker DNS     |
| Not portable            | Portable                 |
| Difficult in production | Production-ready         |

---

# Prerequisites

Install Docker:

```bash
sudo apt update

sudo apt install docker.io docker-compose-v2 -y
```

Enable Docker:

```bash
sudo systemctl enable docker
sudo systemctl start docker
```

Verify:

```bash
docker --version
docker compose version
```

---

# Step 1 — Go To Project Directory

```bash
cd ~/shubh
```

---

# Step 2 — Edit Docker Compose File

```bash
nano docker-compose.yml
```

---

# Step 3 — Start Containers

```bash
docker compose up -d --build
```

---

# Step 4 — Verify Running Containers

```bash
docker ps
```

Expected:

```text
shubh-app
cloudflared-isr
```

---

# Step 5 — Verify Docker Network

```bash
docker network ls
```

Expected Network:

```text
isr-network
```

---

# Step 6 — Test Internal Service Discovery

Enter cloudflared container:

```bash
docker exec -it cloudflared-shubh sh
```

Test connectivity:

```bash
wget -qO- http://shubh-app:3000
```

OR

```bash
curl http://shubh-app:3000
```

If HTML response appears:

```text
Service discovery working successfully
```

Exit:

```bash
exit
```

---

# Step 7 — Configure Cloudflare Tunnel

Open:

[Cloudflare Zero Trust Dashboard](https://one.dash.cloudflare.com?utm_source=shubhsalunke)

Navigate:

```text
Networks
 → Tunnels
   → Your Tunnel
     → Public Hostnames
```

Add Public Hostname:

| Field     | Value        |
| --------- | ------------ |
| Subdomain | shubh          |
| Domain    | main.com     |
| Type      | HTTP         |
| URL       | shubh-app:3000 |

Final URL:

```text
http://shubh-app:3000
```

---

# Step 8 — Restart Tunnel Container

```bash
docker restart cloudflared-isr
```

---

# Step 9 — Check Tunnel Logs

```bash
docker logs -f cloudflared-shubh
```

Expected Output:

```text
Connected to Cloudflare
Tunnel established
Route propagating
```

---

# Step 10 — Open Website

```text
https://shubh.main.com
```

---

# Understanding Docker Service Discovery

Docker automatically creates internal DNS for containers connected to the same network.

Example:

```text
shubh-app
```

acts like:

```text
192.168.x.x
```

internally.

Docker resolves:

```text
isr-app
```

to actual container IP automatically.

No need to manually manage private IPs.

---

# Useful Commands

## Check Container IP

```bash
docker inspect isr-app
```

---

## Check Internal DNS Resolution

```bash
docker exec -it cloudflared-shubh nslookup shubh-app
```

---

## Test Ping Between Containers

```bash
docker exec -it cloudflared-isr ping shubh-app
```

---

## View Logs

```bash
docker logs -f shubh-app
```

```bash
docker logs -f cloudflared-shubh
```

---

# Final Result

You now have:

* Cloudflare Tunnel
* Docker Service Discovery
* Internal Docker DNS
* Production-style Container Communication
* No Static Private IP Dependency

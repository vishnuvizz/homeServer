# 🚀 Proxmox LXC Setup for Nextcloud AIO (Debian + Docker)

## 📦 Container Configuration

```bash
pct create 105 local-lvm \
  --hostname nextcloud-docker \
  --cores 2 \
  --memory 4096 \
  --swap 512 \
  --rootfs local-lvm:30 \
  --ostype debian \
  --net0 name=eth0,bridge=vmbr0,ip=192.168.0.61/24,gw=192.168.0.1 \
  --nameserver 192.168.0.51 \
  --unprivileged 0 \
  --features nesting=1,keyctl=1
```

> 🔐 **Note:** No password is set — use `pct exec` to access the container.

---

## ▶️ Start & Access the Container

```bash
pct start 105
pct exec 105 -- bash
```

---

## 🐳 Install Docker Inside the Container

```bash
apt update && apt install -y curl docker.io docker-compose
```

---

## ☁️ Install Nextcloud All-in-One

```bash
curl -fsSL https://get.nextcloud-aio.dev | bash
```

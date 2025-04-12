# 🚀 Proxmox LXC Setup for Nextcloud AIO (Debian + Docker)

## 📦 Container Configuration

```bash
pct create 105 local:vztmpl/debian-12-standard_12.7-1_amd64.tar.zst \
  --hostname nextcloud-docker \
  --cores 2 \
  --memory 4096 \
  --swap 512 \
  --rootfs local-lvm:30 \
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

## ☁️ Install Nextcloud All-in-One (skiped)

```bash
# For Linux and without a web server or reverse proxy (like Apache, Nginx, Caddy, Cloudflare Tunnel and else) already in place:
sudo docker run \
--init \
--sig-proxy=false \
--name nextcloud-aio-mastercontainer \
--restart always \
--publish 80:80 \
--publish 8080:8080 \
--publish 8443:8443 \
--volume nextcloud_aio_mastercontainer:/mnt/docker-aio-config \
--volume /var/run/docker.sock:/var/run/docker.sock:ro \
ghcr.io/nextcloud-releases/all-in-one:latest
```

**Note**: this command is taken from NextCloud's offical website[https://github.com/nextcloud/all-in-one]


## Docker compose method
- I used docker copose method as per this GitHub [issue](https://github.com/nextcloud/all-in-one/discussions/2845#discussioncomment-6423237)

```docker compose
version: "3.8"

services:
  nextcloud:
    image: ghcr.io/nextcloud-releases/all-in-one:latest
    restart: always
    container_name: nextcloud-aio-mastercontainer 
    volumes:
      - nextcloud_aio_mastercontainer:/mnt/docker-aio-config 
      - /var/run/docker.sock:/var/run/docker.sock:ro 
    ports:
      - 80:80 
      - 8080:8080
    environment: 
      - APACHE_PORT=11000 
      - APACHE_IP_BINDING=0.0.0.0
      - SKIP_DOMAIN_VALIDATION=true

volumes:
  nextcloud_aio_mastercontainer:
    name: nextcloud_aio_mastercontainer
```


- start containers
  `docker-compose up -d`

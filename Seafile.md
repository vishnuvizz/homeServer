# Seafile

## Seafile instll
- its installed via lxc


## access via traefik
- reason? seafile run by default in port `8000`, if we setup domain in seafile, then we cannot use it in local, so we routed LAN seafile traefik via Adguard DNS to point to traefik and then to seafile lxc.
- traefik.yml
```
entryPoints:
  web:
    address: ":80"

  websecure:
    address: ":443"

  traefik:
    address: ":8080"  # Dashboard port

api:
  dashboard: true
  insecure: true     # Optional: disables auth, good for local only

providers:
  file:
    filename: "/etc/traefik/dynamic.yml"
    watch: true

certificatesResolvers:
  cloudflare:
    acme:
      email: "e.vishnu55@gmail.com"
      storage: /etc/traefik/certs/acme.json
      dnsChallenge:
        provider: cloudflare
```

- dynamic.yml

```
root@traefik:/etc/traefik# cat dynamic.yml
http:
  routers:
    seafile:
      rule: "Host(`seafile.vishmohomes.click`)"
      entryPoints:
        - websecure
      service: seafile-svc
      tls:
        certResolver: cloudflare

  services:
    seafile-svc:
      loadBalancer:
        servers:
          - url: "http://192.168.0.63:8000"
```

- point traefik config with command `traefik --configFile=/etc/traefik/traefik.yml`


## Seafile lxc config

- open file - `root@proxmox:/etc/pve/lxc# cat 113.conf`
- 113.conf
```
#<div align='center'>
#  <a href='https%3A//Helper-Scripts.com' target='_blank' rel='noopener noreferrer'>
#    <img src='https%3A//raw.githubusercontent.com/community-scripts/ProxmoxVE/main/misc/images/logo-81x112.png' alt='Logo' style='width%3A81px;height%3A112px;'/>
#  </a>
#
#  <h2 style='font-size%3A 24px; margin%3A 20px 0;'>Seafile LXC</h2>
#
#  <p style='margin%3A 16px 0;'>
#    <a href='https%3A//ko-fi.com/community_scripts' target='_blank' rel='noopener noreferrer'>
#      <img src='https%3A//img.shields.io/badge/&#x2615;-Buy us a coffee-blue' alt='spend Coffee' />
#    </a>
#  </p>
#
#  <span style='margin%3A 0 10px;'>
#    <i class="fa fa-github fa-fw" style="color%3A #f5f5f5;"></i>
#    <a href='https%3A//github.com/community-scripts/ProxmoxVE' target='_blank' rel='noopener noreferrer' style='text-decoration%3A none; color%3A #00617f;'>GitHub</a>
#  </span>
#  <span style='margin%3A 0 10px;'>
#    <i class="fa fa-comments fa-fw" style="color%3A #f5f5f5;"></i>
#    <a href='https%3A//github.com/community-scripts/ProxmoxVE/discussions' target='_blank' rel='noopener noreferrer' style='text-decoration%3A none; color%3A #00617f;'>Discussions</a>
#  </span>
#  <span style='margin%3A 0 10px;'>
#    <i class="fa fa-exclamation-circle fa-fw" style="color%3A #f5f5f5;"></i>
#    <a href='https%3A//github.com/community-scripts/ProxmoxVE/issues' target='_blank' rel='noopener noreferrer' style='text-decoration%3A none; color%3A #00617f;'>Issues</a>
#  </span>
#</div>
arch: amd64
cores: 2
features: keyctl=1,nesting=1
hostname: seafile
memory: 2048
nameserver: 192.168.0.51
net0: name=eth0,bridge=vmbr0,gw=192.168.0.1,hwaddr=BC:24:11:D7:7C:29,ip=192.168.0.63/24,type=veth
onboot: 1
ostype: debian
rootfs: local-lvm:vm-113-disk-0,size=20G
swap: 512
tags: community-script;documents;seafile
unprivileged: 1
lxc.mount.entry: 192.168.0.60:/mnt/hdd2TBdisk/SeaFile /mnt/seafile nfs defaults 0 0
```

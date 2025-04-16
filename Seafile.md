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
middlewares:
  seafile-headers:
    headers:
      customRequestHeaders:
        X-Forwarded-Proto: "https"

http:
  routers:
    seafile:
      rule: "Host(`seafile.vishmohomes.click`)"
      entryPoints:
        - websecure
      tls:
        certResolver: myresolver
      service: seafile
      middlewares:
        - seafile-headers
```

- point traefik config with command `traefik --configFile=/etc/traefik/traefik.yml`

## NFS in seafile for storage
- I took the route where, nfs is mounted to proxmox and then use it in lxc.
- Below is proxmox fstab entry for reference.
- ```
  # <file system> <mount point> <type> <options> <dump> <pass>
/dev/pve/root / ext4 errors=remount-ro 0 1
UUID=35F9-32DE /boot/efi vfat defaults 0 1
/dev/pve/swap none swap sw 0 0
proc /proc proc defaults 0 0
#truenas.local:/mnt/hdd2TBdisk/MediaFiles /mnt/nfs_mediaFiles nfs vers=4.1,_netdev 0 0
192.168.0.60:/mnt/hdd2TBdisk/MediaFiles /mnt/nfs_mediaFiles nfs vers=4.1,_netdev 0 0
192.168.0.60:/mnt/hdd2TBdisk/PhotosAndVideos /mnt/photos nfs vers=4.1,_netdev 0 0
192.168.0.60:/mnt/hdd2TBdisk/Files /mnt/files nfs vers=4.1,_netdev 0 0
192.168.0.60:/mnt/hdd2TBdisk/SeaFile  /mnt/seafile_nfs  nfs4  defaults,_netdev,noatime  0  0
  ```


## Seafile lxc config

- open file - `root@proxmox:/etc/pve/lxc# cat 113.conf`
- 113.conf
```
root@proxmox:/etc/pve/lxc# cat 113.conf
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
#lxc.mount.entry%3A 192.168.0.60%3A/mnt/hdd2TBdisk/SeaFile /mnt/seafile nfs defaults 0 0
arch: amd64
cores: 2
features: keyctl=1,nesting=1
hostname: seafile
memory: 2048
mp0: /mnt/seafile_nfs,mp=/mnt/seafile
nameserver: 192.168.0.51
net0: name=eth0,bridge=vmbr0,gw=192.168.0.1,hwaddr=BC:24:11:D7:7C:29,ip=192.168.0.63/24,type=veth
onboot: 1
ostype: debian
rootfs: local-lvm:vm-113-disk-0,size=20G
swap: 512
tags: community-script;documents;seafile
unprivileged: 1
root@proxmox:/etc/pve/lxc#
```

## Initial login
- username and pass for initial login can be found in lxc home path



## Configure Seafile
- Run command from home of lxc to enable domain - `./domain.sh seafile.vishmohomes.click`
- adjust external storage as per our need
```
root@seafile:~# cat external-storage.sh
#!/bin/bash
STORAGE_DIR="/mnt/seafile"

# Move the seafile-data folder to external storage
mv /opt/seafile/seafile-data $STORAGE_DIR/seafile-data

# Create a symlink for access
ln -s $STORAGE_DIR/seafile-data /opt/seafile/seafile-data
root@seafile:~#
```

## Adjusted seafile config
- `/opt/seafile/conf/seahub_settings.py`

```
# -*- coding: utf-8 -*-
SECRET_KEY = "pp106m27cne!#wg^n#xlwp%xrs(k^!ab@=r(9)vsz4yeo4a6(6"
#SERVICE_URL = "seafile.vishmohomes.click"
SERVICE_URL = 'https://seafile.vishmohomes.click'


DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.mysql',
        'NAME': 'seahub_db',
        'USER': 'seafile',
        'PASSWORD': '0xts1viRuN3C2',
        'HOST': '127.0.0.1',
        'PORT': '3306',
        'OPTIONS': {'charset': 'utf8mb4'},
    }
}

CACHES = {
    'default': {
        'BACKEND': 'django_pylibmc.memcached.PyLibMCCache',
        'LOCATION': '127.0.0.1:11211',
    },
}

FILE_SERVER_ROOT = "seafile.vishmohomes.click/seafhttp"
CSRF_TRUSTED_ORIGINS = ["seafile.vishmohomes.click/"]
ALLOWED_HOSTS = [".seafile.vishmohomes.click"]
CSRF_TRUSTED_ORIGINS = ['http://192.168.0.63/']
SECURE_PROXY_SSL_HEADER = ('HTTP_X_FORWARDED_PROTO', 'https')
```


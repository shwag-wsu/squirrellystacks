# SquirrellyStacks Home Lab

Welcome to **SquirrellyStacks**, a secure, cloud-enabled, self-hosted home lab stack built on an old gaming PC. This setup leverages Docker, Cloudflare Tunnels, Pi-hole, Portainer, and Unbound, with identity-based login and DNS-level security.


<a href="https://squirrellystacks.dev/">
  <img src="logo.png" alt="Squirrelly Stacks" width="200"/>
</a>

https://squirrellystacks.dev/


---

## 🧾 System Overview

| Component       | Value                        |
|----------------|-------------------------------|
| Host OS        | Ubuntu Server 22.04 LTS       |
| Hostname       | labinternalserver             |
| Primary User   | ME                            |
| Static IP      | 10.0.0.100                    |
| Domain         | squirrellystacks.dev          |

---

## 🐳 Services Deployed

| Service     | Stack           | Port   | URL                                        |
|-------------|------------------|--------|---------------------------------------------|
| Docker      | System-wide      | —      | —                                           |
| Portainer   | Docker container | 9000   | https://portainer.squirrellystacks.dev      |
| Pi-hole     | Docker container | 8080   | https://pihole.squirrellystacks.dev         |
| Unbound     | Docker container | 5335   | Used as upstream DNS                       |
| Landing Page| Nginx container  | 8081   | https://squirrellystacks.dev                |

---

## 🐳 Docker Commands Used

### Portainer:
```bash
docker volume create portainer_data

docker run -d \
  -p 9000:9000 -p 8000:8000 \
  --name=portainer \
  --restart=always \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v portainer_data:/data \
  portainer/portainer-ce
```

### Pi-hole:
```bash
docker network create dns_net

mkdir -p /mnt/labdata/pihole/etc-pihole
mkdir -p /mnt/labdata/pihole/etc-dnsmasq.d

docker run -d \
  --name pihole \
  --hostname pihole \
  --network dns_net \
  -p 53:53/tcp -p 53:53/udp \
  -p 8080:80 \
  -e TZ="America/New_York" \
  -e WEBPASSWORD="YourSecurePasswordHere" \
  -v /mnt/labdata/pihole/etc-pihole:/etc/pihole \
  -v /mnt/labdata/pihole/etc-dnsmasq.d:/etc/dnsmasq.d \
  --restart=always \
  --cap-add=NET_ADMIN \
  pihole/pihole
```

### Unbound:
```bash
mkdir -p /mnt/labdata/unbound
# Add config and root.hints before starting

curl -o /mnt/labdata/unbound/root.hints https://www.internic.net/domain/named.root

docker run -d \
  --name=unbound \
  --restart=always \
  -v /mnt/labdata/unbound:/opt/unbound/etc/unbound \
  -p 5335:5335/tcp -p 5335:5335/udp \
  mvance/unbound
```

### Landing Page:
```bash
mkdir -p /mnt/labdata/landing-page
# Create index.html and logo.png in that directory

docker run -d \
  --name landing \
  -v /mnt/labdata/landing-page:/usr/share/nginx/html:ro \
  -p 8081:80 \
  nginx
```

---

## 🌐 Cloudflare Tunnel Config

### File: `~/.cloudflared/config.yml`
```yaml
tunnel: squirrellylab
credentials-file: /home/sharris/.cloudflared/044f33c8-795e-4651-8b41-d29e65d5c527.json

ingress:
  - hostname: portainer.squirrellystacks.dev
    service: http://localhost:9000
  - hostname: pihole.squirrellystacks.dev
    service: http://localhost:8080
  - hostname: squirrellystacks.dev
    service: http://localhost:8081
  - hostname: www.squirrellystacks.dev
    service: http://localhost:8081
  - service: http_status:404
```

### Start tunnel as service:
```bash
sudo systemctl edit cloudflared.service
# Make sure it points to the correct config.yml path
# Restart after editing:
sudo systemctl daemon-reload
sudo systemctl restart cloudflared
```

---

## 🔐 Cloudflare Access Policies

- **Login method**: GitHub
- **Allowed users**: Specific GitHub usernames only
- **Optional IP restriction**: Home/public IP range

### Example Rule:
```
Include → GitHub Username: your-github-username
Require → IP Ranges: 1.2.3.4/32
```

---

## 🧠 Notes

- Docker data stored in `/mnt/labdata`
- Static IP assigned manually for reliable LAN resolution
- Internal DNS via Pi-hole + custom entries for service discovery
- Cloudflare provides external HTTPS access without exposing ports

---

## 📦 Future Enhancements

- Add `Uptime Kuma` for monitoring
- Enable `Watchtower` for container auto-updates
- Add `Netdata` or `Grafana` for resource metrics
- Backups with `rsync` or `restic`

---

Built with 🛠️ on a repurposed gaming PC. Stay squirrelly! 🐿️

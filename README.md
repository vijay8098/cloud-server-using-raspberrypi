# Nextcloud + Cloudflare Tunnel on Raspberry Pi

This project demonstrates how to self-host Nextcloud on a Raspberry Pi and securely expose it to the internet using Cloudflare Tunnel (no port forwarding required). It works even behind mobile hotspot or ISP NAT.

## Features

* Private cloud storage & file sharing
* Secure HTTPS using Cloudflare
* No need to open router ports
* Works on home Wi-Fi or mobile hotspot

---

## Requirements

* Raspberry Pi (3B/4/5 recommended)
* Raspberry Pi OS (Debian-based)
* Cloudflare account with a domain
* Stable internet connection

---

## 1. Install Nextcloud (Snap Version)

```bash
sudo apt update
sudo apt install snapd -y
sudo snap install nextcloud
```

Check nextcloud services:

```bash
sudo snap services nextcloud
```

---

## 2. Access Nextcloud Locally

Find Pi IP:

```bash
hostname -I
```

Open in browser:

```
http://<raspberry_pi_ip>
```

Default admin setup screen should appear.

---

## 3. Install Cloudflared

```bash
curl -L https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-arm64.deb -o cloudflared.deb
sudo dpkg -i cloudflared.deb
```

Login Cloudflare:

```bash
cloudflared tunnel login
```

Create tunnel:

```bash
cloudflared tunnel create nextcloud
```

Note the generated `TUNNEL_ID`.

---

## 4. Configure Tunnel

Create config file:

```bash
sudo nano /etc/cloudflared/config.yml
```

Example config:

```yaml
tunnel: <TUNNEL_ID>
credentials-file: /etc/cloudflared/<TUNNEL_ID>.json

ingress:
  - hostname: cloud.example.com
    service: http://localhost
  - service: http_status:404
```

Start tunnel service:

```bash
sudo systemctl enable cloudflared
sudo systemctl start cloudflared
```

---

## 5. Add DNS in Cloudflare Dashboard

Go to **DNS** → Add Record:

```
Type: CNAME
Name: cloud
Target: <TUNNEL_ID>.cfargotunnel.com
Proxy: ON (Orange Cloud)
```

---

## 6. Configure Nextcloud Trusted Domain

```bash
sudo /snap/bin/nextcloud.occ config:system:set trusted_domains 1 --value=cloud.example.com
sudo /snap/bin/nextcloud.occ config:system:set overwritehost --value=cloud.example.com
sudo /snap/bin/nextcloud.occ config:system:set overwriteprotocol --value=https
sudo snap restart nextcloud
```

Access:

```
https://cloud.example.com
```

---

## 7. Disable Cloudflare & Use Local Access (Optional)

```bash
sudo systemctl stop cloudflared
sudo systemctl disable cloudflared
```

Remove forced redirect:

```bash
sudo /snap/bin/nextcloud.occ config:system:delete overwritehost
sudo /snap/bin/nextcloud.occ config:system:delete overwriteprotocol
sudo snap restart nextcloud
```

Local access:

```
http://<raspberry_pi_ip>
```

---

## Troubleshooting

| Issue                            | Fix                                                     |
| -------------------------------- | ------------------------------------------------------- |
| Cloudflare tunnel fails to start | Check `config.yml` formatting                           |
| Page not loading locally         | Another service may occupy port 80 (`sudo lsof -i :80`) |
| Pi-hole conflicts                | `pihole uninstall` or stop lighttpd                     |

---

## License

Open source — use and modify freely.

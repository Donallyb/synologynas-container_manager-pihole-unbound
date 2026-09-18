# Synology Pi-hole + Unbound — 2026 deployment

## Final layout

- Pi-hole LAN IP: `192.168.81.200`
- Pi-hole Docker bridge IP: `192.168.72.2`
- Unbound Docker bridge IP: `192.168.72.3`
- Pi-hole upstream: `unbound#5335`
- Unbound has no LAN-facing IP
- Unbound runs recursively; no `8.8.8.8`, `1.1.1.1`, or other forwarder is configured.

## Before starting

Export a Pi-hole Teleporter backup.

SSH to the NAS and back up the current deployment:

```bash
cd /volume1/docker/synologynas-container_manager-pihole-unbound
sudo cp -a pihole/pihole pihole/pihole-backup-2026
sudo cp -a pihole/dnsmasq.d pihole/dnsmasq.d-backup-2026
cp docker-compose.yaml docker-compose.pre-2026.yaml
cp .env .env.pre-2026
```

## Copy the new files

Replace/add these files in the project directory:

- `docker-compose.yaml`
- `.env`
- `.env.example`
- `.gitignore`

Before deployment, edit `.env` and replace:

```text
PIHOLE_WEBPASSWORD=CHANGE_ME
```

with your chosen Pi-hole admin password.

Do not commit `.env`.

## Verify the Synology interface

The supplied config uses:

```text
NETWORK_INTERFACE=eth0
```

Confirm with:

```bash
ip addr
ip route
```

If your NAS uses another NIC or a bond, change `NETWORK_INTERFACE` in `.env`.

## Stop the old stack

Do not delete the persistent `pihole/` directories.

```bash
sudo docker compose down
```

If DSM still uses the legacy command:

```bash
sudo docker-compose down
```

## Validate

```bash
sudo docker compose config
```

Verify that the rendered config shows:

- Pi-hole: `192.168.81.200`
- Pi-hole bridge: `192.168.72.2`
- Unbound bridge: `192.168.72.3`
- Pi-hole upstream: `unbound#5335`

## Pull and start

```bash
sudo docker compose pull
sudo docker compose up -d
sudo docker compose ps
```

## Check Unbound

```bash
sudo docker exec unbound /usr/local/unbound/sbin/healthcheck.sh
sudo docker exec unbound drill @127.0.0.1 -p 5335 pi-hole.net
```

## Check Pi-hole

```bash
sudo docker exec pihole pihole-FTL --config dns.upstreams
```

Expected upstream:

```text
unbound#5335
```

From your Mac:

```bash
dig @192.168.81.200 pi-hole.net
nslookup pi-hole.net 192.168.81.200
```

Then open:

```text
http://192.168.81.200/admin/
```

## Google / Nest WiFi

Once the direct Pi-hole tests work, configure:

```text
Primary DNS: 192.168.81.200
Secondary DNS: leave blank
```

if the Google Home app allows that on your setup.

## Pi-hole v5 -> v6 migration note

This first deployment intentionally keeps your old `/etc/dnsmasq.d` mount and sets:

```text
FTLCONF_misc_etc_dnsmasq_d=true
```

because your old Compose used that directory. Pi-hole recommends retaining it for the first v6 migration when upgrading from a v5 deployment that used it.

Once the upgrade is stable, we can inspect whether you actually need the old dnsmasq files and remove this compatibility setting if not.

## Troubleshooting

```bash
sudo docker compose ps
sudo docker logs --tail 100 pihole
sudo docker logs --tail 100 unbound
sudo docker network ls
```

## Rollback

```bash
sudo docker compose down
mv docker-compose.pre-2026.yaml docker-compose.yaml
mv .env.pre-2026 .env
sudo docker compose up -d
```

# homelab
My Homelab config & docs

# About
Mostly old desktop hardware.  Notably storage, case & GPU were bought specifically for server purposes.

Case: SilverStone CS382
CPU: Ryzen 7 1700
CPU Cooler: Noctua NH-D12L
GPU: Intel Arc A310
RAM: 32GB DDR4 2666MHz (4x 8GB)
Motherboard: Gigabyte B450M DS3H
PSU: EVGA Supernova G2 750W
HBA: LSI 9207-8i
Disks:
  - 1x 500GB Crucial P1 NVMe SSD
  - 2x 1TB Crucual MX500 Sata SSD
  - 2x 4TB Western Digital WD40EFRX HDD
  - 3x 18TB Western Digital WD180EDGZ HDD
  - 3x 18Tb Seagate ST18000NM000J HDD
Host OS: OpenSUSE Leap 15.6

### Disk Setup
There's a lot of disks in this machine, and this is how they are currently configured:

- 500GB NVMe SSD -> Host OS.
- 2x 1TB Sata SSD -> ZFS mirror.  Used for anything desiring faster storage.  For example container config, logs, databases, etc.
- 2x 4TB HDD -> ZFS mirror.  Exposed as a SMB share for my personal document storage.
- 6x 18TB HDD -> ZFS raidz1.  General bulk storage.  Mounted into containers that need it.

### Network Setup
I am using Traefik as a reverse proxy.  Configured with three entrypoints:
- `web-internal` listening on port 80
- `websecure-internal` listening on port 443
- `websecure-external` listening on port 8443

`web-internal` only forwards to port 443 & HTTPS.  `websecure-internal` is for traffic connecting from inside my LAN, while `websecure-external` is for traffic connecting from the Internet.
The way this works is that I use my pfsense router as my DNS resolver, and inside of it I have host overrides for each service.  For example if I go to `jellyfin.<DOMAIN>` my pfsense DNS resolver points this traffic directly at the LAN IP of the server so the traffic never has to leave my LAN.  This means I have a guarantee that traffic landing on port 443 in Traefik originated from inside my LAN.
Then for Internet traffic, I have a port forward configured for `443 -> 8443` to my server.  Because of this I know that any traffic on port 8443 in Traefik originated from the Internet.  Thanks to this I have a pretty decent safeguard against accidentally exposing a service to the Internet that I only want available on my LAN, as I would need to explicitly add the `websecure-external` entrypoint to that service.

### Docker Setup
This is a pretty straight forward docker + docker-compose setup.  I am utilising a `.env` file to hold information that I don't want checked in to this repository, such as email address, API keys, etc.

I am taking a simplistic approach at first and intend to put _all_ of my services in one docker-compose template.  If I feel any pain from this I will look at splitting them up, but for a single-user home server situation simple is good.

Also I am taking the route of running as many linuxserver.io containers as possible to make use of their patterns.  I will use UID & GID of 1000 for simple filesystem permissions as well.  Again, for a single-user home server keeping this simple removes headaches.

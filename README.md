# DeployCat — Unraid Community Apps Template

This repository contains the Unraid Community Apps template for the [DeployCat](https://deploycat.app) Docker container.

## What is DeployCat?

DeployCat is a game server manager that makes it easy to create, configure, and manage multiple game servers from a single web interface.

**Supported games:**
- Minecraft (Java + Bedrock + Modded)
- Steam games: ARK, Rust, Palworld, Satisfactory, CS2, Valheim, V Rising, and 25+ more

**Features:**
- SteamCMD pre-installed
- Automatic UPnP port forwarding
- Web-based configuration
- Backup scheduling
- Tailscale integration (DeployCat Connect)

A free 3-day trial is included. A paid subscription is required for ongoing use. See [deploycat.app](https://deploycat.app) for details.

---

## Quick Start

```bash
docker run -d \
  --name deploycat \
  --network host \
  --stop-timeout 40 \
  --device /dev/net/tun \
  --cap-add NET_ADMIN \
  -v /path/to/your/data:/data \
  -e PORT=8732 \
  -e TZ=America/New_York \
  -e PUID=99 \
  -e PGID=100 \
  meowsers/deploycat:latest
```

Then open **http://your-server-ip:8732** in your browser.

---

## Network Mode — READ THIS FIRST

**DeployCat works best with Host networking.** (actually this is a MUST) Host mode lets game servers use any port without manual port mapping, and enables UPnP auto port-forwarding to your router.

If you must use Bridge mode, you need to manually map every port for every game server you create. UPnP will NOT work in bridge mode.

---

## Volume Mappings

| Host Path | Container Path | Required | Description |
|-----------|---------------|----------|-------------|
| `/path/to/your/data` | `/data` | **Yes** | Where DeployCat stores ALL data — servers, config, backups, SteamCMD, Java runtimes, logs. This is the only required mount. |
| `/path/to/drive2` | `/mnt/drive2` | No | Second drive for installing game servers. Useful if your primary drive is small or you want servers on an SSD. |
| `/path/to/backups` | `/mnt/backupdrive` | No | Another location for backups. DeployCat can write backups here so they survive a drive failure. |
| `/var/run/tailscale` | `/var/run/tailscale` | No | Host Tailscale socket. Only needed if your HOST runs Tailscale AND you use host networking. Lets DeployCat use the host's Tailscale daemon instead of spawning a conflicting one. |

### Example with all mounts:

```bash
docker run -d \
  --name deploycat \
  --network host \
  --stop-timeout 40 \
  --device /dev/net/tun \
  --cap-add NET_ADMIN \
  -v /mnt/user/appdata/deploycat:/data \
  -v /mnt/user/appdata2/ssdservers:/mnt/drive2 \
  -v /mnt/remotes/backups/dcbackups:/mnt/backupdrive \
  -v /var/run/tailscale:/var/run/tailscale \
  -e PORT=8732 \
  -e TZ=America/Chicago \
  -e PUID=99 \
  -e PGID=100 \
  -e TS_SOCKET=/var/run/tailscale/tailscaled.sock \
  meowsers/deploycat:latest
```

---

## Environment Variables

| Variable | Default | Required | Description |
|----------|---------|----------|-------------|
| `PORT` | `8732` | **Yes** | Port for the DeployCat web interface. Access the UI at `http://your-ip:PORT`. |
| `TZ` | `UTC` | No | Timezone for scheduling (backup schedules, start/stop schedules). Use [IANA format](https://en.wikipedia.org/wiki/List_of_tz_database_time_zones) e.g. `America/New_York`, `Europe/London`. Without this, all schedules run in UTC. |
| `PUID` | `99` | Recommended | User ID for file ownership. Defaults to `99` (Unraid convention). On Ubuntu/Debian, change to `1000` to match your host user. Run `id -u` on your host to find yours. |
| `PGID` | `100` | Recommended | Group ID for file ownership. Defaults to `100` (Unraid convention). On Ubuntu/Debian, change to `1000` to match your host group. Run `id -g` on your host to find yours. |
| `SERVER_PORT_RANGE` | `25565-25575` | No | Port range for Java Edition Minecraft servers. Only change if you need more than 11 Minecraft servers. |
| `BEDROCK_PORT` | `19132` | No | Port for Bedrock Edition Minecraft servers (UDP). |
| `HOST_IP` | _(empty)_ | No | Your server's LAN IP (e.g. `192.168.1.100`). Shown in DeployCat so you know what IP players use to connect. Auto-detected if not set. |
| `TS_SOCKET` | `/var/run/tailscale/tailscaled.sock` | No | Path to the host's Tailscale daemon socket. Only used when the Host Tailscale Socket volume is mapped. Lets DeployCat control the host's Tailscale (auth, status, connect) without running a duplicate daemon. |

### Data Directory Overrides (Advanced)

These are set in the Dockerfile and rarely need changing. All point inside `/data`:

| Variable | Default | Description |
|----------|---------|-------------|
| `DEPLOYCAT_DATA` | `/data/servers` | Where game servers are installed |
| `DEPLOYCAT_CONFIG_DIR` | `/data/config` | App configuration and settings |
| `DEPLOYCAT_BACKUP_DIR` | `/data/backups` | Default backup location |
| `DEPLOYCAT_IMPORT_DIR` | `/data/import` | Drop server backups here to import |
| `DEPLOYCAT_TMP_DIR` | `/data/Tempdl` | Temporary download staging |
| `DEPLOYCAT_JAVAS_DIR` | `/data/StoredJavas` | Downloaded Java runtimes |
| `DEPLOYCAT_STEAMCMD_DIR` | `/data/SteamCMD` | SteamCMD installation and cache |
| `DEPLOYCAT_LOG_DIR` | `/data/logs` | Application logs |

---

## Docker Compose

```yaml
version: '3.8'

services:
  deploycat:
    image: meowsers/deploycat:latest
    container_name: deploycat
    restart: unless-stopped
    network_mode: host
    stop_grace_period: 40s
    devices:
      - /dev/net/tun:/dev/net/tun
    cap_add:
      - NET_ADMIN
    volumes:
      - /path/to/your/data:/data
      # Optional: second drive for game servers
      # - /path/to/drive2:/mnt/drive2
      # Optional: separate backup location
      # - /path/to/backups:/mnt/backupdrive
      # Optional: host Tailscale socket
      # - /var/run/tailscale:/var/run/tailscale
    environment:
      - PORT=8732
      - TZ=America/New_York
      - PUID=99
      - PGID=100
      # Optional: host Tailscale socket path
      # - TS_SOCKET=/var/run/tailscale/tailscaled.sock
```

Start with:

```bash
docker compose up -d
```

---

## Bridge Mode (Not Recommended)

If you can't use host networking, use bridge mode and map ports manually. **UPnP will NOT work.** You must add a port mapping for every game server port you create.

```bash
docker run -d \
  --name deploycat \
  --restart unless-stopped \
  --stop-timeout 40 \
  --device /dev/net/tun \
  --cap-add NET_ADMIN \
  -p 8732:8732 \
  -p 25565-25575:25565-25575 \
  -p 19132:19132/udp \
  -p 7777:7777/udp \
  -p 7778:7778/udp \
  -p 8211:8211/udp \
  -p 27015:27015/udp \
  -p 27016:27016/udp \
  -p 16261:16261/udp \
  -p 16262:16262/udp \
  -p 8888:8888/tcp \
  -p 25575:25575/tcp \
  -v /path/to/your/data:/data \
  -e PORT=8732 \
  -e TZ=America/New_York \
  -e PUID=99 \
  -e PGID=100 \
  meowsers/deploycat:latest
```

**Common game ports you may need to map (UDP unless noted):**

| Port | Game(s) |
|------|---------|
| 25565-25575 | Minecraft Java (default range) |
| 19132 | Minecraft Bedrock |
| 7777 | ARK, The Isle, Palworld, Satisfactory, Core Keeper |
| 7778 | Query port (many games) |
| 8211 | Palworld (default) |
| 27015-27016 | Rust, Sons of the Forest, Steam query |
| 16261-16262 | Project Zomboid (16262 TCP+UDP is REQUIRED) |
| 9876-9877 | V Rising |
| 8888 (TCP) | RCON (The Isle, Palworld) |
| 25575 (TCP) | RCON (Rust, Palworld) |
| 27020 (TCP) | ARK RCON |
| 10000 (TCP) | The Isle queue |

---

## Unraid

DeployCat is available in the Unraid Community Applications store. The template pre-configures all volume mappings, ports, and environment variables. Key settings:

- **Network Type:** Host
- **Repository:** `meowsers/deploycat`
- **Data Location:** `/mnt/user/appdata/deploycat` → `/data`
- **PUID:** `99` / **PGID:** `100`
- **Extra Parameters:** `--stop-timeout=40 --device=/dev/net/tun --cap-add=NET_ADMIN`

---

## Supported Games

**Steam Games (25+):** ARK: Survival Ascended, ARK: Survival Evolved, Palworld, Rust, Valheim, V Rising, Satisfactory, CS2, Project Zomboid, Enshrouded, The Isle, Sons of the Forest, 7 Days to Die, Conan Exiles, Core Keeper, DayZ, Team Fortress 2, Left 4 Dead 2, Garry's Mod, Terraria, tModLoader, The Forest, Unturned, Factorio, ASKA, Necesse, American Truck Simulator, Euro Truck Simulator 2, Windrose

**Minecraft:** Java Edition (Vanilla, Paper, Purpur, Fabric, Forge, NeoForge), Bedrock Edition

**Other:** Hytale (when available)

---

## Hardware Requirements

| | Minimum | Recommended |
|--|---------|-------------|
| RAM | 4 GB | 16+ GB |
| Storage | 50 GB free | 200+ GB free |
| CPU | 2 cores | 4+ cores |
| Network | Broadband | Wired Ethernet |

Storage needs vary heavily by game — Minecraft needs ~2 GB, ARK needs ~130 GB.

---

## Troubleshooting

**Web UI not loading?**
- Make sure you're using `http://` not `https://`
- Check that PORT (8732) is not blocked by a firewall
- If using bridge mode, make sure you mapped `-p 8732:8732`

**Game servers not accessible from the internet?**
- Use host networking mode — bridge mode blocks UPnP
- Check that your router supports UPnP and it's enabled
- Some ISPs block inbound connections (CGNAT) — contact your ISP

**Permission errors on files?**
- Make sure PUID and PGID match your host user
- Run `id -u` and `id -g` on your host to find the correct values
- On Unraid, use PUID=99 and PGID=100

**Container keeps restarting?**
- Check logs: `docker logs deploycat`
- Make sure `/data` is mounted to a writable location
- Make sure you have enough disk space (50+ GB recommended)

---

## License

DeployCat is proprietary software owned by Hightexperts LLC. By downloading, installing, or using this software, you agree to the [Terms of Service and End User License Agreement](https://deploycat.app/terms).

**Free trial:** Fully unlocked 3-day trial, no payment info required. After the trial, a paid subscription (monthly or annual) is required to continue.

**You may NOT:** resell, redistribute, reverse engineer, or offer DeployCat as a hosted service without written permission.

---

## Links

- **Website:** [deploycat.app](https://deploycat.app)
- **Terms of Service:** [deploycat.app/terms](https://deploycat.app/terms)
- **Support:** Join our Discord — [discord.gg/gVcjQHfhUS](https://discord.gg/gVcjQHfhUS)

---

## Files

| File | Description |
|------|-------------|
| `ca_profile.xml` | Community Apps repository profile (required for CA submission) |
| `Deploycat.xml` | Container template (Unraid CA format) |
| `icon.png` | Container icon shown in Unraid UI |

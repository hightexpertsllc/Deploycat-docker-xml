# DeployCat — Unraid Community Apps Template

This repository contains the Unraid Community Apps template for the [DeployCat](https://deploycat.app) Docker container.

## What is DeployCat?

DeployCat is a game server manager that makes it easy to create, configure, and manage multiple game servers from a single web interface.

**Supported games:**
- Minecraft (Java + Bedrock)
- Steam games: ARK, The Isle, Rust, Palworld, Satisfactory, CS2, Valheim, V Rising, and 25+ more

**Features:**
- SteamCMD pre-installed
- Automatic UPnP port forwarding
- Web-based configuration
- Backup scheduling
- Tailscale integration

A free 3-day trial is included. A paid subscription is required for ongoing use. See [deploycat.app](https://deploycat.app) for details.

## Installation

1. Install the DeployCat container from the Unraid Community Apps tab.
2. Set the **Data Location** path (default: `/mnt/user/appdata/DeployCat`).
3. Set the **WebUI Port** (default: `8732`).
4. Set **PUID** and **PGID** to match your Unraid user (defaults: `99` / `100`).
5. Set **TZ** to your timezone (e.g., `America/New_York`).
6. Start the container and access the web UI at `http://[UNRAID_IP]:8732/`.

> **Network mode:** The container must run in **Host** network mode for UPnP port forwarding and game server ports to function correctly.

## Support

- **Discord:** [discord.gg/deploycat](https://discord.gg/deploycat)
- **Website:** [deploycat.app](https://deploycat.app)

## Files

| File | Description |
|------|-------------|
| `ca_profile.xml` | Community Apps repository profile (required for CA submission) |
| `Deploycat.xml` | Container template (Unraid CA format) |
| `icon.png` | Container icon shown in Unraid UI |

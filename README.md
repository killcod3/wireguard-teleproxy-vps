# WireGuard & Teleproxy VPS Setup

Bash startup scripts for setting up a WireGuard VPN, a Teleproxy Telegram MTProto proxy, or both on a fresh VPS. The scripts have been tested successfully by the maintainer.

WireGuard setup creates five separate client configurations and terminal QR codes. Teleproxy setup creates a random secret and a systemd service. Connection details are saved on the server under `/root/`.

## Choose a script

| Setup | Startup script | Verification commands | Server summary file |
| --- | --- | --- | --- |
| Both services on DigitalOcean | [UserData-DigiOcean.txt](UserData-DigiOcean.txt) | [verification-DigiOcean.txt](verification-DigiOcean.txt) | `/root/NEW-VPS-INFO.txt` |
| Both services with public-IP detection fallbacks | [UserData-generic.txt](UserData-generic.txt) | [verification-generic.txt](verification-generic.txt) | `/root/NEW-VPS-INFO.txt` |
| WireGuard only | [UserData-wireguard.txt](WireGuard-Teleproxy-VPS-Scripts/UserData-wireguard.txt) | [verification-wireguard.txt](WireGuard-Teleproxy-VPS-Scripts/verification-wireguard.txt) | `/root/WIREGUARD-VPS-INFO.txt` |
| Teleproxy only | [UserData-teleproxy.txt](WireGuard-Teleproxy-VPS-Scripts/UserData-teleproxy.txt) | [verification-teleproxy.txt](WireGuard-Teleproxy-VPS-Scripts/verification-teleproxy.txt) | `/root/TELEPROXY-VPS-INFO.txt` |

The `.txt` startup files contain Bash scripts. Use one startup script for the desired setup. The generic and standalone scripts try DigitalOcean metadata, public IP lookup services, and AWS metadata to detect the server's public IPv4 address.

## Requirements

- A fresh Debian/Ubuntu VPS using `apt-get` and systemd, with root access.
- A reachable public IPv4 address and outbound access to package repositories and download/IP lookup services.
- An x86_64/amd64 server for any setup that includes Teleproxy: the scripts download `teleproxy-linux-amd64`.
- Cloud-init or equivalent shell user-data support when installing during VPS creation.
- Available ports for the selected services.

The exact tested OS versions and VPS images are not recorded yet. These requirements describe what the scripts use; they are not a claim that every compatible image has been tested.

## Firewall ports

Allow the relevant inbound ports in your provider firewall/security group and any host firewall:

| Purpose | Port |
| --- | --- |
| SSH access | `22/tcp` (or your configured SSH port) |
| WireGuard | `51820/udp` |
| Teleproxy | `443/tcp` |

The WireGuard scripts configure forwarding and NAT using iptables. The scripts list required inbound ports but do not configure provider firewalls or open host INPUT rules. Teleproxy's internal proxy option is set to `8888`; the documented public connection uses `443`.

## Installation

### During VPS creation

1. Open the startup script for your chosen setup.
2. Paste its complete contents, including `#!/bin/bash`, into your provider's shell user-data/startup-script field.
3. Configure the required firewall ports and create the VPS.
4. Connect over SSH and wait for startup to finish. On a cloud-init image, inspect:

   ```bash
   sudo cloud-init status --long
   sudo tail -n 100 /var/log/cloud-init-output.log
   ```

5. Read the summary file listed in the script table. For a combined installation:

   ```bash
   sudo cat /root/NEW-VPS-INFO.txt
   ```

### Manually on a fresh VPS

Copy the chosen startup file to the server, then run it as root. For example, with `UserData-generic.txt` in your current directory:

```bash
sudo bash UserData-generic.txt
```

Startup scripts install packages and write system configuration. They are intended for initial provisioning: rerunning them regenerates WireGuard keys and/or Teleproxy credentials and overwrites configuration. Existing client access details may become invalid. For an existing installation, use the verification commands to inspect its state.

## Verify the installation

Open the matching `verification-*.txt` file and paste its full contents into a shell on the VPS with appropriate root access. These files contain commands that create a `verify-*.sh` script and run it; they are not just the body of the verification script. The DigitalOcean verification file assumes a root shell. The other variants invoke `sudo` for the generated script.

Checks cover service status, listening ports, generated files, and, for WireGuard, the expected five peers. The generated verifier exits with status `0` when all its checks pass and `1` when any check fails.

A passing verification checks local server state. Confirm actual connectivity by importing a WireGuard profile or using the Telegram proxy link from a client device.

## Connect with WireGuard

Each device should use a different client configuration. Files are generated at:

```text
/etc/wireguard/clients/client1.conf ... client5.conf
/etc/wireguard/clients/client1-qr.txt ... client5-qr.txt
```

Display a QR code on the VPS:

```bash
sudo cat /etc/wireguard/clients/client1-qr.txt
```

Or, from your computer, download a profile if root SSH access is enabled, replacing `SERVER_IP` with the server's public IPv4 address:

```bash
scp root@SERVER_IP:/etc/wireguard/clients/client1.conf .
```

Import the profile or scan the QR code using a WireGuard client. The profiles route all IPv4 traffic through the VPN (`AllowedIPs = 0.0.0.0/0`), use DNS `1.1.1.1`, and configure a 25-second persistent keepalive. IPv6 tunneling is not configured.

## Connect with Telegram

For generic combined and standalone Teleproxy installations, the server summary includes a link in this format:

```text
https://t.me/proxy?server=SERVER_IP&port=443&secret=SECRET
```

For the DigitalOcean combined installation, use the public IP and secret in `/root/NEW-VPS-INFO.txt` to build the same link. Open it in Telegram and enable the proxy. The secret is also stored in `/etc/teleproxy/teleproxy.env`.

Keep generated private keys, client profiles, QR codes, proxy secrets, and server summary files private. Do not commit them to GitHub or include them in issue screenshots or logs.

## Troubleshooting

For a combined installation:

```bash
sudo systemctl is-active wg-quick@wg0 teleproxy
sudo wg show
sudo ss -tulpn
sudo journalctl -u wg-quick@wg0 -n 100 --no-pager
sudo journalctl -u teleproxy -n 100 --no-pager
```

For a standalone installation, inspect only the service you installed. If local checks pass but clients cannot connect, check the public IP and inbound firewall rules. Inspect cloud-init output for installation errors when using user data.

Teleproxy is downloaded from its GitHub `latest` release during installation, so the downloaded version can change between deployments. The scripts do not pin a release or verify a download checksum.

## Contributing

Keep changes focused and preserve the tested setup behavior. Describe which script variants are affected and how changes were verified. Syntax checks alone do not establish that a deployment works; changes to installation behavior need verification on a fresh VPS.

AI coding agents should read [AGENTS.md](AGENTS.md) before editing.

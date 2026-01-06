---
title: "Network"
---

## Basics

## Tailscale

[Tailscale](https://tailscale.com/) has proprietary control plane, but the data plane is based on open source WireGuard. It creates a mesh VPN between your devices, making them accessible to each other securely over the internet. There is opensource alternative for control plane [Headscale](https://headscale.net/).

It will create virtual network interface `tailscale0` on each device. Each device gets its own stable IP address in the `100.x.y.z` range. If you enable `--ssh` you can SSH to other devices using Tailscale IP or hostname (e.g. `device-name.tailcale.net`).

```console
sudo dnf install tailscale
sudo systemctl up tailscaled --ssh # one time with SSH
sudo systemctl enable tailscaled   # on boot
ip addr show tailscale0

tailscale ping <other-device>
```

## Desktop sharing

Server: [Sunshine](https://github.com/LizardByte/Sunshine)
Client: [Moonlight](https://moonlight-stream.org/)

Great with combination with Tailscale for secure access over the internet. Even for gaming.

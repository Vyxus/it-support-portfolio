# Firewall & Network Edge (pfSense)

## Problem
Every environment needs a network edge that separates the internal LAN from the outside world and controls what traffic is allowed in and out. This project builds that edge from scratch.

## What I Built
- pfSense installed as a virtual router/firewall
- WAN and LAN interfaces configured and assigned
- LAN subnet `192.168.50.0/24` established as the backbone for the entire lab
- Firewall rules reviewed and configured
- Verified outbound internet access from an internal VM through the firewall

## Lab Context
- **VM role/name:** pfSense edge firewall
- **Network design:** NAT-facing WAN + host-only LAN (`192.168.50.0/24`)
- **Downstream dependencies:** AD, file server, ticketing, mail, and SIEM-integrated servers all rely on this routing path

## How It Works
![Host-only network config](./screenshots/Host-only%20network%20config.png)
![pfSense dashboard](./screenshots/pfSense%20dashboard.png)
![Firewall rule](./screenshots/firewall%20rule.png)

The pfSense VM sits between the host-only "LAN" network (where all other lab VMs live) and the WAN-facing NAT network (internet access). All other servers in this portfolio — the domain controllers, file server, ticketing system, and mail server — sit behind this firewall on the `192.168.50.0/24` subnet.

## Challenges & Troubleshooting
Getting the WAN/LAN interface assignment correct on first boot required matching VirtualBox's host-only adapter to pfSense's expected LAN interface — an easy point to get backwards, which would silently put the firewall's LAN-facing rules on the wrong interface.

## Validation Performed
- Confirmed firewall web UI and interface status from dashboard
- Confirmed LAN-side VM egress through pfSense to internet
- Confirmed the LAN subnet used by all later modules is reachable behind this edge

## What I'd Do Differently / Next Steps
- Add outbound rules scoped per-VLAN if network segmentation (see Week 7 scope note) is revisited later
- Enable pfBlockerNG or a similar package for basic DNS-based filtering as a stretch goal

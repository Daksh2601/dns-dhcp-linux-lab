# DNS & DHCP Server Lab — Ubuntu Server 24.04 LTS

> Deployed BIND9 DNS and ISC-DHCP Server on Ubuntu Server 24.04 inside VirtualBox.
> Configured authoritative DNS zone, dynamic IP allocation, static interface via Netplan,
> and remote SSH management — simulating a real corporate network infrastructure environment.

📄 **[View Full Lab Documentation (PDF)](./DNS_DHCP_Lab_Documentation.pdf)**

---

## Objective

Deploy a fully functional DNS and DHCP server on Linux to simulate the core infrastructure
services found in every corporate IT environment — hostname resolution and dynamic IP management.

---

## Environment

| Component | Details |
|-----------|---------|
| Host OS | macOS (Apple Silicon M-series) |
| Hypervisor | Oracle VirtualBox 7.1+ |
| Server OS | Ubuntu Server 24.04.4 LTS (aarch64) |
| DNS Software | BIND9 (named) v9.18.39 |
| DHCP Software | ISC-DHCP-Server v4.4.3-P1 |
| Network | VirtualBox Host-only (192.168.56.0/24) |
| Remote Access | SSH via OpenSSH from Mac Terminal |

---

## Network Design

| Host | IP Address | Role |
|------|-----------|------|
| DNS-DHCP Server | 192.168.56.2 | DNS + DHCP Server (static via Netplan) |
| VirtualBox Host | 192.168.56.1 | Default Gateway |
| DHCP Clients | 192.168.56.100–200 | Dynamically assigned |

---

## What Was Configured

### DNS — BIND9
- Installed and configured BIND9 as authoritative DNS for `lab.local`
- Created forward zone file with A records for all lab hosts
- Configured DNS forwarders (8.8.8.8 / 8.8.4.4) for external resolution
- Validated config with `named-checkconf` and `named-checkzone`
- Verified resolution with `dig server.lab.local @192.168.56.2`

### DHCP — ISC-DHCP-Server
- Installed ISC-DHCP and configured scope for `192.168.56.0/24`
- Set IP range: `192.168.56.100 – 192.168.56.200`
- Pushed DNS server, domain name (`lab.local`), and default gateway via DHCP options
- Bound DHCP service to `enp0s9` (host-only interface)
- Verified with `systemctl status` and `dhcpd.leases`

### Linux Administration
- Assigned static IP to `enp0s9` using Netplan (`50-cloud-init.yaml`)
- Managed services with `systemctl` (start, enable, status, restart)
- Configured and enabled OpenSSH for remote management
- Diagnosed and resolved 7 real configuration and networking issues

---

## Verification Screenshots

| Screenshot | What It Proves |
|-----------|---------------|
| `bind9_status.png` | BIND9 active and listening on 192.168.56.2#53 |
| `dhcp_status.png` | ISC-DHCP active on enp0s9/192.168.56.0/24 |
| `dig_resolution.png` | server.lab.local resolves to 192.168.56.2 in 0ms |
| `dhcpd_conf.png` | Clean validated DHCP configuration |
| `dhcpd_leases.png` | Lease file initialised, server DUID generated |

---

## Key Config Files

### /etc/bind/named.conf.options
```
options {
    directory "/var/cache/bind";
    recursion yes;
    allow-query { any; };
    forwarders { 8.8.8.8; 8.8.4.4; };
    dnssec-validation no;
    listen-on { any; };
};
```

### /etc/bind/zones/db.lab.local
```
$TTL    604800
@   IN  SOA ns1.lab.local. admin.lab.local. ( 2024010101 )
@       IN  NS  ns1.lab.local.
ns1     IN  A   192.168.56.2
server  IN  A   192.168.56.2
client  IN  A   192.168.56.100
```

### /etc/dhcp/dhcpd.conf
```
default-lease-time 86400;
max-lease-time 172800;
authoritative;

subnet 192.168.56.0 netmask 255.255.255.0 {
    range 192.168.56.100 192.168.56.200;
    option domain-name-servers 192.168.56.2;
    option domain-name "lab.local";
    option routers 192.168.56.1;
    option broadcast-address 192.168.56.255;
}
```

---

## Troubleshooting Log

Real issues encountered and resolved during this lab:

| Issue | Resolution |
|-------|-----------|
| VirtualBox boot loop | Attached ARM64 ISO to optical drive, set boot order to Optical first |
| Wrong ISO architecture | AMD64 ISO incompatible with Apple M-series — switched to ARM64 |
| Host-only adapter missing | Created vboxnet0 manually in VirtualBox Network Manager |
| Interface enp0s9 DOWN | Assigned static IP via Netplan and applied with `netplan apply` |
| DHCP config corruption | Unicode characters from copy-paste broke config — rewrote using `echo` pipe commands |
| SSH connection refused | Installed openssh-server and enabled the service |
| DNSSEC validation failure | Set `dnssec-validation no` — root servers unreachable in isolated NAT VM |

---

## Skills Demonstrated

- Linux server administration (Ubuntu Server 24.04)
- DNS configuration and zone management (BIND9)
- DHCP scope configuration and lease management (ISC-DHCP)
- Network interface configuration via Netplan
- SSH remote server management
- Service management with systemctl
- Real-world troubleshooting and diagnostics

---

## Tools Used

`Ubuntu Server 24.04` · `BIND9` · `ISC-DHCP-Server` · `VirtualBox` · `OpenSSH` · `Netplan` · `dig` · `systemctl`

---

## Part of My IT Portfolio

This is **Project 2** of my hands-on networking and IT infrastructure portfolio.

| Project | Description | Repo |
|---------|------------|------|
| Project 1 | VLAN & Inter-VLAN Routing (Cisco Packet Tracer) | [vlan-intervlan-routing-lab](https://github.com/Daksh2601/vlan-intervlan-routing-lab) |
| Project 2 | DNS & DHCP Server on Linux (this repo) | — |
| Project 3 | Active Directory Home Lab (coming soon) | — |

---

*Daksh Patel · CCNA Certified · [LinkedIn](https://www.linkedin.com/in/pateldaksh) · [Portfolio](https://daksh2601.github.io/Portfolio/)*

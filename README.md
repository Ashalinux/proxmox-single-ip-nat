<div align="center">

# Proxmox Single-IP NAT

**Run all your Proxmox VE VMs and containers behind one single public IPv4.**

![Proxmox](https://img.shields.io/badge/Proxmox%20VE-7%20%7C%208-E57000?logo=proxmox&logoColor=white)
![Debian](https://img.shields.io/badge/Debian-based-A81D33?logo=debian&logoColor=white)
![License](https://img.shields.io/badge/license-MIT-green)

</div>

Most rented servers come with just one IPv4 address. This setup creates a private NAT bridge (`vmbr1`) for your guests, gives them internet access via masquerading and makes individual services reachable from outside via port forwarding.

```
Internet ──► public IP (vmbr0) ──► NAT / port forwarding ──► vmbr1 (10.0.0.0/29)
                                                               ├── VM/CT 10.0.0.1
                                                               ├── VM/CT 10.0.0.2
                                                               └── ...
```

> ⚠️ This setup does **not** include a firewall. Secure your host with `ufw`, `iptables` or the Proxmox firewall. Best applied to a freshly installed Proxmox VE.

## 📂 Files

| File | Purpose |
|---|---|
| `etc/network/interfaces` | Host network config: public bridge `vmbr0` + private NAT bridge `vmbr1` |
| `vms_cts` | Network settings to use inside each VM/container |
| `rc.local` | Port-forwarding rules (public port → guest) |

## ⚙️ Setup

1. **Host:** copy `etc/network/interfaces` to `/etc/network/interfaces`, replace the `####` placeholders with your public IP and gateway, then run `ifreload -a` (or reboot).
2. **Guests:** give each VM/CT a network device on `vmbr1` with an IP from `10.0.0.1`–`10.0.0.6` (`/29`) – see `vms_cts`.
3. **Port forwarding:** add one rule per service to `/etc/rc.local` and make it executable:

```bash
# Example: public port 221 → SSH on 10.0.0.1
iptables -t nat -A PREROUTING -i vmbr0 -p tcp --dport 221 -j DNAT --to-destination 10.0.0.1:22
```

```bash
ssh user@<public-ip> -p 221
```

> 💡 A `/29` gives you 6 usable guest addresses. Use a bigger subnet (e.g. `/24`) if you need more.

## 📄 License

Released under the [MIT License](LICENSE).

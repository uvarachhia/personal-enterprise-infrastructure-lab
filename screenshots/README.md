# Screenshots

Evidence for each build step and troubleshooting entry. Files starting `p1`–`p4` match problems 1–4 in the [troubleshooting log](../docs/03-troubleshooting-log.md).

| File | What it shows |
| --- | --- |
| [01-labnet-natnetwork-list.png](01-labnet-natnetwork-list.png) | `VBoxManage natnetwork list`: LabNet on 10.10.10.0/24, gateway 10.10.10.1, DHCP server disabled |
| [02-dc01-nic-attached-labnet.png](02-dc01-nic-attached-labnet.png) | `VBoxManage showvminfo`: NIC 1 attached to NAT Network 'LabNet' |
| [03-dc01-apipa-before-static-ip.png](03-dc01-apipa-before-static-ip.png) | Fresh server with a random hostname and an APIPA 169.254.x.x address (no DHCP on LabNet, as designed) |
| [04-dc01-static-ip-and-ping.png](04-dc01-static-ip-and-ping.png) | Static IP 10.10.10.10, gateway 10.10.10.1, DNS 127.0.0.1, and 4/4 ping replies from the gateway |
| [05-p1-failed-to-create-machine-folder.png](05-p1-failed-to-create-machine-folder.png) | Problem 1: VirtualBox failed to create the VM folder inside its program directory |
| [06-p1-verr-path-not-found.png](06-p1-verr-path-not-found.png) | Problem 1: `VERR_PATH_NOT_FOUND`, the clue that the VM was only half-created |
| [07-p2-stuck-restarting.png](07-p2-stuck-restarting.png) | Problem 2: server stuck on "Restarting" after Guest Additions |
| [08-p2-display-fix-vboxsvga.png](08-p2-display-fix-vboxsvga.png) | Problem 2: VM settings after the fix (VBoxSVGA, 128 MB video memory, NAT Network LabNet) |
| [09-p3-promotion-hung-prerequisites.png](09-p3-promotion-hung-prerequisites.png) | Problem 3: `Install-ADDSForest` stuck on "Verifying prerequisites" |
| [10-p3-dcpromo-log-missing.png](10-p3-dcpromo-log-missing.png) | Problem 3: `dcpromo.log` didn't exist, so the promotion had never started |
| [11-p3-resolve-dnsname-10054.png](11-p3-resolve-dnsname-10054.png) | Problem 3: `Resolve-DnsName` error 10054, confirming DNS was pointing at a server that didn't exist yet |
| [12-p3-dns-working-after-fix.png](12-p3-dns-working-after-fix.png) | Problem 3: after pointing DNS at 1.1.1.1, lookups return A and AAAA records |
| [13-dc01-promotion-verified.png](13-dc01-promotion-verified.png) | Final verification: lab.local / LAB, NTDS, DNS and Netlogon running, DNS back to 127.0.0.1, SOA from dc01.lab.local |
| [14-p4-login-screen-no-shutdown.png](14-p4-login-screen-no-shutdown.png) | Problem 4: the LAB\Administrator login screen, which offers no shutdown option on Windows Server by default |

Every screenshot is checked before upload for passwords, product keys or personal details. Password prompts only ever show masked characters.

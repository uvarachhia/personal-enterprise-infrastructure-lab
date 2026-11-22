# 02 – Building DC01

## VM configuration

| Setting | Value |
| --- | --- |
| Name in VirtualBox | `DC01-V2` (Windows hostname: `DC01`) |
| OS | Windows Server 2025 Standard Evaluation (**Desktop Experience**) |
| vCPU / RAM / Disk | 2 / 3 GB / 60 GB dynamically allocated |
| Firmware | UEFI |
| Display | VBoxSVGA, 128 MB video memory, 3D acceleration off (see [troubleshooting](03-troubleshooting-log.md)) |
| Network | NAT Network `LabNet` |

I skipped VirtualBox's unattended install so I could choose the edition myself. The edition without "Desktop Experience" is Server Core, which has no GUI.

I took snapshots at each milestone (`dc01-fresh-install`, `dc01-pre-promotion`) so I could roll back safely instead of reinstalling.

## Before configuring: check the starting point

```powershell
hostname
Get-NetAdapter
ipconfig
```

The server had a random hostname and a `169.254.x.x` address with no gateway. That's an **APIPA** address: Windows couldn't find a DHCP server (expected, because LabNet's DHCP is off) and assigned itself a fallback address.

## Static IP

A domain controller needs a fixed address because every client has to find it in the same place.

```powershell
New-NetIPAddress -InterfaceAlias "Ethernet" -IPAddress 10.10.10.10 -PrefixLength 24 -DefaultGateway 10.10.10.1
Set-DnsClientServerAddress -InterfaceAlias "Ethernet" -ServerAddresses 127.0.0.1
ipconfig /all
ping 10.10.10.1
```

Result: DHCP disabled, IPv4 `10.10.10.10`, mask `255.255.255.0`, gateway `10.10.10.1`, DNS `127.0.0.1`, and 4/4 ping replies from the gateway.

`ipconfig /all` also showed **NetBIOS over TCP/IP: Enabled**. That's a default worth noting: NetBIOS and LLMNR are commonly disabled in hardening guides because attackers can abuse them for credential capture. I've left them on for now so I can observe them in packet captures later.

## Rename and activate

```powershell
Rename-Computer -NewName DC01 -Restart
```

Then I activated Windows (Settings → System → Activation). Evaluation copies must be activated online within 10 days or they start shutting down automatically.

I also set the time zone to Eastern Time. Kerberos rejects authentication when clocks differ by more than about 5 minutes, so correct time matters before building a domain.

## Promote to a domain controller

```powershell
Install-WindowsFeature AD-Domain-Services -IncludeManagementTools
Install-ADDSForest -DomainName "lab.local" -DomainNetbiosName "LAB" -InstallDns -CreateDnsDelegation:$false
```

- `Install-WindowsFeature` installs the AD DS software and admin tools. It doesn't create a domain.
- `Install-ADDSForest` creates a new forest and the `lab.local` domain, installs the DNS Server role and the `lab.local` zone (including the SRV records clients use to locate a DC), builds the AD database (`NTDS.dit`) and the `SYSVOL` share, then reboots.
- It prompts for a **DSRM** (Directory Services Restore Mode) password, which is a separate recovery credential used only if AD itself needs repair. I generated it locally and stored it separately from the Administrator password.
- `-CreateDnsDelegation:$false` skips registering `lab.local` with a parent `.local` zone, which doesn't exist. The first attempt hung without it; see the [troubleshooting log](03-troubleshooting-log.md#3-domain-controller-promotion-hung-on-verifying-prerequisites).

## Verification

```powershell
Get-ADDomain | Select-Object DNSRoot, NetBIOSName
Get-Service NTDS, DNS, Netlogon | Select-Object Name, Status
Get-DnsClientServerAddress -InterfaceAlias "Ethernet"
Resolve-DnsName lab.local
```

| Check | Result | Meaning |
| --- | --- | --- |
| Domain | `lab.local` / `LAB` | The domain exists |
| NTDS, DNS, Netlogon | All Running | AD database, name resolution and logon services are up |
| DC01's DNS client | `127.0.0.1` and `::1` | Promotion switched DNS back to itself, replacing my temporary setting |
| `lab.local` lookup | SOA record, primary server `dc01.lab.local` | DC01 is authoritative for the zone |

## Next steps

- Add a DNS forwarder so DC01 can resolve internet names: `Add-DnsServerForwarder -IPAddress 1.1.1.1`
- Confirm the DC locator SRV record: `Resolve-DnsName -Type SRV _ldap._tcp.dc._msdcs.lab.local`
- Snapshot: `dc01-promoted`
- Install and authorize DHCP

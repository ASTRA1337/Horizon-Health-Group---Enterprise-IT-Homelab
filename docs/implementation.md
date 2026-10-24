# Implementation: First domain controller

## Objective

Deploy and verify a Domain Controller and DNS service for the fictional organization `corp.horizonhealth.test` without disturbing the household network.

## Work completed

1. Created `HHG-DC01` in VirtualBox from the Windows Server 2025 Evaluation ISO, using Desktop Experience and manual installation.
2. Assigned **2 vCPUs**, **4 GiB RAM**, an **80 GB dynamically allocated disk**, and **UEFI**.
3. Attached VM network adapter 1 to VirtualBox **Internal Network** `HHG-LAN`, not Bridged Adapter.
4. Renamed the Windows computer `HHG-DC01`.
5. Configured static IPv4 `10.20.30.10/24`, no default gateway, and preferred DNS `10.20.30.10`.
6. Captured a pre-promotion VM snapshot.
7. Installed the **Active Directory Domain Services** role and promoted the server into a new forest, `corp.horizonhealth.test`, with DNS and Global Catalog enabled.
8. Verified directory and DNS functionality after promotion.

## Validation commands

Run inside the Windows Server VM in an elevated PowerShell window:

```powershell
whoami
Get-NetIPConfiguration -InterfaceAlias 'Ethernet'
Get-ADDomain
Resolve-DnsName -Name 'HHG-DC01.corp.horizonhealth.test' -Server 10.20.30.10
# The first character after "_" in "_ldap" is a lowercase L (l), not digit one (1).
Resolve-DnsName -Name '_ldap._tcp.dc._msdcs.corp.horizonhealth.test' -Type SRV -Server 10.20.30.10
nltest /dsgetdc:corp.horizonhealth.test /force
dcdiag /test:DNS /DnsRecordRegistration /s:HHG-DC01
```

## Observed results

| Check | Observation |
| --- | --- |
| `whoami` | `corp\administrator` |
| `Get-ADDomain` | Returned expected forest/domain and PDC emulator `HHG-DC01.corp.horizonhealth.test` |
| DNS hostname lookup | Returned `10.20.30.10` |
| Correctly spelled LDAP SRV query | Returned SRV record targeting the DC, port 389 |
| `nltest` | Located the DC and completed successfully |
| `dcdiag` | Reported DNS tests passed |

## Remaining verification

These server-side tests do not prove that clients can join the domain or that access policies work. Both require later tests from `HHG-CLIENT01` with synthetic user accounts.

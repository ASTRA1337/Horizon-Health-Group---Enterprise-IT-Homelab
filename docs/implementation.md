# Infrastructure implementation and validation

## Objective

Deploy an isolated Windows enterprise environment for the fictional `corp.horizonhealth.test` domain, then verify authentication, workstation management, and departmental file access from a separate VM.

## Domain controller: HHG-DC01

1. Created a Windows Server 2025 Evaluation VM in VirtualBox (Desktop Experience; 2 vCPUs; 4 GiB RAM; 80 GB dynamically allocated disk; UEFI).
2. Connected its adapter to Internal Network `HHG-LAN` and configured hostname `HHG-DC01`, IPv4 `10.20.30.10/24`, preferred DNS `10.20.30.10`, and no default gateway.
3. Captured a baseline snapshot before Active Directory deployment.
4. Installed AD DS and DNS and promoted the server as the initial domain controller and Global Catalog for `corp.horizonhealth.test`.
5. Created organizational units and departmental global security groups.

### Server validation

Run these commands inside **HHG-DC01** in elevated PowerShell:

~~~powershell
whoami
Get-NetIPConfiguration -InterfaceAlias 'Ethernet'
Get-ADDomain
Resolve-DnsName -Name 'HHG-DC01.corp.horizonhealth.test' -Server 10.20.30.10
# In "_ldap", the first character is lowercase L (l), not digit one (1).
Resolve-DnsName -Name '_ldap._tcp.dc._msdcs.corp.horizonhealth.test' -Type SRV -Server 10.20.30.10
nltest /dsgetdc:corp.horizonhealth.test /force
dcdiag /test:DNS /DnsRecordRegistration /s:HHG-DC01
~~~

**Observed:** The expected domain and PDC emulator were returned; the server name resolved to `10.20.30.10`; LDAP SRV discovery returned the domain controller on port **389**; `nltest` and `dcdiag` passed. See the [DNS troubleshooting case study](dns-troubleshooting.md).

## Domain workstation: HHG-CLIENT01

1. Installed Windows 11 Enterprise Evaluation in a separate VirtualBox VM with 2 vCPUs, 4 GiB RAM, 80 GB dynamic disk, UEFI/TPM.
2. Returned the VM to the isolated `HHG-LAN` network after a temporary activation step using NAT.
3. Set hostname `HHG-CLIENT01`, IPv4 `10.20.30.20/24`, and DNS `10.20.30.10`.
4. Validated DNS connectivity and LDAP SRV discovery from the client, joined `corp.horizonhealth.test`, and moved the computer object into `HHG/Workstations`.
5. Signed in with a synthetic domain account, completed the required initial password change, and verified the domain identity and logon server.

### Client verification

Run in **HHG-CLIENT01**:

~~~powershell
Test-NetConnection 10.20.30.10 -Port 53
Resolve-DnsName -Name '_ldap._tcp.dc._msdcs.corp.horizonhealth.test' -Type SRV -Server 10.20.30.10
whoami
hostname
$env:LOGONSERVER
Get-ItemPropertyValue -Path 'HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System' -Name 'InactivityTimeoutSecs'
~~~

**Observed:** DNS port 53 was reachable; the client found the LDAP SRV record; a domain user session showed `HHG-CLIENT01` with logon server `\\HHG-DC01`. The workstation inactivity policy value was **600 seconds**. An actual timed automatic lock remains unverified.

See [workstation and Group Policy validation](workstation-policy.md).

## Departmental access-control validation

On `HHG-DC01`, a folder at `C:\HHG-LabShares\Billing` was published as the `Billing` SMB share. Access was granted to the `CORP\GG_Billing` group at the share and NTFS layers. Verification from `HHG-CLIENT01` showed:

| Test account role | Expected | Observed |
| --- | --- | --- |
| Billing account | Read/write access | Opened share; created, saved, and reopened a text file |
| Patient Registration account | Denied | Windows reported insufficient permissions |

These tests used the IP-based UNC path `\\10.20.30.10\Billing`. The hostname-based UNC path still needs a separate post-configuration test.

See [SMB troubleshooting and permission verification](smb-share-troubleshooting.md).

## Remaining work

- Verify physical screen lock after ten idle minutes.
- Verify hostname-based SMB access and document name-resolution/authentication behavior.
- Implement account lifecycle automation, security monitoring, and backup/recovery tests.

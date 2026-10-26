# Enterprise IT Infrastructure & Security Homelab

A hands-on **simulated small-business IT environment** built with VirtualBox, Windows Server, and an isolated virtual network. The scenario follows **Horizon Health Group (HHG)**, a fictional 35-employee outpatient healthcare organization, to demonstrate infrastructure deployment, identity administration, endpoint policy, access control, and structured troubleshooting.

> **Scope:** This is an educational simulation, not a production deployment or professional IT employment. All accounts and business scenarios are synthetic.

## Business objective

Provide centrally managed identities, reliable domain services, consistent workstation controls, and least-privilege departmental file access. Verify each capability from a separate client instead of relying on configuration alone.

## Current architecture

~~~mermaid
flowchart TB
    LAN["HHG-LAN<br/>VirtualBox Internal Network<br/>10.20.30.0/24"]
    DC["HHG-DC01<br/>Windows Server 2025 Evaluation<br/>AD DS + DNS + lab SMB share<br/>10.20.30.10"]
    CLIENT["HHG-CLIENT01<br/>Windows 11 Enterprise Evaluation<br/>Domain-joined workstation<br/>10.20.30.20"]
    LAN --- DC
    LAN --- CLIENT
~~~

**Domain:** `corp.horizonhealth.test` (NetBIOS: `CORP`). Both VMs currently use the isolated `HHG-LAN` network with no default gateway. The file share is hosted on the DC for this small lab only; a dedicated file server is preferred in production.

## Verified milestones

- [x] Deploy `HHG-DC01` with Windows Server 2025 Evaluation and static networking.
- [x] Deploy AD DS and DNS for `corp.horizonhealth.test`; validate domain discovery and DNS SRV registration.
- [x] Create `HHG/Employees`, `HHG/Groups`, and `HHG/Workstations` OUs and departmental security groups.
- [x] Provision synthetic accounts for multiple departments and configure group membership.
- [x] Deploy `HHG-CLIENT01`, join the domain, move the computer object to `HHG/Workstations`, and verify domain-user sign-in and initial password change.
- [x] Configure a workstation inactivity GPO; verify the client has an effective timeout value of **600 seconds**.
- [x] Publish the `Billing` SMB share with departmental permissions and verify its share and NTFS ACLs.
- [x] Validate **allow**: a Billing account can open the share and create, save, and reopen a file.
- [x] Validate **deny**: a Patient Registration account receives a permission error for the Billing share.
- [ ] Observe the 10-minute automatic lock in a live idle-session test.
- [ ] Verify the hostname-based SMB path after deployment; direct IP-based access is verified.
- [ ] Automate account lifecycle tasks, implement logging/monitoring, and perform backup/recovery exercises.

## Documentation

- [Architecture and design decisions](docs/architecture.md)
- [Infrastructure implementation and validation](docs/implementation.md)
- [Identity administration and access model](docs/identity-administration.md)
- [Workstation join and Group Policy](docs/workstation-policy.md)
- [Troubleshooting case study: LDAP SRV transcription error](docs/dns-troubleshooting.md)
- [Troubleshooting case study: missing SMB share and access verification](docs/smb-share-troubleshooting.md)

## Technology

**Virtualization:** Oracle VirtualBox  
**Operating systems:** Windows Server 2025 Evaluation; Windows 11 Enterprise Evaluation  
**Infrastructure:** Active Directory Domain Services, DNS, Group Policy, SMB, NTFS ACLs  
**Tools:** PowerShell, Server Manager, Active Directory Users and Computers, Git, GitHub

## Security and publishing practices

No passwords, secrets, VM images, real employee details, or real patient data are committed. Account scenarios use generic departmental roles. The isolated lab and evaluation editions are not substitutes for production licensing, security architecture, or operational controls.

## Next phases

**Windows identity and access foundation → PowerShell administration → Security monitoring and recovery → Synthetic healthcare interoperability experiments (FHIR/HL7).**

Each phase records observed results separately from unverified or planned work.

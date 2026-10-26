# Architecture and design decisions

## Business scenario

Horizon Health Group is a fictional 35-employee outpatient healthcare organization with Billing, Clinical Operations, Patient Registration, and IT Administration departments. This lab models centralized authentication, workstation policy, and departmental resource access without storing real employee or patient records.

## Deployed virtual infrastructure

| Component | State | Configuration |
| --- | --- | --- |
| `HHG-LAN` | Deployed | VirtualBox Internal Network, `10.20.30.0/24`; no default gateway |
| `HHG-DC01` | Deployed | Windows Server 2025 Evaluation, 2 vCPUs, 4 GiB RAM, 80 GB dynamic disk, UEFI |
| Domain/DNS | Operational | `corp.horizonhealth.test` / `CORP`; DNS at `10.20.30.10` |
| `HHG-CLIENT01` | Deployed | Windows 11 Enterprise Evaluation, 2 vCPUs, 4 GiB RAM, 80 GB dynamic disk |
| Client networking | Configured | `10.20.30.20/24`, DNS `10.20.30.10`; Internal Network `HHG-LAN` |
| Workstation management | Tested | Domain join, sign-in, computer placed in `HHG/Workstations`, inactivity GPO applied |
| Departmental file access | Tested | Lab `Billing` SMB share on `HHG-DC01`; group-based share and NTFS permissions |

## Logical architecture

~~~text
corp.horizonhealth.test
└── HHG (OU)
    ├── Employees (synthetic accounts)
    ├── Groups (departmental global security groups)
    └── Workstations
        └── HHG-CLIENT01

HHG-LAN (10.20.30.0/24)
├── HHG-DC01     10.20.30.10   AD DS / DNS / temporary lab SMB
└── HHG-CLIENT01 10.20.30.20   Domain-joined Windows workstation
~~~

## Design rationale and tradeoffs

### Network isolation

VirtualBox Internal Network connects the guest systems without exposing the lab to the physical network. The client temporarily used a NAT adapter for Windows evaluation activation, then returned to `HHG-LAN`. Its current lab configuration has no default gateway.

### Single domain controller

One domain controller supports basic deployment and troubleshooting exercises but provides no high availability. A real organization should plan DC redundancy, secure backups, monitoring, privileged access controls, and disaster recovery.

### DNS and identity

The domain controller has a static IPv4 address, and the workstation uses it for DNS. DNS SRV records support domain controller discovery. Employee accounts belong to departmental security groups; authorization is applied to resources, not assumed from OU placement alone.

### Group Policy

A GPO linked to `HHG/Workstations` sets the interactive-logon machine inactivity limit to **600 seconds**. The effective value was confirmed on the client. A timed idle-lock observation remains outstanding.

### SMB file sharing

A folder stored at `C:\HHG-LabShares\Billing` is published as `\\HHG-DC01\Billing` through its SMB share name. Network tests succeeded using `\\10.20.30.10\Billing`. The share and NTFS ACLs grant department-level access, and separate synthetic user accounts confirmed an allowed read/write scenario and a denied access scenario.

Hosting departmental shares on a domain controller is a **temporary lab simplification**, not a production recommendation. In a production design, the file-serving role should be separated and recovery, auditing, and file-level access reviewed.

### Evaluation software

Evaluation editions are used strictly for learning, with licensing and time limits respected.

## Planned extensions

- Verify the hostname-based share path and live inactivity lock behavior.
- Automate employee provisioning/deprovisioning with PowerShell.
- Introduce monitoring, event-log analysis, and recovery tests.
- Extend the scenario with synthetic healthcare interoperability services using FHIR/HL7.

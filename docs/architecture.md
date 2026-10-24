# Architecture and design decisions

## Business scenario

**Horizon Health Group** is a fictional 35-employee outpatient healthcare organization with Clinical Operations, Patient Registration, Billing, and IT Administration departments. It needs centralized sign-in, controlled access to business resources, reliable name resolution, manageable workstations, and recovery capability.

No actual healthcare, patient, or employee data is stored in the lab.

## Virtual infrastructure

The environment runs as virtual machines managed by Oracle VirtualBox. The architecture focuses on the lab's guest systems, virtual network, and deployed services.

| Component | Status | Configuration |
| --- | --- | --- |
| `HHG-LAN` | Deployed | VirtualBox Internal Network; `10.20.30.0/24` |
| `HHG-DC01` | Deployed | Windows Server 2025 Evaluation; 2 vCPUs; 4 GiB RAM; 80 GB dynamic disk |
| IPv4 | Configured | `10.20.30.10/24` with no default gateway |
| Active Directory | Deployed | `corp.horizonhealth.test`; one forest and domain |
| DNS | Deployed | DNS on `HHG-DC01`, addressed at `10.20.30.10` |
| `HHG-CLIENT01` | Planned | Windows workstation for domain-join and policy tests |

## Design rationale and tradeoffs

### Isolated virtual networking

VirtualBox's **Internal Network** mode allows the lab VMs to communicate on `HHG-LAN`. No default gateway or external connectivity is configured at this stage. A controlled update path can be designed in a later phase.

### Single domain controller

A single DC is sufficient for foundational learning and service verification. It is **not** a recommended resilient production design; a real organization would evaluate redundancy, separate recovery copies, monitoring, patching, and privileged-access policies.

### Static IP and DNS

The DC has a predictable IPv4 address, and lab clients will use it for DNS. Active Directory uses DNS SRV records to help clients locate domain services.

### Evaluation software

Windows Server Evaluation is used only for learning. Its evaluation limits must be observed; it is not a production license.

## Next architecture decisions

1. Model organizational units based on policy and administrative boundaries.
2. Create departmental security groups separately from the OU hierarchy.
3. Domain-join a Windows workstation on `HHG-LAN` and test authentication.
4. Introduce controlled update access and security monitoring after reviewing network boundaries.

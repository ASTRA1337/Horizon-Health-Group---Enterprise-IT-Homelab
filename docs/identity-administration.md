# Identity administration and departmental access model

## Business objective

Horizon Health Group requires a repeatable way to organize accounts, assign departmental roles, enforce workstation settings, and limit access to department-specific resources. All identities used in this lab are synthetic; this documentation intentionally describes them by **role**, not by personal name or account identifier.

## Verified Active Directory structure

Domain: `corp.horizonhealth.test`

~~~text
corp.horizonhealth.test
└── HHG (OU)
    ├── Employees (OU)
    │   └── Synthetic departmental user accounts
    ├── Groups (OU)
    │   ├── GG_Billing
    │   ├── GG_ClinicalOps
    │   ├── GG_PatientRegistration
    │   └── GG_ITStaff
    └── Workstations (OU)
        └── HHG-CLIENT01
~~~

These are **global security groups** for organizational roles, not privileged domain administrator groups.

## Procedure and validation

1. Created the `HHG` organizational unit with `Employees`, `Groups`, and `Workstations` child OUs.
2. Created the four departmental global security groups.
3. Provisioned synthetic accounts for the **Billing** and **Patient Registration** departments.
4. Added each account to its corresponding departmental group; the Patient Registration account was not assigned to `GG_Billing`.
5. Domain-joined `HHG-CLIENT01` and moved its computer object into `HHG/Workstations`.
6. Confirmed a domain employee could sign in to the workstation, perform the required initial password change, and establish a session associated with `HHG-DC01`.
7. Validated resource access from the workstation: the Billing account could read/write the Billing SMB share, while the Patient Registration account was denied access.

## Authentication versus authorization

**Authentication:** The domain controller validates the synthetic employee's identity for Windows sign-in.

**Authorization:** The share and NTFS permissions determine whether that signed-in account can access departmental files.

Account membership alone does not grant access unless it is referenced in a resource ACL. The `Billing` share explicitly grants access to `CORP\GG_Billing`, not to all authenticated domain users. See the [SMB access-control case study](smb-share-troubleshooting.md).

## Security rationale

- OUs provide organization and Group Policy targeting.
- Security groups encode business roles separately from the OU hierarchy.
- Access to a departmental resource is granted only to the relevant group and administrative principals.
- Both authorized and unauthorized test cases are required to validate least privilege.
- No account passwords, real employee records, or identifying synthetic employee names are published.

## Future extensions

Automate joiner/mover/leaver account administration, validate group-membership changes, add security event auditing, and design a dedicated file-server VM for later phases.

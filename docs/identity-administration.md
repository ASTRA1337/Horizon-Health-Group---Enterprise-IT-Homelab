# Identity administration: initial configuration

## Business objective

Horizon Health Group needs a consistent way to organize user accounts and grant departmental access. This milestone establishes the first Active Directory structure and assigns a fictional employee to a department group.

## Verified directory structure

Domain: `corp.horizonhealth.test`

```text
corp.horizonhealth.test
└── HHG (organizational unit)
    ├── Employees (organizational unit)
    │   └── Maya Carter (maya.carter)
    ├── Groups (organizational unit)
    │   ├── GG_Billing
    │   ├── GG_ClinicalOps
    │   ├── GG_PatientRegistration
    │   └── GG_ITStaff
    └── Workstations (organizational unit)
```

The four department groups are intended as **global security groups**, not privileged domain administrator groups. The completed user account is fictional and belongs to `GG_Billing`.

## Procedure completed

1. Created `HHG` OU and child OUs `Employees`, `Groups`, and `Workstations` with Active Directory Users and Computers.
2. Created departmental security groups and organized them inside `HHG/Groups`.
3. Created fictional user `maya.carter` inside `HHG/Employees`, meeting the domain password policy.
4. Added the user to `GG_Billing` and confirmed the membership in Active Directory.

## What has not yet been tested

- No domain-joined workstation has authenticated as Maya.
- `GG_Billing` does not yet imply access to any specific file share or application.
- Group Policy, employee offboarding, and automated provisioning remain future tasks.

## Security rationale

The OUs organize objects for administration and later Group Policy targeting. Departmental security groups represent business roles. Resource-level permissions must be configured and tested separately to implement least privilege.

This is a simulated lab using synthetic identities. No credentials are documented.

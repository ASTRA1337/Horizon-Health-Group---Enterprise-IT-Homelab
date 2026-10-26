# Case study: Missing SMB share and departmental access verification

## Objective

Provide a network folder for a fictional Billing department and verify that a Billing account can read/write files while a Patient Registration account cannot.

**Lab-only design:** The folder is hosted on the domain controller to keep this small simulation simple. A separate file server would be preferable in production.

## Symptom

The Billing folder had been created on `HHG-DC01`, but opening the intended network resource from `HHG-CLIENT01` did not initially reach the folder. Windows Network browsing was also empty because network discovery was disabled; that browsing view is distinct from a direct UNC connection.

The initial investigation focused on **whether the server had published an SMB share**, rather than assuming folder existence implied network availability.

## Evidence and root cause

On **HHG-DC01**, elevated PowerShell returned:

~~~powershell
Test-Path 'C:\HHG-LabShares\Billing'
# True

Get-SmbShare -Name 'Billing'
# No matching MSFT_SMBShare object found
~~~

The folder existed locally, but **no network share named Billing existed**. A local disk path and an SMB share name are separate objects.

## Resolution

Create the SMB share and explicitly set share-level permissions:

~~~powershell
New-SmbShare -Name 'Billing' -Path 'C:\HHG-LabShares\Billing' -ChangeAccess 'CORP\GG_Billing' -FullAccess 'BUILTIN\Administrators'
~~~

Then confirm that Windows actually published it:

~~~powershell
Get-SmbShare -Name 'Billing'
Get-SmbShareAccess -Name 'Billing'
~~~

**Observed:** `Billing` mapped to `C:\HHG-LabShares\Billing`. `CORP\GG_Billing` had **Change** share access; `BUILTIN\Administrators` had **Full** share access.

### Why the network path omits the parent directory

- Physical location on server: `C:\HHG-LabShares\Billing`
- Published share name: `Billing`
- UNC path: `\\HHG-DC01\Billing`
- IP-based UNC path: `\\10.20.30.10\Billing`

The SMB server maps the share name to its physical directory. Clients do **not** prepend `HHG-LabShares` unless that parent directory has separately been published as a share.

## NTFS permission review

SMB share permissions and NTFS permissions are **independent layers**. Both restrict effective access for a network connection.

Read the folder ACL on **HHG-DC01**:

~~~powershell
icacls 'C:\HHG-LabShares\Billing'
~~~

**Initial observation:** The intended `CORP\GG_Billing` Modify permission was present, but the ACL also contained broader `BUILTIN\Users` grants.

Remove the unnecessary *granted* permissions on the lab's Billing folder, then re-check:

~~~powershell
icacls 'C:\HHG-LabShares\Billing' /remove:g 'BUILTIN\Users'
icacls 'C:\HHG-LabShares\Billing'
~~~

**Final observed entries:**

| Principal | NTFS rights |
| --- | --- |
| `CORP\GG_Billing` | Modify `(M)`, inheritable to files/subfolders |
| `BUILTIN\Administrators` | Full `(F)` |
| `NT AUTHORITY\SYSTEM` | Full `(F)` |
| `CREATOR OWNER` | Inherit-only Full Control on child items |

No explicit Deny entry was required for the non-Billing role. The ACL change targeted the Billing folder, not unrelated directories; no recursive `/T` option was used.

## End-to-end validation

The tests were performed on **HHG-CLIENT01** using the IP-based UNC path `\\10.20.30.10\Billing`:

| Test identity | Action | Actual result |
| --- | --- | --- |
| Synthetic Billing employee | Open share, create/save/reopen a text document | **Allowed** |
| Synthetic Patient Registration employee | Open the same share | **Denied** with Windows permission error |

This verifies both the successful and unsuccessful access path for two different departmental roles. The initial folder listing was empty, which was expected before a test document was created.

## Diagnostic lessons

1. A directory existing on disk does **not** mean it has been shared.
2. Use `Test-Path` to validate local existence, `Get-SmbShare` to validate publication, `Get-SmbShareAccess` to inspect share rights, and `icacls` to inspect NTFS rights.
3. Windows Network discovery and direct access by UNC path are different mechanisms; lack of browsing results does not establish that SMB is unavailable.
4. Use role-appropriate test accounts to demonstrate both authorized and unauthorized access.
5. Do not confuse the share name with the folder's physical parent path.
6. Record a result as verified only when a test actually demonstrated it.

## Follow-up

Verify `\\HHG-DC01\Billing` by hostname **after** the share exists, and compare name-resolution/authentication behavior with the IP-based path. The successful end-to-end access tests above used the IP address. Review dedicated file-server architecture, auditing, and backup controls before treating this design as production-like.

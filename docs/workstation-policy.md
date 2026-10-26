# Workstation domain join and Group Policy verification

## Purpose

Demonstrate that an independently deployed Windows workstation can discover Active Directory, join the simulated company domain, authenticate a synthetic employee, and receive centrally managed security settings.

## Environment

| Item | Setting |
| --- | --- |
| VM | `HHG-CLIENT01` |
| OS | Windows 11 Enterprise Evaluation |
| Network | VirtualBox Internal Network `HHG-LAN` |
| IPv4 | `10.20.30.20/24` |
| DNS | `10.20.30.10` |
| Domain | `corp.horizonhealth.test` |
| Workstation OU | `HHG/Workstations` |

The VM briefly used NAT for evaluation activation before returning to the isolated Internal Network. No default gateway is configured in the current lab network.

## Verification before joining

On the **client**, the following commands tested service reachability and directory discovery:

~~~powershell
Test-NetConnection 10.20.30.10 -Port 53
Resolve-DnsName -Name '_ldap._tcp.dc._msdcs.corp.horizonhealth.test' -Type SRV -Server 10.20.30.10
~~~

**Observed:** TCP port 53 was reachable and the corrected DNS SRV query returned the domain controller, port 389.

The domain join used Windows System Properties with authorized domain administrator credentials. The join produced a welcome confirmation, followed by a reboot. The computer object was then moved from the default Computers container into `HHG/Workstations`.

## Domain logon validation

After an initial mandatory password change, a synthetic departmental account signed in successfully. Client-side checks included:

~~~powershell
whoami
hostname
$env:LOGONSERVER
~~~

**Observed:** The account belonged to the `CORP` domain, the computer name was `HHG-CLIENT01`, and the logon server was `\\HHG-DC01`.

## Workstation security baseline

A GPO named **HHG - Workstation Security Baseline** was linked to `HHG/Workstations`.

Configured path:

~~~text
Computer Configuration
  > Policies
  > Windows Settings
  > Security Settings
  > Local Policies
  > Security Options
  > Interactive logon: Machine inactivity limit
~~~

The inactivity limit was set to **600 seconds** (10 minutes). On the client, policy refresh and a restart were part of the deployment procedure. The effective registry policy value was checked with:

~~~powershell
Get-ItemPropertyValue -Path 'HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System' -Name 'InactivityTimeoutSecs'
~~~

**Observed:** `600`.

**Not yet verified:** A complete ten-minute idle test showing the workstation actually locking. The registry value confirms the configured policy reached the client, but is not a substitute for the behavioral test.

## Practical lessons

- Verify client-side domain discovery before attempting a join.
- Group Policy must be tested on the target computer, not just created in the management console.
- Keep computer-targeted policy linked to the correct computer OU.
- Distinguish configuration evidence from behavior observed during a live test.

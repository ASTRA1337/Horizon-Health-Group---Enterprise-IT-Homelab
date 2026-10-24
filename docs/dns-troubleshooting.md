# Case study: LDAP SRV query failure caused by a transcription error

## Symptom

After Active Directory promotion, the server resolved the DC hostname but repeated DNS queries for the LDAP SRV service returned **DNS name does not exist**. Domain configuration appeared correct, creating a misleading symptom of broken service discovery.

## Why the check matters

Active Directory relies on DNS SRV records to advertise services used in Domain Controller discovery. A genuine missing record could interfere with client domain joins and authentication.

## Diagnostic trail

- `Get-ADDomain` returned the intended domain and PDC emulator.
- `Get-DnsServerZone` showed both `corp.horizonhealth.test` and `_msdcs.corp.horizonhealth.test`.
- `Get-DnsServerResourceRecord` showed LDAP and Kerberos SRV records in DNS.
- `dcdiag /test:DNS /DnsRecordRegistration /s:HHG-DC01` passed.
- `nltest /dsgetdc:corp.horizonhealth.test /force` discovered `HHG-DC01`.
- DNS was listening on `10.20.30.10:53`; an SOA query succeeded.
- Netlogon restart and DNS cache/zone refresh did not resolve the observed query failure.

## Root cause

The typed DNS name used **digit one (`1`) instead of lowercase letter L (`l`)** in the LDAP service label.

**Correct:** `_ldap` — lowercase letter L.  
**Incorrect:** `_1dap` — digit one.

The server was behaving as expected: the mistyped query requested a name that did not exist. The issue was not missing AD records or a DNS outage.

## Correct command

```powershell
# "_ldap" begins with lowercase L (l), not the number one (1).
Resolve-DnsName -Name '_ldap._tcp.dc._msdcs.corp.horizonhealth.test' -Type SRV -Server 10.20.30.10
```

**Observed:** The corrected query returned an SRV answer targeting `HHG-DC01.corp.horizonhealth.test` with LDAP on port **389**.

## Lessons learned

1. Verify exact query strings and distinguish ambiguous characters **before** changing infrastructure.
2. Separate evidence about stored DNS records from evidence about a requested DNS name.
3. Prefer low-impact, read-only diagnostics before service restarts or server configuration changes.
4. Record failed assumptions alongside successful resolutions.

This case illustrates a troubleshooting process and its correction; it does not imply production incident-response experience.

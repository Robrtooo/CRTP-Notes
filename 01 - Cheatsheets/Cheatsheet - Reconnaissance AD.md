# Cheatsheet - Reconnaissance AD

> [!summary] Objectif
> First-pass recon : domaine, SID, DC, policy et services AD exposés, avant LDAP / SMB / Kerberos / BloodHound. Début de lab, pour valider la cible.

### DNS / basic network
```bash
nslookup <DOMAIN>
nslookup <DC_IP>
nmap -sV -Pn -p 53,88,135,139,389,445,464,636,3268,3269 <DC_IP>
```

### PowerView - current domain
```powershell
. C:\AD\Tools\PowerView.ps1
Get-Domain
Get-DomainSID
Get-DomainController
Get-DomainPolicyData
```

### AD module - same checks
```powershell
Get-ADDomain
(Get-ADDomain).DomainSID
Get-ADDomainController
```

### Another domain in lab
```powershell
Get-Domain -Domain moneycorp.local
Get-ADDomainController -DomainName moneycorp.local -Discover
```

> [!note] PowerView et AD Module sont les deux pivots montrés en parallèle. Garder domaines et DC séparés dès qu'on passe en multi-domaines / forêts.

## Related
[[Enumeration]] · [[CheetSheet]] · [[Cheatsheet - LDAP SMB Kerberos]] · [[Cheatsheet - BloodHound SharpHound]]

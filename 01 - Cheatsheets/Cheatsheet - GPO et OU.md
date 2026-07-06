# Cheatsheet - GPO et OU

> [!summary] Objectif
> Lister les OU, relier une OU à sa GPO (`gplink`) et recouper les ACL des policies. Ordre à garder : OU → gplink → GPO → ACL.

### OU enumeration

```powershell
. C:\AD\Tools\PowerView.ps1
Get-DomainOU | Select-Object -ExpandProperty name
(Get-DomainOU -Identity DevOps).distinguishedname | %{Get-DomainComputer -SearchBase $_} | Select-Object name
```

### List and read GPOs

```powershell
Get-DomainGPO
Get-DomainGPO -Identity "DevOps Policy"
```

### Resolve a GPO from OU gplink

```powershell
(Get-DomainOU -Identity DevOps).gplink
Get-DomainGPO -Identity (Get-DomainOU -Identity DevOps).gplink.substring(11,(Get-DomainOU -Identity DevOps).gplink.length-72)
```

> [!note] Ne pas confondre `displayname` et `name/GUID`. BloodHound (`Search: Applocked / DevOps` → Inbound Object Control) lit plus vite `GenericAll` / `WriteDACL` sur les policies.

## Related
- [[Cheatsheet - ACL et droits AD]]
- [[Cheatsheet - Monitoring et détections]]
- [[Synthèse - ACL et GPO]]

# Cheatsheet - GPO et OU

> [!summary] Objectif
> Lister les OU, relier une OU à sa GPO (`gplink`) et recouper les ACL des policies. Ordre à garder : OU → gplink → GPO → ACL.

> [!abstract] Pourquoi · Chaînage
> Couche **identité (AD)**. **Repérer** quelle GPO frappe quelle OU (gplink) = flag · **Exploiter** = droit d'écriture sur la GPO → SharpGPOAbuse → code sur toutes les machines de l'OU.
> **Prérequis** write sur la GPO → **débloque** exec/admin sur les machines liées.

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

Pour lister les GPO d'une OU, lister les OU et récuperer les GUID pour en lister les GPO
```powershell
Get-DomainGPO -Identity "{7478F170-6A0C-490C-B355-9E4618BC785D}"
```

> [!note] Ne pas confondre `displayname` et `name/GUID`. BloodHound (`Search: Applocked / DevOps` → Inbound Object Control) lit plus vite `GenericAll` / `WriteDACL` sur les policies.

## Related
- [[Cheatsheet - ACL et droits AD]]
- [[Cheatsheet - Monitoring et détections]]
- [[Synthèse - ACL et GPO]]

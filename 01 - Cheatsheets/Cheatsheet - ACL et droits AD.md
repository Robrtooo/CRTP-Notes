# Cheatsheet - ACL et droits AD

> [!summary] Objectif
> Repérer rapidement les droits intéressants sur groupes, users, computers et policies.

> [!warning] Cadre d'utilisation
> À utiliser uniquement dans un environnement de lab autorisé, dans le cadre de l'apprentissage et de la préparation CRTP.

## Quick use

- `WriteDACL`
- `GenericAll`
- `GenericWrite`
- `ForceChangePassword`
- object control in BloodHound

## Core commands

### Read ACLs on a sensitive object

```powershell
. C:\AD\Tools\PowerView.ps1
Get-DomainObjectAcl -Identity "Domain Admins" -ResolveGUIDs -Verbose
```

Commande de base du PDF pour lire les ACE d’un objet important.

### Find interesting ACLs for a compromised principal

```powershell
Find-InterestingDomainAcl -ResolveGUIDs | Where-Object {$_.IdentityReferenceName -match "studentx"}
Find-InterestingDomainAcl -ResolveGUIDs | Where-Object {$_.IdentityReferenceName -match "RDPUsers"}
```

Le lab manual montre ce workflow exact : user direct, puis groupe du user.

### BloodHound pivot

```text
BloodHound CE / Legacy
- Search: studentx
- Outbound Object Control
- Search: Applocked / DevOps
- Inbound Object Control
```

Permet de voir vite les droits les plus exploitables sans parser toute la sortie PowerView.

## PDF notes

- Dans le support, `RDPUsers` ressort sur des objets intéressants même quand `studentx` seul ne remonte rien.
- Toujours noter : `ObjectDN`, droit exact, principal, impact concret.

## Related

- [[Cheatsheet - BloodHound SharpHound]]
- [[Cheatsheet - GPO et OU]]
- [[Cheatsheet - RBCD]]
- [[Synthèse - ACL et GPO]]


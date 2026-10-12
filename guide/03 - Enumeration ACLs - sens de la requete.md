# 03 — Énumération ACLs : le SENS de la requête

Le piège n°1 sur les ACLs : confondre **qui détient le droit** (principal / IdentityReference) et **sur quel objet** (cible / Object).

## Les deux cmdlets

### `Find-InterestingDomainAcl`
Balaye **tout le domaine** et sort les ACEs "intéressantes" (droits abusables : `GenericAll`, `WriteDacl`, `WriteOwner`, `AllExtendedRights`...).
Filtrer sur le **principal** → répond : *« qu'est-ce que X peut faire ? »*
```powershell
Find-InterestingDomainAcl -ResolveGUIDs | ?{$_.IdentityReferenceName -match "RDPUsers"}
```

### `Get-DomainObjectAcl -Identity <cible>`
Liste **toutes** les ACEs **posées SUR** un objet précis.
Filtrer par cible → répond : *« qui a des droits sur Y ? »*
```powershell
Get-DomainObjectAcl -Identity "Domain Admins" -ResolveGUIDs -Verbose
```

## Tableau mémo
| Objectif | Cmdlet | On filtre sur |
|---|---|---|
| Ce que **X peut faire** | `Find-InterestingDomainAcl` | le **principal** (identity) |
| Qui peut agir **sur Y** | `Get-DomainObjectAcl -Identity Y` | la **cible** (object) |

> [!WARNING] "droits DE A SUR B"
> Ex : *ActiveDirectoryRights for **RDPUsers** on **ControlxUser***.
> - principal (identity) = **RDPUsers**
> - cible (object) = **ControlxUser**
> Décide QUI est quoi AVANT de choisir la cmdlet et le champ de filtre.

## Différence clé
- `Find-InterestingDomainAcl` ne montre **QUE** les ACEs jugées intéressantes (filtrées). Un droit "banal" n'apparaît pas → si vide, repasse par `Get-DomainObjectAcl` qui montre **TOUT**.

Suite : [[04 - Debug des requetes ACL]] · [[00 - Index]]

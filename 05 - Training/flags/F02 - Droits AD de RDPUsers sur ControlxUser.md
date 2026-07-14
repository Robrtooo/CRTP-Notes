---
flag: 2
machine: Student VM
statut: validé
technique: ACL enumeration
LO: "[[LO03 - ACLs]]"
---

# F02 - Droits AD de RDPUsers sur ControlxUser

> [!summary] Objectif
> Identifier le droit AD exact que le groupe **RDPUsers** possède sur l'utilisateur cible `ControlxUser` (placeholder — le vrai samAccountName est à découvrir).

## Contexte
- **Étape attack flow**: reconnaissance ACL — repérer un droit abusable sur un principal contrôlable (couche identité/AD).
- **Ce qu'on a**: `studentx` membre du groupe `RDPUsers`. PowerView chargé.
- **Ce qu'on cherche**: le nom du droit ADRights (ex. `GenericAll`, `GenericWrite`, `WriteDACL`, `ForceChangePassword`) porté par `RDPUsers` sur l'objet user `Controlxxxx`.

## Solution

### Étape 1: SID du principal (RDPUsers)
```powershell
. C:\AD\Tools\PowerView.ps1
Get-DomainGroup -Identity RDPUsers | select samaccountname,objectsid
```

### Étape 2: chercher toutes les ACEs où RDPUsers est le principal
```powershell
# Balayer les ACL "interessantes" du domaine et filtrer par SID du principal
Find-InterestingDomainAcl -ResolveGUIDs |
  Where-Object { $_.SecurityIdentifier -eq "<SID_RDPUsers A REMPLIR DEPUIS LAB>" } |
  Select-Object ObjectDN, ActiveDirectoryRights, AceQualifier
```
→ Voir [[Cheatsheet - ACL et droits AD]] pour variantes (`-match "RDPUsers"`).

### Étape 3 (pivot BloodHound)
```cypher
// Outbound object control de RDPUsers — flag F02
MATCH p=(g:Group)-[r]->(n) WHERE g.name STARTS WITH "RDPUSERS@" RETURN p
```
→ Voir [[Cheatsheet - BloodHound SharpHound]].

## Réponse

| Élément | Valeur |
|---------|--------|
| Droit ADRights de RDPUsers sur `ControlxUser` | `GenericAll` |
| samAccountName cible réel | `<A REMPLIR DEPUIS LAB>` |

`GenericAll` sur un user → reset de son password sans connaître l'ancien (`Set-DomainUserPassword`), ou add à un groupe si le user est membre d'un groupe qu'on peut manipuler.

## Pièges et OPSEC
- `-Identity` de `Get-DomainObjectAcl` = la **cible** (objet dont on lit l'ACL), **jamais le principal**.
- Le principal est dans le champ `SecurityIdentifier`, pas dans `ObjectDN`.
- `ControlxUser` = **placeholder** dans l'énoncé, pas un vrai samAccountName. Le vrai nom se déduit de l'`ObjectDN` de l'ACE.
- `AccessAllowed` = valeur de `AceQualifier` (Allow vs Deny), **PAS un droit AD**. Le droit est dans `ActiveDirectoryRights`.
- Bruit fréquent : ACE `ListChildren` / `ReadProperty` sur des OU = non exploitable, ignorer.

## Refs
- [[Cheatsheet - ACL et droits AD]]
- [[Cheatsheet - BloodHound SharpHound]]
- [[Synthèse - ACL et GPO]]
- [[LO03 - ACLs]]

[[Flags - Index]] · [[Journal - Index]]

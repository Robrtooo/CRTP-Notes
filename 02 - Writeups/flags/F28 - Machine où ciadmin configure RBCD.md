---
flag: 28
machine: dcorp-dc
statut: à faire
technique: RBCD (write ACL discovery)
LO: "[[LO17 - RBCD]]"
---

# F28 - Machine ou ciadmin configure RBCD

> [!summary] Objectif
> Trouver le compte machine sur lequel l'user `ciadmin` a un droit d'ecriture (`GenericWrite`/`GenericAll`/`WriteOwner`/`WriteDACL`) — c'est la cible RBCD car ce droit permet d'ecrire `msDS-AllowedToActOnBehalfOfOtherIdentity`.

## Contexte
- **Etape attack flow**: mise en place RBCD ([[LO17 - RBCD]]) — on cherche cote LDAP la relation "user contrôlé → GenericWrite sur computer" pour ensuite s'ajouter dans `msDS-AllowedToActOnBehalfOfOtherIdentity` et faire S4U.
- **Ce qu on a**: creds/hash de `ciadmin` (obtenus en amont), shell studentx avec PowerView.
- **Ce qu on cherche**: le samAccountName du computer object sur lequel `ciadmin` a le droit d'ecriture.

## Solution

### Etape 1: reperer les ACL interessantes de ciadmin
```powershell
. C:\AD\Tools\PowerView.ps1
Find-InterestingDomainAcl | Where-Object {$_.IdentityReferenceName -match 'ciadmin'}
```
→ Voir [[Cheatsheet - RBCD]].

### Etape 2: filtrer sur les objets computer
```powershell
Find-InterestingDomainAcl | ? {$_.IdentityReferenceName -match 'ciadmin'} `
  | ? {$_.ObjectClass -eq 'computer' -or $_.ObjectDN -match 'CN=Computers'} `
  | Select-Object ObjectDN,ActiveDirectoryRights,IdentityReferenceName
```
→ Voir [[Cheatsheet - ACL et droits AD]].

### Etape 3 (verif): BloodHound
Requete Cypher :
`MATCH p=(u:User {name:'CIADMIN@DOLLARCORP.MONEYCORP.LOCAL'})-[:GenericWrite|GenericAll|WriteOwner|WriteDacl]->(c:Computer) RETURN p`
→ Voir [[Cheatsheet - BloodHound SharpHound]].

## Reponse

| Element | Valeur |
|---------|--------|
| Computer object ou ciadmin peut configurer RBCD | <A REMPLIR DEPUIS LAB> |

## Pieges et OPSEC
- Un droit `WriteOwner` seul suffit aussi (on se met owner puis on ecrit) — ne filtre pas uniquement sur `GenericWrite`.
- `Find-InterestingDomainAcl` est bruyant en LDAP mais souvent whitelist en lab.
- Pour configurer effectivement le RBCD ensuite : `Set-DomainRBCD -Identity <computer_cible> -DelegateFrom 'dcorp-studentx$'` (voir [[Cheatsheet - RBCD]]).
- Bien noter les 4 elements pour la suite : **objet controle** (ciadmin), **machine deleguee** (studentx$), **SPN cible** (`http/<computer>` typiquement), **user impersonne** (Administrator).

## Refs
- [[Cheatsheet - RBCD]]
- [[Cheatsheet - ACL et droits AD]]
- [[Cheatsheet - BloodHound SharpHound]]
- [[LO17 - RBCD]]
- [[Synthèse - Kerberos]]

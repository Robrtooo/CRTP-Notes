---
flag: 2
machine: Student VM
statut: validé
---
# F02 · Droits AD de RDPUsers sur ControlxUser

> [!question] LO2 — Student VM
> ActiveDirectory Rights for RDPUsers group on the users named ControlxUser

## Commande qui marche
```powershell
# 1. SID du principal (RDPUsers)
Get-DomainGroup -Identity RDPUsers | select samaccountname,objectsid

# 2. balayer les ACEs où RDPUsers est le principal
Find-InterestingDomainAcl -ResolveGUIDs |
 ?{$_.SecurityIdentifier -eq "S-1-5-21-719815819-3726368948-3917688648-1123"} | 
 
 -----
 Get-DomainObjectAcl -Identity "S-1-5-21-719815819-3726368948-3917688648-1123" -ResolveGUIDs -Verbose
 
```
Réponse : `GenericAll` (sur le user cible ; `ListChildren`/`ReadProperty` sur les OU = bruit).

> [!warning] Pièges
> - `-Identity` = la **cible** (objet dont on lit l'ACL), jamais le principal.
> - le principal est dans le champ **`SecurityIdentifier`** (pas `ObjectDN`).
> - "ControlxUser" = **placeholder**, pas un vrai nom → à découvrir via l'ObjectDN.
> - `AccessAllowed` = `AceQualifier` (Allow/Deny), PAS un droit.

## Fiches liées
[[Cheatsheet - ACL et droits AD]]

[[Flags - Index]] · [[Journal - Index]]

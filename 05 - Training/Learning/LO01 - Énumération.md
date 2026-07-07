---
lo: 1
statut: fait
---
# LO1 - Énumération

> [!info] Objectif
> Énumérer users, computers, Domain Admins, Enterprise Admins. BloodHound : plus court chemin vers Domain Admins. Trouver un partage où studentX a le droit d'écriture.

## Fiches liées

- [[Cheatsheet - Reconnaissance AD]]
- [[Cheatsheet - BloodHound SharpHound]]

## Ce que j'ai fait
- Tenté d'utiliser Get-DomainSID
- Get-DomainGroupMember -Identity "Enterprise Admins" -Domain dcorp.local
--> dcorp.local est un subdomain du domain souhaité, donc le groupe Enterprise Admins n'exite pas dedans
- Get-DomainGroupMember -Identity "Enterprise Admins" -Domain moneycorp.local

_(à remplir pendant le lab)_


## Commande qui marche

```powershell
Get-DomainGroupMember -Identity "Enterprise Admins" -Domain moneycorp.local
```

## Piège / à retenir

Faire attention au scope sur lequel on cherche l'information

## Voir aussi
[[Journal - Index]] · [[2. Attack flow]]

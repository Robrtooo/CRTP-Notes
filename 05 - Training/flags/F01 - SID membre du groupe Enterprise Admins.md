---
flag: 1
machine: Student VM
statut: validé
---
# F01 · SID membre du groupe Enterprise Admins

> [!question] LO1 — Student VM
> SID of the member of the Enterprise Admins group

## Commande qui marche
```powershell
Get-DomainGroupMember -Identity "Enterprise Admins" -Domain moneycorp.local
```
Réponse : `S-1-5-21-335606122-960912869-3279953914-500`

## Fiches liées
[[Cheatsheet - Reconnaissance AD]] · [[Cheatsheet - BloodHound SharpHound]]

[[Flags - Index]] · [[Journal - Index]]

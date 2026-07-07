---
flag: 3
machine: Student VM
statut: validé
---
# F03 · GPO appliquée sur StudentMachines OU

> [!question] LO3 — Student VM
> Display name of the GPO applied on StudentMachines OU

## Commande qui marche
1. Récuperer le GUID de l'OU StudentMachines
```powershell
Get-DomainOU 
```
2. Lister les GPO associées à cette OU 
```powershell
Get-DomainGPO -Identity "{7478F170-6A0C-490C-B355-9E4618BC785D}"
```

## Fiches liées
[[Cheatsheet - GPO et OU]]

[[Flags - Index]] · [[Journal - Index]]

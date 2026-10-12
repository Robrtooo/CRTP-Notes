# 01 — Charger PowerView

## Symptôme
```
Get-DomainSID : The term 'Get-DomainSID' is not recognized as the name of a cmdlet...
CommandNotFoundException
```

## Cause
`Get-DomainSID`, `Get-DomainUser`, `Find-InterestingDomainAcl`, etc. sont des **fonctions PowerView**, PAS des cmdlets natives Windows. L'erreur = PowerView n'est pas chargé dans la session.

> [!warning] AMSI bloque dès le départ (autre erreur, à ne pas confondre)
> Si au dot-source tu obtiens `malicious content` / `cannot be loaded` (et non `not recognized`), c'est **AMSI**, pas un oubli de chargement. Lance **InviShell** (`RunWithRegistryNonAdmin.bat`) ou patche AMSI **AVANT** de charger PowerView. *C'est le tout premier blocage du lab.* Détail : [[Cheatsheet - Évasion et OPSEC]].

## Fix — dot-source le script
```powershell
# 1. bypass execution policy (session courante)
powershell -ep bypass

# 2. dot-source (point + espace obligatoire)
. C:\AD\Tools\PowerView.ps1
```

> [!IMPORTANT] Dot-source vs call
> `.` + espace (`. .\PowerView.ps1`) charge les fonctions dans le scope courant.
> `& .\PowerView.ps1` NE les rend PAS disponibles ensuite.

## Vérifier que c'est chargé
```powershell
Get-Command Get-DomainSID
```

## Si AMSI / AppLocker bloque
```powershell
# charger en mémoire
iex (Get-Content C:\AD\Tools\PowerView.ps1 -Raw)
```
Ou utiliser une version obfusquée type `PowerView_dev.ps1`.

## Alternative native (module AD / RSAT)
```powershell
Get-ADDomain | select DomainSID
```

Voir aussi [[00 - Index]]

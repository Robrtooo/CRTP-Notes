# Cheatsheet - Delegation attacks

> [!summary] Objectif
> Détecter et abuser les délégations Kerberos. Ne jamais mélanger les 4 cas : unconstrained / constrained user / constrained computer / RBCD.

### Unconstrained delegation

> Compromettre ce serveur permet de capturer le **TGT complet** de tout user qui s'y connecte (y compris un DA) et de le rejouer pour l'usurper intégralement — combiné à une coercition (Printer Bug), on force même le DC à s'y authentifier pour voler son TGT.

> [!warning]- Required
> Accès (code execution / admin local) sur le serveur en unconstrained delegation, pour y capturer les TGT entrants — voir [[Cheatsheet - Lateral movement]] / [[Cheatsheet - Privilege escalation]] pour l'obtenir.

```powershell
. C:\AD\Tools\PowerView.ps1
Get-DomainComputer -Unconstrained | Select-Object -ExpandProperty Name
```

### Constrained delegation

> Contrôler ce compte permet de s'authentifier **en tant que n'importe quel user du domaine** (via S4U2Self, sans connaître son mot de passe) auprès du service listé dans `msDS-AllowedToDelegateTo` — en ciblant un DA, ça donne un accès admin à ce service précis.

> [!warning]- Required
> Hash ou clé du compte qui porte la délégation contrainte (le `<svc>` de la commande ci-dessous) — s'obtient par DCSync ou extraction locale, voir [[Cheatsheet - Persistence]].

```powershell
Get-DomainUser -TrustedToAuth
Get-DomainComputer -TrustedToAuth
```

### Abuse constrained (S4U)

```powershell
Rubeus.exe s4u /user:<svc> /rc4:<hash> /impersonateuser:administrator /msdsspn:<spn> /altservice:<svc> /ptt
```

> [!note] Deux commandes à retenir d'abord : `Get-DomainComputer -Unconstrained` et `Get-DomainUser -TrustedToAuth`. RBCD (délégation définie côté cible, pas côté délégué) → fiche dédiée. Coercition (Printer Bug / MS-WSP / MS-DFSNM) et abus unconstrained → [[1. Cheatsheet - Toutes les commandes]].

## Related
[[Cheatsheet - RBCD]] · [[Cheatsheet - Trusts et forêts]] · [[Synthèse - Kerberos]]

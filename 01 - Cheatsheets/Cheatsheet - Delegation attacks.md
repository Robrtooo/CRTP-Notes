# Cheatsheet - Delegation attacks

> [!summary] Objectif
> Vérifier rapidement les prérequis delegation vus dans le PDF : unconstrained, constrained et comptes/machines concernés.

> [!warning] Cadre d'utilisation
> À utiliser uniquement dans un environnement de lab autorisé, dans le cadre de l'apprentissage et de la préparation CRTP.

## Quick use

- Unconstrained delegation
- Constrained delegation
- user vs computer delegation

## Core commands

### Unconstrained delegation

```powershell
. C:\AD\Tools\PowerView.ps1
Get-DomainComputer -Unconstrained | Select-Object -ExpandProperty Name
```

Le PDF l’utilise pour trouver le serveur pivot avant la suite des actions Kerberos.

### Constrained delegation - users

```powershell
Get-DomainUser -TrustedToAuth
Get-DomainUser -LDAPFilter "(msDS-AllowedToDelegateTo=*)"
```

Repère les comptes user portant les attributs utiles.

### Constrained delegation - computers

```powershell
Get-DomainComputer -TrustedToAuth
Get-ADComputer -Filter * -Properties msds-allowedtodelegateto | Where-Object {$_.msds-allowedtodelegateto}
```

Repère les machines avec delegation configurée.

### Abuse path reminder

```text
PDF topics to chain after enumeration:
- Rubeus s4u
- monitor / target DC$
- RBCD if machine control is available
```

Cette fiche sert surtout à préparer la phase suivante, pas à la mélanger avec tout le reste.

## PDF notes

- Toujours séparer `Unconstrained`, `Constrained user`, `Constrained computer`, `RBCD`.
- `Get-DomainComputer -Unconstrained` et `Get-DomainUser -TrustedToAuth` sont les deux commandes à retenir en premier.

## Related

- [[Cheatsheet - RBCD]]
- [[Cheatsheet - Trusts et forêts]]
- [[Synthèse - Kerberos]]


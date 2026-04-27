# Cheatsheet - Delegation attacks

> [!summary] Objectif
> Résumer les vérifications de base autour des délégations Kerberos en lab, sans approfondir au-delà du cadre pédagogique CRTP.

> [!warning] Cadre d'utilisation
> À utiliser uniquement dans un environnement de lab autorisé, dans le cadre de l'apprentissage et de la préparation CRTP.

## Quand utiliser cette fiche

- Quand l'énumération du domaine fait apparaître des comptes ou machines avec délégation.
- Pour lier les attributs AD aux scénarios de délégation vus en cours.

## Prérequis

- Un compte du lab avec visibilité LDAP suffisante.
- PowerView ou le module Active Directory.

## Commandes

### Objectif

Lister les machines avec délégation contrainte.

### Commande

```powershell
Get-DomainComputer -TrustedToAuth
Get-DomainComputer -Unconstrained | Select-Object -ExpandProperty Name
Get-ADComputer -Filter * -Properties msds-allowedtodelegateto | Where-Object {$_.msds-allowedtodelegateto}
```

### Explication

Ces commandes identifient les objets machines impliqués dans des scénarios de délégation contrainte ou non contrainte.

### Contexte d’utilisation

À utiliser pour comprendre où des chemins d'attaque liés à la délégation peuvent exister dans le lab.

### Précautions

Rester sur l'analyse des prérequis et documenter les attributs observés.

### Objectif

Lister les comptes utilisateur liés à la délégation.

### Commande

```powershell
Get-DomainUser -TrustedToAuth
Get-DomainUser -LDAPFilter "(msDS-AllowedToDelegateTo=*)"
```

### Explication

Permet de voir quels comptes utilisateur portent des attributs liés à la délégation.

### Contexte d’utilisation

Pratique pour recouper les données LDAP avec BloodHound ou d'autres outils d'analyse du lab.

### Précautions

Ne pas extrapoler sans vérifier le type exact de délégation en jeu.

### Notes associées

- [[Enumeration]]
- [[CheetSheet]]
- [[Cheatsheet - LDAP SMB Kerberos]]
- [[Cheatsheet - BloodHound SharpHound]]
- [[Synthèse - Kerberos]]

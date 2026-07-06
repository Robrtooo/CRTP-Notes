# Cheatsheet d'énumération AD

> [!summary] Résumé
> Note historique d'énumération centrée sur PowerView et le module Active Directory. Le fichier reste nommé `CheetSheet` pour préserver l'existant. Cours : [[Énumération AD]].

## Outils couverts

## PowerView

## Active Directory Module

## Commandes de base

### Objectif

Identifier le domaine courant et un autre domaine du lab.

### Commande

```powershell
Get-Domain
Get-ADDomain

Get-Domain -Domain moneycorp.local
Get-ADDomain -Identity moneycorp.local
```

### Explication

Compare les sorties de PowerView et du module Active Directory pour la même information de domaine.

### Contexte d’utilisation

Utile au début d'une session d'énumération pour confirmer le périmètre AD étudié.

### Précautions

Remplacer `moneycorp.local` par le domaine réellement concerné dans le lab.

### Objectif

Récupérer le SID du domaine et identifier les contrôleurs de domaine.

### Commande

```powershell
Get-DomainSID
(Get-ADDomain).DomainSID

Get-DomainController
Get-ADDomainController

Get-DomainController -Domain moneycorp.local
Get-ADDomainController -DomainName moneycorp.local -Discover
```

### Explication

Ces informations servent de base à la cartographie AD et aux vérifications d'accès.

### Contexte d’utilisation

À utiliser après l'identification du domaine pour savoir où adresser les requêtes LDAP, SMB et Kerberos.

### Précautions

Noter distinctement les DC du domaine courant et ceux des domaines voisins du lab.

### Objectif

Consulter la politique de domaine.

### Commande

```powershell
Get-DomainPolicyData
```

### Explication

Expose des paramètres de politique utiles pour la compréhension du comportement du domaine.

### Contexte d’utilisation

Utile pour préparer la suite de l'énumération et relier les paramètres globaux aux comportements observés.

### Précautions

Documenter les valeurs pertinentes dans les notes de lab plutôt que de garder une sortie brute uniquement.

### Objectif

Lister les utilisateurs et leurs propriétés importantes.

### Commande

```powershell
Get-DomainUser
Get-DomainUser -Identity student1

Get-ADUser -Filter * -Properties *
Get-ADUser -Identity student1 -Properties *

Get-DomainUser -Identity student1 -Properties *
Get-DomainUser -Properties samaccountname, logonCount

Get-ADUser -Filter * -Properties * | Select-Object -First 1 | Get-Member -MemberType *Property | Select-Object Name
Get-ADUser -Filter * -Properties * | Select-Object name,logoncount,@{expression={[datetime]::fromFileTime($_.pwdlastset)}}
```

### Explication

Ces commandes permettent d'énumérer les utilisateurs, leurs attributs et certains indicateurs utiles pour l'analyse en lab.

### Contexte d’utilisation

Pertinent quand il faut repérer des comptes intéressants, de service ou peu utilisés.

### Précautions

Vérifier les sorties attribut par attribut avant d'en tirer une conclusion.

> [!warning] Point d'analyse sur `logonCount`
> En lab, un `logonCount` très faible peut indiquer un compte rarement utilisé ou un leurre pédagogique. Un `logonCount` élevé peut signaler un compte de service ou d'automatisation. Ce n'est qu'un indicateur, pas une preuve.

### Objectif

Rechercher une chaîne spécifique dans les attributs utilisateurs.

### Commande

```powershell
Get-DomainUser -LDAPFilter "Description=*built*" | Select-Object name,Description
Get-ADUser -Filter 'Description -like "*built*"' -Properties Description | Select-Object name,Description
```

### Explication

Les descriptions et attributs proches contiennent parfois des indices utiles dans les exercices de lab.

### Contexte d’utilisation

À utiliser lorsqu'une étape demande d'identifier un indice textuel dans les métadonnées du domaine.

### Précautions

Traiter ces champs comme des indices à confirmer, pas comme des faits définitifs.

### Notes associées

- [[Cheatsheet - Reconnaissance AD]]
- [[Cheatsheet - LDAP SMB Kerberos]]
- [[CheetSheet]]
- [[PowerShell - MOC]]
- [[Active Directory - MOC]]

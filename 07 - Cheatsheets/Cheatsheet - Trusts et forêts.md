# Cheatsheet - Trusts et forêts

> [!summary] Objectif
> Préparer les vérifications de base sur les relations de confiance, domaines et forêts avant d'aborder les attaques cross-trust.

> [!warning] Cadre d'utilisation
> À utiliser uniquement dans un environnement de lab autorisé, dans le cadre de l'apprentissage et de la préparation CRTP.

## Quand utiliser cette fiche

- Quand le lab bascule vers plusieurs domaines ou plusieurs forêts.
- Quand il faut distinguer une relation `WITHIN_FOREST` d'une relation `FILTER_SIDS`.

## Prérequis

- PowerView ou module Active Directory.
- Les noms de domaine / forêt concernés.

## Commandes

### Objectif

Lister les trusts du domaine courant.

### Commande

```powershell
. C:\AD\Tools\PowerView.ps1
Get-DomainTrust
Get-ADTrust -Filter *
```

### Explication

Compare les sorties PowerView et du module AD sur les relations de confiance visibles.

### Contexte d’utilisation

Point de départ avant toute étude des chemins d'attaque inter-domaines.

### Précautions

Noter `TrustType`, `TrustAttributes` et `TrustDirection`.

### Objectif

Lister les domaines d'une forêt.

### Commande

```powershell
Get-ADForest
(Get-ADForest).Domains
Get-ForestDomain | %{Get-DomainTrust -Domain $_.Name}
```

### Explication

Permet d'identifier la surface de la forêt et les domaines à étudier.

### Contexte d’utilisation

Utile pour organiser les notes avant un module cross-trust.

### Précautions

Toujours préciser depuis quelle forêt la commande a été lancée.

### Objectif

Filtrer les trusts selon leurs attributs.

### Commande

```powershell
Get-DomainTrust | Where-Object {$_.TrustAttributes -eq "FILTER_SIDS"}
Get-ForestDomain | %{Get-DomainTrust -Domain $_.Name} | Where-Object {$_.TrustAttributes -eq "FILTER_SIDS"}
Get-ADTrust -Filter '(intraForest -ne $True) -and (ForestTransitive -ne $True)'
Get-ADTrust -Filter * -Server eurocorp.local
```

### Explication

Prépare la lecture des trusts externes ou filtrés, et reprend explicitement la bascule d'énumération vers `eurocorp.local` montrée dans le PDF.

### Contexte d’utilisation

Très utile avant d'ouvrir les notes sur trusts tickets ou trust key abuse.

### Précautions

Interpréter les attributs de trust avec le contexte complet domaine/forêt.

### Notes associées

- [[Synthèse - Trusts et forêts]]
- [[Cheatsheet - Delegation attacks]]
- [[Cheatsheet - AD CS et certificats]]
- [[Attaques AD - MOC]]

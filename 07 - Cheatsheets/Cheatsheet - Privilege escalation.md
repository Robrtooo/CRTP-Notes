# Cheatsheet - Privilege escalation

> [!summary] Objectif
> Réunir les vérifications de contexte utiles pour comprendre une élévation de privilèges dans un lab Windows ou Active Directory.

> [!warning] Cadre d'utilisation
> À utiliser uniquement dans un environnement de lab autorisé, dans le cadre de l'apprentissage et de la préparation CRTP.

## Quand utiliser cette fiche

- Après l'obtention d'un premier accès sur une machine du lab.
- Quand il faut mesurer les privilèges réels d'un compte avant toute étape suivante.

## Prérequis

- Une session sur l'hôte du lab.
- Accès PowerShell ou ligne de commande.
- Outils du lab disponibles si l'exercice le prévoit, par exemple `PowerUp.ps1` ou `winPEASx64.exe`.

## Commandes

### Objectif

Inspecter le contexte de sécurité actuel.

### Commande

```cmd
whoami /all
```

### Explication

Affiche l'utilisateur courant, les groupes, les privilèges et le niveau d'intégrité.

### Contexte d’utilisation

Commande de base à lancer dès qu'un accès est obtenu dans un exercice.

### Précautions

Conserver la sortie pour la révision et le rapprochement avec les droits attendus dans le lab.

### Objectif

Vérifier l'appartenance aux groupes locaux ou de domaine.

### Commande

```powershell
Get-LocalGroupMember -Group Administrators
Get-DomainGroupMember -Identity "Domain Admins"
```

### Explication

Permet d'évaluer rapidement si des privilèges importants sont déjà disponibles.

### Contexte d’utilisation

Utile pour éviter de supposer une élévation là où un droit est déjà présent.

### Précautions

Comparer les groupes locaux et de domaine au contexte exact de la machine étudiée.

### Objectif

Contrôler rapidement les pistes locales prévues par le lab.

### Commande

```powershell
. C:\AD\Tools\PowerUp.ps1
Invoke-AllChecks
```

### Explication

`PowerUp` fait partie des outils du support pour accélérer la revue des mauvaises configurations locales Windows.

### Contexte d’utilisation

Utile lorsque le lab prévoit une phase de privilèges locaux avant pivot ou mouvement latéral.

### Précautions

Lire les résultats comme aide d'analyse et éviter toute modification locale non prévue par le scénario.

### Notes associées

- [[General Notes]]
- [[Enumeration]]
- [[Cheatsheet - Lateral movement]]
- [[Cheatsheet - Post-exploitation]]
- [[Checklist - Méthodologie CRTP]]

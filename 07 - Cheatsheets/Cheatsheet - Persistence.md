# Cheatsheet - Persistence

> [!summary] Objectif
> Fournir des points de contrôle utiles pour étudier la persistance dans un lab, avec un angle pédagogique et de compréhension des surfaces d'accroche.

> [!warning] Cadre d'utilisation
> À utiliser uniquement dans un environnement de lab autorisé, dans le cadre de l'apprentissage et de la préparation CRTP.

## Quand utiliser cette fiche

- Pendant une phase de révision des mécanismes de persistance vus en formation.
- Pour analyser des artefacts existants dans un lab prévu à cet effet.

## Prérequis

- Une session autorisée sur l'hôte du lab.
- Accès PowerShell ou CMD.

## Commandes

### Objectif

Lister les tâches planifiées existantes.

### Commande

```powershell
Get-ScheduledTask | Select-Object TaskName, TaskPath, State
```

### Explication

Aide à repérer les tâches importantes ou inattendues à analyser dans le lab.

### Contexte d’utilisation

Utile dans les exercices où il faut comprendre comment une persistance pourrait être implantée ou détectée.

### Précautions

Rester sur l'observation et ne pas modifier les tâches sans consigne explicite du lab.

### Objectif

Inspecter les clés `Run` classiques.

### Commande

```powershell
Get-ItemProperty "HKLM:\Software\Microsoft\Windows\CurrentVersion\Run"
Get-ItemProperty "HKCU:\Software\Microsoft\Windows\CurrentVersion\Run"
```

### Explication

Ces emplacements servent souvent d'exemple pédagogique pour la persistance Windows.

### Contexte d’utilisation

À utiliser pour comparer théorie et artefacts concrets dans une VM de lab.

### Précautions

Ne pas écrire dans le registre hors scénario explicitement prévu.

### Notes associées

- [[Cheatsheet - Post-exploitation]]
- [[Cheatsheet - Privilege escalation]]
- [[What is PowerShell]]
- [[PowerShell - MOC]]
- [[Attaques AD - MOC]]


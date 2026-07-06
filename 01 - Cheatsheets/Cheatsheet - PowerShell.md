# Cheatsheet - PowerShell

> [!summary] Objectif
> Regrouper les commandes PowerShell de base utiles pour travailler les labs CRTP et naviguer dans les modules AD sans complexifier inutilement les notes.

> [!warning] Cadre d'utilisation
> À utiliser uniquement dans un environnement de lab autorisé, dans le cadre de l'apprentissage et de la préparation CRTP.

## Quand utiliser cette fiche

- Quand une note mentionne un module ou une cmdlet sans rappel rapide.
- En révision pour retrouver les commandes PowerShell essentielles.

## Prérequis

- Une session PowerShell sur un hôte Windows du lab.
- Les modules requis, par exemple ActiveDirectory si nécessaire.

## Commandes

### Objectif

Identifier les cmdlets AD disponibles.

### Commande

```powershell
. C:\AD\Tools\PowerView.ps1
Import-Module ActiveDirectory
Get-Command -Module ActiveDirectory
Get-Domain
```

### Explication

Charge les outils les plus utilisés dans les supports CRTP et valide la disponibilité des cmdlets AD.

### Contexte d’utilisation

Très utile sur un poste d'administration ou une VM de lab Windows.

### Précautions

Vérifier la présence du module avant d'écrire des commandes plus spécifiques.

### Objectif

Contrôler la politique d'exécution et le contexte de la session.

### Commande

```powershell
Get-ExecutionPolicy -List
$PSVersionTable
```

### Explication

Donne le contexte d'exécution PowerShell et la version disponible sur l'hôte.

### Contexte d’utilisation

À utiliser pour comprendre les différences entre plusieurs machines de lab.

### Précautions

Documenter les différences de version avant de comparer le comportement de scripts.

### Notes associées

- [[What is PowerShell]]
- [[2. Good practices]]
- [[PowerShell]]
- [[PowerShell - MOC]]
- [[General Notes]]

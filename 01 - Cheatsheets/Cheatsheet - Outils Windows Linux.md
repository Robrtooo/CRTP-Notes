# Cheatsheet - Outils Windows Linux

> [!summary] Objectif
> Rassembler quelques outils de travail courants côté Windows et Linux pour les labs CRTP, avec des exemples simples et révisables.

> [!warning] Cadre d'utilisation
> À utiliser uniquement dans un environnement de lab autorisé, dans le cadre de l'apprentissage et de la préparation CRTP.

## Quand utiliser cette fiche

- Pour savoir rapidement quel outil choisir selon l'hôte de travail.
- Pour préparer un lab mixte Windows/Linux.

## Prérequis

- Un environnement de lab autorisé.
- Les binaires ou modules nécessaires déjà disponibles.

## Commandes

### Objectif

Scanner rapidement un hôte depuis Linux.

### Commande

```bash
nmap -sV -Pn <TARGET>
```

### Explication

Récupère un inventaire simple des services visibles sur une cible du lab.

### Contexte d’utilisation

Bon point d'entrée quand la machine d'attaque du lab est Linux.

### Précautions

Limiter la portée du scan à la cible de l'exercice.

### Objectif

Lister des informations AD depuis Windows.

### Commande

```powershell
Get-ADUser -Filter * -Properties *
Get-ADComputer -Filter *
```

### Explication

Exemple de commandes utiles quand le travail se fait depuis un poste Windows joint au domaine.

### Contexte d’utilisation

Permet de comparer rapidement les approches Linux et Windows pendant les révisions.

### Précautions

Réduire les propriétés demandées si une sortie plus ciblée est suffisante.

### Notes associées

- [[What is PowerShell]]
- [[PowerShell]]
- [[Enumeration]]
- [[CheetSheet]]
- [[Cheatsheets - MOC]]


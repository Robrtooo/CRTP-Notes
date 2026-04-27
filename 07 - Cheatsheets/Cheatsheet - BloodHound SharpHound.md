# Cheatsheet - BloodHound SharpHound

> [!summary] Objectif
> Résumer la collecte minimale utile pour visualiser les relations AD dans BloodHound pendant les labs CRTP.

> [!warning] Cadre d'utilisation
> À utiliser uniquement dans un environnement de lab autorisé, dans le cadre de l'apprentissage et de la préparation CRTP.

## Quand utiliser cette fiche

- Après l'énumération de base du domaine.
- Quand il faut visualiser les relations entre utilisateurs, groupes, machines et droits.

## Prérequis

- Un poste Windows ou Linux de lab.
- Les collecteurs BloodHound ou SharpHound dans un répertoire autorisé.
- Des identifiants du lab si la collecte authentifiée est requise.

## Commandes

## SharpHound

### Objectif

Lancer une collecte SharpHound sur un hôte Windows.

### Commande

```powershell
C:\AD\Tools\Loader.exe -Path C:\AD\Tools\Sharphound\SharpHound.exe -args --collectionmethods All
```

### Explication

Utilise les chemins fournis dans le lab pour lancer une collecte compatible BloodHound CE via `Loader.exe`.

### Contexte d’utilisation

À utiliser dans un lab quand on veut comprendre rapidement les chemins d'attaque possibles.

### Précautions

Documenter l'hôte et le contexte de collecte pour savoir d'où proviennent les données.

### Objectif

Lancer une collecte PowerShell.

### Commande

```powershell
cd C:\AD\Tools\BloodHound-4.0.3_old\BloodHound-master\Collectors
Invoke-BloodHound -CollectionMethod All
```

### Explication

Alternative PowerShell cohérente avec le Lab Manual quand BloodHound Legacy 4.0.3 est utilisé.

### Contexte d’utilisation

Pertinent pour comparer une collecte exécutable et une collecte PowerShell.

### Précautions

Rester dans le périmètre du lab et noter la méthode utilisée pour la révision.

## BloodHound

### Objectif

Utiliser les interfaces déjà fournies par le lab.

### Commande

```text
BloodHound Legacy : C:\AD\Tools\BloodHound-win32-x64\BloodHound-win32-x64
BloodHound CE : instance partagée du lab en lecture seule
```

### Explication

Le lab fournit déjà une UI locale Legacy et une UI CE préremplie, ce qui permet de privilégier l'analyse plutôt que la reconfiguration.

### Contexte d’utilisation

Très utile pour vérifier rapidement un chemin vers `Domain Admins` ou relire un contrôle d'objet.

### Précautions

Faire correspondre le collecteur et la version de l'interface utilisée.

### Notes associées

- [[Enumeration]]
- [[CheetSheet]]
- [[Cheatsheet - Reconnaissance AD]]
- [[Cheatsheets - MOC]]
- [[Active Directory - MOC]]

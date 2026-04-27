# Cheatsheet - Monitoring et détections

> [!summary] Objectif
> Préparer les points de contrôle sur les mécanismes de journalisation, AppLocker, CLM et contexte PowerShell/WinRM vus dans le module de défenses.

> [!warning] Cadre d'utilisation
> À utiliser uniquement dans un environnement de lab autorisé, dans le cadre de l'apprentissage et de la préparation CRTP.

## Quand utiliser cette fiche

- Quand le parcours arrive sur `Monitoring and Detections`.
- Quand il faut comprendre le contexte d'une session PowerShell ou WinRM dans le lab.

## Prérequis

- Session PowerShell sur l'hôte concerné.
- Droits suffisants pour lire les paramètres locaux.

## Commandes

### Objectif

Vérifier la version de PowerShell et le mode de langage.

### Commande

```powershell
$PSVersionTable
$ExecutionContext.SessionState.LanguageMode
```

### Explication

Donne le contexte PowerShell de la machine et permet d'identifier un éventuel `ConstrainedLanguage`.

### Contexte d’utilisation

Utile avant tout travail sur modules, chargement de scripts ou remoting.

### Précautions

Comparer les résultats entre la machine locale et la cible distante si besoin.

### Objectif

Chercher des hôtes où l'accès PSRemoting admin local est déjà possible.

### Commande

```powershell
. C:\AD\Tools\Find-PSRemotingLocalAdminAccess.ps1
Find-PSRemotingLocalAdminAccess
```

### Explication

Le PDF utilise cette séquence avant d'étudier un hôte comme `dcorp-adminsrv`.

### Contexte d’utilisation

Très utile quand le module passe des accès locaux aux contraintes de remoting.

### Précautions

Garder le nom exact de l'hôte trouvé pour la suite de l'exercice.

### Objectif

Contrôler la politique AppLocker effective.

### Commande

```powershell
reg query HKLM\Software\Policies\Microsoft\Windows\SRPV2
Get-AppLockerPolicy -Effective | Select-Object -ExpandProperty RuleCollections
```

### Explication

Le PDF passe d'abord par `reg query` puis confirme avec `Get-AppLockerPolicy` pour comprendre une allowlist AppLocker.

### Contexte d’utilisation

Très utile quand le lab traite d'Applocker ou d'un comportement inattendu en exécution.

### Précautions

Lire la politique avant toute tentative de contour de lab pour éviter les faux diagnostics.

### Objectif

Vérifier le contexte de remoting et de logs sur une session distante.

### Commande

```powershell
Enter-PSSession dcorp-adminsrv
$ExecutionContext.SessionState.LanguageMode
```

### Explication

Permet de voir si la session distante se comporte différemment de la session locale.

### Contexte d’utilisation

Utile dans les parties du cours où WinRM, CLM et remoting sont liés aux défenses, avec un exemple concret sur `dcorp-adminsrv`.

### Précautions

Documenter le contexte de la session distante dans les notes, surtout si le comportement change.

### Notes associées

- [[Cheatsheet - PowerShell]]
- [[Cheatsheet - GPO et OU]]
- [[Synthèse - Monitoring et détections]]
- [[PowerShell - MOC]]

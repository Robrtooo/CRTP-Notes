# Cheatsheet - Lateral movement

> [!summary] Objectif
> Rassembler les commandes de déplacement latéral étudiées en lab pour comprendre les prérequis d'accès distant et de contexte utilisateur.

> [!warning] Cadre d'utilisation
> À utiliser uniquement dans un environnement de lab autorisé, dans le cadre de l'apprentissage et de la préparation CRTP.

## Quand utiliser cette fiche

- Quand un compte du lab donne accès à une autre machine.
- Pour comparer plusieurs mécanismes d'administration distante vus pendant les labs.

## Prérequis

- Identifiants valides sur la machine cible du lab.
- Connectivité réseau vers `<TARGET>`.
- Services distants autorisés et actifs dans le lab.

## Commandes

### Objectif

Ouvrir une session PowerShell distante.

### Commande

```powershell
Enter-PSSession -ComputerName <TARGET> -Credential <DOMAIN>\<USER>
```

### Explication

Utilise PowerShell Remoting pour interagir avec l'hôte cible dans un scénario de lab.

### Contexte d’utilisation

Utile lorsque WinRM est activé et fait partie du chemin prévu dans l'exercice.

### Précautions

Valider que l'accès distant est explicitement prévu dans le lab avant usage.

### Objectif

Exécuter une commande distante via WinRS.

### Commande

```cmd
winrs -r:<TARGET> cmd /c hostname
```

### Explication

Permet de vérifier un accès distant basique et de confirmer le contexte d'exécution sur la machine cible avec une commande explicitement utilisée dans le lab.

### Contexte d’utilisation

Pratique pour valider rapidement un accès dans un exercice guidé.

### Précautions

Si les identifiants sont déjà présents dans la session du lab, éviter d'ajouter des secrets en ligne de commande.

### Notes associées

- [[Cheatsheet - Privilege escalation]]
- [[Cheatsheet - Post-exploitation]]
- [[What is PowerShell]]
- [[PowerShell - MOC]]
- [[Attaques AD - MOC]]

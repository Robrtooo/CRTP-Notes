# Cheatsheet - AS-REP Roasting

> [!summary] Objectif
> Réviser l'identification des comptes sans préauthentification Kerberos et la récupération contrôlée d'AS-REP dans un lab.

> [!warning] Cadre d'utilisation
> À utiliser uniquement dans un environnement de lab autorisé, dans le cadre de l'apprentissage et de la préparation CRTP.

## Quand utiliser cette fiche

- Lorsqu'un compte du lab semble configuré sans préauthentification Kerberos.
- Pour réviser la différence entre Kerberoasting et AS-REP Roasting.

## Prérequis

- Connectivité au domaine.
- Outils AD ou Kerberos disponibles en environnement autorisé.

## Commandes

### Objectif

Identifier les comptes sans préauthentification requise.

### Commande

```powershell
Get-DomainUser -PreauthNotRequired
Get-ADUser -Filter * -Properties DoesNotRequirePreAuth | Where-Object {$_.DoesNotRequirePreAuth}
```

### Explication

Ces requêtes ciblent les comptes dont la configuration Kerberos permet la récupération d'un AS-REP dans le lab.

### Contexte d’utilisation

À utiliser après une énumération des utilisateurs et des attributs de sécurité.

### Précautions

Valider le contexte du compte avant de lancer la suite des commandes.

### Objectif

Récupérer un AS-REP pour étude hors ligne.

### Commande

```powershell
C:\AD\Tools\Loader.exe -Path C:\AD\Tools\Rubeus.exe -args asreproast /user:<USER> /outfile:C:\AD\Tools\asrephashes.txt
```

### Explication

La commande suit le mode de travail du lab pour produire une sortie réutilisable pendant l'exercice et la révision.

### Contexte d’utilisation

Utile dans les labs où la faiblesse visée concerne directement la configuration Kerberos d'un compte.

### Précautions

Conserver une trace claire du compte étudié et du prérequis identifié.

### Notes associées

- [[Enumeration]]
- [[CheetSheet]]
- [[Synthèse - Kerberos]]
- [[Cheatsheet - Kerberoasting]]
- [[Attaques AD - MOC]]

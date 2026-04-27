# Cheatsheet - MSSQL et PowerUpSQL

> [!summary] Objectif
> Préparer les commandes de découverte SQL Server et de crawl des database links avant d'attaquer ce module du lab.

> [!warning] Cadre d'utilisation
> À utiliser uniquement dans un environnement de lab autorisé, dans le cadre de l'apprentissage et de la préparation CRTP.

## Quand utiliser cette fiche

- Quand le lab bascule vers `PowerUpSQL`, `database links` ou `xp_cmdshell`.
- Quand un compte de domaine possède des droits sur un ou plusieurs serveurs SQL.

## Prérequis

- Module `PowerUpSQL` présent sur la VM du lab.
- Connectivité vers les serveurs SQL du scénario.

## Commandes

### Objectif

Charger le module et découvrir les instances SQL du domaine.

### Commande

```powershell
Import-Module C:\AD\Tools\PowerUpSQLmaster\PowerupSQL.psd1
Get-SQLInstanceDomain | Get-SQLServerinfo -Verbose
```

### Explication

Retourne les instances SQL visibles et plusieurs informations utiles sur leur configuration.

### Contexte d’utilisation

Point d'entrée du module SQL quand aucune cartographie n'a encore été faite.

### Précautions

Documenter les instances, versions et types d'authentification observés.

### Objectif

Lister les database links à la main.

### Commande

```sql
select * from master..sysservers
select * from openquery("DCORP-SQL1",'select * from master..sysservers')
select * from openquery("DCORP-SQL1",'select * from openquery("DCORPMGMT",''select * from master..sysservers'')')
```

### Explication

Ces requêtes viennent directement du PDF et montrent le chaînage `DCORP-MSSQL -> DCORP-SQL1 -> DCORP-MGMT`.

### Contexte d’utilisation

Très utile pour comprendre ce que `Get-SQLServerLinkCrawl` automatise ensuite.

### Précautions

Conserver le chemin SQL exact remonté par chaque requête.

### Objectif

Explorer automatiquement les database links.

### Commande

```powershell
Get-SQLServerLinkCrawl -Instance dcorpmssql.dollarcorp.moneycorp.local -Verbose
```

### Explication

Parcourt les liens entre serveurs SQL pour montrer les chemins d'accès disponibles.

### Contexte d’utilisation

Très utile pour visualiser un chaînage multi-serveurs avant tout test plus spécifique.

### Précautions

Noter clairement le chemin de liens remonté par la commande.

### Objectif

Exécuter une requête simple sur le chaînage SQL.

### Commande

```powershell
Get-SQLServerLinkCrawl -Instance dcorp-mssql -Query 'SELECT @@version'
```

### Explication

Permet de vérifier le contexte d'exécution et la portée réelle de la chaîne SQL.

### Contexte d’utilisation

À utiliser avant les étapes plus poussées du module SQL.

### Précautions

Commencer par une requête de lecture simple avant toute autre action du lab.

### Notes associées

- [[Synthèse - MSSQL]]
- [[Cheatsheet - Trusts et forêts]]
- [[Cheatsheet - Outils Windows Linux]]
- [[Labs - MOC]]

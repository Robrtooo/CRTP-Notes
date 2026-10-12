---
flag: 37
machine: dcorp-mssql
statut: à faire
technique: MSSQL link crawl
LO: "[[LO22 - MSSQL links]]"
---

# F37 · Premier SQL Server lié à dcorp-mssql

> [!summary] Objectif
> Identifier le premier SQL Server *lié* à `dcorp-mssql` (linked server directement accessible depuis l'instance de départ).

## Contexte
- **Etape attack flow**: enumeration transverse via MSSQL — pivot vers un autre serveur SQL sans admin de domaine.
- **Ce qu on a**: un compte de domaine standard (student utilisateur RDP) accepté par l'authentification Windows sur `dcorp-mssql`.
- **Ce qu on cherche**: le nom du premier `srvname` renvoyé par `master..sysservers` sur `dcorp-mssql` (hors `dcorp-mssql` lui-meme).

## Solution

### Etape 1: Découvrir les instances SQL du domaine
```powershell
Import-Module C:\AD\Tools\PowerUpSQLmaster\PowerupSQL.psd1
Get-SQLInstanceDomain | Get-SQLServerinfo -Verbose
```
→ Voir [[Cheatsheet - MSSQL et PowerUpSQL]] pour la variante de découverte.

### Etape 2: Lire les links depuis dcorp-mssql
```powershell
Get-SQLServerLinkCrawl -Instance dcorp-mssql -Verbose
```
Ou directement en T-SQL depuis `dcorp-mssql`:
```sql
select srvname, isremote from master..sysservers
```
Le premier `srvname` distinct de `dcorp-mssql` est la reponse.

## Reponse

| Element | Valeur |
|---------|--------|
| Premier SQL Server lié | <A REMPLIR DEPUIS LAB> |

## Pieges et OPSEC
- La première ligne de `sysservers` est presque toujours l'instance locale (`isremote=0`) — ignorer et prendre le premier `isremote=1`.
- `Get-SQLServerLinkCrawl` cascade automatiquement — bien lire la sortie hiérarchique pour distinguer le lien *direct* d'un lien *transitif*.
- Event ID 33205 (SQL audit) si audit configuré ; xEvents `sql_statement_completed` visible côté DBA.

## Refs
- [[Cheatsheet - MSSQL et PowerUpSQL]]
- [[Synthèse - MSSQL]]
- [[LO22 - MSSQL links]]
- [[Flags - Index]] · [[Journal - Index]]

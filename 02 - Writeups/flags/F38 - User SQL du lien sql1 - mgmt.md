---
flag: 38
machine: dcorp-mssql
statut: à faire
technique: MSSQL link crawl
LO: [[LO22 - MSSQL links]]
---

# F38 · User SQL du lien sql1 / mgmt

> [!summary] Objectif
> Identifier le login SQL Server utilisé pour établir le lien entre `dcorp-sql1` et `dcorp-mgmt` (colonne `name` de `sysservers` côté `dcorp-sql1`).

## Contexte
- **Etape attack flow**: crawl des database links pour comprendre *qui* s'authentifie sur chaque saut — un lien peut utiliser un compte SQL local (`sa`, service) ou passer l'auth Windows.
- **Ce qu on a**: la chaine `dcorp-mssql → dcorp-sql1 → dcorp-mgmt` déjà identifiée (F37).
- **Ce qu on cherche**: le nom du user SQL configuré pour le lien sortant `dcorp-sql1 → dcorp-mgmt`.

## Solution

### Etape 1: Crawler la chaine
```powershell
Import-Module C:\AD\Tools\PowerUpSQLmaster\PowerupSQL.psd1
Get-SQLServerLinkCrawl -Instance dcorp-mssql -Verbose
```
→ Voir [[Cheatsheet - MSSQL et PowerUpSQL]] pour la version manuelle avec `openquery`.

### Etape 2: Lire le login utilisé côté dcorp-sql1
```powershell
Get-SQLServerLinkCrawl -Instance dcorp-mssql -Query "select system_user"
```
Ou en T-SQL imbriqué manuel — la valeur remontée depuis `dcorp-sql1` révèle sous quelle identité on frappe `dcorp-mgmt`:
```sql
select * from openquery("DCORP-SQL1",'select system_user')
```

## Reponse

| Element | Valeur |
|---------|--------|
| Login SQL du lien sql1 → mgmt | <A REMPLIR DEPUIS LAB> |

## Pieges et OPSEC
- `system_user` renvoie l'identité *courante* d'exécution — si le lien est configuré en "current security context", ça affiche le compte AD qui a initié la chaine ; s'il utilise "remote login", ça affiche le compte SQL statique (souvent `sa`).
- Ne pas confondre `system_user` (identité de session) et `original_login()` (identité initiale).
- xp_cmdshell non requis ici — pure lecture, moins bruyant.

## Refs
- [[Cheatsheet - MSSQL et PowerUpSQL]]
- [[Synthèse - MSSQL]]
- [[LO22 - MSSQL links]]
- [[Flags - Index]] · [[Journal - Index]]

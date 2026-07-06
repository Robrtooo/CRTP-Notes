# Cheatsheet - Monitoring et détections

> [!summary] Objectif
> Lire le contexte défensif d'une session avant d'agir : langage PowerShell (CLM), politique AppLocker effective, et si le comportement change en remoting.

### Check PowerShell context
```powershell
$PSVersionTable
$ExecutionContext.SessionState.LanguageMode
```
Un `ConstrainedLanguage` limite fortement ce qui est exécutable (pas de `Add-Type`, réflexion bridée) — à vérifier avant tout chargement de module.

### Find hosts with local admin PSRemoting access
```powershell
. C:\AD\Tools\Find-PSRemotingLocalAdminAccess.ps1
Find-PSRemotingLocalAdminAccess
```

### Read effective AppLocker policy
```powershell
reg query HKLM\Software\Policies\Microsoft\Windows\SRPV2
Get-AppLockerPolicy -Effective | Select-Object -ExpandProperty RuleCollections
```
Passer d'abord par `reg query` avant `Get-AppLockerPolicy` pour confirmer l'allowlist réellement appliquée.

### Compare local vs remote session
```powershell
Enter-PSSession dcorp-adminsrv
$ExecutionContext.SessionState.LanguageMode
```
Le `LanguageMode` peut différer entre la session locale et une session distante sur le même compte — à documenter, ça conditionne les outils utilisables une fois latéralisé.

**Link** — [[Cheatsheet - PowerShell]] · [[Cheatsheet - GPO et OU]] · [[Synthèse - Monitoring et détections]]

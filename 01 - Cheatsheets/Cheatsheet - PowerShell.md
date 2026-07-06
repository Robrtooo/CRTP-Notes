# Cheatsheet - PowerShell

> [!summary] Objectif
> Recharger vite les cmdlets AD et vérifier le contexte d'exécution d'un hôte de lab.

### Load AD tooling

```powershell
. C:\AD\Tools\PowerView.ps1
Import-Module ActiveDirectory
Get-Command -Module ActiveDirectory
```

### Check execution context

```powershell
Get-ExecutionPolicy -List
$PSVersionTable
```

Utile pour comparer le comportement d'un script entre deux machines de lab (version, policy).

**Link**
- [[What is PowerShell]]
- [[2. Good practices]]
- [[PowerShell - MOC]]

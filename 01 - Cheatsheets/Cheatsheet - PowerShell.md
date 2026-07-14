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

### Depuis Linux (attaquant hors domaine)

```bash
nmap -sV -Pn <TARGET>
```

### Depuis Windows (joint au domaine)

```powershell
Get-ADUser -Filter * -Properties *
Get-ADComputer -Filter *
```

Réduire les propriétés demandées (`-Properties <attribut>`) dès qu'une sortie ciblée suffit plutôt que `*`.

## Voir aussi
- [[What is PowerShell]]
- [[2. Good practices]]
- [[Cheatsheet - Reconnaissance AD]]
- [[Tools list]]

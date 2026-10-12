# 06 — Méthode : utiliser un script .ps1 (PowerView & co)

Recette générique pour tout script `.ps1` à fonctions (PowerView, PowerUp, Find-PSRemotingLocalAdminAccess, Invoke-Mimi…).

## Les 4 temps
```powershell
# 0. (1x par session) autoriser les scripts
Set-ExecutionPolicy Bypass -Scope Process -Force

# 1. DOT-SOURCE (point + espace + chemin)
. C:\AD\Tools\PowerView.ps1

# 2. VÉRIFIER + DÉCOUVRIR
Get-Command Get-Domain*            # fonctions apparues
Get-Command <Fonction> -Syntax     # signature

# 3. LIRE l'usage
Get-Help <Fonction> -Full
Get-Help <Fonction> -Examples

# 4. EXÉCUTER
<Fonction> -Param ...
```

## Le point crucial : dot-source
> [!IMPORTANT] `. .\x.ps1` vs `.\x.ps1`
> - `. .\x.ps1` (**point + espace**) → charge les fonctions dans **ta** session. ✅
> - `.\x.ps1` (sans le point devant) → exécute dans un **scope enfant** qui meurt aussitôt → la fonction n'existe pas → `CommandNotFound`. ❌
> - `& .\x.ps1` → pareil, ne persiste pas.

## Lister ce qu'un script expose
```powershell
# depuis le fichier
Select-String -Path .\Script.ps1 -Pattern '^\s*function\s+([\w-]+)'

# ou diff avant/après dot-source
$b = Get-Command -CommandType Function
. .\Script.ps1
Get-Command -CommandType Function | ? { $_ -notin $b }
```

## Dépannage
| Symptôme | Cause | Fix |
|---|---|---|
| `not recognized` après l'appel | pas dot-sourcé (`.\` seul) | `. .\x.ps1` (point espace) |
| `cannot be loaded / disabled` | execution policy | `Set-ExecutionPolicy Bypass -Scope Process -Force` (cf [[05 - Lancer PowerShell en bypass]]) |
| `malicious content` au chargement | **AMSI** | bypass AMSI puis re-dot-source (InviShell / Amsi-Byp.txt) |
| `.exe not recognized` | cwd pas dans le PATH | préfixe `.\` : `.\Tool.exe` |

## Cas .exe (SharpHound, Rubeus, mimikatz…)
Pas de dot-source : c'est un binaire. Juste le préfixe `.\` car PowerShell ne cherche pas dans le dossier courant :
```powershell
.\Rubeus.exe kerberoast
```

Voir aussi [[01 - Charger PowerView]] · [[05 - Lancer PowerShell en bypass]] · [[00 - Index]]

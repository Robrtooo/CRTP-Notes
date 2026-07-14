---
flag: 5
machine: Student VM
statut: validé
technique: Service Windows faible (PowerUp)
LO: [[LO05 - Privesc locale et Jenkins]]
---

# F05 - Service abusé pour privesc locale

> [!summary] Objectif
> Identifier le service Windows tiers de la student VM abusable pour élever en admin local / SYSTEM.

## Contexte
- **Etape attack flow**: privesc locale hôte (couche invisible dans BloodHound).
- **Ce qu on a**: shell PowerShell Medium Integrity en tant que `studentX` (utilisateur non-admin) sur la student VM.
- **Ce qu on cherche**: le **nom du service** non-Microsoft qui autorise l abus (répond au flag) et sert de vecteur vers SYSTEM.

## Solution

### Etape 1 - Repérer la surface locale
```powershell
Get-CimInstance win32_service | ?{$_.PathName -notmatch 'C:\\Windows'} | select Name,StartName,State,PathName
```
Filtre les services hors `C:\Windows` → seuls les services tiers ressortent.
→ Voir [[Cheatsheet - Privilege escalation]] §"Découvrir la surface d attaque LOCALE".

### Etape 2 - Automated checks avec PowerUp
```powershell
. C:\AD\Tools\PowerUp.ps1 ; Invoke-AllChecks
```
PowerUp nomme la faiblesse (`Modifiable Service`, `Unquoted Service Path`, `CanRestart: True`) et le service ciblé. Le **Name** du service = réponse du flag.
→ Voir [[Cheatsheet - Privilege escalation]] §"Automated local checks".

### Etape 3 (bonus) - Abus vers SYSTEM
```powershell
Invoke-ServiceAbuse -Name <svc> -Command "cmd /c echo RAN > C:\Users\Public\p.txt"
```
Le mode `-Command` reconfigure `binPath` (aucun fichier écrasé, rien à restaurer). `1053 / timeout 30s` sur `sc start` = normal, la charge a tourné.

## Reponse

| Element | Valeur |
|---------|--------|
| Service abusé | AbyssWebServer |

## Pieges et OPSEC
- Ne pas confondre `-BinaryPath` (écrase l .exe → nécessite `Restore-ServiceBinary`) et `-Command` (change juste binPath, rien à restaurer).
- Après `-Command`, `Restart-Service -Force` échoue (SCM ne dialogue pas avec un `cmd /c`) → utiliser `sc.exe stop <svc>` puis rejeu.
- Charge `.exe` on-disk = signature Defender ; script PS = AMSI. Un simple `cmd /c echo RAN` prouve l exec SYSTEM sans AV.
- **Events**: 7040 (config service modifiée), 7045 (service installé), 4697, 1053 (timeout SCM).

## Refs
- [[Cheatsheet - Privilege escalation]]
- [[Cheatsheet - Post-exploitation]]
- [[LO05 - Privesc locale et Jenkins]]
- [[Flags - Index]] · [[Journal - Index]]

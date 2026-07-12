---
flag: 10
machine: dcorp-mgmt
statut: validé
---
# F10 · Process utilisant svcadmin

> [!question] LO7 — dcorp-mgmt
> Process using svcadmin as service account

## Commande qui marche
Repérer où svcadmin (DA) a une session/compte de service — via Remote Registry, **non-admin OK** :
```powershell
C:\AD\Tools\InviShell\RunWithRegistryNonAdmin.bat
. C:\AD\Tools\Invoke-SessionHunter.ps1
Invoke-SessionHunter -NoPortScan -RawResults | select Hostname,UserSession,Access
```
→ **svcadmin (DA) sur `dcorp-mgmt`**. Chaîne complète (dump → clé) : [[LO07 - Session DA et escalade]].

## Fiches liées
[[Cheatsheet - Post-exploitation]] · [[Cheatsheet - Lateral movement]] · [[LO07 - Session DA et escalade]]

[[Flags - Index]] · [[Journal - Index]]

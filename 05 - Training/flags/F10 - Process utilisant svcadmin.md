---
flag: 10
machine: dcorp-mgmt
statut: validé
technique: Session hunting (Remote Registry)
LO: [[LO07 - Session DA et escalade]]
---

# F10 · Process utilisant svcadmin

> [!summary] Objectif
> Identifier le processus / la session sur laquelle `svcadmin` (Domain Admin) est actif — c'est la box à compromettre pour dumper LSASS et récupérer sa clé.

## Contexte
- **Étape attack flow** : après privesc locale + accès à Jenkins (ciadmin), on cherche une **session DA** exploitable côté domaine.
- **Ce qu'on a** : shell non-admin `studentx` sur la student VM.
- **Ce qu'on cherche** : sur quel **hôte** un process tourne sous `svcadmin` → pivot cible du dump LSASS.

## Solution

### Étape 1 : session hunting sans admin (Remote Registry)
```powershell
C:\AD\Tools\InviShell\RunWithRegistryNonAdmin.bat
. C:\AD\Tools\Invoke-SessionHunter.ps1
Invoke-SessionHunter -NoPortScan -RawResults | Select-Object Hostname,UserSession,Access
```
→ Voir [[Cheatsheet - Reconnaissance AD]] et [[LO07 - Session DA et escalade]] étape 1 pour la chaîne complète.

### Étape 2 : recouper avec le groupe Domain Admins
```powershell
Get-DomainGroupMember -Identity "Domain Admins" | Select-Object MemberName
```
→ confirme que `svcadmin` fait bien partie des DAs (donc la session sur son hôte = jackpot).

### Étape 3 : (optionnel) énumérer les services WMI côté cible
```powershell
Get-WmiObject -Class Win32_Service -ComputerName <HOTE_CIBLE> `
  | Where-Object { $_.StartName -like "*svcadmin*" } `
  | Select-Object Name,StartName,PathName
```

## Réponse

| Élément | Valeur |
|---------|--------|
| Hôte où `svcadmin` a une session | `dcorp-mgmt` |
| Process / service utilisant svcadmin | <A REMPLIR DEPUIS LAB> |

## Pièges et OPSEC
- **Remote Registry** doit être atteignable sur la cible (allumé + FW OK) — `Invoke-SessionHunter` tombe silencieusement sinon. `-NoPortScan` évite du bruit.
- `dcorp-adminsrv` est une piste **séparée** (AppLocker/GPO) — ne pas confondre avec `dcorp-mgmt` où vit la session DA.
- Traces : Event ID **4624** (logon type 3) côté cible ; l'ouverture de `winreg` peut aussi être auditée.

## Refs
- [[Cheatsheet - Reconnaissance AD]]
- [[Cheatsheet - Post-exploitation]]
- [[Cheatsheet - Lateral movement]]
- [[LO07 - Session DA et escalade]]

[[Flags - Index]] · [[Journal - Index]]

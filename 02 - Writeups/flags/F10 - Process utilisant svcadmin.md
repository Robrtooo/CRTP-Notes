---
flag: 10
machine: dcorp-mgmt
statut: validé
technique: Session hunting (Remote Registry)
LO: "[[LO07 - Session DA et escalade]]"
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
| **Service / process utilisant svcadmin (CRTP)** | **`MSSQLSERVER`** ✅ |

> [!info] Interprétation
> Le service Windows `MSSQLSERVER` (moteur SQL Server) tourne sous le compte de domaine `dcorp\svcadmin` sur dcorp-mgmt → sa session logon est en LSASS = jackpot dump. Le walkthrough gitbook (dev-angelist) proposait `SQLSERVERAGENT` (SQL Server Agent) — c'est un service **différent** (agent d'ordonnancement de SQL), rejeté par CRTP sur ce pod. Le bon = **`MSSQLSERVER`** (le moteur DB lui-même).

## Commande de validation

Depuis session admin sur dcorp-mgmt (via ciadmin Jenkins ou OPtH svcadmin) :
```powershell
Invoke-Command -ComputerName dcorp-mgmt.dollarcorp.moneycorp.local -ScriptBlock {
  Get-WmiObject Win32_Service | ? { $_.StartName -match "svcadmin" } | select Name, DisplayName, StartName, PathName | fl
}
```
→ bloc `Name : MSSQLSERVER` avec `StartName : dcorp\svcadmin`.

## Pièges et OPSEC

> [!failure]- Réponses testées avant validation
> - `SQLSERVERAGENT` (walkthrough gitbook) → **rejeté** (mauvais service — c'est l'Agent, pas le moteur DB)
> - `MSSQLSERVER` → ✅ **accepté**

- **Remote Registry** doit être atteignable sur la cible (allumé + FW OK) — `Invoke-SessionHunter` tombe silencieusement sinon. `-NoPortScan` évite du bruit.
- `dcorp-adminsrv` est une piste **séparée** (AppLocker/GPO) — ne pas confondre avec `dcorp-mgmt` où vit la session DA.
- Ne pas confondre `MSSQLSERVER` (moteur DB, F10) et `SQLSERVERAGENT` (agent d'ordonnancement, service séparé qui peut tourner sous un compte différent).
- Traces : Event ID **4624** (logon type 3) côté cible ; l'ouverture de `winreg` peut aussi être auditée.

## Refs
- [[Cheatsheet - Reconnaissance AD]]
- [[Cheatsheet - Post-exploitation]]
- [[Cheatsheet - Lateral movement]]
- [[LO07 - Session DA et escalade]]

[[Flags - Index]] · [[Journal - Index]]

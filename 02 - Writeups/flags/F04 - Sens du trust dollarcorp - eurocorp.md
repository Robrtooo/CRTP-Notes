---
flag: 4
machine: Student VM
statut: validé
technique: Trust enumeration
LO: "[[LO04 - Trusts et forêt]]"
---

# F04 - Sens du trust dollarcorp / eurocorp

> [!summary] Objectif
> Identifier la **TrustDirection** du trust entre `dollarcorp.moneycorp.local` et `eurocorp.local` (Inbound / Outbound / Bidirectional).

## Contexte
- **Étape attack flow**: énumération inter-domaine — savoir si l'attaquant peut traverser le trust dans un sens ou dans l'autre (couche identité/AD, cross-forest).
- **Ce qu'on a**: `studentx` dans `dollarcorp.moneycorp.local`. `eurocorp.local` est une **autre forêt** liée par un external/forest trust.
- **Ce qu'on cherche**: la valeur du champ `TrustDirection` — utile pour choisir entre trust key attack, referral ticket, ou SID history.

## Solution

### Étape 1: énumération PowerView
```powershell
. C:\AD\Tools\PowerView.ps1
Get-DomainTrust
```
Champs clés : `SourceName`, `TargetName`, `TrustType`, **`TrustDirection`**, `TrustAttributes`.

### Étape 2: étendre à toute la forêt
```powershell
# Enumerer trusts de TOUS les domaines de la foret moneycorp
Get-ForestDomain | ForEach-Object { Get-DomainTrust -Domain $_.Name }
```
→ Voir [[Cheatsheet - Trusts et forêts]] pour variantes AD module.

### Étape 3 (pivot AD module)
```powershell
Import-Module C:\AD\Tools\ADModulemaster\ActiveDirectory\ActiveDirectory.psd1
Get-ADTrust -Filter *
```
Champ `Direction` = `BiDirectional`, `Inbound`, ou `Outbound`.

## Réponse

| Élément | Valeur |
|---------|--------|
| TrustDirection dollarcorp ↔ eurocorp | `Bidirectional` |
| TrustType | `<A REMPLIR DEPUIS LAB>` (External ou Forest) |
| TrustAttributes | `<A REMPLIR DEPUIS LAB>` (vérifier `FILTER_SIDS`) |

Bidirectional = trust utilisable dans les deux sens (les principals des deux domaines peuvent s'authentifier chez l'autre, sous réserve des SIDs filtrés).

## Pièges et OPSEC
- `TrustDirection` de PowerView est un **entier** : `1=Inbound`, `2=Outbound`, `3=Bidirectional`. Certaines versions affichent le nom, d'autres le nombre — vérifier.
- Ne pas confondre `TrustDirection` (sens du trust) et `TrustAttributes` (flags dont `FILTER_SIDS` qui bloque SID history cross-forest).
- Un trust **inter-forêt** est **non transitif** par défaut : une referral error est attendue si on interroge des domaines "en profondeur" depuis l'autre côté — normal, pas un bug.
- Trust child → parent (`dcorp` → `moneycorp`) = **intra-forêt**, sortie différente (`ParentChild`), pas ce flag.

## Refs
- [[Cheatsheet - Trusts et forêts]]
- [[Synthèse - Trusts et forêts]]
- [[LO04 - Trusts et forêt]]
- [[Fondamentaux AD]]

[[Flags - Index]] · [[Journal - Index]]

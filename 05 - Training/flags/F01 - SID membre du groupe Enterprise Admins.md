---
flag: 1
machine: Student VM
statut: validé
technique: Enumeration groupes
LO: "[[LO01 - Énumération]]"
---

# F01 - SID membre du groupe Enterprise Admins

> [!summary] Objectif
> Récupérer le SID du (des) membre(s) du groupe **Enterprise Admins** de la forêt `moneycorp.local`.

## Contexte
- **Étape attack flow**: reconnaissance initiale, phase énumération large (couche identité/AD).
- **Ce qu'on a**: compte de domaine `studentx` sur la VM student, PowerView chargé depuis `C:\AD\Tools`.
- **Ce qu'on cherche**: la valeur SID d'un membre du groupe `Enterprise Admins` — groupe qui n'existe **que dans le domaine racine** de la forêt (`moneycorp.local`), pas dans l'enfant `dollarcorp`.

## Solution

### Étape 1: charger PowerView
```powershell
. C:\AD\Tools\PowerView.ps1
Get-Domain
Get-DomainSID
```
→ Voir [[Cheatsheet - Reconnaissance AD]] pour les checks basiques.

### Étape 2: cibler le domaine racine
```powershell
# Enterprise Admins vit dans le ROOT domain de la forêt, pas dans le child
Get-DomainGroupMember -Identity "Enterprise Admins" -Domain moneycorp.local
```
Le champ `MemberSID` de la sortie donne la réponse.

### Étape 3 (alternative AD module)
```powershell
Import-Module C:\AD\Tools\ADModulemaster\ActiveDirectory\ActiveDirectory.psd1
Get-ADGroupMember -Identity "Enterprise Admins" -Server moneycorp.local
```

## Réponse

| Élément | Valeur |
|---------|--------|
| SID membre Enterprise Admins | `S-1-5-21-335606122-960912869-3279953914-500` |

Le suffixe `-500` = compte `Administrator` du domaine racine.

## Pièges et OPSEC
- Ne pas requêter `-Domain dcorp.local` ni `dollarcorp.moneycorp.local` : `Enterprise Admins` n'existe **que dans la racine** de la forêt. Erreur classique = `no group found`.
- `Get-DomainGroupMember` **résout** le SID mais expose déjà la valeur brute dans `MemberSID` — pas besoin de re-résoudre.
- Aucun droit particulier requis : tout compte authentifié lit LDAP. Trace = 4662/4661 sur DC racine.

## Refs
- [[Cheatsheet - Reconnaissance AD]]
- [[Cheatsheet - BloodHound SharpHound]]
- [[Synthèse - Énumération]]
- [[LO01 - Énumération]]
- [[Fondamentaux AD]]

[[Flags - Index]] · [[Journal - Index]]

# 04 — Debug : "mon filtre ACL renvoie toujours vide"

Checklist ordonnée. Fais chaque étape AVANT la suivante.

## A — La cible existe-t-elle VRAIMENT ?
```powershell
Get-DomainUser | ?{$_.distinguishedname -match "ControlxUser"}
```
- Vide → le nom n'est pas celui que tu crois. Ne filtre plus, **liste et regarde** :
```powershell
Get-DomainUser | select samaccountname,distinguishedname
```
> [!TIP] Placeholders du cours
> Un nom type `ControlxUser` est souvent un **placeholder** (« le user X »), pas le nom littéral. Repère le vrai `samaccountname`.

## B — Sortir UNE ACE brute complète
```powershell
Find-InterestingDomainAcl -ResolveGUIDs | ?{$_.ObjectDN -match "CN=<vrai_nom>"} | fl *
```
- Vide → `Find-InterestingDomainAcl` **filtre** les ACEs non-intéressantes. Passe à `Get-DomainObjectAcl -Identity <vrai_nom>` (montre TOUT). Cf [[03 - Enumeration ACLs - sens de la requete]].
- Non vide → note les **noms de champs** et si l'identité est un **nom** ou un **SID**.

## C — Nom vs SID (cause n°1 des filtres vides)
Si l'identité est stockée en **SID**, alors `-match "RDPUsers"` ne matchera **JAMAIS**.
Récupère d'abord le SID :
```powershell
Get-DomainGroup -Identity RDPUsers | select samaccountname,objectsid
ConvertFrom-SID S-1-5-21-...-1123   # SID -> nom, pour vérifier à qui il appartient
```
Puis filtre sur le **bon champ SID**.

## Champs réels de `Find-InterestingDomainAcl`
Vérifie toujours avec `| Get-Member`, mais en général :
- **cible** identifiée par `ObjectDN` (un DN, **pas** un champ `objectsid`)
- **principal** (détenteur du droit) : SID dans **`SecurityIdentifier`**, nom dans `IdentityReferenceName`

> [!DANGER] Erreurs classiques
> - `grep` **n'existe pas** en PowerShell. Utiliser `?{}` / `Where-Object` / `Select-String`.
> - Filtrer `$_.objectsid` sur `Find-InterestingDomainAcl` → champ souvent **inexistant** → toujours vide.
> - Filtrer `$_.IdentityReferenceName` sur `Get-DomainUser` → ce champ **n'existe pas** sur un user → toujours vide (pas un bug du lab).
> - `-match` = **regex** + substring. Sur un SID exact, préférer `-eq` sur le champ SID.

## Mapping SID ↔ rôle ↔ champ
1. `ConvertFrom-SID <sid>` → c'est le principal ou la cible ?
2. principal (ex RDPUsers) → filtre `SecurityIdentifier`, puis vérifie `ObjectDN`.
3. cible (ex ControlxUser) → filtre `ObjectDN -match "CN=..."`.

Voir aussi [[03 - Enumeration ACLs - sens de la requete]] · [[00 - Index]]

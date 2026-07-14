---
flag: 3
machine: Student VM
statut: validé
technique: Enumeration GPO/OU
LO: "[[LO02 - OUs et GPOs]]"
---

# F03 - GPO appliquée sur StudentMachines OU

> [!summary] Objectif
> Récupérer le **displayName** de la GPO liée à l'OU `StudentMachines` du domaine `dollarcorp.moneycorp.local`.

## Contexte
- **Étape attack flow**: énumération OU/GPO — repérer une GPO qui frappe une OU cible pour, plus tard, chasser un droit d'écriture (SharpGPOAbuse).
- **Ce qu'on a**: compte de domaine `studentx`, PowerView chargé.
- **Ce qu'on cherche**: le champ `displayName` d'un objet `groupPolicyContainer` (`Get-DomainGPO`), pas son GUID.

## Solution

### Étape 1: lire l'attribut `gplink` de l'OU
```powershell
. C:\AD\Tools\PowerView.ps1
Get-DomainOU -Identity StudentMachines | Select-Object distinguishedname, gplink
```
Le `gplink` a la forme `[LDAP://cn={GUID},cn=policies,cn=system,DC=...;0]` → le GUID entre `{}` est celui de la GPO liée.

### Étape 2: résoudre le GUID en GPO
```powershell
# Récupérer le GUID depuis gplink puis résoudre en displayname
Get-DomainGPO -Identity "{<GPO_GUID A REMPLIR DEPUIS LAB>}" | Select-Object displayname, name
```
→ Voir [[Cheatsheet - GPO et OU]] pour la formule `substring` qui parse le `gplink` en une commande.

### Étape 3 (raccourci one-liner)
```powershell
Get-DomainGPO -Identity (Get-DomainOU -Identity StudentMachines).gplink.substring(11,(Get-DomainOU -Identity StudentMachines).gplink.length-72) |
  Select-Object displayname
```

### Étape 4 (pivot AD module)
```powershell
Import-Module C:\AD\Tools\ADModulemaster\ActiveDirectory\ActiveDirectory.psd1
Get-GPInheritance -Target (Get-ADOrganizationalUnit -Filter 'name -eq "StudentMachines"').DistinguishedName
```

## Réponse

| Élément | Valeur |
|---------|--------|
| DisplayName de la GPO liée à `StudentMachines` OU | `<A REMPLIR DEPUIS LAB>` |
| GUID GPO | `<A REMPLIR DEPUIS LAB>` |

## Pièges et OPSEC
- Ne pas confondre **`displayName`** (lisible, ex. "StudentMachines Policy") et **`name`/`cn`** (GUID `{...}`) — le flag demande le displayName.
- Une OU peut avoir **plusieurs** GPO liées (le `gplink` est une liste). Vérifier avec `Get-GPInheritance`.
- Une GPO peut aussi **hériter** depuis une OU parente : `gplink` ne montre que le lien direct.
- Bruit BloodHound : recherche `StudentMachines` → onglet **Inbound Object Control** de l'OU pour recouper.

## Refs
- [[Cheatsheet - GPO et OU]]
- [[Cheatsheet - ACL et droits AD]]
- [[Synthèse - ACL et GPO]]
- [[LO02 - OUs et GPOs]]

[[Flags - Index]] · [[Journal - Index]]

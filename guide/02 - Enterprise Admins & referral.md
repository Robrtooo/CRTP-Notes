# 02 — Enterprise Admins & "referral returned"

## Symptôme
```powershell
Get-DomainGroupMember -Identity "Enterprise Admins" -Domain dcorp.local
# WARNING: Error searching for group ... "A referral was returned from the server."
```

## Cause
Le groupe **Enterprise Admins vit dans le domaine RACINE de la forêt**, pas dans un domaine enfant.
`dcorp.local` (dollarcorp) = **child domain** → le DC renvoie un *referral* vers la racine.

Lab CRTP : forest root = **moneycorp.local**, child = dollarcorp (dcorp.local).

## Fix — cibler le domaine racine
```powershell
Get-DomainGroupMember -Identity "Enterprise Admins" -Domain moneycorp.local

# récursif (résout groupes imbriqués -> tous les SIDs membres)
Get-DomainGroupMember -Identity "Enterprise Admins" -Domain moneycorp.local -Recurse
```
Le champ **`MemberSID`** = le SID cherché.

## Trouver la racine si le nom est inconnu
```powershell
Get-ForestDomain
(Get-Forest).RootDomain
Get-DomainSID -Domain moneycorp.local
```

> [!NOTE] Well-known RIDs
> Enterprise Admins SID = `<ForestRootSID>-519`
> Domain Admins SID = `<DomainSID>-512`

Voir aussi [[00 - Index]]

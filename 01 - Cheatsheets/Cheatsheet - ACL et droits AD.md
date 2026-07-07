# Cheatsheet - ACL et droits AD

> [!summary] Objectif
> Repérer les droits exploitables (`WriteDACL`, `GenericAll`, `GenericWrite`, `ForceChangePassword`) sur groupes, users, computers et policies.

> [!abstract] Pourquoi · Chaînage
> Couche **identité (AD)**, visible BloodHound. **Repérer** un droit abusable (GenericAll/WriteDACL/ForceChangePassword) = souvent le flag · **Exploiter** = agir sur l'objet (reset pwd, add member, RBCD).
> **Prérequis** un principal compromis → **débloque** le contrôle d'un compte/groupe plus privilégié.

### Read ACLs on a sensitive object

```powershell
. C:\AD\Tools\PowerView.ps1
Get-DomainObjectAcl -Identity "Domain Admins" -ResolveGUIDs -Verbose
```
`

### Special Acc
```powershell
Find-InterestingDomainAcl -ResolveGUIDs | ?{$_.ObjectDN -match "CN=ControlxUser"}
```

### Find interesting ACLs for a compromised principal

```powershell
Find-InterestingDomainAcl -ResolveGUIDs | Where-Object {$_.IdentityReferenceName -match "studentx"}
Find-InterestingDomainAcl -ResolveGUIDs | Where-Object {$_.IdentityReferenceName -match "RDPUsers"}
```


### BloodHound pivot

```text
Search: studentx / Applocked / DevOps
Outbound Object Control · Inbound Object Control
```

> [!note] Workflow du lab : user direct puis groupes du user — `RDPUsers` ressort souvent là où `studentx` seul ne remonte rien. Toujours noter : ObjectDN, droit exact, principal, impact.

## Related
- [[Cheatsheet - BloodHound SharpHound]]
- [[Cheatsheet - GPO et OU]]
- [[Cheatsheet - RBCD]]
- [[Synthèse - ACL et GPO]]

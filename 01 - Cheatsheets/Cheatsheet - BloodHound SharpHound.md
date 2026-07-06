# Cheatsheet - BloodHound SharpHound

> [!summary] Objectif
> Collecter les relations AD (SharpHound) et lire les chemins d'attaque (shortest path, object control, ACL, delegation). Le vrai intérêt est la lecture des paths, pas la collecte.

### SharpHound CE via Loader

```powershell
C:\AD\Tools\Loader.exe -Path C:\AD\Tools\Sharphound\SharpHound.exe -args --collectionmethods All
```

### BloodHound Legacy collector

```powershell
cd C:\AD\Tools\BloodHound-4.0.3_old\BloodHound-master\Collectors
Invoke-BloodHound -CollectionMethod All
```

### Lower-noise collection

```text
Group,GPOLocalGroup,Session,Trusts,ACL,Container,ObjectProps,SPNTargets
```

### Views utiles

```text
Shortest Path to Domain Admins
Inbound / Outbound Object Control   (Applocked, DevOps, studentx, RDPUsers)
```

> [!note] Attention à la compatibilité collecteur/UI entre Legacy et CE.

## Related
- [[Enumeration]]
- [[Cheatsheet - ACL et droits AD]]
- [[Cheatsheet - GPO et OU]]
- [[Cheatsheet - Delegation attacks]]

# Cheatsheet - BloodHound SharpHound

> [!summary] Objectif
> Collecter et lire rapidement les relations AD avec les outils vus dans le PDF.

> [!warning] Cadre d'utilisation
> À utiliser uniquement dans un environnement de lab autorisé, dans le cadre de l'apprentissage et de la préparation CRTP.

## Quick use

- Après l’énumération de base
- Pour shortest path / object control / ACL / delegation

## Core commands

### SharpHound CE via Loader

```powershell
C:\AD\Tools\Loader.exe -Path C:\AD\Tools\Sharphound\SharpHound.exe -args --collectionmethods All
```

Le chemin le plus direct montré dans le support pour une collecte CE.

### BloodHound Legacy collector

```powershell
cd C:\AD\Tools\BloodHound-4.0.3_old\BloodHound-master\Collectors
Invoke-BloodHound -CollectionMethod All
```

Version PowerShell utile pour BloodHound Legacy 4.0.3.

### Lower-noise collection from the PDF

```text
Group,GPOLocalGroup,Session,Trusts,ACL,Container,ObjectProps,SPNTargets
```

Le PDF mentionne explicitement des collectes moins bruyantes que `All`.

### UI / analysis

```text
BloodHound Legacy : C:\AD\Tools\BloodHound-win32-x64\BloodHound-win32-x64
BloodHound CE : instance partagée read-only du lab
Views utiles :
- Shortest Path to Domain Admins
- Outbound Object Control
- Inbound Object Control
```

Le gros intérêt n’est pas juste la collecte, c’est la lecture des rights / paths derrière.

## PDF notes

- Le support insiste sur la compatibilité collecteur/UI entre Legacy et CE.
- Pour `Applocked`, `DevOps`, `studentx`, `RDPUsers`, les vues `Inbound/Outbound Object Control` sont les plus utiles.

## Related

- [[Enumeration]]
- [[Cheatsheet - ACL et droits AD]]
- [[Cheatsheet - GPO et OU]]
- [[Cheatsheet - Delegation attacks]]


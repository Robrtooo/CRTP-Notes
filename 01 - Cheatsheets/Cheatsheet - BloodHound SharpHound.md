# Cheatsheet - BloodHound SharpHound

> [!summary] Objectif
> Collecter les relations AD (SharpHound) et lire les chemins d'attaque (shortest path, object control, ACL, delegation). Le vrai intérêt est la lecture des paths, pas la collecte.

> [!important] Chemins réels (VM student) — cf [[Cheatsheet - Chemins des outils]]
> Le collector est **double-niché** :
> - `.exe` : `C:\AD\Tools\BloodHound-master\BloodHound-master\Collectors\SharpHound.exe`
> - `.ps1` : `C:\AD\Tools\BloodHound-master\BloodHound-master\Collectors\SharpHound.ps1`
> - raccourci : `C:\AD\Tools\Sliver\SharpHound.exe`
> - furtif (ADWS) : `C:\AD\Tools\SOAPHound.exe`
> - GUI (visualisation) : `C:\AD\Tools\BloodHound-win32-x64\`

### Collecte — .exe (simple)

```powershell
C:\AD\Tools\BloodHound-master\BloodHound-master\Collectors\SharpHound.exe -c All -d dollarcorp.moneycorp.local
# ou le raccourci Sliver :
C:\AD\Tools\Sliver\SharpHound.exe -c All -d dollarcorp.moneycorp.local
```

### Collecte — .ps1 (dot-source obligatoire)

```powershell
. C:\AD\Tools\BloodHound-master\BloodHound-master\Collectors\SharpHound.ps1
Invoke-BloodHound -CollectionMethod All -Domain dollarcorp.moneycorp.local
```

### Collecte furtive — SOAPHound (ADWS 9389)

```powershell
.\SOAPHound.exe --buildcache -c C:\Users\Public\cache.txt
.\SOAPHound.exe -c C:\Users\Public\cache.txt --bhdump -o C:\Users\Public\bh-output
```

### Ancien collector (Legacy 4.0.3)

```powershell
cd C:\AD\Tools\BloodHound-4.0.3_old\BloodHound-master\Collectors
. .\SharpHound.ps1
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

## Visualiser (après la collecte)

## NEO4J passwd : cyber
```powershell
# 1. base de données neo4j  (chemin DOUBLE-niché !)
C:\AD\Tools\neo4j-community-4.4.5-windows\neo4j-community-4.4.5\bin\neo4j.bat console
#   laisse cette fenêtre ouverte ; 

# http://localhost:7474
# http://localhost:7687
  
#(1er login neo4j/neo4j -> change le mdp)

# 2. GUI BloodHound
C:\AD\Tools\BloodHound-win32-x64\BloodHound-win32-x64\BloodHound.exe
#   login creds neo4j (bolt://localhost:7687)

# 3. importer : drag-drop du .zip SharpHound (ou bouton "Upload Data")
```

- **Marque tes acquis** : clic droit sur un node possédé → *Mark as Owned* (débloque les "from Owned").
- **Export image** : icône *Export Graph* → PNG. À déposer dans `attachments/bloodhound/` de la vault.

## Requêtes utiles (onglet Analysis + Cypher)

Pré-faites (onglet **Analysis**) :
```text
Shortest Path to Domain Admins
Shortest Paths to High Value Targets
Find Principals with DCSync Rights
Find Computers with Unconstrained Delegation
Shortest Paths from Owned Principals
```

Cypher custom (barre "Raw Query" en bas) :
```cypher
// tout ce que RDPUsers CONTRÔLE (outbound object control) — utile flag F02
MATCH p=(g:Group)-[r]->(n) WHERE g.name STARTS WITH "RDPUSERS@" RETURN p

// users kerberoastables (SPN) — flag F23
MATCH (u:User {hasspn:true}) RETURN u.name, u.serviceprincipalnames

// AS-REP roastables (pas de préauth)
MATCH (u:User {dontreqpreauth:true}) RETURN u.name

// unconstrained delegation — flag F24/F25
MATCH (c:Computer {unconstraineddelegation:true}) RETURN c.name

// constrained delegation (allowedtodelegateto) — flag F26/F27
MATCH (c) WHERE c.allowedtodelegate IS NOT NULL RETURN c.name, c.allowedtodelegate

// DCSync (GetChanges + GetChangesAll) vers le domaine — flag F21
MATCH p=(n)-[:GetChanges|GetChangesAll*1..]->(d:Domain) RETURN p

// chemin depuis TON user vers Domain Admins
MATCH p=shortestPath((u:User {name:"STUDENTX@DOLLARCORP.MONEYCORP.LOCAL"})-[*1..]->(g:Group {name:"DOMAIN ADMINS@DOLLARCORP.MONEYCORP.LOCAL"})) RETURN p
```

> [!tip] Noms en MAJUSCULES
> Dans neo4j les `name` sont en **UPPER@FQDN** (ex `RDPUSERS@DOLLARCORP.MONEYCORP.LOCAL`). Respecte la casse dans le Cypher sinon 0 résultat.

## Related
- [[Cheatsheet - Chemins des outils]]
- [[Enumeration]]
- [[Cheatsheet - ACL et droits AD]]
- [[Cheatsheet - GPO et OU]]
- [[Cheatsheet - Delegation attacks]]

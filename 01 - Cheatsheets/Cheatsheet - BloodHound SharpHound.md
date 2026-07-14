# Cheatsheet - BloodHound SharpHound

> [!summary] Objectif
> Collecter les relations AD (SharpHound) et lire les chemins d'attaque (shortest path, object control, ACL, delegation). Le vrai intérêt est la lecture des paths, pas la collecte.

> [!abstract] À quoi ça sert — et à quoi PAS
> Carte de la couche **identité/AD** seulement (ACL, sessions, admin rights, delegation, chemins → DA). **Aveugle** à la couche hôte (services Windows, Jenkins, web) → pour ça, énum locale ([[Cheatsheet - Privilege escalation]]).
> Exploiter = clic sur une **arête** → onglet **Abuse Info** = les commandes.

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

## Visualiser — lancer Neo4j PUIS BloodHound

> [!important] Où lancer Neo4j (EN PREMIER, avant d'ouvrir la GUI ; laisse la fenêtre ouverte = c'est le serveur de base, la fermer coupe la GUI)
> Sur la **VM student**, chemin **double-niché** (cf bloc ci-dessous). Console web `http://localhost:7474`, Bolt `bolt://localhost:7687`.
> 1er login `neo4j`/`neo4j` → force le changement. **Mdp de ce pod : `cyber`.**

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

### Bloc GPO — tout voir (LO06 / F03 / F09)

> [!important] Les GPO edges nécessitent `-c Container` (inclus dans `-c All`)
> `-c Default` **n'inclut PAS** `GpLink` ni `Contains` → `MATCH ()-[:GpLink]->() RETURN count(*)` = 0 = ré-collecte requise avec `-c All` (ou au minimum `-c Container,GPOLocalGroup,ACL`).

```cypher
// juste les nodes GPO
MATCH (g:GPO) RETURN g

// GPOs + leur cible (OU/Domain)
MATCH p=(g:GPO)-[:GpLink]->(o) RETURN p

// chaîne complète GPO → OU → tous les objets impactés
MATCH p=(g:GPO)-[:GpLink]->(container)-[:Contains*1..]->(n) RETURN p LIMIT 300

// qui CONTRÔLE une GPO (Inbound Object Control) — modif = RCE sur toutes les machines liées
MATCH p=(n)-[:GenericAll|GenericWrite|WriteDacl|WriteOwner]->(g:GPO) RETURN p

// GPO ciblant une OU précise (ex StudentMachines → F03)
MATCH p=(g:GPO)-[:GpLink]->(o:OU) WHERE toUpper(o.name) CONTAINS "STUDENT" RETURN p

// diagnostic collecte
MATCH (g:GPO) RETURN count(g)                    // nb de GPOs
MATCH ()-[r:GpLink]->() RETURN count(r)          // nb d'edges GpLink (0 = pas de -c Container)
```

Panneau **Analysis** pré-baked :
- `Find GPOs with Interesting Permissions` = équivalent visuel de la req "qui contrôle une GPO".

> [!warning] Le compteur "N GPOs" dans le Node Info ne rend rien au clic
> Cause fréquente : filtre de node type ou GPOs très à l'écart hors viewport. Options :
> - **Menu ☰ → Settings → Node Types** : coche `GPO` (peut être décoché par défaut selon version).
> - Après clic sur le compteur, **Fit to Screen** (icône ⛶ ou `Ctrl+A`) pour recentrer.
> - Bypass : lance directement le Cypher `MATCH p=(g:GPO)-[:GpLink]->(o)-[:Contains*1..]->(n {name:"<CIBLE>"}) RETURN p`.

### Reset de la vue graphe

Pas de "home button" — la vue **change** avec chaque nouvelle requête, elle ne s'accumule pas.
- Clear search bar (X) → tape le nom du domain → clic.
- Ou icône **↻ Refresh** en haut de la sidebar.
- Ou Cypher fourre-tout : `MATCH (n) RETURN n LIMIT 300`.
- **Wipe complet DB** (à faire seulement avant ré-upload propre) : `MATCH (n) DETACH DELETE n`.

> [!tip] Noms en MAJUSCULES
> Dans neo4j les `name` sont en **UPPER@FQDN** (ex `RDPUSERS@DOLLARCORP.MONEYCORP.LOCAL`). Respecte la casse dans le Cypher sinon 0 résultat.

> [!warning] Debug — « je vois des nodes mais la recherche dit no data / user introuvable »
> Dans l'ordre :
>
> 1. **Compte cherché absent = compte LOCAL, pas de domaine.** Un user créé sur une box (ex. `john` via abus de service) n'est **jamais** dans SharpHound → c'est normal, pas un bug. La search ne connaît que l'AD.
> 2. **`RETURN u.name` renvoie un tableau, pas un graphe** → la vue graphe affiche « no data ». Pour voir des nodes, retourne le node : `MATCH (u:User) RETURN u LIMIT 100`.
> 3. **Vérifie l'ingest réel** : `MATCH (n) RETURN labels(n)[0], count(n)`. Si `User = 0` alors que le zip contient des users → **import partiel**.
> 4. **Cause fréquente d'import partiel : BOM UTF-8** (`EF BB BF`) en tête des `.json` (export via redirection PowerShell) — certaines versions n'ingèrent pas. Fix : retirer le BOM puis ré-importer (wipe la db avant).
> 5. **Recherche insensible casse / partielle** : `MATCH (n) WHERE toUpper(n.name) CONTAINS "SVCADMIN" RETURN n`.

## Related
- [[Cheatsheet - Chemins des outils]]
- [[Enumeration]]
- [[Cheatsheet - ACL et droits AD]]
- [[Cheatsheet - GPO et OU]]
- [[Cheatsheet - Delegation attacks]]

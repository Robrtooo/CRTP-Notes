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

# http://localhost:7474
# http://localhost:7687
  
#(1er login neo4j/neo4j -> change le mdp)

# 2. GUI BloodHound
C:\AD\Tools\BloodHound-win32-x64\BloodHound-win32-x64\BloodHound.exe
#   login creds neo4j (bolt://localhost:7687)

# 3. importer : drag-drop du .zip SharpHound (ou bouton "Upload Data")
```

- **Marque tes acquis** : clic droit sur un node possédé → *Mark as Owned* (débloque les "from Owned").
- **Export image** : icône *Export Graph* → PNG. À déposer dans `attachments/bloodhound/` de la vault.

## Requêtes utiles — Analysis + Cypher (toggles par thème)

> [!tip] Noms en MAJUSCULES + Raw Query en bas d'écran
> Les `name` neo4j sont en **UPPER@FQDN** (ex `RDPUSERS@DOLLARCORP.MONEYCORP.LOCAL`). Respecte la casse sinon 0 résultat. Recherche insensible : `WHERE toUpper(n.name) CONTAINS "..."`.

**Panneau `Analysis`** — pré-baked à cliquer :
```text
Shortest Path to Domain Admins
Shortest Paths to High Value Targets
Find Principals with DCSync Rights
Find Computers with Unconstrained / Constrained Delegation
Shortest Paths from Owned Principals
Find GPOs with Interesting Permissions
```

> [!example]- 🎯 Marquage — Owned / High Value (BATCH via Cypher)
> Le right-click *Mark as Owned / High Value* marche node-par-node (chiant). Pour tagger en masse : Cypher `SET`. **Un `SET` ne rend rien visuellement → toujours ajouter `RETURN count()` pour confirmer, puis refresh la vue.**
> ```cypher
> // tag une catégorie entière comme High Value (= cibles path-finding)
> MATCH (g:GPO) SET g.highvalue = true RETURN count(g)
> MATCH (u:User) WHERE u.name STARTS WITH "SVC" SET u.highvalue = true RETURN count(u)
>
> // tag comme Owned (= point de départ "Shortest Paths from Owned")
> MATCH (u:User {name:"STUDENTX@DOLLARCORP.MONEYCORP.LOCAL"}) SET u.owned = true RETURN u
> MATCH (u:User {name:"SVCADMIN@DOLLARCORP.MONEYCORP.LOCAL"}) SET u.owned = true RETURN u
>
> // unset
> MATCH (n) WHERE n.highvalue = true SET n.highvalue = false RETURN count(n)
> MATCH (n) WHERE n.owned = true SET n.owned = false RETURN count(n)
> ```
> Piège : right-click sur la **ligne de résultat** (mode data/tableau) = pas de menu. Le menu n'est dispo que sur le **node dans le graphe visuel**. Batch Cypher = plus fiable.

> [!example]- 🏰 GPO — tout voir (LO06 / F03 / F09)
> **Prérequis : `-c Container` dans la collecte** (inclus dans `-c All`, PAS dans `-c Default`).
> `MATCH ()-[:GpLink]->() RETURN count(*)` = 0 → ré-collecte requise avec `-c All` (ou minimum `-c Container,GPOLocalGroup,ACL`).
> ```cypher
> // juste les nodes GPO
> MATCH (g:GPO) RETURN g
>
> // GPOs + leur cible (OU/Domain)
> MATCH p=(g:GPO)-[:GpLink]->(o) RETURN p
>
> // chaîne complète GPO → OU → tous les objets impactés
> MATCH p=(g:GPO)-[:GpLink]->(container)-[:Contains*1..]->(n) RETURN p LIMIT 300
>
> // qui CONTRÔLE une GPO (Inbound Object Control) — modif = RCE sur toutes les machines liées
> MATCH p=(n)-[:GenericAll|GenericWrite|WriteDacl|WriteOwner]->(g:GPO) RETURN p
>
> // GPO ciblant une OU précise (ex StudentMachines → F03)
> MATCH p=(g:GPO)-[:GpLink]->(o:OU) WHERE toUpper(o.name) CONTAINS "STUDENT" RETURN p
> ```
> Piège Node Info panel : le compteur "N GPOs" ne rend rien au clic si type GPO filtré. **Menu ☰ → Settings → Node Types → coche GPO**, ou bypass Cypher direct.

> [!example]- 🔑 ACL / droits AD (LO03 / F02)
> ```cypher
> // outbound object control d'un principal (ex RDPUsers → F02)
> MATCH p=(g:Group)-[r]->(n) WHERE g.name STARTS WITH "RDPUSERS@" RETURN p
>
> // qui a des droits abusables sur MON user
> MATCH p=(n)-[:GenericAll|GenericWrite|WriteDacl|WriteOwner|AddMember|ForceChangePassword]->(u:User {name:"STUDENTX@DOLLARCORP.MONEYCORP.LOCAL"}) RETURN p
>
> // tous les principals avec GenericAll sur quelque chose
> MATCH p=(n)-[:GenericAll]->(m) RETURN p LIMIT 200
>
> // AddSelf / AddMember (self-service groupe)
> MATCH p=(n)-[:AddSelf|AddMember]->(g:Group) RETURN p
>
> // WriteSPN (kerberoast forcé)
> MATCH p=(n)-[:WriteSPN]->(u:User) RETURN p
> ```

> [!example]- 👑 Kerberos — SPN / AS-REP / RC4 downgrade (F23)
> ```cypher
> // users kerberoastables (SPN présent) — F23
> MATCH (u:User {hasspn:true}) RETURN u.name, u.serviceprincipalnames
>
> // AS-REP roastables (DONT_REQ_PREAUTH)
> MATCH (u:User {dontreqpreauth:true}) RETURN u.name
>
> // comptes forcés en RC4 (downgrade)
> MATCH (u:User) WHERE u.supportedencryptiontypes CONTAINS "rc4" RETURN u.name
>
> // users protégés (Protected Users group) — NE PEUVENT PAS être kerberoastés en RC4
> MATCH (u:User)-[:MemberOf*1..]->(g:Group {name:"PROTECTED USERS@DOLLARCORP.MONEYCORP.LOCAL"}) RETURN u.name
> ```

> [!example]- 🎭 Delegation — Unconstrained / Constrained / RBCD (F24-F28)
> ```cypher
> // unconstrained delegation — F24/F25
> MATCH (c:Computer {unconstraineddelegation:true}) RETURN c.name
> MATCH (u:User {unconstraineddelegation:true}) RETURN u.name
>
> // constrained delegation (allowedtodelegateto) — F26/F27
> MATCH (n) WHERE n.allowedtodelegate IS NOT NULL RETURN n.name, n.allowedtodelegate
>
> // constrained avec protocol transition (T2A4D → RCE any service)
> MATCH (n) WHERE n.allowedtodelegate IS NOT NULL AND n.trustedtoauth = true RETURN n.name
>
> // RBCD (msDS-AllowedToActOnBehalfOfOtherIdentity) — F28
> MATCH p=(n)-[:AllowedToAct]->(c:Computer) RETURN p
>
> // qui peut ÉCRIRE l'attribut RBCD sur un target (= abus RBCD possible)
> MATCH p=(n)-[:AddAllowedToAct|WriteAccountRestrictions]->(c:Computer) RETURN p
> ```

> [!example]- ⚡ DCSync / Chemins vers DA (LO08 / F21)
> ```cypher
> // qui a DCSync (GetChanges + GetChangesAll) — F21
> MATCH p=(n)-[:GetChanges|GetChangesAll*1..]->(d:Domain) RETURN p
>
> // shortest path depuis TON user vers Domain Admins
> MATCH p=shortestPath((u:User {name:"STUDENTX@DOLLARCORP.MONEYCORP.LOCAL"})-[*1..]->(g:Group {name:"DOMAIN ADMINS@DOLLARCORP.MONEYCORP.LOCAL"})) RETURN p
>
> // shortest path depuis N'IMPORTE QUEL owned vers DA
> MATCH p=shortestPath((s {owned:true})-[*1..]->(g:Group {name:"DOMAIN ADMINS@DOLLARCORP.MONEYCORP.LOCAL"})) RETURN p
>
> // shortest path vers Enterprise Admins (forest root moneycorp.local)
> MATCH p=shortestPath((s {owned:true})-[*1..]->(g:Group {name:"ENTERPRISE ADMINS@MONEYCORP.LOCAL"})) RETURN p
> ```

> [!example]- 👥 Users / Groups / Computers — énum de base (F01)
> ```cypher
> // membres Domain Admins
> MATCH (u:User)-[:MemberOf*1..]->(g:Group {name:"DOMAIN ADMINS@DOLLARCORP.MONEYCORP.LOCAL"}) RETURN u.name
>
> // membres Enterprise Admins (forest root) — F01
> MATCH (u:User)-[:MemberOf*1..]->(g:Group {name:"ENTERPRISE ADMINS@MONEYCORP.LOCAL"}) RETURN u.name, u.objectid
>
> // computers avec description non vide (souvent mots de passe en clair)
> MATCH (c:Computer) WHERE c.description IS NOT NULL AND c.description <> "" RETURN c.name, c.description
> MATCH (u:User) WHERE u.description IS NOT NULL AND u.description <> "" RETURN u.name, u.description
>
> // users passwordnotrequired
> MATCH (u:User {passwordnotrequired:true}) RETURN u.name
>
> // users pwd never expires
> MATCH (u:User {pwdneverexpires:true}) RETURN u.name
> ```

> [!example]- 🖥️ Sessions / Admin rights (LO07 / F10)
> ```cypher
> // où svcadmin a une session (F10)
> MATCH p=(u:User {name:"SVCADMIN@DOLLARCORP.MONEYCORP.LOCAL"})<-[:HasSession]-(c:Computer) RETURN p
>
> // où N'IMPORTE QUEL DA a une session (session hunting global)
> MATCH (u:User)-[:MemberOf*1..]->(:Group {name:"DOMAIN ADMINS@DOLLARCORP.MONEYCORP.LOCAL"}) MATCH (u)<-[:HasSession]-(c:Computer) RETURN u.name, c.name
>
> // où MON user est admin local (direct ou via groupe)
> MATCH p=(u:User {name:"STUDENTX@DOLLARCORP.MONEYCORP.LOCAL"})-[:AdminTo|MemberOf*1..]->(c:Computer) RETURN p
>
> // machines où RDPUsers peut RDP
> MATCH p=(g:Group {name:"RDPUSERS@DOLLARCORP.MONEYCORP.LOCAL"})-[:CanRDP]->(c:Computer) RETURN p
> ```

> [!example]- 🌐 Trusts inter-domaine / forêt (LO04 / F04)
> ```cypher
> // tous les trusts sortants
> MATCH p=(d1:Domain)-[:TrustedBy]->(d2:Domain) RETURN p
>
> // trust dollarcorp ↔ eurocorp (F04)
> MATCH p=(d1:Domain)-[r]-(d2:Domain) WHERE d1.name CONTAINS "EUROCORP" OR d2.name CONTAINS "EUROCORP" RETURN p
> ```

> [!example]- 🩺 Diagnostic collecte / ingest (à lancer AVANT toute enquête)
> ```cypher
> // combien de nodes par type ? (test rapide ingest complet)
> MATCH (n) RETURN labels(n)[0] AS type, count(n) ORDER BY count(n) DESC
>
> // combien d'edges par type ?
> MATCH ()-[r]->() RETURN type(r) AS edge, count(r) ORDER BY count(r) DESC
>
> // check GPO edges présents (0 = -c Container manquant lors de la collecte)
> MATCH ()-[r:GpLink]->() RETURN count(r)
> MATCH ()-[r:Contains]->() RETURN count(r)
>
> // check sessions présentes (0 = -c Session manquant ou pas d'user connecté)
> MATCH ()-[r:HasSession]->() RETURN count(r)
> ```
> Interprétation : label à 0 alors que le zip contient de la donnée → **import partiel**. Cause fréquente = **BOM UTF-8** (`EF BB BF`) en tête des `.json` (export via redirection PowerShell). Fix : retire le BOM + wipe + ré-importe.

### Reset de la vue graphe

Pas de "home button" — la vue **change** avec chaque requête, elle ne s'accumule pas.
- Clear search bar (X) → tape le nom du domain → clic.
- Ou icône **↻ Refresh** en haut de la sidebar.
- Ou Cypher fourre-tout : `MATCH (n) RETURN n LIMIT 300`.
- **Wipe complet DB** (avant ré-upload propre) : `MATCH (n) DETACH DELETE n`.

> [!warning] Debug — « je vois des nodes mais no data / user introuvable »
> 1. **Compte LOCAL** (créé sur une box via abus service) → **jamais dans SharpHound**. Normal.
> 2. **`RETURN u.name` renvoie un tableau**, pas un graphe → mode data, pas visualisation. Retourne le node : `MATCH (u:User) RETURN u LIMIT 100`.
> 3. **Ingest partiel** : cf toggle "🩺 Diagnostic" ci-dessus.
> 4. **BOM UTF-8** en tête des `.json` (export PS avec redirection) → certaines versions n'ingèrent pas. Fix : retire BOM + wipe + ré-importe.

## Related
- [[Cheatsheet - Chemins des outils]]
- [[Enumeration]]
- [[Cheatsheet - ACL et droits AD]]
- [[Cheatsheet - GPO et OU]]
- [[Cheatsheet - Delegation attacks]]
